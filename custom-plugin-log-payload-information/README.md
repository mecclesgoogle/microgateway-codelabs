# Develop a custom plugin to log information about the request and reponse

## Environment variables

_Assuming nodejs and npm is installed globally:_

1. Set the following environment variables
```
export NODE_MODULES_HOME=`npm list -g | head -1`/node_modules
export EDGEMICRO_HOME=${NODE_MODULES_HOME}/edgemicro
```

## Steps

2. Create the plugin shell
under $EDGEMICRO_HOME/plugins create a new directory with the name 'log-payload'
E.g.
`mkdir $EDGEMICRO_HOME/plugins/log-payload && cd $EDGEMICRO_HOME/plugins/log-payload` 

3. Run `npm init` and enter the defaults

4. Create a file called index.js and paste in the following code:
```
'use strict';
var os = require('os');

module.exports.init = function(config, logger, stats) {

  const formatHeaders = (headers) => {
    let requestHeadersString = "";
    for(var header in headers) {
      requestHeadersString += os.EOL;
      if ((/authorization/i).test(header)) {
        requestHeadersString += `${header}: ***`;
      } else {
        requestHeadersString += `${header}: ${headers[header]}`;
      }
    }
    return requestHeadersString;
  }

  return {

    onend_request: function(req, res, data, next) {
      logger.info(`Request Headers: ${formatHeaders(req.headers)}`);
      logger.info(`Request payload:${os.EOL}${data}`);
      next(null, data);
    },

    onend_response: function(req, res, data, next) {
      logger.info(`Response Headers: ${formatHeaders(res.getHeaders())}`);
      logger.info(`Response Status Code: ${res.statusCode}`);
      logger.info(`Response payload:${os.EOL}${data}`);
      next(null, data);
    },

    onerror_response: function(req, res, err, next) {
      logger.info(`Response Headers: ${formatHeaders(res.getHeaders())}`);
      logger.info(`Response Status Code: ${res.statusCode}`);
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
...
plugins:
    sequence:
      - accumulate-request
      - log-payload
      - accumulate-response
...
```

6. Restart Edge Microgateway

7. Call an API through Edge Microgateway. Once the API call is processed, you should be able to see additional information in the log files.

`$EDGEMICRO_LOGDIR/edgemicro-<hostname>-<uuid>-api.log`

For example:

```
...
1556252675337 info Request Headers:
host: localhost:8000
user-agent: curl/7.54.0
accept: */*
authorization: ***
content-length: 15
content-type: application/x-www-form-urlencoded
client_received_start_timestamp: 1556252675315
x-authorization-claims: ***
target_sent_start_timestamp: 1556252675333
client_received_end_timestamp: 1556252675336
1556252675337 info Request payload:
{"key":"value"}
...
1556253557024 info Response Headers:
x-powered-by: Apigee
access-control-allow-origin: *
x-frame-options: ALLOW-FROM RESOURCE-URL
x-xss-protection: 1
x-content-type-options: nosniff
content-type: application/json; charset=utf-8
etag: W/"7ea-+yc8UnqiiWEYi/+PtKzqAg"
date: Fri, 26 Apr 2019 04:39:16 GMT
via: 1.1 google
x-response-time: 372
1556250978352 info Response Status Code:
200
1556250978352 info Response payload:
{"headers":{"user-agent":"curl/7.54.0","accept":"*/*","authorization":"Bearer *","content-type":"application/x-www-form-urlencoded","client_received_start_timestamp":"1556250978158","x-authorization-claims":"*","target_sent_start_timestamp":"1556250978165","x-request-id":"9fff9420-67d6-11e9-849b-01f8caca77d9.3ed99ff0-67d7-11e9-849b-01f8caca77d9","x-forwarded-proto":"http","x-forwarded-host":"localhost:8000","host":"mocktarget.apigee.net","transfer-encoding":"chunked","x-cloud-trace-context":"4ab7babda6f7fac445225f39699d0647/551900080969811934","via":"1.1 localhost, 1.1 google","x-forwarded-for":"::1, 58.164.7.113, 35.227.194.212","connection":"Keep-Alive"},"method":"POST","url":"/","body":"{\"key\":\"value\"}"}
1556252675553 info Response Status Code: 200
...
```

## Caveats

This plugin does not consider [Data masking and hiding rules](https://docs.apigee.com/api-platform/security/data-masking). The developer is responsible for masking out sensitive headers or payload fragments in the custom plugin code. The code provided in the example already masks out authorization headers.
