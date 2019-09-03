# Omelas Data API

Omelas offers programmatic API access to our proprietary database of curated content from social media and other publicly
available data sources. The API also provides access to descriptive and predictive analytics about the content and the users
producing that content. Contact Omelas at our [contact page](https://www.omelas.co/contact) for more information and access.

#### Versions

- Current version: v2019-04-15:
    - Includes endpoints:
        - location
        - content
        - entity
        - relationship
        - interaction
    

## Quickstart

Basic steps:
1. Get token from AuthO API using client_id and client_credentials (provided by Omelas)
1. Call API with token (as Bearer authorization)

Python code for calling the content endpoint:

```python
import http.client
import json

# Request Token
conn = http.client.HTTPSConnection("omelas.auth0.com")

payload = "{\"client_id\":\"CLIENT_ID\",\"client_secret\":\"CLIENT_SECRET\",\"audience\":\"omelas-user-auth-data-api\",\"grant_type\":\"client_credentials\"}"

headers = { 'content-type': "application/json" }

conn.request("POST", "/oauth/token", payload, headers)

# Extract token
res = conn.getresponse()
data = res.read()
data = data.decode("utf-8")
data_dict = json.loads(data)

access_token = data_dict['token_type'] + ' ' + data_dict['access_token']
headers = {'authorization': access_token}

# Call full content API with token
api_conn = http.client.HTTPSConnection("7wo9u54nd3.execute-api.us-gov-west-1.amazonaws.com")

# Use token to call API with desired parameters, potential calls are below as well as all the example queries for each endpoint

# Most recent content
api_conn.request('GET','/prod/v2019-04-15/content/vkontakte/', headers=headers)

# Single content ID
# api_conn.request('GET','/dev/v2019-04-15/content/vkontakte/CONTENT_ID', headers=headers)

# Single content ID
# api_conn.request('GET','/dev/v2019-04-15/location/twitter/LOCATION_ID', headers=headers)

# Note that timestamps need to be encoded as URLs, generally, this just means replacing spaces with %20.
# Timestamps are parsed with dateutil.parser.parse
# Specify content start time
# api_conn.request('GET','/dev/v2019-04-15/content/twitter/?most_recent_content_timestamp=TIMESTAMP', headers=headers)

# Process and decode response
api_res = api_conn.getresponse()
api_data = api_res.read()
api_data_string = api_data.decode('utf-8')
final_data_dict = json.loads(api_data_string)
```

## Resource endpoints

Omelas currently supports the following endpoints: 
- Content
  - Text from various sources eg. social media posts
- Location
  - Information and scores about locations of various sizes including recent content about those locations. 
- Entity
  - Metadata about a social media user or group
- Relationship
  - Metadata about relationship between two entities, entities are allowed to have multiple types of relationship
- Interactions
  - Information about specific interactions between two entities 


### Content

This endpoint provides access to content objects either single item or a list of the most recent items.

Content endpoints are prefaced by /content and followed by the particular platform. For example: `/content/twitter` and `/content/vkontakte`. 

This endpoint can be accessed at `https://e9bz5rf9tc.execute-api.us-gov-west-1.amazonaws.com/dev/v2019-04-15/content/{platform_name}`. See below for examples. 

#### Available platforms
The following platforms are available, in the listed format. Formatting is all lower-case, ie. 'twitter'.
- Twitter: 'twitter'
- VKontakte/VK: 'vkontakte'
- Reddit: 'reddit'
- Odnoklassniki/OK.RU: 'okru'
- YouTube: 'youtube'
- RSS: 'rss'
- Instagram: 'instagram'
- Telegram: 'telegram'
- Aparat: 'aparat'
- Soroush: 'soroush'

#### Return objects

The content endpoint returns content objects as a list along with metadata about the returned data and the user's request. 

##### Overall return

```
{'content_list': List of Content Objects,
 'requested_timestamp': start_time requested, text
 'oldest_timestamp': oldest timestamp on content object, text,
 'actor_id': Id number of actor requested, Null if not called, int
 'content_length': length of content list, int
 'platform_name': name of called platform, text
}
```

##### Content Object
```
    {'content_id': internal id, int,
    'entity_name': content originator text,
    'text': content text, text
    'translated_text': translated text, text, - Only included for users who have the translation plan.
    'language_key': Two or three letter language code, text
    'date_posted': datetime, text,
    'content_url': text,
    'locations': list of dictionaries of form 
                {'location_id': int
                'location_name: text},
    'platform_id': int,
    'platform_name': text
    }
```

#### Path parameters

- content_id
  - An integer to return a single content object of the same ID.

#### Query parameters

The content API can be queried with the parameters below to access particular pieces of content

- content_id (int)
  - Unique identifier assigned by Omelas
  - multiple items can be passed `/?content_id=1&content_id=2`, up to 50 items are allowed
- limit (int)
  - Number of items to return, no more than 50
  - Defaults to 50
- most_recent_content_timestamp (timestamp)
  - Most recent time to get content from, for example, set to midnight today, will get the last content published yesterday
  - Defaults to now
  - Timestamp is parsed by Python library dateutil.parser.parse and needs to be parseable as a URL parameter, ie. %20 replacing spaces.
- actor_id (int)
  - Id for actor associated with desired content
  - Actors are associated to content from Entity -> Brand -> Portfolio -> Actor 
- content_keyword (text)
  - Search term that must be included in all returned content (case insensitive)
  - Searches raw and translated content for users on the translation plan and only raw content for others.
  - Multiple keywords can be included as long as they are separated with either an and operator (`&`) or an or operator (`|`). 
  These operators can be combined with parenthesis if needed. 
    - All content keywords must be URL encoded, your package or tool of choice may do this automatically. This [encoding tutorial](https://www.w3schools.com/tags/ref_urlencode.asp) can provide assistance.
    - A query for content including the terms Russia and Iran but not Iraq would initially be `(Russia & Iran) & !Iraq` 
    and the URL would be `%28Russia%26Iran%29+%26+%21Iraq`.
    - No spaces will be permitted in the content query.

#### Example queries



`/v2019-04-15/content/twitter/` - Returns the most recent 50 content objects from Twitter.

`/v2019-04-15/content/twitter/CONTENT_ID` - Returns a single content object with the associated content ID, CONTENT_ID must be from Twitter.

`/v2019-04-15/content/twitter/?content_id=CONTENT_ID_1&content_id=CONTENT_ID_2` - Returns two content objects with the corresponding IDs.

`/v2019-04-15/content/vkontakte/?limit=3` - Returns most recent 3 content items from VK

`/v2019-04-15/content/vkontakte/?actor_id=10` - Returns most recent content from actor_id 10   

`/v2019-04-15/content/vkontakte/?content_keyword=Russia` - Returns most recent 50 content items that include the exact term 'Russia'

`/v2019-04-15/content/vkontakte/?content_keyword=Russia+%26+Putin` - Returns most recent 50 content items that include both the exact terms Russia and Putin

`/v2019-04-15/content/vkontakte/?content_keyword=Europe+%7C+Asia` - Returns most recent 50 content items that include either Europe or Asia

`/v2019-04-15/content/vkontakte/?most_recent_content_timestamp=2018-01-01` - Return most recent content from VK before 2018-01-01

### Location

This endpoint provides information about locations in the Omelas database. These include countries, regions/provinces and cities
as well as various other facilities including mines, oil fields and hospitals. The endpoint provides a return object (detailed below)
that includes select proprietary scores and recent content about those locations.

There are two ways to access locations: one, with a location ID directly and the other with search terms that will look for locations with the same name.

This endpoint can be accessed at `https://e9bz5rf9tc.execute-api.us-gov-west-1.amazonaws.com/dev/v2019-04-15/location/` for requests with IDs.

The ability to search for locations is located here: `https://e9bz5rf9tc.execute-api.us-gov-west-1.amazonaws.com/dev/v2019-04-15/location/search/`.

#### Return object

The location endpoint returns a list of Location objects as well as additional metadata about the request and the information returned.

##### Overall return

```
{'location_list': list of Location Objects (see below),
'include_related': user passed indicator, boolean ,
'num_locations': number of location objects returned, int,
'content_timestamp': user query for most recent content object, Null if user didn't pass a timestamp
}
```
##### Location Object
```
{'location_id': internal id, int,
'country_id': Country of Location, int,
'country_name': Name of Location's country, text,
'location_population': Population of location, int,
'location_type': Type of location, primarily City, Region, Country 
'location_name': name, text,
'recent_location_content': list of up to 50 tuples of (content ids, date_posted, platform_name) associated with location, list of ints,
'related_locations': list of dictionaries of contained locations
                                            {'related_location_name':name
                                            'related_location_id':id} ,
'location_scores': dictionaries of scores associated with the location
                            {score_type: {'score': score, int, 'score_calculated_time': time, timestamp}}
}
```

#### Path parameters

- location_id (int)
  - Unique identifier associated with a location
  - Can be combined with any query parameter below
  
#### Query parameters

The API accepts different query parameters to help filter and pinpoint particular data.

- get_related_locations (boolean)
  - Whether to return ids for locations contained within the called location. ie Baghdad for Iraq
  - Defaults to False
  - True/true should be passed to get the related locations
    
- most_recent_content_timestamp (timestamp)
   - Most recent time to get content from, for example, set to midnight today, will get the last content published yesterday
   - Defaults to now
   - Timestamp is parsed by Python library dateutil.parser.parse and needs to be parseable as a URL parameter, ie. %20 replacing spaces.
   
- location_name (string)
  - This can only be used with the `/v2019-04-15/location/search` endpoint.
  - The API will look up the name in the database and return all matching objects
    - The lookup is not case sensitive
    - Search will only return exact matches
  - This can be called with multiple names (see below for an example)
    - Only 50 names can be called at one time.


##### Example queries

Base endpoint:

`/v2019-04-15/location/LOCATION_ID` - Gets the location object for LOCATION_ID

`/v2019-04-15/location/LOCATION_ID?get_related_locations=True` - Related locations for LOCATION_ID

`/v2019-04-15/location/LOCATION_ID?content_timestamp=2018-01-31` - Only content for LOCATION_ID before 2018-01-31

`/v2019-04-15/location/LOCATION_ID?get_related_locations=True&content_timestamp=2019-01-01 13:00` - Related locations and only before 2019-01-01 13:00

Search endpoint:

`/v2019-04-15/location/search/?location_name=Baghdad` - Gets all locations with the name Baghdad

`/v2019-04-15/location/search/?location_name=Baghdad&location_name=Springfield` - Gets all locations named either Baghdad or Springfield

`/v2019-04-15/location/search/?location_name=Baghdad&get_related_locations=True` - Get all locations named Baghdad and includes related locations.

### Entity

This endpoint provides access to one or more entity objects.

This endpoint can be accessed at `https://e9bz5rf9tc.execute-api.us-gov-west-1.amazonaws.com/dev/v2019-04-15/entity/`. See below for examples. 

#### Return objects

The entity endpoint returns a list of entity objects and some metadata about the request and the number of objects returned.

##### Overall return
```
{
	"entity_list":"list", List of entity objects
	"num_entities": "int", Number of entities in the entity list
}
```

##### Entity Object
```
 {"platform_id": "int", Internal Omelas ID
 "platform_name": "string",
 "platform_account_id": "string",
 "entity_name": "string",
 "entity_url": "string",
 "entity_id": "int"}
```

#### Path parameters

- entity_id
  - An integer to return a single entity object with the same ID.

#### Query parameters

The content API can be queried with the parameters below to access particular pieces of content

- entity_id (int)
  - Unique identifier assigned by Omelas
  - multiple items can be passed `/?entity_id=1&entity_id=2`, up to 50 items are allowed

#### Example queries

`/v2019-04-15/entity/ENTITY_ID` - Returns a single entity object with the provided entity ID

`/v2019-04-15/entity/?entity_id=ENTITY_ID_1&entity_id=ENTITY_ID_2` - Returns two entity objects with the corresponding IDs.

### Interactions

This endpoint provides access to one or more interaction objects. The interactions logged here are Retweets, Reply and Quotes.

This endpoint can be accessed at `https://e9bz5rf9tc.execute-api.us-gov-west-1.amazonaws.com/dev/v2019-04-15/interaction/`. See below for examples. 

#### Return objects

This endpoint returns interaction objects by either the entity who was being interacted with or the entity doing the interaction.
It also includes metadata about when the first and last interaction returned occurred.
 
##### Overall return

```
{'interaction_list': list of Interaction Objects,
 'interacting_entities': list of int of interacting entity IDs
 'primary_entities': list of entity IDs who were interacted with
 'actual_start_time': 'datetime', earliest interaction returned
 'start_time_requested': 'datetime', start time requested
 'end_time_requested': 'datetime', most recent interaction requested
 'actual_end_time': 'datetime', most recent interaction returned
 'interaction_count': 'int', number of interactions returned in the interaction_list}
```

##### Interaction Object
```
{
'primary_entity_id':'int',
'interacting_entity_id':'int',
'interaction_type_id':'int', Int
'interaction_type':'string', Text of the interaction type (Retweet, Reply, Quote)
'interaction_time':'datetime',
'interaction_content_id':'int' 
}
```

#### Query parameters

The interaction API can be queried with the given parameters below. Timestamps are parsed by Python library dateutil.parser.parse and needs to be parseable as a URL parameter, ie. %20 replacing spaces.

- /
  - Parameters
    - primary_entity_id 
      - Either this or interacting is required
      - Entity ID for user who is interacted with
      - An ID to return all all interactions that other entities have initiated with the provided entity ID.
      - Multiple entities can be passed, up to 50
    - interacting_entity_id
      - Either this or primary is required
      - Entity ID for user who interacts with people
      - An ID that returns all interactions that an entity has initiated with other entities.
      - Multiple entities can be passed, up to 50
    - most_recent_time
      - Optional, unless oldest_time is also passed
      - Time of the last interaction to return
      - Defaults to now
    - oldest_time
      - Optional
      - Time of the first interaction to return
      - Defaults to 1 day before end_time
      - No greater than 7 days between start and end time
      - Only 50 interactions will be returned at a time
    - interaction_type_id
      - Optional
      - ID of interaction type to pull
      - Only filters if an ID is passed
      - Retweet: 1, Reply: 2, Quote: 3

#### Example queries

`/v2019-04-15/interactions/?primary_entity_id=ENTITY_ID_1` - Returns interactions over the last 7 days where entity_id_1 was being interacted with

`/v2019-04-15/interactions/?interacting_entity_id=ENTITY_ID_2` - Returns interactions over the last 7 days where entity_id_2 was interacting with other users

`/v2019-04-15/interactions/?primary_entity_id=ENTITY_ID_1&interaction_type_id=INTERACTION_TYPE_ID` - Returns interactions with entity_id_1 of the given type

`/v2019-04-15/interactions/?interacting_entity_id=ENTITY_ID_2&oldest_time=TIME_1&most_recent_time=TIME_2` - Returns interactions between time_1 and time_2 where entity_id_2 was interacting with other users

### Relationships

This endpoint provides access to one or more relationship objects. The relationships logged are Friend, Interactor, Follower and Commenter.
Relationships are only logged if a user interacts with another users. ie. an entity is only logged as a follower if the follower comments on the entity they follow.

This endpoint can be accessed at `https://e9bz5rf9tc.execute-api.us-gov-west-1.amazonaws.com/dev/v2019-04-15/relationships/`. See below for examples. 

#### Return objects

This endpoint returns relationship objects called by either the entity who is being interacted with (Followed, Commented On, Interacted with) or the interacting entity (Follower, Commenter, Interactor).
Friends are logged in both directions. Each call returns up to 50 relationship objects.

##### Overall return

```
{
	"relationship_list": list of Relationship Objects,
	"num_relationships": "int", length of relationship list,
	"most_recent_logged_returned": "datetime", most recent relationship time returned, this can be used to paginate,
	"oldest_logged_returned": "datetime",oldest relationship time returned, this can be used to paginate
}
```

##### Relationship Object

```
{
     "primary_entity_id": "int", Omelas created entity identifier, primary ID (Followee, Interactee, Commented on)
     "secondary_entity_id": "int", Secondary entity (Follower, Commentor, Interactor)
     "relationship_type": "string", Type of relationship (inc. Friend, Interactor, Follower, Commenter)
     "relationship_type_id": "int", 
     "first_logged": "datetime", The first time the relationship was logged by Omelas,
     "relationship_id": "int", Omelas produced identifier for a particular relationship
}
```

#### Path parameters

- relationship_id
  - An integer to return a single relationship object with the same ID.


#### Query parameters

The relationship API can be queried with the given parameters below

- / 
  - Parameters
    - primary_entity_id (int)
      - Entity ID to get friends and followers from
      - Either primary or secondary entity is required
      - ie. A user that is followed or commented on
    - secondary_entity_id (int)
      - Entity ID for secondary entities
      - ie. find all accounts a user follows, comments or interacts with
        - Note that only followers who interact with the user are logged as followers
    - most_recent_relationship_start (datetime)
      - Most recent time to look for relationship's initial logged time. This can be use to paginate through relationships.
      - Timestamp is parsed by Python library dateutil.parser.parse and needs to be parseable as a URL parameter, ie. %20 replacing spaces.
    - relationship_type_id (int)
      - Types of relationships to return (Friend: 1, Interactor: 2, Follower: 3, Commenter: 4)


#### Example queries

`/v2019-04-15/relationships/?primary_entity_id=ENTITY_ID_1` - Returns the 500 earliest relationships logged by Omelas where ENTITY_ID_1 is the primary user.

`/v2019-04-15/relationships/?secondary_entity_id=ENTITY_ID_2` - Returns the 500 earliest relationship logged by Omelas where ENTITY_ID_2 is the secondary user.

`/v2019-04-15/relationships/?primary_entity_id=ENTITY_ID_1&relationship_type_id=RELATIONSHIP_TYPE_ID` - Returns relationships of type RELATIONSHIP_TYPE_ID with ENTITY_ID_1 as a primary entity

`/v2019-04-15/relationships/?primary_entity_id=ENTITY_ID_1&earliest_relationship_logged=EARLIEST_TIME` - Returns relationships with ENTITY_ID_1 as a primary that were logged after EARLIEST_TIME.


## Authentication and Authorization

The Omelas Data API is secured with OAuth 2.0 tokens provided by AuthO. These tokens provide access for 7 days and can be refreshed
using the provided client ID and client secret. 


The overall structure of the authorization flow below is borrowed from the AuthO documentation.

![auth_example](auth-sequence-m2m-flow.png)

### Calling the Auth API

The client_id and client_secret are used to call for the API token. The two examples below print out the token and can be changed to store it as a variable.

A base plan from Omelas allows calling the Auth API 30 times in a given month.

The call will return the json object below:
```json
{"access_token": "TOKEN",
 "scope": "PERMISSIONS",
 "expires_in": 86400,
 "token_type": "Bearer"}
```

Pulling access token in bash:
```bash
curl --request POST \
  --url https://omelas.auth0.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"CLIENT_ID","client_secret":"CLIENT_SECRET","audience":"omelas-user-auth-data-api","grant_type":"client_credentials"}'
```

Pulling access code in Python:
```python
import http.client
import json

conn = http.client.HTTPSConnection("omelas.auth0.com")

payload = "{\"client_id\":\"CLIENT_ID\",\"client_secret\":\"CLIENT_SECRET\",\"audience\":\"omelas-user-auth-data-api\",\"grant_type\":\"client_credentials\"}"

headers = { 'content-type': "application/json" }

conn.request("POST", "/oauth/token", payload, headers)

res = conn.getresponse()
data = res.read()

data = data.decode("utf-8")
data_dict = json.loads(data)
access_token = data_dict['token_type'] + ' ' + data_dict['access_token']

```

This returns a token that can be used to call the API directly with curl (see below). This can be done in code as long as the authorization
header includes Bearer followed by the token. 


```bash
curl --request GET \
  --url http://path_to_your_api/ \
  --header 'authorization: Bearer TOKEN'
```
