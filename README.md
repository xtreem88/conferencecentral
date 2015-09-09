# Conference Organization App API Project

### About

This project is a scalable application to support a range of applications including web based and android for conference organization.  The API supports the following functionality:

- User authentication (via Google accounts)
- User profiles
- Conference information
- Session information
- User wishlists

The API is hosted on Google App Engine as application ID [conference-central-1058](https://conference-central-1058.appspot.com).

### Design and Improvement Tasks

#### Task 1: Add Sessions to a Conference

I added the following endpoint methods:

- `createSession`: given a conference, creates a session.
- `getConferenceSessions`: given a conference, returns all sessions.
- `getConferenceSessionsByType`: given a conference and session type, returns all applicable sessions.
- `getSessionsBySpeaker`: given a speaker, returns all sessions across all conferences.



```python

class Session(ndb.Model):
    """Session -- session object"""
    name = ndb.StringProperty()
    highlights = ndb.StringProperty(repeated=True)
    speaker = ndb.StringProperty()
    duration = ndb.IntegerProperty()
    typeOfSession = ndb.StringProperty(default='NOT_SPECIFIED')
    date = ndb.DateProperty()
    startTime = ndb.IntegerProperty()

class SessionForm(messages.Message):
    """Session Form -- Session outbound from message"""
    name = messages.StringField(1)
    highlights = messages.StringField(2, repeated=True)
    speaker = messages.StringField(3)
    duration = messages.IntegerField(4)
    typeOfSession = messages.EnumField('SessionType', 5)
    date = messages.StringField(6)
    startTime = messages.IntegerField(7)
    websafeKey = messages.StringField(8)

class Speaker(ndb.Model):
    name = ndb.StringProperty(required=True)
    details = ndb.StringProperty()

class Wishlist(ndb.Model):
    user = ndb.StringProperty(required=True)
    websafeSessionKey = ndb.StringProperty()

```


In order to represent the one `conference` to many `sessions` relationship, I used the ancestor relationship.  This allows for strong consistent querying, as sessions can be queried by their conference ancestor.

I used the has_a relationship for the Speaker and the Session models because i the speakers should be changeable at any point

Session types (e.g. talk, lecture) were implemented more in a "tag" representation, with sessions able to receive multiple different types.


#### Task 2: Add Sessions to User Wishlist

`Wishlist` uses both types of relationships, ancestor for the user and has_a for the session key
I added two endpoint methods to the API:

- `addSessionToWishlist`: given a session websafe key, saves a session to a user's wishlist.
- `getSessionsInWishlist`: return a user's wishlist.

#### Task 3: Indexes and Queries

I added two endpoint methods for additional queries that I thought would be useful for this application:

-`getAvailableSessions`: returns a conference's sorted list of sessions yet to occur.
-`getIncompleteSessions`: returns sessions missing time/date information. This is a useful tool for the admin to find sessions that were not completed by the creators so they can be resolved

The query goes against the first rule of datastore queries : an inequality filter can be applied to at most one property.
I decided to use the 'startTime' filter, then i filtered the session types with python


#### Task 4: Add Featured Speaker

I modified the `createSession` endpoint to cross-check if the speaker appeared in any other of the conference's sessions.  If so, the speaker name and relevant session names were added to the memcache under the `featured_speaker` key.  I added a final endpoint, `getFeaturedSpeaker`, which would check the memcache for the featured speaker.  If empty, it would simply pull the next upcoming speaker.

### Setup Instructions

Requirements
-[Google App Engine SDK for Python](https://cloud.google.com/appengine/downloads)
-Python 2.7

--Run using Google App Engine SDK for Python


