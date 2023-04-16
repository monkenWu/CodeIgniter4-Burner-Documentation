---
title: RoadRunner Commands
---

# RoadRunner Commands

### start server

When you do not pass any parameters, it will be preset to start the server.

```
php spark burner:start
```

By default, burner reads the default driver written in `app/Burner.php`. Of course, you can force Burner to execute commands with the `RoadRunner` driver by using a parameter like this：

```
php spark burner:start --driver RoadRunner
```

{% info Note %}

`--driver RoadRunner` This parameter also applies to all the commands mentioned below.

{% end %}

You can also use the following parameters to construct your commands according to your needs.

* -c: path to the config file.
* -w: set the working directory.
* --dotenv: populate the process with env variables from the .dotenv file.
* -d: start a pprof server. Note, this is not debug, to use debug logs level, please, use logs: https://roadrunner.dev/docs/plugins-logger/2.x/en
* -s: silent mode.
* -o: to override configuration keys with your values, e.g.
  ```
  -o=http.address=:8080
  ```
  will override the http.address from the .rr.yaml.

{% info Note %}

Burner already uses the `-c`, `-p` and `-w` parameters and you need to avoid using the same parameters again.

{% end %}

Let RoadRunner work in the background.

When you run the server with this option, Burner will ignore the Automatic reload setting.

```
php spark burner:start --daemon
```

Using this mode Burner will direct the output to `/dev/null` and you must define your `log_output` in `.rr.yaml` to look like this:

```yaml
logs:
  mode: development
  output: stdout
  file_logger_options:
    log_output: "{{log_path}}"
    max_size: 100
    max_age: 1
    max_backups : 5
    compress: false
```

### stop server

This command runs in daemon mode only.

```
php spark burner:stop
```

Force the server to close.

```
php spark burner:stop -f
```

### workers status

Get the current running information of all Workers.

```
php spark burner:rr workers
```

Continuously updated interaction mode every second.

```
php spark burner:rr workers -i
```

### more command

Run commands directly to RoadRunner's rr binary.

```
php spark burner:rr [rr_comands]
```

You can refer to the official [RoadRunner documentation](https://roadrunner.dev/docs/app-server-cli/2.x/en) to construct your commands. 

{% info Note %}

Burner already uses the `-c`, `-p` and `-w` parameters and you need to avoid using the same parameters again.

{% end %}
