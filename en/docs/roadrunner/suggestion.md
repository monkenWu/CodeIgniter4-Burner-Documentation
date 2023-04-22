---
title: RoadRunner Suggestion
---

# RoadRunner Suggestion

### Automatic reload

In the default circumstance of RoadRunner, you must restart the server everytime after you revised any PHP files so that your revision will effective.
It seems not that friendly during development.

You can revise your `.rr.yaml` configuration file, add the settings below and start the development mode with `-d`. RoadRunner Server will detect if the PHP files were revised or not, automatically, and reload the Worker instantly.

```yaml
reload:
  interval: 1s
  patterns: [ ".php" ]
  services:
    http:
      recursive: true
      ignore: [ "vendor" ]
      patterns: [ ".php", ".go", ".md" ]
      dirs: [ "." ]
```

## Debugging with only one Worker

Since the RoadRunner has fundamentally difference with other server software(i.e. Nginx, Apache), every Codeigniter4 will persist inside RAMs as the form of Worker, HTTP requests will reuse these Workers to process. Hence, we have better develop and test stability under the circumstance with only one Worker to prove it can also work properly under serveral Workers in the formal environment.

You can reference the `.rr.yaml` settings below to lower the amount of Worker to the minimum:

```yaml
http:
  address: "0.0.0.0:8080"
  static:
    dir: "./public"
    forbid: [".htaccess", ".php"]
  pool:
    num_workers: 1
    # max_jobs: 64
    # debug: true
```
