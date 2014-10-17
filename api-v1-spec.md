FORMAT: 1A

<div style="position: fixed; top: 10px; left: 10px;" markdown="1">
    
# TOC
<!-- MarkdownTOC depth=1 autolink=true bracket=round -->

- [CherryMusic Server API](#cherrymusic-server-api)
- [Group Admin](#group-admin)
- [Group User](#group-user)
- [Group Media](#group-media)
- [Search](#/search{?query})
- [Group Playlist](#group-playlist)
- [Group Art](#group-art)
- [Group Session](#group-session)
- [Group Settings](#group-settings)
- [Group Stats](#group-stats)
- [Group Download](#group-download)
- [Group Action](#group-action)
- [Group Lyrics](#group-lyrics)
- [Group Tags](#group-tags)
- [Group Taglink](#group-taglink)

<!-- /MarkdownTOC -->
</div>

# CherryMusic Server API

This is the **second draft spec** of the upcoming CherryMusic server API.
(Here's the [first draft](restful-interface) for comparison.)

This document is still under development. Please note that **the interface does not exist yet**!

Enjoy this documentation in [API Blueprint](http://apiblueprint.org/) format. (And render it with [aglio](https://github.com/danielgtaylor/aglio) maybe?)


## RESTfulness

The API is a restful interface. It uses HTTP requests to transfer files and JSON representations of CherryMusic objects between client and server.

## Authentication

Unless otherwise specified, every request requires authentication to be successful. Unauthenticated requests will yield a response of `401 Unauthorized`.

[TODO] &hellip;

## Generic Resource Model

All models (resource representations) have the following attributes:

- `_id` (string)
- `_cls` (string)
- `_url` (string)

Sometimes, resources are embedded in other resources; for example, the `owner` of a *Session* resource is an embedded *User*. To keep overhead low, embedded resources often contain only a selection of all their attributes. They will, however, always contain the attributes mentioned above. This should make it possible to set up a proxy (and caching) mechanism, if necessary.

- `_created_at`
- `_modified_at`


## Collection resources 

Collection resources contain any number of sub-resources. As such, they do not have explicit `_id` and timestamp attributes. They do have `_cls` and `_url`, on the other hand.

All subresources use one of their attributes as their **key**, which is used to identify them within the URL scheme (`collection_resource/key`). Examples for key attributes that might get used are `_id` or `name`.

### Generic Query Parameters

Unless otherwise noted, all collection resources support the following query parameters to give some control over which subresources get returned.

- `keys` (JSON array) ... keys of the resources to get
- `desc` (true|false) ... reverses the sorting order
- `skip` (int) ... number of results to skip
- `limit` (int) ... maximum number of results
- `first` (int) ... first key to include in results (if exists)
- `last` (int) ... last key to include in results (if exists)


## Media Resources

Media resources deal with playable media. They are either 

- single, playable **tracks**, or 
- **collections** which can contain any number of tracks and even child collections.

For definitions and examples, see the section about the [/media resource](#group-media). Note, however, that other resources (like [search](#search)) can return collections as well.

Possible media collection types are:

- `Collection`: generic collection
- `Compact`: super collection, groups together other collections when there are too many
- `Playlist`: supports ownership and privacy, but can only contain tracks
- `Search`: top level collection of search results

# Group Admin

The `admin` resource collection is a separate namespace which duplicates the other API resources. It allows special read and write access beyond the usual ownership constraints.

Access is restricted to users with the `"admin"` role. 



# Group User

Resource representation of CherryMusic users.

## User [/users/{name}]

Attributes:

- *_id*
- *_url*
- *name*
- *roles\**
- *stats\**

*Italics* mean readonly, appended \* mean visibility is restricted to owners and admins.

+ Model (application/json)

        {
            "_id": 1,
            "_url": "/users/angus",
            "name": "angus",
            "roles": ["admin", "downloader"],
            "stats": {
                "_url": "/stats/users/angus",
                "time_last_seen": "2014-10-05T07:15:00Z"
            }
        }
        

### GET

+ Response 200

    [User][]


## POST /users/{name}/new_password

+ Parameters
    + name (string, "angus") ... the username

+ Request

        {
            "old_password": "12345",
            "new_password": "54321"
        }

+ Response 201





# Group Media

Browse the media collection in a directory-like structure.

Subresources are either collections (*groups*) or single tracks; the base resource is a collection.

Common Attributes:

- _id
- _url
- name
- path

Group Attributes:

- count = `{"children": 0, "tracks": 0}`
- children (list of groups)
- tracks (list of tracks)

Track Attributes:

- format = `"mp3" | "ogg" | "m4a" | ...` (string)
- length_ms (number)
- start_ms = `0` (number)

## Media [/media/{+path}]

+ Parameters
    + path (string, `path/to/track.mp3`) ... sub-resources: URL-encoded directory and file names, joined by "/"

+ Model

        {
            "_id": "123",
            "_cls": "Collection",
            "_url": "/media/path",
            "name": "path",
            "count": {
                "children": 0,
                "tracks": 1
            },
            "children": [],
            "tracks":  [
                    {
                        "_id": "456",
                        "_url": "/media/path/track.mp3",
                        "_cls": "Track",
                        "format": "mp3",
                        "name": "Track",
                        "path": "/path/track.mp3"
                    }
            ]
        }



# Search [/search{?query}]

Searching for text strings returns a Media group that contains matching groups and single tracks.

+ Parameters
    + query (string, `shibboleth`) 

+ Model

        {
            "_cls": "Search",
            "_url": "/search?query=shibboleth",
            "query": "shibboleth",
            "name": "shibboleth",
            "count": {
                "children": 0,
                "tracks": 0
            },
            "children": [],
            "tracks": []
        }

## GET

+ Response 200

    [Search][]


# Group Playlist

## Playlist [/playlists/{name}/{user}]

+ Parameters
    + name (required, string, `foo songs`)
    + owner (required, string, `angus`)

+ Model

        {
            "_id": 123,
            "_cls": "Playlist",
            "_url": "/playlists/foo+songs/angus",
            "name": "foo songs",
            "public": true,
            "owner": {
                "_id": "1",
                "_cls": "User",
                "_url": "/users/angus",
                "name": angus
            },
            "count": {"children": 0, "tracks": 1},
            "children": [],
            "tracks": [
                    {
                        "_id": "456",
                        "_cls": "Track",
                        "_url": "/media/path/track.mp3",
                        "format": "mp3",
                        "name": "Track",
                        "path": "/path/track.mp3",
                        "length_ms": 123456,
                        "start_ms": 0
                    }
            ]
        }

## Playlists Collection [/playlists]

+ Model

        {
            "_url": "/playlists",
            "_cls": "Collection",
            "name": "playlists",
            "count": {
                "children": 1,
                "tracks": 0
            },
            "children": [
                {
                    "_id": "123",
                    "_cls": "Playlist"
                    "_url": "/playlists/foo+songs/angus",
                    "name": "foo songs",
                    "public": true,
                    "owner": {
                        "_id": "1",
                        "_cls": "User",
                        "_url": "/users/angus",
                        "name": "angus"
                    },
                    "count": ...,
                }
            ],
            "tracks": []
        }

### GET

### POST



# Group Art

Search and set media art.

## Search for art [/art/search{?query,source}]

+ Parameters
    + query (required, string, `"abbey%20road"`) ... query string to find related images for
    + source = `"google"` (optional, string) ... specify where to look for images
        + values
            + `"amazon"`
            + `"bestbuy"`
            + `"google"`  

### GET

+ Response 200

        {
            "query": "abbey road",
            "source": "google",
            "urls": [
                "http://example.com/img.png", 
                "http://example.com/img2.jpeg"
            ]
        }

## Set art [/art/for/local/{+path}]

+ Parameters
    + path (string, `"path/to/target"`) ... path to local media resource

### PUT

+ Request

        {
            "_url": "http://example.com/img.png"
        }

+ Response 201

        {
            "link": "/art/eW91cl9tb20="
        }

# Group Session

Session management; login and logout.

## Session [/sessions/{_id}]

+ Model (application/json)

        {
            "_id": "1234567890",
            "_cls": "Session",
            "_url": "/sessions/1234567890",
            "created_at": "2014-10-05T07:15:00Z",
            "modified_at": "2014-10-05T07:15:00Z",
            "expires_at": "2064-01-01T00:00:00Z",
            "owner": {
                "_id": "1",
                "_cls": "User"
                "_url": "/users/angus",
                "name": "angus"
            },
            "server": {
                "version": "0.34.0",
                "resources": {
                    "audio": {"_url": "/audio"},
                    "transcoder": {
                        "_url": "/trans", 
                        "encoders": ["mp3"], 
                        "decoders": ["mp3"]
                    }
                }
            },
            "motd": {
                "wisdom": {
                    "type": "wisdom",
                    "text": "lalelu"
                },
                "update" : {
                    "type": "update",
                    "version": "3.4.1",
                    "features": ["unlimited bandwidth", "NSA knockout"],
                    "fixes": ["no more bugs"]
                }
            }
        }


### Quit a session (logout) [DELETE]

+ Response 204


### GET

+ Response 200

    [Session][]


## Session Collection

### Establish a session (login) [POST]

+ Request (application/json)

        {
            "user": {
                "name": "angus",
                "password": "12345"
            }
        }


+ Response 201 (application/json)

    [Session][]



# Group Settings

Serverside storage for user-specific settings. Settings are simple key/value pairs which are also bound to a context, allowing different client apps to have their own settings namespace.

## Settings [/settings/{context}/{user}]

+ Parameters
    + context (string) ... `global` or client app UUID
    + user (string) ... username

### GET

### POST

### PATCH


# Group Stats

Statistics for people and other things.

## Stats [/stats/{type}/{_id}]

+ Parameters
    + type (string, "users") ... what kind of thing are the stats for?
        + Values
            + `users`
            + `media`
            + `server`
    + id (string, "angus") ... the id of the thing the stats are for

+ Model 

        {
            "_url": "/stats/users/angus",
            "time_last_seen": "2014-10-05T07:15:00Z"
        }




# Group Download

[TODO] &hellip;

# Group Action

## Action [/actions]

Attributes:

- status = `new | running | completed | error | cancelled | aborted`
- progress = `[1..100]`
- eta = `2069-12-31T00:00:00Z`
- command = `media.update`
- params = `{"path": "/path/to/update"}`
- owner


+ Model

        {
            "_id": "123",
            "_cls": "Action",
            "_url": "/actions/123",
            "_created_at": "2069-12-31T00:00:00Z",
            "_modified_at": "2069-12-31T00:00:00Z",
            "status": "completed",
            "progress": 100,
            "eta": "2069-12-31T00:00:00Z",
            "command": "media.update",
            "params": {"path": "/path/to/update"},
            "owner": {
                "_id": "1",
                "_cls": "User",
                "_url": "/users/angus",
                "name": "angus"
            }
        }

# Group Lyrics

Set and retrieve track lyrics.

## Lyrics

+ Model

        {
            "_id": "123",
            "_cls": "Lyrics",
            "_url": "/lyrics/for/%2Bthemes/popeye.mp3",
            "title": "The Popeye Theme Song",
            "html": "<p>I'm strong to the finich<br/>Cause I eats me spinach<br />I'm Popeye, the Sailor Man!</p>"
        }

### GET

+Response 200

    [Lyrics][]

### PUT

### DELETE


# Group Tags

Tags are typed, textual metadata that can be linked with tracks. The `tags` resource is a readonly collection of aggregate data; the actual creating, linking and deleting is done via the [`tagged` resource](#group-tagged).

Attributes:
    - type (string, `"album"`) ... alphanumerical incl. `_`
    - text (string, `"Greatest Hits"`)
    - items (list of tracks) ... tracks this tag is linked with
    - users (list of users) ... users that are using this tag

## Tag [/tags/{type}/{text}]

+ Parameters
    + type (string, `album`) ... alphanumerical incl. `_`
    + text (string, `Greatest Hits`)

+ Model 

        {
            "_id": "123",
            "_url": "/tags/album/Greatest+Hits",
            "_cls": "Tag",
            "type": "album",
            "text": "Greatest Hits"
            "items": [],
            "users": [],
        }


# Group Taglink

This resource collects the links between tags and tracks.

## Taglink [/taglinks/{user}/{tag_id}/{track_id}]

A Taglink binds together a Tag and a Track on behalf of a User. The combination of `(tag, track, user)` is unique.

+ Parameters
    + tag_id (string, `123`)
    + user (string, `angus`)
    + track_id (string, `456`)

+ Model

        {
            "_id": "123",
            "_url": "/taglinks/angus/123/456"
            "_cls": "Taglink",
            "type": "album",
            "text": "Greatest Hits",
            "owner": {
                "_id": "1",
                "_cls": "User",
                "_url": "/users/angus",
                "name": "angus"
            },
            "item": {
                "_id": "456",
                "_cls": "Track",
                "_url": "/media/path/track.mp3",
                "name": "Track"
            }
        }

### GET

+ Response 200

    [Taglink][]

### DELETE

+ Response 204


## Taglink Collection [/taglinks{?type,text,owner,item}]

+ Model

        {
            "_url": "",
            "_cls": "Collection"
        }

### GET



### POST

+ Request

        {
            "type": "album",
            "text": "Greatest Hits",
            "owner": {"_id": "1"};
            "item": {"_id": "456"};
        }

+ Response 201

    [Taglink][]
