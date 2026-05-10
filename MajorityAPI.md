# JSON-based API
These page contains the *Documentation* of a `Majority Touro`.
The logged connection requests can be found [here](MajorityData.md).

## Preliminary Information

### Time
The radio queries a NTP server at `time.wifiradiofrontier.com`.

Tampering DNS to use a different server works.

### Firmware Updates
Radio seems to query a server at `update.wifiradiofrontier.com`, see also https://github.com/cweiske/frontier-silicon-firmwares.

### Host Name
The host name where the radio expect the JSON-based API is `airable.wifiradiofrontier.com`. The `A` record points to `49.13.34.80` (May 2026) and this IP is also serves the older XML-API.

Forging DNS requests is used to redirect `.wifiradiofrontier.com` to alternative API implementations like [Radio-API](https://github.com/KIMB-technologies/Radio-API).

### Supported Technologies
The Majority radio supports HTTPS and requires the API to be served via HTTPS. However, the radio does not validate any SSL certificates, so self-signed certificates work.
Beside not verifying certificate chains, also common names are not verified.
As HTTPS is supported, the radio can stream from HTTPS urls, but will also stream from HTTP.

I do not know if the radio supports IPv6, as the DNS records are all IPv4 and my DNS tampering uses only IPv4. As it is not possible to set IPv6 DNS servers in the radio's settings, I assume no IPv6 support.

The radio supports MP3 and other well known audio formats streams over HTTP. The radio does **not support** audio streams via **HLS/M3U**. The latter in contrast to older XML-based Radios.

## API Documentation
- Host (main)
	- https://airable.wifiradiofrontier.com
	- 49.13.34.80 (10. May 2026)
- Normal HTTP 1.0 requests via SSL/ TLS on port 443
- SSL certificate is not checked/ verified

### Request Headers
HTTP headers used by the radio:

- `Authorization: base64(<Radio-ID>:<??>)`
	- The authentication mechanism of the radio, slightly inspired by HTTP basic auth.
	- There is a Radio-ID, also visible in the radio's web interface, that provides the *username*. After a `:` (for separation) some kind of token (32 chars, hex range) is provided as *password*.
	- I the assume the token being an md5 hash, it seems to be time-based, i.e., it changes over time for requests to the same URL.
- `Accept-Language: de-DE` 
	- The language of the server should respond in, e.g., `de-DE`, `en-US`.

More headers sent by the radio:
- `Accept: audio/aac;mp3;dash`: Even if the radio expects JSON
	- The `dash` may tell us that the radio supports DASH instead of HLS/M3U?
- `User-Agent: ir-cui-...`: Radio's firmware version


### Request: Airable Directory
```
	https://airable.wifiradiofrontier.com/
```

TODO: soon :D
