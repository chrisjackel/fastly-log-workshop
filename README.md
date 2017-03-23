# fastly-log-workshop
Code samples for Altitude NYC 2017


## Useful things to log

```
resp.http.Set-Cookie
server.datacenter
server.identity
server.region
if(req.http.Fastly-FF, "1", "0")
resp.http.X-Cache
fastly_info.state
regsub(fastly_info.state, "-.*", "")
obj.hits
geoip.continent_code
geoip.country_code
geoip.region (code for region within country)
req.restarts
```

--- Img prefix selector from UI ---

## Examples of different prefixes

`<134>2017-03-19T13:13:33Z cache-jfk8147 foobarlogs2[453796]: {"client_ip":"207.237.0.218","req_url":"/"}`
`<134>1 2017-03-19T13:13:33Z cache-jfk8147 - 453796 foobarlogs2 - {"client_ip":"207.237.138.218","req_url":"/"}`
`114 <134>1 2017-03-19T13:13:33+00:00 cache-jfk8147 - 453796 foobarlogs2 - {"client_ip":"207.237.138.218","req_url":"/"}`
`{"client_ip":"207.237.138.218","req_url":"/"}`

--- JSON logs

Sample hand-coded (set prefix to 'blank')

```
log "syslog 60idOs66l4AbzuqvgBypS2 foobar logs :: 
{"
  {""client_ip":""}    client.ip   {"","}
  {""req_url":""}      req.url     {"""}
"}";

```

Better still, use

https://github.com/fastly/vcl-json-generate




## Conditional Logging

### Only log errors

```
if (beresp.status == 500 || beresp.status == 503) {
     log {"syslog …etc… "};
} 
```

### Random sample of logs

```
if (randombool(std.atoi(table.lookup(logging, "percentage"))), 100) {
    log {"syslog <service_ID> <endpoint-name> :: "} var.logstr;
}
```

Then, to modify the dictionary item, use the edge dictionary API

```
curl -X PATCH -H "Fastly-Key: <api_token>" -d "item_value=5" "https://api.fastly.com/service/<service_id>/dictionary/<dict_id>/item/percentage"
```

### Only log specific URLs (via dictionary)

```
 table panic_mode_logging {
“/my_broken_url” : “1”
}

if (table.lookup(panic_mode_logging, req.url)) {
   log syslog 4rqBj4oy1YChTPgdapiWS4 gcs-test :: 
}
```


### Shard across multiple endpoints

```
  declare local var.logstr STRING;

  set var.logstr = …;

  declare local var.endpoint STRING;
  set var.endpoint = randomstr(1, "1234");

  if (var.endpoint == "1") {
    log {"syslog 4rqBj4oy1YChTPgdapiWS4 gcs-test1 :: "} var.logstr;
  } else if (var.endpoint == "2") {
    log {"syslog 4rqBj4oy1YChTPgdapiWS4 gcs-test2 :: "} var.logstr;
  } else if (var.endpoint == "3") {
    log {"syslog 4rqBj4oy1YChTPgdapiWS4 gcs-test3 :: "} var.logstr;
  } else { /* only "4" left */
    log {"syslog 4rqBj4oy1YChTPgdapiWS4 gcs-test4 :: "} var.logstr;
  }
  ```
  
  
  


Appendix A: URLs
Dogs of Fastly (with lots of Luna) 
https://www.instagram.com/explore/tags/dogsoffastly/

All streaming logging documentation
https://docs.fastly.com/guides/streaming-logs/

Log format variables:
https://docs.fastly.com/guides/streaming-logs/custom-log-formats#version-2-log-format

VCL request flow debugging:
https://www.fastly.com/blog/how-solve-anything-vcl-part-1-collecting-data-edge

Changing log line prefixes:
https://docs.fastly.com/guides/streaming-logs/changing-log-line-formats
