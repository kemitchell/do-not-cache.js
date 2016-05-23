```javascript
var assert = require('assert')
var http = require('http')

var doNotCache = require('./')

var receivedResponse = false

http.createServer()
  .on('request', function(request, response) {
    doNotCache(response)
    response.end() })
  .listen(0 /* random high port */, function() {
    var server = this
    var port = server.address().port
    http.request({ port: port })
      .on('response', function(response) {
        var headers = response.headers
        receivedResponse = true
        // RFC 1945 (HTTP 1.0)
        assert.equal(headers['pragma'], 'no-cache') // Section 10.12
        assert.equal(headers['expires'], '0') // 10.7
        // RFC 2616 (HTTP 1.1)
        assert(headers['cache-control'].includes('no-cache')) // Section 14.9.1
        assert(headers['cache-control'].includes('no-store')) // 14.9.2
        assert(headers['cache-control'].includes('must-revalidate')) // 14.9.4
        server.close() })
      .end() })

process.on('exit', function() {
  assert.equal(receivedResponse, true)
  console.log('Tests passed') })
```
