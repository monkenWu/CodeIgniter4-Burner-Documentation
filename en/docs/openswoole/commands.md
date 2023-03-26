---
title: OpenSwoole Commands
---

# OpenSwoole Commands

### Start Server

```
php spark burner:start
```

By default, Burner will read the default driver from `app/Burner.php`. Of course, you can force the `OpenSwoole` driver to start the server in the command, like this:

```
php spark burner:start --driver OpenSwoole
```

{% info Note %}

The `--driver OpenSwoole` parameter also applies to all the commands below.

{% end %}

### Daemon Mode

Run the server in the background.

```
php spark burner:start --daemon
```

{% info Note %}

When your server is running in daemon mode, Burner will ignore the automatic reload you have enabled.

{% end %}

### Stop Server

Burner will use the `SIGTERM` signal to close the program, waiting for the current task to complete before stopping the server, and will not kill the program directly.

```
php spark burner:stop
```

### Reload Worker

As the [official document](https://openswoole.com/docs/modules/swoole-server-reload#hot-code-linux-signal-trigger) says, this mode will only reload the running Worker. Please note that this mode may not handle all cases. For example: When you make changes to the core PHP files of your project through `composer require/update`.

```
php spark burner:reload --mode worker
```

```
php spark burner:reload --mode task_worker
```

### Restart Worker

Burner will stop the server for you and then restart the server.

Whether your server is running in daemon mode or development mode, you can use this command to restart.

```
php spark burner:restart
```