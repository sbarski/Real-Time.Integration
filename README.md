# Real-Time Platform Integration

The Real-Time platform is designed to easily integrate with various content consuming third-party systems. Real-Time makes an extensive use of well-known industry standards such [atom](http://tools.ietf.org/html/rfc4287)/[rss](http://validator.w3.org/feed/docs/rss2.html) feeds to transmit data and notify other systems of changes and events. 

This document gives a high level overview of the steps needed to successfully integrate Real-Time and a content consumption platform. It assumes basic knowledge of http, atom/rss and [json](http://tools.ietf.org/html/rfc7159).  

Please note that the following information is subject to change while the development of the Real-Time platform takes place. Also, sample URIs given below might not include the domain and/or the URI scheme. The production domain will be given at a later date. The URI scheme can be assumed to be HTTPS in all circumstances.  

## Taxonomy

Third-party consuming platforms (**clients**) use the Real-Time integration API to access/download premium content produced by content creators on the platform.

Content creators using the Real-Time platform publish content within a **channel** which groups content of a related topic (for example, soccer or tennis).

Each item published within a channel is called a **story** and comprises a sequentially ordered set of rich content pieces, including:

 - headings
 - text
 - images
 - audio
 - video
 - social media references

Each client that integrates with Real-Time is able to push end-users **comments** relating to each story, back into the platform. These comments may be moderated (depending on the story's comment mode configured by the content creator), and are then made available to clients.

A set of atom/rss **feeds** is made available to clients, such that these systems can automatically discover stories that are published/modified/unpublished. These feeds can be filtered in a variety of ways, such as accessing stories by channel.

## Pull vs Push

Currently, the API implements a pull-based model, where clients can poll the feeds and pull required content at their convenience.

A push-based approach is under active development and will be documented soon. In this model, much of the feed and content structure described below will be used, where a feed of change notifications will be actively pushed to subscribed clients.


## Authentication

Clients authenticate with Real-Time using a bearer token that must be included in the header of every request (apart from the initial login). 


### Initial Handshake

When joining the Real-Time platform, a client organisation is provided a username and a password that must be used during an initial handshake to authenticate with the platform. The response from this initial handshake will include a bearer token that the client will then need to include with each subsequent request. 

To conduct the first handshake with Real-Time - the client will need to perform a POST request to '/api/token'. 
The content-type must be set to 'application/x-www-form-urlencoded'. 

The following must be included in the body of the request:
* client_id (set this to 'server')
* grant_type (set this to 'password')
* password (set this to your password)
* username (set this to your username)
 
### Subsequent Requests

The initial request returns a bearer token that must be included in inside the Authorization header of each subsequent request.

A sample authorization header with a bearer token may look like this:

```
Authorization:Bearer nr24jianJanrI3EnVx6RjTa2Xp-UQzH9XXP4Vuj8gnn43471QHalCtQVH9ZqG5JOsUTeJqBHw3t4UCmfH5AqHYfPRufKFHOUoOw0WudyJ6nZd7MB2OuEWlck-HEAsPRvPZlFF5Bzp6PbVIXf-iA6disg7_WaP1-x0uq8dkwljjtyHSHwcBiNyKxWFh7gJdEm6Ty4z9LSh5PeMQnUx-Kn2MTlO-sU6hTBpGJhVNRQNPRyOXY5GRsN5slT-AIVRXfsE88NcmMRpiu8qZXckvuKn8wRtGE7n3AFV9-YTafPp9O8ST2DayMKof6nGyohWTTWFjv4oLBYz34nImqG_KtH2NPL4Tt4vqubtGnWnw84JoMFjvmnr9ptFqPmYGu0BlqzQPpmYdFkjtsZXM16KwQl58lb6iSGjG1ZXLWX4VvCJCM
```

The bearer token will automatically expire after 60 days (an 400 http status code will be sent) and the client will have to re-authorize again to recieve a new token.

To secure **non-idempotent requests** (such as posting a comment) against replay attacks, 2 additional parameters are also required to be included in the request's querystring:

1. nonce: an arbitrary random string that can only be used once to make an API request (128 bits max)
2. timestamp: current time in EPOCH time


## Requesting a List of Feeds

A client has access to an 'updated' feed that supplies information about all published stories sorted by (published) date and a 'deleted' feed that provides a list of unpublished stories. 

Each story may also have zero or more comments and in a similar manner the client has access to two feeds that provide all published comments and all unpublished comments. 

Feeds are provided in Atom (default) and RSS formats. To request an RSS feed specifically the client needs to include the 'application/rss+xml' in the Accepts header of the request. The client may also use the 'application/atom+xml' in the Accepts header to request an Atom feed (although this is not required as Atom is the default format).

**Execute a GET request to '/api/notifications/topics' to get a JSON payload that describes the available feeds.**

An example of such a payload is as follows:

```
{
  "version": "0.1",
  "created": "2014-10-27T06:13:26.275614Z",
  "title": "Topics Discovery Profile",
  "feeds": [
    {
      "stories": {
          "updated": "https://realtime.api/api/topics/e19a7c4d-4fb4-417f-a399-a391010e47b7/updated/stories",
          "deleted": "https://realtime.api/api/topics/e19a7c4d-4fb4-417f-a399-a391010e47b7/deleted/stories"
       },
       "comments": {
          "updated": "https://realtime.api/api/topics/e19a7c4d-4fb4-417f-a399-a391010e47b7/updated/comments",
          "deleted": "https://realtime.api/api/topics/e19a7c4d-4fb4-417f-a399-a391010e47b7/deleted/comments"
       }
    }
  ],
  "channels" : [
    {
        "guid": "a27e73f0-3d7d-435a-860e-5b061a5bfbdc",
        "title": "Soccer Channel"
    },
    {
        "guid": "d55b7bed-47a7-42d1-a560-5cc1b15a23dc",
        "title": "Tennis Channel"
    }
  ]
}

```

## The Feed

The client system can inspect the feeds to find out which stories have been recently published and unpublished. The client can poll these feeds at regular intervals to determine if content has changed. 

A sample ('updated') stories feed looks as follows (please note that this applies to the 'deleted' feed as well):
```
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
  <title type="text">Updated Documents</title>
  <subtitle type="text">Updated Documents</subtitle>
  <id>uuid:d6ae516a-df73-4305-8c4b-0b1a44fb1f5f;id=1</id>
  <updated>2014-10-27T06:59:52Z</updated>
  <link href="http://realtime.api/api/topics/e19a7c4d-4fb4-417f-a399-a391010e47b7/updated/stories" rel="self" />
  <entry>
    <id>ef11c5e2-7085-4a30-ac4f-022f3258703c</id>
    <channel>d6ae116a-df73-4g7f-a399-022f3258703c</channel>
    <title type="text">Basketball Game</title>
    <updated>2014-10-27T06:23:32+11:00</updated>
    <link rel="alternate" href="http://realtime.api/api/docs/ef11c5e2-7085-4a30-ac4f-022f3258703c" />
  </entry>
</feed>
```
Please note that by default the feed will reveal the top 50 last stories. Each entry in the feed represents a story. The feed is ordered from most recently published story to least recently published story. 

The most important aspects to note are the id of the story, the updated date/time and the link to the story. 

An example of ('updated') comments feed:
```
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
  <title type="text">Updated Documents</title>
  <subtitle type="text">Updated Documents</subtitle>
  <id>uuid:d6ae516a-df73-4305-8c4b-0b1a44fb1f5f;id=1</id>
  <updated>2014-10-27T06:59:52Z</updated>
  <link href="http://realtime.api/api/topics/e19a7c4d-4fb4-417f-a399-1111111aaaaa/updated/comments" rel="self" />
  <entry>
    <id>ef11c5e2-7085-4a30-ac4f-222222211111</id>
    <story>ef11c5e2-7085-4a30-ac4f-022f3258703c</story>
    <channel>d6ae116a-df73-4g7f-a399-022f3258703c</channel>
    <title type="text">Comment on Basketball Game</title>
    <content type="text">What a great game!</content
    <updated>2014-10-27T06:23:32+11:00</updated>
  </entry>
</feed>
```

### Filtering Feeds
Both story and comment feeds may be filtered in a variety of different ways.

The story 'updated' and 'deleted' feeds may be queried by the channel Id, to find stories within particular channels. The comment 'updated' and 'deleted' feeds may be queried by channel and/or story Ids, to find comments related to particular stories or channels.

For example, to get a feed of stories filtered by the channel Id perform a GET request as follows:

https://realtime.api/api/topics/e19a7c4d-4fb4-417f-a399-a391010e47b7/updated/stories?channel=d6ae116a-df73-4g7f-a399-022f3258703c

To get a feed of all comments for a given story the client can issue a GET request as follows:

https://realtime.api/api/topics/e19a7c4d-4fb4-417f-a399-a391010e47b7/updated/comments?story=ef11c5e2-7085-4a30-ac4f-022f3258703c

### Query Parameters
#### Updated/Deleted Stories Feed
1.	Since (optional). E.g. http://url/feed?since=2014-10-27T07:41:23Z. Updated timestamp after which to look for new entries. It is recommended that polling clients populate this parameter with “updated” field from last previous response. 
2.	Limit (optional). E.g. http://url/feed?limit=20. Number of entries to return, up to 100. Default to 50
3.	Offset (optional). E.g. http://url/feed?offset=100. Number of entries to skip over for paging. Default to 0
4.	Channel (optional). E.g. http://url/feed?channel=d6ae116a-df73-4g7f-a399-022f3258703c. The id of the channel of which the returned stories must reside within.

#### Updated/Deleted Comments Feed
1.	Since (optional). E.g. http://url/feed?since=2014-10-27T07:41:23Z. Updated timestamp after which to look for new entries. It is recommended that polling clients populate this parameter with “updated” field from last previous response. 
2.	Limit (optional). E.g. http://url/feed?limit=20. Number of entries to return, up to 100. Default to 50
3.	Offset (optional). E.g. http://url/feed?offset=100. Number of entries to skip over for paging. Default to 0
4.	Channel (optional). E.g. http://url/feed?channel=d6ae116a-df73-4g7f-a399-022f3258703c. The id of the channel of which the returned comments must reside within.
5.	Story (optional). E.g. http://url/feed?story=ef11c5e2-7085-4a30-ac4f-022f3258703c. The id of the story of which the returned comments must reside within.

### Cache Control
To save bandwidth, feed requests can be cached using etags.

1.	Feed response returns an “Etag” response header. 

2.	Client implementation should include this etag value as “If-None-Match” header on their subsequent requests

3.	Subsequent requests: if nothing has changed, an empty 304 response will be returned. Otherwise a normal 200 response (with a new “Etag” header) will be returned

4.	This is a default behaviour both on most web-browsers as well as native http-client libraries, so all the steps above may just work out of the box without requiring any extra client implementation.

## Fetching Story Content

The client can access the content of the story by issuing a GET request to the story URI provided in the feed.

The response is a JSON payload that describes the structure of the story. Please note that the client can also request an HTML version of the story by setting the Accepts header to 'text/html'. The HTML version of the content can be embedded within a browser and rendered immediately.


### JSON Payload

An example of a JSON payload is as follows:

```
{
    "version": "1.0",
    "attributes": {
        "created": "2014-08-24T06:24:03.593Z",
        "modified": "2014-10-27T06:23:32.85Z",
        "guid": "ef11c5e2-7085-4a30-ac4f-022f3258703c",
        "items": [
            {
                "content": {
                    "default": {
                        "html": "Sample Heading"
                    }
                },
                "type": "heading"
            },
            {
                "content": {
                    "default": {
                        "href": "http://farm2.staticflickr.com/1103/567229075_2cf8456f01_s.jpg",
                        "width": 75,
                        "height": 75
                    },
                    "square": {
                        "href": "http://farm2.staticflickr.com/1103/567229075_2cf8456f01_s.jpg",
                        "width": 75,
                        "height": 75
                    },
                    "medium": {
                        "href": "http://farm2.staticflickr.com/1103/567229075_2cf8456f01.jpg",
                        "width": 500,
                        "height": 375
                    }
                },
                "type": "social",
                "meta": {
                    "caption": "placeholder caption",
                    "mediaType": "flickr",
                    "user": "70607220@N04",
                    "mediaid": "15011159293",
                    "rotation": 90,
                    "owner": {
                        "realname": "Octavius Caeser",
                        "username": "caeser01"
                    },
                    "title": "My victory over Antony",
                    "description": "Seens at Egypt following Antony's death",
                    "visibility" : {
                        "public": true,
                        "friend": false,
                        "family": false
                    },
                    "datePosted": "2014-10-10T18:00:00Z"
                },
                "contentType": "image"
            },
            {
                "content": {
                    "default": {
                        "html": "<p>Sample</p><p>Content</p>"
                    }
                },
                "type": "text",
                "meta": {}
            },
            {
                "content": {
                    "default": {
                        "href": "http://az656831.vo.msecnd.net/2098ccdc-d633-4d7e-b694-aa319abb61a5/channel/5a346e7c-0398-4e57-865a-a89038b288a9.png"
                    }
                },
                "meta": {
                    "caption": "this is the default caption",
                    "width": 300,
                    "height": 300
                },
                "type": "image"
            },
            {
                "content": {
                    "default": {
                        "href": "http://az656831.vo.msecnd.net/2098ccdc-d633-4d7e-b694-aa319abb61a5/channel/132bbb38-47b6-46ae-b9ad-06705555c1a1.mp4/manifest(format=m3u8-aapl)"
                    },
                    "hls": {
                        "href": "http://az656831.vo.msecnd.net/2098ccdc-d633-4d7e-b694-aa319abb61a5/channel/132bbb38-47b6-46ae-b9ad-06705555c1a1.mp4/manifest(format=m3u8-aapl)"
                    },
                    "Smooth Streaming": {
                        "href": "http://az656831.vo.msecnd.net/2098ccdc-d633-4d7e-b694-aa319abb61a5/channel/132bbb38-47b6-46ae-b9ad-06705555c1a1.mp4/manifest"
                    },
                    "MPEG DASH": {
                        "href": "http://az656831.vo.msecnd.net/2098ccdc-d633-4d7e-b694-aa319abb61a5/channel/132bbb38-47b6-46ae-b9ad-06705555c1a1.mp4/manifest(format=mpd-time-csf)"
                    }
                },
                "meta": {
                    "caption": "video caption"
                },
                "type": "video"
            }
        ]
    }
}
```

The JSON payload has a number of mandatory fields such as the guid, (date) created, (date) modified and items. The items array will consist of a number of objects that represent various sections of the article. Typically these will be - heading, text, image, video, audio or social. 

Each section contains a meta component, a type, and the actual content that is consumed by the end-user. In the case of a text section, this could be an HTML fragment. In a case of an image, it will be a link to an image. 

### HTML Response

The HTML response can be embedded and displayed in a browser. Currently, the Real-Time platform decides which links to embed in the HTML - i.e. the client doesn't have an option whether to include a high, medium or low version of an image or to specify a different format of a video. Until such support is available, a client may issue a JSON request to retrieve the more detailed version of the story and replace URLs if required. 

## Social Media Content

A social media content item will include identifying information (such as a tweet id in the case of Twitter) as well as a set of richer meta-data about the embedded resource from the social media site's API. In the example JSON above we demonstrate what a flickr media content item may look like. It includes the hrefs the various sizes available for the image under the content field as well as rich meta data from the api about the picture such as the posters' name, the rotation of the photo (ie. portrait, landscape) and the description provided for the photo. 

This additional information gives clients the ability to easily present rich content and metadata related to the social media item, without having to integrate against each social media site's API.

```
            {
                "content": {
                    "default": {
                        "href": "http://farm2.staticflickr.com/1103/567229075_2cf8456f01_s.jpg",
                        "width": 75,
                        "height": 75
                    },
                    "square": {
                        "href": "http://farm2.staticflickr.com/1103/567229075_2cf8456f01_s.jpg",
                        "width": 75,
                        "height": 75
                    },
                    "medium": {
                        "href": "http://farm2.staticflickr.com/1103/567229075_2cf8456f01.jpg",
                        "width": 500,
                        "height": 375
                    }
                },
                "type": "social",
                "meta": {
                    "caption": "placeholder caption",
                    "mediaType": "flickr",
                    "user": "70607220@N04",
                    "mediaid": "15011159293",
                    "rotation": 90,
                    "owner": {
                        "realname": "Octavius Caeser",
                        "username": "caeser01"
                    },
                    "title": "My victory over Antony",
                    "description": "Seens at Egypt following Antony's death",
                    "visibility" : {
                        "public": true,
                        "friend": false,
                        "family": false
                    },
                    "datePosted": "2014-10-10T18:00:00Z"
                    
                },
                "contentType": "image"
            },
```

## Rich Media Content

The Real-Time platform hosts rich media (video and audio) using Windows Azure Media Services. The Real-Time platform provides the same video in three different formats to accomodate different devices and platforms as well as varying bandwidth conditions.

The three types of formats that are provided are:
 - HLS
 - Smooth Streaming
 - MPEG Dash
 
The format is represented below:

```
{
    "content": {
        "default": {
            "href": "http://az656831.vo.msecnd.net/2098ccdc-d633-4d7e-b694-aa319abb61a5/channel/132bbb38-47b6-46ae-b9ad-06705555c1a1.mp4/manifest(format=m3u8-aapl)"
        },
        "hls": {
            "href": "http://az656831.vo.msecnd.net/2098ccdc-d633-4d7e-b694-aa319abb61a5/channel/132bbb38-47b6-46ae-b9ad-06705555c1a1.mp4/manifest(format=m3u8-aapl)"
        },
        "Smooth Streaming": {
            "href": "http://az656831.vo.msecnd.net/2098ccdc-d633-4d7e-b694-aa319abb61a5/channel/132bbb38-47b6-46ae-b9ad-06705555c1a1.mp4/manifest"
        },
        "MPEG DASH": {
            "href": "http://az656831.vo.msecnd.net/2098ccdc-d633-4d7e-b694-aa319abb61a5/channel/132bbb38-47b6-46ae-b9ad-06705555c1a1.mp4/manifest(format=mpd-time-csf)"
        }
    }
}
```

The client can choose which format to serve depending on platform requirements.

## Comments

The Real-Time platform supports creation of user comments for each story. The comments are provided via an atom/rss feed that is similar to the story feed. A client can subscribe to the comment 'updated' and 'deleted' feeds and monitor addition and removal of comments for each story.

A client can also submit a new comment (it is usually created by the end-user of the system). This comment is provided as a JSON structure and has the following requirements:

```
{
  "version": "1.0",
  "story": "ef11c5e2-7085-4a30-ac4f-022f3258703c",
  "created" : "2014-10-27T07:08:18.3871533Z",
  "content" : "This is the comment content"
}
```

A client can perform a POST request to 'api/topics/{channel}/comment' (where {channel} is the guid of a channel) to submit a new comment.

Nonce and Timestamp request parameters are required to be supplied, as detailed within the Authorisation section.
New comments can go through a moderation process and, once approved, appear in the comments 'updated' feed. The client can then consume these comments. 

Note that there are 3 possible comment modes that can be set on a story.
- Moderated (default): comments go through an approval process. Comments may be modified and accepted/rejected. Accepted comments appear in the comments feed. 
- Public: all comments are allowed and do not go through the moderation process.
- Disabled: no comments are allowed for the story.
