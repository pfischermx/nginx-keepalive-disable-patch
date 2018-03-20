# nginx-keepalive-disable-patch
Patch to be able to disable Keep-Alive on nginx via a nginx variable.

Nginx does not let you override the Connection: header (if you do it gets duplicated), so when
you are running keep-alive and have a lot of long-running clients it can take quiet a bit to drain
a host without stopping nginx on the host.

One possible way to do it is by writing some tools that modify the keepalive_timeout and reload nginx on
certain conditions but the other way is to have a patch that tells nginx when it should send Connection: keep-alive.

This patch does "that" (patches nginx). It is very simple the way it is done, because you want to disable
keep-alive more on runtime rather than with a setting then we add a new nginx variable: $disable_keep_alive_now.

Nginx will check the value of $disable_keep_alive_now every time it needs to send the Connection: header to the client, if
the value is "yes" then it will send a Connection: close otherwise it follows its normal path which can be sending a keep-alive
or a close (if the keep-alive connection timed out).

An example:

```
location / {
  if (!-f "healthcheck/path") {
    set $disable_keep_alive_now "yes";
  }
}
```

Enjoy!
