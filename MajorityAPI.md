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

This is the first request by the radio to the server.
It seems to be some kind of server compatibility check and the response needs to be quite exact.

So make sure to include `airable` and `index` in `id`. Also, the radio expect two urls/ content entries:
- The feeds (podcasts) site, a Directory Listing (below)
- The radios (stations) site, a Directory Listing (see below)

The response below is from Radio-APIs implementation, which points to two times to the same content entry (directory).
In doing so, we can afterwards have subdirectories for podcasts, radio stations and everything else. 

The radio will follow the url in the `url` field of each entry.

```json
{
    "id": [
        "airable",
        "directory",
        "index"
    ],
    "title": "Index",
    "url": "https://radio.example.com",
    "content": {
        "entries": [
            {
                "id": [
                    "frontiersmart",
                    "service",
                    "feed"
                ],
                "title": "Podcast",
                "url": "https://radio.example.com/index"
            },
            {
                "id": [
                    "frontiersmart",
                    "service",
                    "radio"
                ],
                "title": "Radio",
                "url": "https://radio.example.com/index"
            }
        ]
    }
}
```

### Request: Directory Listing
Shows the content of a directory. A directory can contain further directories or elements to play/ listen to.

- The second value of `id` tells the radio that this is a `directory`.
- The radio will follow the `urls` in the content entries, if they are of type `directory` (again second value in `id`).

Example from Radio-API index, e.g., `https://radio.example.com/index`

```json
{
    "id": [
        "frontiersmart",
        "directory",
        "list"
    ],
    "title": "Liste", // name/ title shown in radio
    "url": "https://radio.example.com/index?go=initial", // url to this page/ site
    "content": {
        "entries": [
            {
                "id": [
                    "frontiersmart",
                    "directory",
                    "podcast"
                ],
                "title": "Podcast", // name to click on
                "url": "https://radio.example.com/list?tid=3" // url will be opened by radio
		    								// (as 'directory' because 'directory' in id above)
            },
            {
                "id": [
                    "frontiersmart",
                    "directory",
                    "radio"
                ],
                "title": "Radiosender",
                "url": "https://radio.example.com/list?tid=1"
            },
            {
                "id": [
                    "frontiersmart",
                    "directory",
                    "radiobrowser"
                ],
                "title": "Radio-Browser",
                "url": "https://radio.example.com/radio-browser?by=none&term=none"
            },
            {
                "id": [
                    "frontiersmart",
                    "directory",
                    "guicodexxxx"
                ],
                "title": "GUI-Code: Zxxx",
                "url": "https://radio.example.com/?go=initial"
            }
        ]
    }
}
```

#### Entries: Radio
So one item representing a radio station in the `content.entries[]` array.

- Second item in `id` is radio, this defines the type!
- Radio will not follow the url in the `url` field!
	- It will concatenate the values in `id` with `/` as separator
	- So to open *My Station* the request will be sent to `https://radio.example.com/frontiersmart/radio/?sSearchtype=3&Search=1000` and expects to get a Radio Station Infos (below).

Example from Radio-API, we use http get values to be better compatible with the older XML-API.
```json
            {
                "id": [
                    "frontiersmart",
                    "radio", 
                    "?sSearchtype=3&Search=1000"
                ],
                "title": "My Station",
                "url": "https://radio.example.com/?sSearchtype=3&Search=1000",
                "images": [
                    {
                        "url": "https://radio.example.com/media/default.png",
                        "size": [
                            150,
                            150
                        ],
                        "type": "cover"
                    }
                ]
            }
```

#### Entries: Podcast
There is a `feed` type, but for Radio-API a podcast is a `directory` of episodes, a this is easier to handle.
So we use `directory` when listing podcasts as entries, but the urls will respond with `feed` types, see Podcast Episodes (below).

### Request: Radio Station Infos
The radio opened an url of the radio type, so second value of `id` is `radio`.

- The image will be shown by radio and Oktiv app.
- The `url` in `streams` must not be a media stream url, instead there is one more level of JSON API (see Play Stream) below
- Only the value for `url` in `stream` seems to be relevant

```json
{
    "id": [
        "frontiersmart",
        "radio",
        1000
    ],
    "title": "Some station title",
    "url": "https://radio.example.com/?sSearchtype=3&Search=1000",
    "description": "some description",
    "images": [
        {
            "url": "https://radio.example.com/media/default.png",
            "size": [
                150,
                150
            ],
            "type": "cover"
        }
    ],
    "streams": [
        {
            "url": "https://radio.example.com/?sSearchtype=3&Search=1000&play",
            "codec": {
                "name": "MP3",
                "bitrate": 64
            },
            "reliability": 1
        }
    ]
}
```

