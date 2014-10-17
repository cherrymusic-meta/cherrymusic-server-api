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
