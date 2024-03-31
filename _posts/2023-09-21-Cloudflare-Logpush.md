---
title: Cloudflare Logpush to Grafana Loki Endpoint
date: 2023-09-21 12:00:00 -500
categories: [Cloudflare, Logpush]
tags: [log]     # TAG names should always be lowercase
---

[Cloudflare Logpush](https://developers.cloudflare.com/logs/about/)

## Set up Grafana

Go to [Grafana](https://grafana.com/) and set up a free account. \
Alternatively, you can host your own instance of Grafana [Install Grafana on Ubuntu](https://iamjjchang.github.io/Install-Grafana-on-Ubuntu/)


## Create a Worker

Create a new Worker with [Cloudflare Wrangler](https://developers.cloudflare.com/workers/wrangler/commands/)

```
wrangler generate YOUR_WORKER_NAME
```

**Index.js**
Edit the `index.js` file:
```
# + Change job label { "job": "cloudflare-logpush-http-requests" } in src/index.js as needed
```


```
export default {
    async fetch(request, env, ctx) {
        const contentEncoding = request.headers.get('content-encoding')
        const buf = await request.arrayBuffer();
        const enc = new TextDecoder("utf-8");
        if (contentEncoding === 'gzip') {
            // Decompress gzipped logpush body to json
            const blob = new Blob([buf])
            const ds = new DecompressionStream('gzip');
            const decompressedStream = blob.stream().pipeThrough(ds);
            const buffer = await new Response(decompressedStream).arrayBuffer();
            const decompressed = new Uint8Array(buffer)
            const ndjson = enc.decode(decompressed)
            const json = ndjson.split('\n')
            const lokiFormat = {
                "streams": [{
                    "stream": {
                        "job": "cloudflare-logpush-httprequests"
                    },
                    "values": []
                }]
            }
            json.forEach(element => {
                const date = element.EdgeStartTimestamp || new Date().getTime() * 1000000
                lokiFormat.streams[0].values.push([date.toString(), element])
            })
            // Post data to Grafana Loki
            return await fetch(`https://${env.LOKI_HOST}/loki/api/v1/push`, {
                method: "POST",
                body: JSON.stringify(lokiFormat),
                headers: {
                    "Content-Type": "application/json",
                    Authorization: 'Basic ' + btoa(`${env.LOKI_USER}:${env.LOKI_API_KEY}`)
                }
            });
        } else {
            // Initial pre-flight Logpush Request to confirm the integration check
            const compressed = new Uint8Array(buf);
            console.log(enc.decode(compressed).trim()) //{"content":"test","filename":"test.txt"}')
            const lokiFormat = {
                "streams": [{
                    "stream": {
                        "job": "cloudflare-logpush-httprequests"
                    },
                    "values": []
                }]
            }
            const unixnano = new Date().getTime() * 1000000
            const json = '{"content":"test","filename":"test.txt"}';
            lokiFormat.streams[0].values.push([unixnano.toString(), json])
            return await fetch(`https://${env.LOKI_HOST}/loki/api/v1/push`, {
                method: "POST",
                body: JSON.stringify(lokiFormat),
                headers: {
                    "Content-Type": "application/json",
                    Authorization: 'Basic ' + btoa(`${env.LOKI_USER}:${env.LOKI_API_KEY}`)
                }
            });
        }
    },
};
```

## Wrangler secret
```
wrangler secret put LOKI_USER # 3XXXXX
wrangler secret put LOKI_API_KEY # eyJxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx==
wrangler secret put LOKI_HOST # logs-prod-0xxxxx.grafana.net
```

## Upload the Worker
```
wrangler publish
```

## Logpush \
Get Log Fields
```
curl -s \
-H "X-Auth-Email: $EMAIL" \
-H "X-Auth-Key: $APIKEY" \
"https://api.cloudflare.com/client/v4/zones/$ZONE_ID/logpush/datasets/http_requests/fields" | jq -r '.result |
keys | join(",")')
```

You should see a comma separated list of available fields you can send to your destination
```
BotDetectionIDs,BotScore,BotScoreSrc,BotTags,CacheCacheStatus,CacheReserveUsed,CacheResponseBytes,
CacheResponseStatus,CacheTieredFill,ClientASN,ClientCountry,ClientDeviceType,ClientIP,ClientIPClass,
ClientMTLSAuthCertFingerprint,ClientMTLSAuthStatus,ClientRegionCode,ClientRequestBytes,ClientRequestHost,
ClientRequestMethod,ClientRequestPath,ClientRequestProtocol,ClientRequestReferer,ClientRequestScheme,
ClientRequestSource,ClientRequestURI,ClientRequestUserAgent,ClientSSLCipher,ClientSSLProtocol,ClientSrcPort,
ClientTCPRTTMs,ClientXRequestedWith,ContentScanObjResults,ContentScanObjTypes,Cookies,EdgeCFConnectingO2O,
EdgeColoCode,EdgeColoID,EdgeEndTimestamp,EdgePathingOp,EdgePathingSrc,EdgePathingStatus,EdgeRateLimitAction,
EdgeRateLimitID,EdgeRequestHost,EdgeResponseBodyBytes,EdgeResponseBytes,EdgeResponseCompressionRatio,
EdgeResponseContentType,EdgeResponseStatus,EdgeServerIP,EdgeStartTimestamp,EdgeTimeToFirstByteMs,
FirewallMatchesActions,FirewallMatchesRuleIDs,FirewallMatchesSources,JA3Hash,OriginDNSResponseTimeMs,OriginIP,
OriginRequestHeaderSendDurationMs,OriginResponseBytes,OriginResponseDurationMs,OriginResponseHTTPExpires,
OriginResponseHTTPLastModified,OriginResponseHeaderReceiveDurationMs,OriginResponseStatus,OriginResponseTime,
OriginSSLProtocol,OriginTCPHandshakeDurationMs,OriginTLSHandshakeDurationMs,ParentRayID,RayID,RequestHeaders,
ResponseHeaders,SecurityLevel,SmartRouteColoID,UpperTierColoID,WAFAction,WAFAttackScore,WAFFlags,WAFMatchedVar,
WAFProfile,WAFRCEAttackScore,WAFRuleID,WAFRuleMessage,WAFSQLiAttackScore,WAFXSSAttackScore,WorkerCPUTime,
WorkerStatus,WorkerSubrequest,WorkerSubrequestCount,WorkerWallTimeUs,ZoneName
```

## Create Logpush Job \ 
Variables
```
export ZONE_ID=yourzoneid.com
export EMAIL=your@email.com
export APIKEY=yourglobalAPIkey
export FIELD=BotDetectionIDs,BotScore,BotScoreSrc,BotTags,CacheCacheStatus,CacheReserveUsed,.....,.......,......
```

```
curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/logpush/jobs" \
-H "X-Auth-Email: $EMAIL" \
-H "X-Auth-Key: $APIKEY" \
-d '{
"name": "grafana-loki",
"logpull_options": "fields='$FIELDS'",
"destination_conf": "https://loki_logpush.myworker.workers.dev?host:example.com,dataset:http_requests",
"max_upload_bytes": 5000000,
"max_upload_records": 1000,
"dataset": "http_requests",
"enabled": true
}' 
```

## Test explorer on Grafana Loki
```
{job="<YOUR_LOKI_JOB_NAME>"} | json | __error__ = ""
```

# Troubleshooting

## Test Grafana Loki endpoint
Make you can first query your Grafana Loki endpoint
```
export LOKI_USER=3XXXXX
export LOKI_API_KEY=eyJxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx==
export LOKI_HOST=logs-prod-0xxxxx.grafana.net
curl -v -H "Content-Type: application/json" -XPOST -s \
"https://$LOKI_USER:$LOKI_API_KEY@$LOKI_HOST/loki/api/v1/push" --data-raw \
'{"streams": [{ "stream": { "foo": "bar2" }, "values": [ [ "1677514484485207000", "fizzbuzz" ] ] }]}'
```

## Test worker endpoint
Use command **wrangler tail** to view live logs or login to dashboard and view worker log stream
```
curl "https://loki_logpush.myworker.workers.dev" -X POST \
--data-raw '{"content":"test","filename":"test.txt"}' \
-H "Content-Type: application/json"
```

Check if [Log Retention](https://developers.cloudflare.com/logs/logpull/enabling-log-retention) is enabled:
```
curl -s -H "X-Auth-Email:<YOUR_EMAIL>" -H "X-Auth-Key:<GLOBAL_API_KEY>" GET "https://api.cloudflare.com/client
/v4/zones/<ZONE_ID>/logs/control/retention/flag" | jq .
```

## Enable Log Retention
```
curl -s -H "X-Auth-Email:<YOUR_EMAIL>" -H "X-Auth-Key:<GLOBAL_API_KEY>" POST "https://api.cloudflare.com/client
/v4/zones/<ZONE_ID>/logs/control/retention/flag" -d'{"flag":true}' | jq .
```