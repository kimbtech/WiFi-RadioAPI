# Hama API Documentation

> Model DIR3100MS

## WiFi Radio Frontier API

- Host
	- http://hama.wifiradiofrontier.com
	- points via CNAME to http://pri.wifiradiofrontier.com
	- 185.71.184.114 (09. October 2018)
- Host II
	- http://hama2.wifiradiofrontier.com
	- points via CNAME to http://sec.wifiradiofrontier.com
	- 185.114.37.112 (09. October 2018)
- Server
	- Microsoft IIS
	- ASP.NET
- Normal HTTP 1.1 Requests on Port 80
- Server sets as ASP.NET Session Cookie, but Radio never responds with it, always a new one

### Request
```
	http://hama.wifiradiofrontier.com/setupapp/hama/asp/BrowseXML/<file>.asp?<params>
```
The `<file>` seems to define the type of the task, the dataset. While the `<params>` identifie the Clients etc.

Sometimes `BrowseXML` is also used in small letters: `browsexml`.

#### Parameter
The radio always adds the following parameter at the end of each URL: 
- `mac` Seems to be the Radio Authetication Token
	- Not related to MAC-Adress
	- At least I didn't find the connection
- `dlang` The Language for the answer, Translation of Strings in XML
	- `ger` is german
	- `eng` is english
- `fver` The firmwareversion
	- mine is `8` 
	- `5` also seen
- `ven` The vendor of the radio.
	- `hama7`
	- `medion2` for some medion radios

```
	&mac=c0d414e16b8a0789796e088387337fab&dlang=ger&fver=8&ven=hama7
```

Somtimes the two paramter `startItems=1&endItems=100` are added to specifie parts of lists (only while browsing the list of all stations, podcast)

>
> Browsing a favorite group:
> ```
> http://hama.wifiradiofrontier.com/setupapp/hama/asp/browsexml/FavXML.asp?empty=&sFavName=<name-of-favorites-group>&startItems=1&endItems=100&mac=c0d414e16b8a0789796e088387337fab&dlang=ger&fver=8&ven=hama7
> ```

### Login
The first request on startup is the following URL, maybe it is to test the internet-connection.
```
http://hama.wifiradiofrontier.com/setupapp/hama/asp/BrowseXML/loginXML.asp?token=0
```
The answer is always the string `<EncryptedToken>3a3f5ac48a1dab4e</EncryptedToken>`.

- This seems to be the only case, when the radio does not append more parameters.

### Startpage
The list show after a successfull connection is just an XML-response. Its the same file as the login `loginXML.asp?gofile=&...`.
The response it a directory containing all item from the list menu of the internet radio.

### Other Lists
All other lists are reachable by the URLs provided over the previous directory requests.
So it seems as the radio only has to know two URLs, the one for the login (connection test)
and the second to get the first list (directory).

### Searches
There are also some URLs, which provide information about shows/ stations if the Show ID or Station ID is known.

Search-Types:
- 3
	- used to get information about a radio statio by their ID
	- used if item with type station has only 3 params and user wants to listen to it
	- `/Search.asp?sSearchtype=3&Search=333002`
	- answer is normal XML with one item (and Previous) and this is the full station with 11 parameter
- 5
	- used to get information about a show/ podcast episode
	- like stations, but using podcasts/ episodes
	- `/Search.asp?sSearchtype=5&Search=P23740X1`

### Responses
- All responses are some type of XML.
- Error messages are sometimes just plain strings.

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
		<!-- more values, up to type -->
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
- has 4 parameter:
- `<Title>`
	- the title to show on the monitor
- `<UrlDir>`
	- the url, where the directory can be found (XML)
- `<UrlDirBackUp>`
	- backup url, on other server

##### `Previous`
- represents the place where to go, if back-button pressed
- always the first item in the XML answer, always existent
- has 4 parameter:
- `<Title>`
	- the title to show on the monitor
- `<UrlPrevious>`
	- the url where to go, if back-button is pressed (XML)
- `<UrlPreviousBackUp>`
	- backup url, on other server

##### `ShowOnDemand`
- represents a show/ podcast, which can be opend to see all episodes
- has 6 parameter:
	- `<ShowOnDemandID>`
		- the ID of the show in the system
		- is an integer, with the letter `P` as first character
	- `<ShowOnDemandName>`
		- the name to show on the monitor
	- `<ShowOnDemandURL>`
		- the url where to get all episodes of the show (XML)
	- `<ShowOnDemandURLBackUp>`
	- `<BookmarkShow>`
		- will be called, it this show should be added to favorites

##### `ShowEpisode`
- represents a episode of a show
- has 12 paramter:
	- `<ShowEpisodeID>`
		- the ID of the episode
		- is the show ID, with the letter `X` and a counter appended
	- `<ShowName>`
		- the name of the show (not the name of the episode)
	- `<Logo>`
		- a url to a logo, (IMAGE), empty for my radio
	- `<ShowEpisodeName>`
		- the name of the episode
	- `<ShowEpisodeURL>`
		- the url where the media-file can be found (AUDIO)
	- `<BookmarkShow>`
		- url to bookmark this episode
	- `<ShowDesc>`
		- may begin with `[truncated]`
		- description of this episode
	- `<ShowFormat>`
		- some type of cassification
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
- represents a radio statio
- has 3 to 11 prameter:
	- `<StationId>`
		- the ID of the station
		- is an integer
	- `<StationName>`
		- the name to show on display
	- `<StationUrl>`*
		- the url to get the data (AUDIO)
	- `<StationDesc>`*
		- a description of the station
	- `<Logo>`*
		- a url to get a logo (IMAGE)
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

* optional fields

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

## Other
- Radio seems to use IPv4 only, also if IPv6 available
- Radio uses no SSL/TLS encryption, seems not capable
- non ASCII-chars are escaped
	- octal unicode
	- e.g.
		- ä = `\303\244`; as parts `"` and `a`
		- ü = `\303\274`
		- ö = `\303\266`
		- ß = `\303\237`