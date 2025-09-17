# JSON logs

There are 2 means of getting logs written in JSON format.

### Use Zeek built-in functionality

In your `local.zeek` file add the following:

```
@load tuning/json-logs

redef LogAscii::use_json=T;
```

This will cause the logs to be written only in JSON format.

### Use the [add-json Zeek package](https://github.com/J-Gras/add-json)

After installing the package using `zeek-pkg install add-json` (you first need to have `zeek-pkg` installed by using the instructions available at <https://docs.zeek.org/projects/package-manager/en/stable/>) add the following to your `local.bro` file:

```
@load add-json
```

This provides greater flexibility than the native JSON format described above. In addition you get logs both in ASCII (tab delimited format) and in JSON format. Full documentation can be found on GitHub: <https://github.com/J-Gras/add-json>.
