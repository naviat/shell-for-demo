## sheel-for-demo

`shell-for-demo.sh` is a pure bash script that will wait on the availability of a host and TCP port.  It is useful for synchronizing the spin-up of interdependent services, such as linked docker containers.  Since it is a pure bash script, it does not have any external dependencies.

## Usage

```
shell-for-demo.sh host:port [-s] [-t timeout] [-- command args]
-h HOST | --host=HOST       Host or IP under test
-p PORT | --port=PORT       TCP port under test
                            Alternatively, you specify the host and port as host:port
-s | --strict               Only execute subcommand if the test succeeds
-q | --quiet                Don't output any status messages
-t TIMEOUT | --timeout=TIMEOUT
                            Timeout in seconds, zero for no timeout
-- COMMAND ARGS             Execute command with args after the test finishes
```

## Examples

For example, let's test to see if we can access port 80 on www.google.com, and if it is available, echo the message `google is up`.

```
$ ./shell-for-demo.sh www.google.com:80 -- echo "google is up"
shell-for-demo.sh: waiting 15 seconds for www.google.com:80
shell-for-demo.sh: www.google.com:80 is available after 0 seconds
google is up
```

You can set your own timeout with the `-t` or `--timeout=` option.  Setting the timeout value to 0 will disable the timeout:

```
$ ./shell-for-demo.sh -t 0 www.google.com:80 -- echo "google is up"
shell-for-demo.sh: waiting for www.google.com:80 without a timeout
shell-for-demo.sh: www.google.com:80 is available after 0 seconds
google is up
```

The subcommand will be executed regardless if the service is up or not.  If you wish to execute the subcommand only if the service is up, add the `--strict` argument. In this example, we will test port 81 on www.google.com which will fail:

```
$ ./shell-for-demo.sh www.google.com:81 --timeout=1 --strict -- echo "google is up"
shell-for-demo.sh: waiting 1 seconds for www.google.com:81
shell-for-demo.sh: timeout occurred after waiting 1 seconds for www.google.com:81
shell-for-demo.sh: strict mode, refusing to execute subprocess
```

If you don't want to execute a subcommand, leave off the `--` argument.  This way, you can test the exit condition of `shell-for-demo.sh` in your own scripts, and determine how to proceed:

```
$ ./shell-for-demo.sh www.google.com:80
shell-for-demo.sh: waiting 15 seconds for www.google.com:80
shell-for-demo.sh: www.google.com:80 is available after 0 seconds
$ echo $?
0
$ ./shell-for-demo.sh www.google.com:81
shell-for-demo.sh: waiting 15 seconds for www.google.com:81
shell-for-demo.sh: timeout occurred after waiting 15 seconds for www.google.com:81
$ echo $?
124
```

