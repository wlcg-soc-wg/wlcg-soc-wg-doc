# Deploying MISP

## Operating System ​

MISP can be deployed on a wide variety of different operating systems. The platform of choice inside WLCG is obviously Linux. Similarly to the myriad of operating systems supported, MISP can be deployed on a variety of different Linux distributions. Sites are free to choose the operating system and the Linux distribution. Nonetheless, we strongly recommend using CentOS 7 / SL 7 / RHEL 7. That's the distribution used at CERN and all provided resources (installation packages, configuration management code, documentation, scripts, etc) have been developed and tested on CentOS 7. MISP has dependencies on some package versions newer than what's available with CentOS 7 out of the box. The EPEL (Extra Packages for Enterprise Linux) repository and Software Collections are needed.

## System Requirements

### Core of MISP

- System type: Preferable to have a VM (or a container), dedicating a physical box for it would be an overkill.
- Number of cores: 2 cores. This would allow 1 core to be used for the UI and 1 core for background jobs.
- Memory: 4 GB of RAM. Most of the time memory usage will be very low. Exports have the potential of taking a lot of memory (depending on the number of events being exported and the export format). Also, some of the more advanced features (such as caching of feeds) can also take considerable amounts of memory.
- Disk space: 10 GB. MISP itself requires very little space. Attachments (payload samples, artefacts dropped, etc) are stored on the filesystem, not in the database. Still these should be relatively small in size.

### MISP dependencies
#### MySQL

MISP is using MySQL as a backend for data storage. The database can be either hosted on the local server or can be hosted remotely, if you have a central MySQL server for example. In case you decide to use a remote MySQL server it's advisable to create a MISP specific username and database in advance of the workshop. The MISP DB user should have access only to the MISP database. There shouldn't be any particular requirements around that (database should be less than 1 GB in size, in total). Of course MariaDB can be used as a good substitute for MySQL.

#### Redis

MISP requires Redis for a number of features (state of background workers, caching of feeds, session data, etc). Again, the Redis server can be either co-hosted on the same system as MISP or a remote Redis server can be used. Depending on the set of features used the memory requirements needed for Redis may be significant (in the order of 1-2 GB of RAM).

## HTTPS certificate

It is highly recommended to use HTTPS (TLS) for your MISP instance. While it's technically possible to use a self generated certificate we highly advice against doing that. Best would be to have a certificate issued by a certificate authority that's part of the bundle that comes with most web browsers and operating systems. If you don't have access to a commercial certificate authority you can use one of the free CAs such as Let's Encrypt. Using a certificate issues by your grid CA would be another possibility although that may pose problems for other peers connecting to your instance and not having your CA's root certificate installed.

Here's how to generate the CSR (certificate signing request) if you're using a third party CA:

```
# To generate csr (to get public facing certificate)
openssl genrsa -out /etc/pki/tls/certs/misp.key 2048
openssl req -new -sha256 -key /etc/pki/tls/certs/misp.key -out /etc/pki/tls/certs/misp.csr

```

## SELinux

The Puppet manifests below assume that you have SELinux enabled. It should work with SELinux disabled or in permissive mode but it's highly recommended that you enable SELinux.

## Masterless Puppet

One of the easiest ways of deploying MISP is by using Puppet in masterless mode.

The first step would be to create a virtual machine. In this case we are going to use CentOS 7, although it should work perfectly fine on other versions of CentOS / RHEL. Nevertheless, note that the MISP module only support those families and is not made for other Linux distributions (e.g. Debian).

1. Install the OS in the VM.

2. Install puppet-agent (the Puppet MISP module requires Puppet 4):

```
yum install https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
yum install puppet-agent
```

If you had Puppet 3 installed before installing Puppet 4, since with version 4 there has been a change of paths, you would either need to log out and log back in or change the `$PATH` variable in your environment to include `/opt/puppetlabs/bin/`.

In addition a symbolic link for the default hiera variables might be needed:

```
ln -s /etc/hiera.yaml /etc/puppet/hiera.yaml
```

3. Install the required modules:

