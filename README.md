# Search Facebook events by location

As Facebook has discontinued the FQL query API for all apps created after 2014-04-30, it has gotten much more complicated to get public Facebook events by passing a location.

This implementation uses regular Facebook Graph API calls in a three-step approach to get the events:

1. Search for places in the radius of the passed coordinate and distance (`/search?type=place&q={query}&center={coordinate}&distance={distance}`)
2. Use the places to query for their events in parallel (`/?ids={id1},{id2},{id3},...`)
3. Unify, filter and sort the results from the parallel calls and return them to the client

###Known limitations

* The Graph API has some "instabilities" with search results. It's possible that the amount of results returned can vary between calls within seconds
* The `/search` endpoint "magically" limits the number of results, independent from the `distance` used (larger distance doesn't guarantee more results)
* [Rate limiting](https://developers.facebook.com/docs/graph-api/advanced/rate-limiting) will apply, but I experienced no call blocks within a reasonable amount of service requests. Be aware that the way this application works, there are potentially hundreds of (counted) Graph API calls per request to `/events`.

## Installation

### As NPM package

The module can be installed via 

`npm install facebook-events-by-location-core --save`

### Git

To clone the repository, use

`git clone https://github.com/tobilg/facebook-events-by-location-core.git`

## Usage

The basic usage pattern of this module is the following:

```
var EventSearch = require("facebook-events-by-location-core");

var es = new EventSearch({
    "lat": 40.710803,
    "lng": -73.964040
});

es.search(function (error, events) {
    if (error) {
        console.error(JSON.stringify(error));
    } else {
        console.log(JSON.stringify(events));
    }
});
```

### Access Tokens

The above example expects that the `ACCESS_TOKEN` environment variable is set to a valid App Access Token. Otherwise, you need to specify the `accessToken` parameter with the request.

### Parameters

Mandatory parameters are the following:

* `lat`: The latitude of the position/coordinate the events shall be returned for
* `lng`: The longitude of the position/coordinate the events shall be returned for

Non-mandatory parameters

* `query`: The term(s) on which you want to narrow down your *location search* (this only filters the places, not the events itself!).
* `access_token`: The **App Access Token** to be used for the requests to the Graph API.
* `distance`: The distance in meters (it makes sense to use smaller distances, like max. 2500). Default is `100`.
* `sort`: The results can be sorted by `time`, `distance`, `venue` or `popularity`. If omitted, the events will be returned in the order they were received from the Graph API.
* `version`: The version of the Graph API to use. Default is `v2.5`.

### Sample output (shortened)

```
{
	"events": [{
		"id": "163958810691757",
		"name": "3Bridge Records presents inTRANSIT w/ David Kiss, Deep Woods, Eric Shans",
		"coverPicture": "https://scontent.xx.fbcdn.net/t31.0-8/s720x720/13679859_10153862492796325_8533542782240254857_o.jpg",
		"profilePicture": "https://scontent.xx.fbcdn.net/v/t1.0-0/c133.0.200.200/p200x200/13872980_10153862492796325_8533542782240254857_n.jpg?oh=a46813bbf28ad7b8bffb88acd82c7c71&oe=581EF037",
		"description": "Saturday, August 20th.\n\nJoin the 3Bridge Records team for another night of sound and shenanigans - as we send Deep Woods & David Kiss out to Burning Man & belatedly celebrate Slav Ka's debut release on the label - \"Endless\" - out May 14th, featuring a remix by Mr. Shans.\n\nDavid Kiss (House of Yes)\nhttps://soundcloud.com/davidkiss\n\nDeep Woods (3Bridge Records)\nhttps://soundcloud.com/deep-woods\n\nEric Shans (3Bridge Records)\nhttps://soundcloud.com/eric-shans\n\nSlav Ka (3Bridge Records)\nhttps://soundcloud.com/slinkyslava\n\nFree before 12, $10 after (+ 1 comp well drink). $5 presale available on RA.\n\nhttps://www.residentadvisor.net/event.aspx?863815\n\nStay dope, Brooklyn.",
		"distance": "203",
		"startTime": "2016-08-20T22:00:00-0400",
		"timeFromNow": 481946,
		"stats": {
			"attending": 44,
			"declined": 3,
			"maybe": 88,
			"noreply": 1250
		},
		"venue": {
			"id": "585713341444399",
			"name": "TBA Brooklyn",
			"coverPicture": "https://scontent.xx.fbcdn.net/v/t1.0-9/s720x720/13932666_1397749103574148_4391608711361541993_n.png?oh=2d82be3a458d1ce9ac8fab47cdbc6e26&oe=585E6545",
			"profilePicture": "https://scontent.xx.fbcdn.net/v/t1.0-1/p200x200/12049351_1300865083262551_8221231831784471629_n.jpg?oh=a30798841ad60dfe5cfabaa4e803c3ad&oe=5854DFB9",
			"location": {
				"city": "Brooklyn",
				"country": "United States",
				"latitude": 40.711217064583,
				"longitude": -73.966384349735,
				"state": "NY",
				"street": "395 Wythe Ave",
				"zip": "11249"
			}
		}
	},
	... 
	],
	"metadata": {
		"venues": 1,
		"venuesWithEvents": 1,
		"events": 4
	}
}
```