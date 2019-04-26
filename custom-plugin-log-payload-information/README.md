# Develop a custom plugin to log information about the request and reponse

## Environment variables

_Assuming nodejs and npm is installed globally:_

1. Set the following environment variables
export NODE_MODULES_HOME=`npm list -g | head -1`/node_modules
export EDGEMICRO_HOME=${NODE_MODULES_HOME}/edgemicro
export EDGEMICRO_KEY=<key>
export EDGEMICRO_SECRET=<secret>


2. Create the plugin shell
under $EDGEMICRO_HOME/plugins create a new directory with the name 'log-payload'
E.g.
`mkdir $EDGEMICRO_HOME/plugins/log-payload && cd $EDGEMICRO_HOME/plugins/log-payload` 

3. Run `npm init` and enter the defaults

4. Create a file called index.js and paste in the following code:
```
'use strict';
var os = require('os');
const util = require('util');

module.exports.init = function(config, logger, stats) {

  return {

    onend_request: function(req, res, data, next) {
      logger.info(`Request Headers:${os.EOL}${util.inspect(req.headers, false, null, true)}`);
      logger.info(`Request payload:${os.EOL}${data}`);
      next(null, data);
    },

    onend_response: function(req, res, data, next) {
      logger.info(`Response Status Code:${os.EOL}${res.statusCode}`);
      logger.info(`Response payload:${os.EOL}${data}`);
      next(null, data);
    },

    onerror_response: function(req, res, err, next) {
      logger.info(`Response Status Code:${os.EOL}${res.statusCode}`);
      logger.info(`Response payload:${os.EOL}${data}`);
      next();
    }
   
  };
}
```

5. Enable the following plugins in `~/.edgemicro/<org>-<env>-config.yaml`.
**Note:** Any existing plugins such as oauth can be left how they are.
**Note:** The order of the plugins is important because the data must be accumulated before the log-payload policy executes. In the response execution, plugins are exected in reverse order so we need to place accumulate-response after log-payload.
```
plugins:
    sequence:
      - accumulate-request
      - log-payload
      - accumulate-response
```

