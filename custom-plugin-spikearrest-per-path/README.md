# Develop a custom plugin to execute spikearrest for a specific path

## Environment variables

_Assuming nodejs and npm is installed globally:_

1. Set the following environment variables
```
export NODE_MODULES_HOME=`npm list -g | head -1`/node_modules
export EDGEMICRO_HOME=${NODE_MODULES_HOME}/edgemicro
```

## Steps

2. Create the plugin shell
under $EDGEMICRO_HOME/plugins create a new directory with the name 'spikearrest-a'
E.g.
`mkdir $EDGEMICRO_HOME/plugins/spikearrest-a && cd $EDGEMICRO_HOME/plugins/spikearrest-a` 

3. Run `npm init` and enter the defaults

4. Create a file called index.js and paste in the following code:
```
'use strict';

const SpikeArrest = require('volos-spikearrest-memory');

module.exports.init = function(config /*, logger, stats */) {

  const spikeArrest = SpikeArrest.create({config});
  const pathPattern = RegExp(config.pathPattern);

  const middlewareA = spikeArrest.connectMiddleware().apply({key : config.pathPattern});

  return {
    onrequest: function(req, res, next) {
      if (pathPattern.test(req.reqUrl.path)) {
        middlewareA(req, res, next); 
      } else {
        next();
      }
    }
  };

}

```

5. Define the plugin for the specific path you want to apply spikearrest to in 
`~/.edgemicro/<org>-<env>-config.yaml`, one level under edgemicro.
timeUnit, allow, and buffersize work in the same way as the standard spikearrest plugin.
**Important** pathPattern is a JavaScript regular expression so be sure to escape where necessary.

```
edgemicro:
...
spikearrest-a:
  timeUnit: minute
  allow: 5
  buffersize: 0
  pathPattern: v1\/customeraccounts
```

6. Enable the following plugins in `~/.edgemicro/<org>-<env>-config.yaml`.

**Note:** Any existing plugins such as oauth can be left how they are.

```
edgemicro:
...
  plugins:
    sequence:
      ...
      - spikearrest-a
      ...
...
```

7. Reload Edge Microgateway

8. Make several calls in quick succession to Edge Microgateway that matches the pathPattern provided in the configuration. Note that it starts returning HTTP 403 after the first couple of requests.

9. Now do the same for an API that does not match the pathPattern. It should *not* be throttled!

10. To achieve the same for additional paths, repeat steps 2-6, each time using a unique name for the plugin. e.g. "spikearrest-b", "spikearrest-c",  etc. Reload MGW at the end.

edgemicro:
...
spikearrest-a:
  timeUnit: minute
  allow: 5
  buffersize: 0
  pathPattern: v1\/customeraccounts
spikearrest-b:
  timeUnit: minute
  allow: 30
  buffersize: 0
  pathPattern: v1\/contacts
spikearrest-c:
  timeUnit: minute
  allow: 100
  buffersize: 0
  pathPattern: v2\/customeraccounts
```

## References
https://www.npmjs.com/package/volos-spikearrest-memory

