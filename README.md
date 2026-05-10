# WiFi-RadioAPI
API on host `wifiradiofrontier.com` for radios with chips from *Frontier Smart*.

The services at `wifiradiofrontier.com` are currently provided by [*airable*](https://www.airablenow.com/tag/frontier-silicon/).

## Alternative API Implementation
There are multiple alternative API implementations.
- Both APIs (older XML-based and newer JSON-based):
	- https://github.com/KIMB-technologies/Radio-API 
		- My (KIMBtech's) implementation, as far as I know the most complete and feature rich implementation
		- Uses Radio-Browser.info as station list and supports custom stations and podcasts, streams from Nextcloud shares, ...
- Older XML-based API
	- https://github.com/milaq/YCast
	- https://github.com/coffeegreg/YTuner
	- https://github.com/compujuckel/librefrontier
- Newer JSON-based API
	- https://github.com/seife/frontier-airable
	- https://github.com/rhaamo/pyrable
	- https://github.com/Half-Shot/fairable


## API Documentation

### XML-based API
The API documentation is based on recordings and tests with:
- Manufacturer: Hama  
- Model: DIR3100MS

The documentation consist of
- [Documentation](HamaAPI.md)
- [Connection Logs](HamaData.md)

The documentations consists of the *daily usage* and how the radio gets information about radio stations, podcasts and how files and streams are played.
Documentation of the firmware update (and firmware files) can be found at: https://github.com/cweiske/frontier-silicon-firmwares

	
### JSON-based API
The API documentation is based on recordings and tests with:
- Manufacturer: Majority  
- Model: Touro

The documentation consist of
- [Documentation](MajorityAPI.md)
- [Connection Logs](MajorityData.md)

The documentations consists of the *daily usage* and how the radio gets information about radio stations, podcasts and how files and streams are played.

## Note
This is a private project and has no connections to Frontier Smart/ Frontier Nuvola/ Frontier Silicon nor airable.