### Request: Play Stream
This is a json response of type `redirect` (second value in `id`) and here we can link the actual media stream.

- The media stream is in `url`
- The `content` contains the values of the, e.g., Radio Station Infos (above) of the stream

```json
{
    "id": [
        "frontiersmart",
        "redirect",
        1000
    ],
    "title": "Play Stream",
    "url": "https://my-stream-server.example.com/some/radio/station.mp3",
    "codec": {
        "name": "MP3",
        "bitrate": 64
    },
    "content": {
        "title": "Some station title",
        "url": "https://radio.example.com/?sSearchtype=3&Search=1000",
        "description": "some description",
        "images": [
            {
                "url": "https://radio.example.com/media/default.png",
                "size": [
                    150,
                    150
                ],
                "type": "cover"
            }
        ],
        "streams": [
            {
                "url": "https://radio.example.com/?sSearchtype=3&Search=1000&play",
                "codec": {
                    "name": "MP3",
                    "bitrate": 64
                },
                "reliability": 1
            }
        ]
    }
}
```

### Request: Podcast Episodes 
The radio opened an url of the decretory (as Radio-API does it) type, but second value of `id` is `feed` to inform radio about this being episodes of a podcast.

- Each entry is of type episode
- Episodes are similar to radio stations
- Again, there is an url for only fetching the episode details (here radio seems to follow the link)
- And a stream url, this stream url needs to be a json redirect like with stations, Play Stream (see above)

```json

{
    "id": [
        "frontiersmart",
        "feed",
        3000
    ],
    "title": "Podcast name",
    "url": "https://radio.example.com/list?tid=3&id=3000",
    "content": {
        "entries": [
            {
                "id": [
                    "frontiersmart",
                    "episode",
                    "?sSearchtype=5&Search=3000X1"
                ],
                "title": "Some Episode Name",
                "url": "https://radio.example.com/?sSearchtype=5&Search=3000X1",
                "description": "Some description",
                "images": [
                    {
                        "url": "https://radio.example.com/image.php?hash=....",
                        "size": [
                            150,
                            150
                        ],
                        "type": "cover"
                    }
                ],
                "streams": [
                    {
                        "url": "https://radio.example.com/?sSearchtype=5&Search=3000X1&play",
                        "codec": {
                            "name": "MP3",
                            "bitrate": 64
                        },
                        "reliability": 1
                    }
                ]
            },
            {
                "id": [
                    "frontiersmart",
                    "episode",
                    "?sSearchtype=5&Search=3000X2"
                ],
		    // next episode, ...
            }
        ]
    }
}
```

#### Single Episode
```json
{
    "id": [
        "frontiersmart",
        "redirect",
        "3000X200"
    ],
    "title": "The title",
    "url": "https://radio.example.com/?sSearchtype=5&Search=3000X200",
    "description": "Some description",
    "images": [
        {
            "url": "https://radio.example.com/image.php?hash=...",
            "size": [
                150,
                150
            ],
            "type": "cover"
        }
    ],
    "streams": [
        {
            "url": "https://radio.example.com/?sSearchtype=5&Search=3000X200&play",
            "codec": {
                "name": "MP3",
                "bitrate": 64
           },
            "reliability": 1
        }
    ]
}
```

#### Play Episode
Like the Play Stream (see above)

```json

{
    "id": [
        "frontiersmart",
        "redirect",
        "3000X200"
    ],
    "title": "Episode abspielen",
    "url": "https://my-stream.example.com/some-file-mp.mp3",
    "codec": {
        "name": "MP3",
        "bitrate": 64
    },
    "content": {
        "title": "The title",
        "url": "https://radio.example.com/?sSearchtype=5&Search=3000X200",
        "description": "Some description",
        "images": [
            {
                "url": "https://radio.example.com/image.php?hash=...",
                "size": [
                    150,
                    150
                ],
                "type": "cover"
            }
        ],
        "streams": [
            {
                "url": "https://radio.example.com/?sSearchtype=5&Search=3000X200&play",
                "codec": {
                    "name": "MP3",
                    "bitrate": 64
                },
                "reliability": 1
            }
        ]
    }
}

```

## Simulating a request
```bash
	curl -k https://radio.example.com \
		-H "Authorization: $(echo -n "AA00BB11CC22:$(echo -n "something"|md5)"|base64 )" \
		-H "Accept-Language: de-DE" \
		-H "Host: airable.wifiradiofrontier.com" 
```