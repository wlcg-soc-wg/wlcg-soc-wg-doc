# JSON logs

There are 2 means of getting logs written in JSON format.

### Use Zeek built-in functionality

In your `local.bro` file add the following:

```
@load tuning/json-logs

redef LogAscii::use_json=T;
```

This will cause the logs to be written only in JSON format.

### Use the [add-json Zeek package](https://github.com/J-Gras/add-json)

After installing the package using `bro-pkg install add-json` (you first need to have `bro-pkg` installed by using the instructions available at <https://bro-package-manager.readthedocs.io/en/stable/>) add the following to your `local.bro` file:

```
@load add-json
```

This provides greater flexibility than the native JSON format described above. In addition you get logs both in ASCII (tab delimited format) and in JSON format. Full documentation can be found on GitHub: <https://github.com/J-Gras/add-json>.