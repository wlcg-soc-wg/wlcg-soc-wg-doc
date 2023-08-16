# GnuPG

To set up GPG fist you need to generate a GPG key, if you don't already have one, with:

```
gpg --gen-key
```

Note that the email should be the email set up in the GnuPG part of the configuration (`gpg_email` parameter), and the same applies for the password (`gpg_password` parameter). There are known cases of errors when using it with a password (instead of passwordless as in the default configuration). If it gives an error run is as root.

Then move the key to the directory set up as home directory (`gpg_homedir` parameter), set the user under which your webserver is running as the owner and group and the SELinux context `httpd_sys_rw_content_t`.

```
mv ~/.gnupg /var/www/MISP/
chown -R apache:apache /var/www/MISP/.gnupg
chcon -R -t httpd_sys_rw_content_t /var/www/MISP/.gnupg
```

Finally export the public key to the webroot.

```
sudo -u apache gpg --homedir /var/www/MISP/.gnupg --export --armor YOUR-EMAIL > /var/www/MISP/app/webroot/gpg.asc
```
