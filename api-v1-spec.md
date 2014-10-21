FORMAT: 1A

# CherryMusic Server API


<div style="display: none; position: fixed; top: 10px; left: 10px;" markdown="1">
<!-- MarkdownTOC depth=1 autolink=true bracket=round -->

- [Group Browse Media](#group-browse-media)
- [Group Search Media](#group-search-media)
- [Search](#/search{?query})
- [Group Admin](#group-admin)
- [Group Users](#group-users)
- [Group Playlists](#group-playlists)
- [Group Album Art](#group-album-art)
- [Group Session Management](#group-session-management)
- [Group User Settings](#group-user-settings)
- [Group Stats](#group-stats)
- [Group Download](#group-download)
- [Group Tasks](#group-tasks)
- [Group Lyrics](#group-lyrics)

<!-- /MarkdownTOC -->
</div>

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

<hr/>

# Group Browse Media

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

## Track [/media/{+filepath}]

+ Parameters
    + filepath (string, `path/to/track.mp3`) ... path to playable audio file, relative to *basedir*

+ Model

        {
            "_id": "987",
            "_cls": "Track",
            "_url": "/media/path/to/track.mp3",
            "name": "Track",
            "path": "/path/to/track.mp3",
            "format": "mp3",
            "length_ms": 232000,
            "start_ms": 0
        }

### GET

+ Response 200

    [Track][]


## Collection [/media/{+dirpath}]

+ Parameters
    + dirpath (string, `path/to`) ... path to subdirectory or other collection, relative to *basedir*

+ Model

        {
            "_id": "123",
            "_cls": "Collection",
            "_url": "/media/path",
            "name": "path",
            "count": {
                "children": 1,
                "tracks": 1
            },
            "children": [
                {
                    "_id": "456",
                    "_cls": "Collection",
                    "_url": "/media/path/to",
                    "name": "to",
                    "path": "/path/to"
                    "count": {
                        "children": 0,
                        "tracks": 1
                    },
                }
            ],
            "tracks":  [
                    {
                        "_id": "654",
                        "_url": "/media/path/other_track.mp3",
                        "_cls": "Other Track",
                        "format": "mp3",
                        "name": "Track",
                        "path": "/path/other_track.mp3"
                    }
            ]
        }

### GET

+ Response 200

    [Collection][]


# Group Search Media

Searching for text strings returns a Media group that contains matching groups and single tracks.

# Search [/search{?query}]

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


# Group Admin

The `admin` resource collection is a separate namespace which duplicates the other API resources. It allows special read and write access beyond the usual ownership constraints.

Access is restricted to users with the `"admin"` role.

## List Roles [/admin/roles{?name,user}]
+ Parameters
    + name (string, `admin`) ... role name
        + Values
            + `admin`
            + `downloader`
    + user (optional, string, `angus`) ... user name

+ Model

        {
            "admin": [{"name": "admin", "user": "angus"}],
            "downloader": []
        }

### GET

+ Response 200

    [List Roles][]


## Grant Role [/admin/roles]

### POST

+ Request

        {"name": "admin", "user": "angus"}

+ Response 200

## Revoke Role [/admin/roles/{name}/user/{user}]

+ Parameters
    + name (string, `admin`)
        + Values
            + `admin`
            + `downloader`
    + user (string, `angus`)

+ Model

        {
            "role": "admin",
            "user": "angus"
        }

### DELETE

+ Response 200


# Group Users

Resource representation of CherryMusic users.

## User [/users/{name}]

Attributes:

- *_id*
- *_url*
- *name*
- *roles\**
- *stats\**

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


### Retrieve details [GET]

+ Response 200

    [User][]


## Change Password [/users/{name}/new_password]


+ Parameters
    + name (string, `angus`) ... the username

### POST

+ Request

        {
            "old_password": "12345",
            "new_password": "54321"
        }

+ Response 201


# Group Playlists

## Playlist [/playlists/{name}/by/{user}]

Attributes:

- name (string)
- public (`true | false`)
- owner (User)
- count.tracks (number)
- tracks
- _created_at (timestamp)
- _modified_at (timestamp)

+ Parameters
    + name (required, string, `foo+songs`)
    + user (required, string, `angus`)

+ Model

        {
            "_id": 123,
            "_cls": "Playlist",
            "_url": "/playlists/foo+songs/angus",
            "_created_at": "",
            "_modified_at": "",
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

### Read [GET]

+ Response 200

    [Playlist][]

### Update [PUT]

+ Request

        {
            "tracks": [
                {"_id": "456"},
                {"_id": "457"}
            ]
        }

+ Response 200

    [Playlist][]


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

### List [GET]

+ Response 200

    [Playlists Collection][]

### Create [POST]

+ Request

        {
            "name": "Bell Bottom Blues",
            "owner": {"_id": "1"},
            "tracks": [
                {"_id": "345"}
            ]
        }

+ Response 201


# Group Album Art

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


# Group Session Management

Session management; login and logout.

## Session [/sessions/{id}]

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
                "_cls": "User",
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


### Quit a session: Logout [DELETE]

+ Response 204


### Get session data [GET]

+ Response 200

    [Session][]


## Session Collection [/sessions]

### Establish a session: Login [POST]

+ Request (application/json)

        {
            "user": {
                "name": "angus",
                "password": "12345"
            }
        }


+ Response 201 (application/json)

    [Session][]

### List [GET]

+ Response 200

        {
            "1234567890": {
                "_id": "1234567890",
                "_cls": "Session",
                "_url": "/sessions/1234567890",
                "created_at": "2014-10-05T07:15:00Z",
                "modified_at": "2014-10-05T07:15:00Z",
                "expires_at": "2064-01-01T00:00:00Z",
                "owner": {
                    "_id": "1",
                    "_cls": "User",
                    "_url": "/users/angus",
                    "name": "angus"
                }
            }
        }


# Group User Settings

Serverside storage for user-specific settings. Settings are simple key/value pairs which are also bound to a context, allowing different client apps to have their own settings namespace.

## Settings [/settings/{context}/{user}]

+ Parameters
    + context (string, `common`) ... `common` or client app UUID
    + user (string, `angus`) ... username

+ Model

```json
{
    "_id": "123",
    "_cls": "Settings",
    "_url": "/settings/common/angus",
    "_context": "common",
    "_owner": {
        "_id": "1",
        "_cls": "User",
        "_url": "/users/angus",
        "name": "angus"
    },
    "keys.prev": 89,
    "kbs.play": 88,
    "kbs.pause": 67,
    "kbs.stop": 86,
    "kbs.next": 66,
    "kbs.search": 83
}
```

### Retrieve all settings [GET]

+ Response 200

    [Settings][]

### Change single setting [PUT]

+ Request

```json
{
    "keys.prev": 111
}
```

+ Response 200

    [Settings][]


# Group Stats

Statistics for people and other things.

## Stats [/stats/{type}/{key}]

+ Parameters
    + type (string, `users`) ... what kind of thing are the stats for?
        + Values
            + `users`
            + `media`
            + `server`
    + key (string, `angus`) ... the id of the thing the stats are for

+ Model

        {
            "_url": "/stats/users/angus",
            "time_last_seen": "2014-10-05T07:15:00Z"
        }




# Group Download

[TODO] &hellip;

# Group Tasks

## Task [/tasks]

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
            "_cls": "Task",
            "_url": "/tasks/123",
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

## Lyrics [/lyrics/for/{+path}]

+ Model

        {
            "_id": "123",
            "_cls": "Lyrics",
            "_url": "/lyrics/for/%2Bthemes/popeye.mp3",
            "title": "The Popeye Theme Song",
            "html": "<p>I'm strong to the finich<br/>Cause I eats me spinach<br />I'm Popeye, the Sailor Man!</p>"
        }

### Retrieve [GET]

+ Response 200

    [Lyrics][]

### Set [PUT]

+ Request

    [Lyrics][]

+ Response 200

### Remove [DELETE]

+ Response 200