```
puppet module install puppetlabs-apache #(dependencies: puppetlabs-stdlib, puppetlabs-concat)
puppet module install puppetlabs-inifile
puppet module install puppetlabs-firewall
puppet module install puppetlabs-vcsrepo -v 1.5.0 # unfortunately the version number needs to be fixed here
puppet module install camptocamp-openssl #(dependencies: puppetlabs-stdlib). Note that you need to enforce version 1.9.0.
puppet module install edestecd-mariadb #(dependencies: puppetlabs-apt, puppetlabs-mysql, puppet-staging, puppetlabs-stdlib)
puppet module install arioch-redis
puppet module install puppet-misp
```

Note: you might need to install ``epel-release`` and ``centos-release-scl`` repositories.

```
# We need some packages from the Extra Packages for Enterprise Linux repository
yum install epel-release

# Since MISP 2.4 PHP 5.5 is a minimal requirement, so we need a newer version than CentOS base provides
# Software Collections is a way do to this, see https://wiki.centos.org/AdditionalResources/Repositories/SCL
yum install centos-release-scl
```

4. Create a Puppet manifest and save it as ``/etc/puppet/manifests/site.pp`` (you may need to create the ``/etc/puppet/manifests/`` directory). An example manifest is:

```puppet
node default {

  # Generate certificates
  class { '::openssl':
    package_ensure         => latest,
    ca_certificates_ensure => latest,
  }

  openssl::certificate::x509 { "${::fqdn}":
    ensure       => present,
    country      => 'CH',
    organization => "${::fqdn}",
    commonname   => "${::fqdn}",
    base_dir     => '/etc/pki/tls/certs/',
    owner        => 'apache',
  }

  package {
    "iptables-services": ensure => latest,
  }

  firewall { '100 allow https':
    proto   => 'tcp',
    dport   => '443',
    action  => 'accept',
    require => Package[iptables-services],
  }

  firewall { '101 allow http':
    proto   => 'tcp',
    dport   => '80',
    action  => 'accept',
    require => Package[iptables-services],
  }

  $mysql_passwd = mysql_password('mispdb')

  class {'mariadb::server':
    root_password => 'mispdb',
    databases   => {
      'misp'  => {
        ensure  => 'present',
        charset => 'utf8',
      },
    },
    users                         => {
      'misp@localhost' => {
        ensure                   => 'present',
        max_connections_per_hour => '0',
        max_queries_per_hour     => '0',
        max_updates_per_hour     => '0',
        max_user_connections     => '0',
        password_hash            => $mysql_passwd,
        tls_options              => ['NONE'],
      },
    },
    grants => {
      'misp@localhost/misp.*' => {
        ensure     => 'present',
        options    => ['GRANT'],
        privileges => ['ALL'],
        table      => 'misp.*',
        user       => 'misp@localhost',
      },
    },
  }

  class {'::misp':
    misp_git_tag               => 'v2.4.84',
    show_correlations_on_index => true,
    showorgalternate           => true,
    timezone                   => 'Europe/Zurich',
    db_password                => 'mispdb',
    org                        => 'ORGNAME',
  }

  ini_setting { 'php56.ini_session':
    ensure  => present,
    require => Package[rh-php56],
    path    => '/etc/opt/rh/rh-php56/php.ini',
    section => 'Session',
    setting => 'session.cookie_httponly',
    value   => 'True',
    notify  => Service[rh-php56-php-fpm],
  }
  ini_setting { 'php56.ini_max_execution_time':
    ensure  => present,
    require => Package[rh-php56],
    path    => '/etc/opt/rh/rh-php56/php.ini',
    section => 'PHP',
    setting => 'max_execution_time',
    value   => '300',
    notify  => Service[rh-php56-php-fpm],
  }
    ini_setting { 'php56.ini_memory_limit':
    ensure  => present,
    require => Package[rh-php56],
    path    => '/etc/opt/rh/rh-php56/php.ini',
    section => 'PHP',
    setting => 'memory_limit',
    value   => '512M',
    notify  => Service[rh-php56-php-fpm],
  }
  ini_setting { 'php56.ini_upload_max_filesize':
    ensure  => present,
    require => Package[rh-php56],
    path    => '/etc/opt/rh/rh-php56/php.ini',
    section => 'PHP',
    setting => 'upload_max_filesize',
    value   => '50M',
    notify  => Service[rh-php56-php-fpm],
  }
  ini_setting { 'php56.ini_post_max_size':
    ensure  => present,
    require => Package[rh-php56],
    path    => '/etc/opt/rh/rh-php56/php.ini',
    section => 'PHP',
    setting => 'post_max_size',
    value   => '50M',
    notify  => Service[rh-php56-php-fpm],
  }

  #Apache

  class { '::apache':
    default_vhost => false,
    serveradmin   => 'root@localhost',
    trace_enable  => 'Off',
  }

  class{'::apache::mod::proxy_fcgi':}

  apache::vhost { "${::fqdn}-http":
    port              => '80',
    serveradmin       => 'root@localhost',
    servername        => "${::fqdn}",
    docroot           => '/var/www/MISP/app/webroot/',
    redirect_status => 'permanent',
    redirect_dest   => "https://${::fqdn}",
    manage_docroot  => false,
  }

  apache::vhost { "${::fqdn}-https":
    port              => 443,
    serveradmin       => 'root@localhost',
    servername        => "${::fqdn}",
    docroot           => '/var/www/MISP/app/webroot/',
    manage_docroot    => false,
    directoryindex    => 'index.php',
    override          => all,
    ssl               => true,
    ssl_cert          => "/etc/pki/tls/certs/${::fqdn}.crt",
    ssl_key           => "/etc/pki/tls/certs/${::fqdn}.key",
    log_level         => 'warn',
    error_log_file    => 'misp.local_error.log',
    access_log_file   => 'misp.local_access.log',
    access_log_format => 'combined',
    ssl_proxyengine   => true,
    setenvif          => ['Authorization "(.*)" HTTP_AUTHORIZATION=$1'],
    proxy_pass_match  => [
      {
        'path'         => '^/(.*\.php(/.*)?)$',
        'url'          => "fcgi://127.0.0.1:9000/var/www/MISP/app/webroot/",
        'reverse_urls' => ["fcgi://127.0.0.1:9000/var/www/MISP/app/webroot/"],
      },
    ],   
  }
}
```

