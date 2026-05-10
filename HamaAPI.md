# XML-based API
These page contains the *Documentation* of a `Hama DIR3100MS`.
The logged connection requests can be found [here](HamaData.md).

## Preliminary Information

### Time
The radio queries a NTP server at `time.wifiradiofrontier.com`.

Tampering DNS to use a different server works.

### Firmware Updates
See https://github.com/cweiske/frontier-silicon-firmwares, the radio queries a server at `update.wifiradiofrontier.com`.

### Host Name
The host names where the radio expects the XML-based API follow the pattern `<manufacturer>.wifiradiofrontier.com`. However, the API is the same for all different `<manufacturer>`. Today (year 2026), all domains are cnamed to `shim-pri.wifiradiofrontier.com` or `shim-sec.wifiradiofrontier.com` and share on IPv4.

Forging DNS requests is used to redirect `.wifiradiofrontier.com` to alternative API implementations like [Radio-API](https://github.com/KIMB-technologies/Radio-API).

### Supported Technologies
The Hama radio only supports IPv4 and does not support any HTTPS. Therefore, all streams and APIs need to be accessible via HTTP, HTTPS to HTTP proxies can help (as implemented in Radio-API).

The radio supports MP3 and other well known audio formats streams over HTTP. The radio also supports audio streams via HLS/M3U. The latter in contrast to newer JSON-based Radios.

## API Documentation
- Host (main)
	- http://hama.wifiradiofrontier.com
	- points via CNAME to http://pri.wifiradiofrontier.com
	- 185.71.184.114 (09. October 2018)
- Host (fallback)
	- http://hama2.wifiradiofrontier.com
	- points via CNAME to http://sec.wifiradiofrontier.com
	- 185.114.37.112 (09. October 2018)
- Server
	- Microsoft IIS
	- ASP.NET
- Normal HTTP 1.1 requests on port 80
- Server sends as ASP.NET session cookie, but radio never responds with it, always a new one
- `Content-Length`-Header is important, otherwise radio will not read the response!

### Request
```
	http://hama.wifiradiofrontier.com/setupapp/hama/asp/BrowseXML/<file>.asp?<params>
```
The `<file>` seems to define the type of the task, the dataset. While the `<params>` identify the clients etc.

- Sometimes `BrowseXML` is also used with lowercase letters `browsexml`
- The value `hama` in hostname and path changes based on the manufacturer of the radio

#### Parameter
The radio always adds the following parameters at the end of each URL: 
- `mac`: Is the radio's authentication token
	- Not related to MAC address
	- At least I didn't find the connection
- `dlang` The language to use in the API's response, translate strings/ text in XML
	- `ger` is German
	- `eng` is English
- `fver` The firmware version
	- mine is `8` 
	- `5` also seen
- `ven` The vendor of the radio.
	- `hama7`
	- `medion2` for some Medion radios
- These parameters are not added when requesting a stream url, so no leaking of MAC to radio stations

```
	&mac=c0d414e16b8a0789796e088387337fab&dlang=ger&fver=8&ven=hama7
```

Sometimes the two parameters `startItems=1&endItems=100` are added to specify parts of lists (only while browsing the list of all stations or podcast)

>
> Browsing a favorite group:
> ```
> http://hama.wifiradiofrontier.com/setupapp/hama/asp/browsexml/FavXML.asp?empty=&sFavName=<name-of-favorites-group>&startItems=1&endItems=100&mac=c0d414e16b8a0789796e088387337fab&dlang=ger&fver=8&ven=hama7
> ```

### Login
The first request on startup is the following URL, maybe it is to test the internet connection and server compatibility.
```
http://hama.wifiradiofrontier.com/setupapp/hama/asp/BrowseXML/loginXML.asp?token=0
```
The answer is always the string `<EncryptedToken>3a3f5ac48a1dab4e</EncryptedToken>`.

- This seems to be the only case, when the radio does not append more parameters, like MAC.
- Will be always the first sent request.
- API has to respond with `EncryptedToken`

### Startpage
The list shown after a successful connection is based on a XML response. Its the same file url as the login `loginXML.asp?gofile=&...`. The API identifies that it has to send the list of items based on the fact that the radio sent a MAC.

The response it a directory containing all items from the list menu of the internet radio.

### Other Lists/ Directories
All other lists are reachable by the URLs provided over the previous directory requests.
The radio know two default URLs, the one for the login (connection test) and the second to get the first list (directory).

### Searches
There are also some URLs, which provide information about shows/ stations if a `Show ID` or `Station ID` is known.

Search types:
- 3
	- Used to get information about a radio station by their ID
	- Used if item with type station has only 3 params and user wants to listen to it
	- `/Search.asp?sSearchtype=3&Search=333002`
	- Response is normal XML with one item (and `Previous`) and this is the full station with 11 attributes
- 5
	- Used to get information about a show/ podcast episode
	- Like stations, but for using podcasts/ episodes
	- `/Search.asp?sSearchtype=5&Search=P23740X1`

### Responses
- All responses are some type of XML:
	- No HTML entities supported
	- Use `&` and not `&amp;`
	- Only `<>` are reserved
	- Non ASCII-chars are escaped
		- some of the UTF-8 range seems to be supported 
		- octal unicode, e.g.:
			- ä = `\303\244`; as parts `"` and `a`
			- ü = `\303\274`
			- ö = `\303\266`
			- ß = `\303\237`
- Error messages are sometimes just plain strings.
- The `Content-Length` header is very important, without the radio does not parse the responses.

Most responses are build like:
```xml
<?xml
	version="1.0"
	encoding="UTF-8"
	standalone="yes"
?>
<ListOfItems>
	<ItemCount>
      	-1 
      </ItemCount>
	<Item>
		<ItemType>...</ItemType>
		<!-- more attributes, up to type -->
	</Item>
	<!-- more items -->
</ListOfItems>
```

- `<ListOfItems>`
	- the list of all items
	- `<ItemCount>`
		- should give the number of items in response
		- is sometimes `-1`, e.g. on startpage, first list
		- the `Previous` item type is uncounted
	- `<Item>`
		- each item in the list
		- `<ItemType>`
			- the type, e.g. `Dir`, `Previous`, `ShowOnDemand`, `ShowEpisode`, `Station`

#### ItemTypes

##### `Dir`
- represents a directory which can be opened and may contain more items
- has 4 parameters:
- `<Title>`
	- the title to show on the monitor
- `<UrlDir>`
	- the url, where the directory can be found (XML response at URL expected)
- `<UrlDirBackUp>`
	- backup url, on other server

##### `Previous`
- represents the place where to go, if back-button pressed
- always the first item in the XML answer, always existent
- has 4 parameter:
- `<Title>`
	- the title to show on the monitor
- `<UrlPrevious>`
	- the url where to go, if back-button is pressed (XML response at URL expected)
- `<UrlPreviousBackUp>`
	- backup url, on other server

##### `ShowOnDemand`
- represents a show/ podcast, which can be opened to see all episodes
- has 6 parameter:
	- `<ShowOnDemandID>`
		- the ID of the show in the system
		- is an integer, with the letter `P` as first character
		- format not relevant, but some radios store at max. 32 bytes/ chars
	- `<ShowOnDemandName>`
		- the name to show on the monitor
	- `<ShowOnDemandURL>`
		- the url where to get all episodes of the show (XML response at URL expected)
	- `<ShowOnDemandURLBackUp>`
	- `<BookmarkShow>`
		- radio will call URL to add this show to favorites

##### `ShowEpisode`
- represents a episode of a show
- has 12 parameters:
	- `<ShowEpisodeID>`
		- the ID of the episode
		- is the show ID, with the letter `X` and a counter appended
	- `<ShowName>`
		- the name of the show (not the name of the episode)
	- `<Logo>`
		- a url to a logo
		- image can be a jpg or png, should not be too large
	- `<ShowEpisodeName>`
		- the name of the episode
	- `<ShowEpisodeURL>`
		- the url where the media-file can be found (AUDIO)
		- This URL will not be request directly if the radio currently lists multiple episodes, i.e, displays a podcast.
			Radio will do a `Search.asp?sSearchtype=5&Search=<ShowEpisodeID>` and expect a XML response with one item of `ShowEpisode` type. From this response, the radio will request the value in `<ShowEpisodeURL>`.
	- `<BookmarkShow>`
		- url to bookmark this episode
	- `<ShowDesc>`
		- may begin with `[truncated]`
		- description of this episode
	- `<ShowFormat>`
		- some type of classification
		- e.g. `Radio Drama`
	- `<Lang>`
		- the language of the episode
		- e.g. `German`
	- `<Country>`
		- the country of origin
		- e.g. `Germany`
	- `<ShowMime>`
		- the mime-type
		- e.g. `MP3`

##### `Station`
- represents a radio station
- has 3 to 11 parameter:
	- `<StationId>`
		- the ID of the station
		- is an integer, but can also be string, radio stores max. 32 bytes/ chars
	- `<StationName>`
		- the name to show on display
	- `<StationUrl>`*
		- the url to get the data (AUDIO)
	- `<StationDesc>`*
		- a description of the station
	- `<Logo>`*
		- a url to get a logo
		- image can be a jpg or png, should not be too large
	- `<StationFormat>`*
		- some type of classification
	- `<StationLocation>`*
		- the location of the station
	- `<StationBandWidth>`*
		- some bandwidth information
		- e.g. `32`
	- `<StationMime>`*
		- the mine-type
		- e.g. `MP3`
	- `<Relia>`*
		- some reliability score
		- seen `5`

* optional fields: These fields are not part of a response while listing multiple radio stations at the same time.
When selecting one of the stations from a list ot play it, the radio does a `Search.asp?sSearchtype=3&Search=<StationId>` request. It then expects a XML with a single item of type `Station` and all 11 parameters. To stream the station's audio the radio will request `StationUrl`.

#### `Search`
- Opens a search box (text input)
- Search Type `sSearchtype=1`
	- `<SearchURL>`
		- The search URL (where the input get send to?; `sSearchtype=1`) 
	- `<SearchURLBackUp>`
	- `<SearchCaption>`
		- The name for the item
	- `<SearchTextbox>`
		- The default/ placeholder value
	- `<SearchButtonGo>`
		- The go button label
	- `<SearchButtonCancel>`
		- The cancel button label
- By now I don't know how the input is send back

## Known files and parameters
- /loginXML.asp?token=0
- /loginXML.asp?gofile=
- /FavXML.asp?empty=
- /navXML.asp?gofile=LocationLevelFour-Europe-Germany&bkLvl=LocationLevelFour-Europe-GermanyOObkLvlTypeOOl
- /navXML.asp?gofile=Radio
- /navXML.asp?gofile=ShowPod
- /AFavXML.asp?empty=
- /help.asp?gofile=
- /FavXML.asp?empty=&startItems=1&endItems=100
- /FavXML.asp?empty=&sFavName=<name>
- /FavXML.asp?empty=&sFavName=<name>&startItems=1&endItems=100
- /FavXML.asp?empty=gofile=
- /GetShowXML.asp?showid=P23740&ShowStatic=&gofile=favorite&sFavName=<name>&showname=<name of show>
- /AddFav.asp?empty=&showid=P23740
- /Search.asp?sSearchtype=3&Search=333002
- /Search.asp?sSearchtype=5&Search=P23740X1
- /AddFav.asp?empty=&showid=P23740
- /AFavXML.asp?empty=&startItems=1&endItems=100
- /RemoveFavs.asp?empty=&ID=P23740&sFavName=<name>
- /RemoveFavs.asp?empty=&ID=73373&ShowID=0&amp;sFavName=<name>
- /RemoveFavs.asp?empty=&ID=P28889&sFavName=<name>