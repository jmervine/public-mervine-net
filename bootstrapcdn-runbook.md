# BootstrapCDN Runbook

### Basic Information

* Production Host: www.bootstrapcdn.com
    * Service: Digital Ocean
    * IP Addr: 192.241.225.157
    * SSH: ssh -p 2020 www.bootstrapcdn.com
    * User: bootstrapcdn
    * Code: /home/bootstrapcdn/bootstrap-cdn
    * Logs: /home/bootstrapcdn/bootstrap-cdn/logs/server.log

* Dev Host: dev.bootstrapcdn.com
    * Service: Digital Ocean
    * IP Addr: 192.241.233.159
    * SSH: ssh -p 2020 dev.bootstrapcdn.com
    * User: bootstrapcdn
    * Code: /home/bootstrapcdn/bootstrap-cdn
    * Logs: /home/bootstrapcdn/bootstrap-cdn/logs/server.log

### Updating BootstrapCDN

Currently, automatic updates are disabled to some funkyness which has yet to be
fully addressed. To update BootstrapCDN to the latest code from github (`master`
branch for production and `develop` branch for dev) do the following.

```
$ ssh -p 2020 www.bootstrapcdn.com
# connection...

# update source
[jmervine@bootstrapcdn01 ~]$ sudo -u bootstrapcdn -c 'cd ~/bootstrap-cdn; git pull'
## git output...

# restart app...
[jmervine@bootstrapcdn01 ~]$ sudo bash -c 'cd ~bootstrapcdn/bootstrap-cdn; ./node_module
s/.bin/forever restart server.js'
info:    Forever restarted process(es):
data:        uid  command       script    forever pid   id logfile         uptime
data:    [0] 9ZVR /usr/bin/node server.js 12998   13029    logs/server.log 2:23:40:28.354
[jmervine@bootstrapcdn01 ~]$

# verify restart
[jmervine@bootstrapcdn01 ~]$ sudo bash -c 'cd ~bootstrapcdn/bootstrap-cdn; ./node_modules/.bin/forever list'
info:    Forever processes running
data:        uid  command       script    forever pid   id logfile         uptime
data:    [0] 9ZVR /usr/bin/node server.js 12998   14638    logs/server.log 0:0:2:22.975
[jmervine@bootstrapcdn01 ~]$
```

### Starting BootstrapCDN

```
[jmervine@bootstrapcdn01 ~]$ sudo bash -c 'cd ~bootstrapcdn/bootstrap-cdn; ./node_modules/.bin/forever stop server.js'
info:    Forever stopped process:
uid  command       script    forever pid   id logfile         uptime
[0] 9ZVR /usr/bin/node server.js 12998   14638    logs/server.log 0:0:3:20.147
[jmervine@bootstrapcdn01 ~]$ sudo bash -c 'cd ~bootstrapcdn/bootstrap-cdn; ./node_modules/.bin/forever start server.js'
warn:    --minUptime not set. Defaulting to: 1000ms
warn:    --spinSleepTime not set. Your script will exit if it does not stay up for at least 1000ms
info:    Forever processing file: server.js
```

### Stopping BootstrapCDN

```
[jmervine@bootstrapcdn01 ~]$ sudo bash -c 'cd ~bootstrapcdn/bootstrap-cdn; ./node_modules/.bin/forever list'
info:    Forever processes running
data:        uid  command       script    forever pid   id logfile         uptime
data:    [0] 9ZVR /usr/bin/node server.js 12998   14638    logs/server.log 0:0:2:22.975
```

### Restarting BootstrapCDN

```
[jmervine@bootstrapcdn01 ~]$ sudo bash -c 'cd ~bootstrapcdn/bootstrap-cdn; ./node_modules/.bin/forever restart server.js'
info:    Forever restarted process(es):
data:        uid  command       script    forever pid   id logfile                 uptime
data:    [0] enz- /usr/bin/node server.js 14670   14672    /root/.forever/enz-.log 0:0:0:6.853
[jmervine@bootstrapcdn01 ~]$
```

### Troubleshooting

> Work in progress.

```
$ ssh -p 2020 www.bootstrapcdn.com
# connection...

# 1. check for running node process
## should see something like...
[jmervine@bootstrapcdn01 ~]$ ps aux | grep node
root     12998  0.0  4.2 662168 21520 ?        Ssl  Feb25   0:00 /usr/bin/node /home/bootstrapcdn/bootstrap-cdn/node_modules/forever/bin/monitor server.js
root     13029  0.0  2.4 658800 12292 ?        Sl   Feb25   0:26 /usr/bin/node /home/bootstrapcdn/bootstrap-cdn/server.js
root     13031  0.0 16.0 945060 80480 ?        Sl   Feb25   0:02 /usr/bin/node app.js
jmervine 14470  0.0  0.1 103232   848 pts/0    S+   06:08   0:00 grep node
[jmervine@bootstrapcdn01 ~]$

## if not, restart

# 2. check for errors in logs
##
[jmervine@bootstrapcdn01 ~]$ sudo su - bootstrapcdn
[sudo] password for jmervine:
[bootstrapcdn@bootstrapcdn01 ~]$ cd ~/bootstrap-cdn/logs/
[bootstrapcdn@bootstrapcdn01 logs]$ less server.log

## Search for "Error" and/or "Trace:" near current time.
```

#### Common Errors

> Work in progress.

##### Missing `_maxcdn.yml`

In certain update conditions, `config/_maxcdn.yml` can get overwritten by the
sample version stored on github. When this occurs you'll see the following
error.

```
Trace: { statusCode: 500,
  data: '{"code":500,"error":{"message":"No consumer with consumer_key \\"KEY\\"","type"
:"RWS\\\\Exception"}}' }
    at /home/bootstrapcdn/bootstrap-cdn/routes/extras.js:49:21
    at /home/bootstrapcdn/bootstrap-cdn/node_modules/maxcdn/index.js:123:9
    at passBackControl (/home/bootstrapcdn/bootstrap-cdn/node_modules/maxcdn/node_module
s/oauth/lib/oauth.js:397:13)
    at IncomingMessage.<anonymous> (/home/bootstrapcdn/bootstrap-cdn/node_modules/maxcdn
/node_modules/oauth/lib/oauth.js:409:9)
    at IncomingMessage.EventEmitter.emit (events.js:117:20)
    at _stream_readable.js:920:16
    at process._tickCallback (node.js:415:13)
Trace: [SyntaxError: Unexpected token u]
    at /home/bootstrapcdn/bootstrap-cdn/routes/extras.js:26:21
    at fs.js:266:14
    at Object.oncomplete (fs.js:107:15)
```

To fix this, run the following as the `bootstrapcdn` user:

```
$ cp /home/bootstrapcdn/_maxcdn.yml /home/bootstrapcdn/bootstrap-cdn/config/_maxcdn.yml
```

##### `bind EADDRINUSE` on startup

Node throws a `bind EADDRINUSE` error when another process (node or otherwise)
is using the http port you're attempting to listen on. In our case, this means
there's currently an instance of BootstrapCDN already running. To fix, stop the
current running process and attempt to start the app again.

Sample log message:

```
Error: bind EADDRINUSE
    at errnoException (net.js:901:11)
    at net.js:1081:30
    at Object.1:1 (cluster.js:592:5)
    at handleResponse (cluster.js:171:41)
    at respond (cluster.js:192:5)
    at handleMessage (cluster.js:202:5)
    at process.EventEmitter.emit (events.js:117:20)
    at handleMessage (child_process.js:318:10)
    at child_process.js:392:7
    at process.handleConversion.net.Native.got (child_process.js:91:7)
```