5. Run puppet:

    puppet apply /etc/puppet/manifests/site.pp

Note that `site.pp` is the name that we gave to the manifest created in step number 4, but it can take whatever other name.

6. Load the DB schema:

    mysql -D misp -u misp -p < /var/www/MISP/INSTALL/MYSQL.sql

The default password is `mispdb`.

Once all this has been done we can connect to the MISP instance in the browser by typing there the ip of the VM. It will ask for login and we can use:

    user       -  admin@admin.test
    password   -  admin

And then change the password as it is asked by the platform.

## Using Puppet in agent / master mode

Another option is to deploy a MISP instance by using Puppet in agent / master mode.

A few things need to be taken into account when creating the Puppet manifest:

- Management of SSL certificates
- Open the firewall both in port 80 and 443
- Installing MISP, via the puppet-misp module in voxpupuli or the existing mirror in your local Puppet infrastructure. Be careful to handle properly the secretes and the configuration files/
- Set up the PHP configuration according to your needs, the default one that should be used is:
    + `openssl.capath => /path/to/the/certificate`
    + `session.cookie_httponly => True`
    + `session.save_handler => redis`
    + `session.save_path => tcp://localhost:6379`
    + `max_execution_time => 300`
    + `memory_limit => 512M`
    + `upload_max_filesize => 50M`
    + `post_max_size => 50M`
- Set up the Shibboleth daemon if you want to use enable Single Sign On in the instance. Note that some variable should be set in `hiera/module` for MISP to be able to establish the mapping between SSO and MISP users / roles.
- Set up Apache (or another webserver of choice) in port 443 and in port 80 redirected to 443.

## Other options
There are other options to deploy a MISP instance. However, these have not been tested therefore using them is under your own risk.

- [MISP Ansible module](https://github.com/MISP/ansible)
- [Manual Installation](https://github.com/MISP/MISP/tree/2.4/INSTALL)
