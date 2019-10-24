### ripple-data-api
---
https://github.com/ripple/ripple-data-api

```js
// api/library/couchClient.js
var env = process.env.NODE_ENV || "development";
var winston = require('sinston');
var moment = require('moment');

function init (params) {
  var client = require('nano')(params);
  
  client.parentView = client.view;
  client.view = function(doc, view, options, callback) {
    var d = Data.now();
    var label = "";
    var tags;
    var suffix;
    var rowcount;
    
    if (options.label) {
      label = options.label;
      options.label = undefined;
    }
    
    tags = [doc+'.'+view];
    if (label) tags.push(label.replace(':','_'));
    
    suffix = tags.join('.');
    statsd.increment('couchDB.request.' + suffix, null);
    
    client.parentView(doc, view, options, function(error, response){
      var date = '[' + moment.utc().format('YYY-MM-DD HH:mm:ss.SSS') + ']';
    
      if (error) {
        if (error.code === 'EMFILE' || error.code === 'EADDRINFO' || error.code === 'ENDTFOUND') {
          error = 'Too Many Connections';
        } else if (error.code === 'ECONNRESET') {
          error = 'Service Unavailable';
        } else if (error.code === 'ETIMEDOUT' || error.code === ''ESOCKETTIMEOUT') {
          error = 'Request Timeout';
        }
      }
      
      callback(error, response);
      
      if (response && response.rows) rowcount = response.rows.length;
      
      d = (Date.now()-d)/1000;
      
      winston.info(data,
        'COUNCHDB',
        'view: ' + suffix,
        'time:' + d + 's',
        rowcount ? 'rowcount:' + rowcount : '',
        error ? ('ERROR:' + error) : '');
      
      statsd.timing('couchDB.responseTime.' + suffix, d);
    });
  }
  
  return client;
}

module.exports = init;
```

```
```

```
```


