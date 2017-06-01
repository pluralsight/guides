## Getting started

This guide describes the componenents that comprise the official Unsplash JSON API, the most powerful photo engine in the world.

If you are familiar with Unsplash and have any problems or requests, please [contact our API team](mailto:api@unsplash.com).

### Creating a developer account

To access the Unsplash API, first [register as a developer](/developers).

### Registering your application

Once your account has been registered for the API, log in and go to the [Developers page](/developers). Go to "Your Applications", click "New Application", and fill in the required details.

Initially, your aplication will be in development mode and will be limited to 50 requests per hour. When you are ready to produce your app, upload screenshots of your application (focusing on the proper attribution and use of Unsplash photos in your app), then click the "Request Approval" button. If approved, your rate limit will be increased to the full amount allowed.

Preference will be given to applications that follow the [API Guidelines](https://community.unsplash.com/developersblog/unsplash-api-guidelines), including [properly providing attribution for the photographer and Unsplash](https://community.unsplash.com/developersblog/how-to-give-proper-unsplash-attribution).

### Libraries

To make it as easy as possible to integrate the Unsplash API, see official libraries for [PHP](https://github.com/unsplash/unsplash-php), [Ruby](https://github.com/unsplash/unsplash_rb), and [Javascript](https://github.com/unsplash/unsplash-js).

### Guidelines & Crediting

The Unsplash API is made available as a free API. To use the API you must [abide by the terms](/api-terms) and [follow the API guidelines](https://community.unsplash.com/developersblog/unsplash-api-guidelines).

### Hotlinking

Unlike most APIs, we **prefer for the image URLs returned by the API to be directly used or embedded in your applications.** This embedding process is generally referred to as *hotlinking*. By using our CDN and embedding the photo URLs in your application, we can better track photo views and pass those stats on to the photographer, providing them with context for how popular their photo is and how it's being used.


## Schema

### Location

The API is available at `https://api.unsplash.com/`. Responses are sent as JSON.

### Version

All requests receive the **v1** version of the API. We encourage you to specifically request this via the `Accept-Version` header:

`Accept-Version: v1`

### Summary objects

When retrieving a list of objects, an abbreviated or summary version of that object is returned - i.e., a subset of its attributes. To get a full detailed version of that object, fetch it individually.

### HTTP Verbs

The Unsplash API uses HTTP verbs appropriate to each action.

  Verb    | Description
----------|----------------------------------
GET       | Retrieving resources.
POST      | Creating resources.
PUT       | Updating resources.
DELETE    | Deleting resources.

### Error messages

If an error occurs, whether on the server or client side, the error message(s) will be returned in an `errors` array. For example:

```
422 Unprocessable Entity
```
```javascript
{
  "errors": ["Username is missing", "Password cannot be blank"]
}
```



## Authorization

### Public Actions

Many actions can be performed without requiring authentication from a specific user. For example, downloading a photo does not require a user to log in.

To authenticate requests in this way, pass your application ID via the HTTP Authorization header:

```
Authorization: Client-ID YOUR_APPLICATION_ID
```

You can also pass this value using a `client_id` query parameter:

```
https://api.unsplash.com/photos/?client_id=YOUR_APPLICATION_ID
```

If only your application ID is sent, attempting to perform non-public actions that require user authorization will result in a `401 Unauthorized` response.


### User Authentication

The Unsplash API uses OAuth2 to authenticate and authorize Unsplash users. Unsplash's OAuth2 paths live at `https://unsplash.com/oauth/`.

#### Authorization workflow

This process is described below in detail. However, many libraries exist to simplify the process. If you are using one of [the Unsplash API client libraries](#libraries), see their documentation for how to handle user authentication.


1. Direct the user to `https://unsplash.com/oauth/authorize` with the following query parameters:

param | Description
------|------------
`client_id` | Your application ID.
`redirect_uri` | A URI you control that handles successful user authorization.
`response_type` | The access response type you are requesting. The authorization workflow Unsplash supports requires the value "code" here.
`scope`         | A `+`-separated list of requested scopes. e.g. `public+read_user`

    If necessary the user will be asked to log in. They will be presented with the list of permission scopes being requested and asked to authorize.

2. If the user accepts the request, the user will be redirected to the `redirect_uri`, with the authorization code in the `code` query parameter.

3. Make a POST request to `https://unsplash.com/oauth/token` with the following parameters:

param | Description
------|-------------
`client_id` | Your application ID.
`client_secret` | Your application secret.
`redirect_uri` | Your application's redirect URI.
`code`         | The authorization code supplied to the callback by Unsplash.
`grant_type`   | Value "authorization_code".

    If successful, the response body will be a JSON representation of your user's access token:


        {
          "access_token": "091343ce13c8ae780065ecb3b13dc903475dd22cb78a05503c2e0c69c5e98044",
          "token_type": "bearer",
          "scope": "public read_photos write_photos",
          "created_at": 1436544465
        }

    Access tokens do not expire.


4. On future requests, send OAuth Bearer access token via the HTTP Authorization header:

    ```
    Authorization: Bearer ACCESS_TOKEN
    ```

#### Permission scopes

To write data on behalf of a user or to access their private data, you must request additional permission scopes from them. The scopes are:

Scope               | Description
--------------------|------------------------------------------
`public`            | Default. Read public data.
`read_user`         | Access user's private data.
`write_user`        | Update the user's profile.
`read_photos`       | Read private data from the user's photos.
`write_photos`      | Update photos on the user's behalf.
`write_likes`       | Like or unlike a photo on the user's behalf.
`write_followers`   | Follow or unfollow a user on the user's behalf.
`read_collections`  | View a user's private collections.
`write_collections` | Create and update a user's collections.

When authorizing your application, the user will be presented with a list of permission scopes being requested.

## Pagination

Requests that return multiple items (a list of all photos, for example) will be paginated into pages of 10 items by default, up to a maximum of 30. The optional `page` and `per_page` query parameters can be supplied to define which page and the number of items per page to be returned, respectively.

If `page` is not supplied, the first page will be returned.

### Pagination headers

Additional pagination information is returned in the response headers:

#### Per-page and Total

The `X-Per-Page` and `X-Total` headers give the number of elements returned on each page and the total number of elements respectively.

#### Link

URL's for the first, last, next, and previous pages are supplied, if applicable. They are comma-separated and differentiated with a `rel` attribute.

For example, after requesting page 3 of the photo list:

```
Link: <https://api.unsplash.com/photos?page=1>; rel="first",
<https://api.unsplash.com/photos?page=2>; rel="prev",
<https://api.unsplash.com/photos?page=346>; rel="last",
<https://api.unsplash.com/photos?page=4>; rel="next"
```

## Rate Limiting

For applications in development mode, the Unsplash API currently places a limit of <%= RateLimit::HourlyRateLimit::DEVELOPMENT_RATE_LIMIT %> requests per hour. After approval for production, this limit is increased to <%= RateLimit::HourlyRateLimit::PRODUCTION_RATE_LIMIT %> requests per hour. On each request, your current rate limit status is returned in the response headers:

```
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

Note that only the JSON requests (i.e., those to `api.unsplash.com`) are counted. Image file requests (`images.unsplash.com`) do not count against your rate limit.

If you think you'll need a higher rate limit, [contact us](mailto:api@unsplash.com).

## Current User

### Get the user's profile

```
GET /me
```

*Note*: To access a user's private data, the user is required to authorize the `read_user` scope. Without it, this request will return a `403 Forbidden` response.

*Note*: Without a Bearer token (i.e. using a Client-ID token) this request will return a `401 Unauthorized` response.

No parameters are necessary here.

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "private_user.json" %>
```

### Update the current user's profile

```
PUT /me
```

*Note*: This action requires the `write_user` scope. Without it, it will return a `403 Forbidden` response.

#### Parameters

All parameters are optional.

  param    | Description
-----------|----------------------------------
`username`            | Username.
`first_name`          | First name.
`last_name`           | Last name.
`email`               | Email.
`url`                 | Portfolio/personal URL.
`location`            | Location.
`bio`                 | About/bio.
`instagram_username`  | Instagram username.

#### Response

Returns the updated user profile.

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "private_user.json" %>
```


## Users

### Link relations

Users have the following link relations:

  rel         | Description
--------------|----------------------------------
`self`        | API location of this user.
`html`        | HTML location of this user.
`photos`      | API location of this user's photos.
`portfolio`   | API location of this user's external portfolio.
`followers`   | API location of this user's followers.
`following`   | API location of users this user is following.

### Get a user's public profile

Retrieve public details on a given user.

```
GET /users/:username
```

#### Parameters

  param    | Description
-----------|----------------------------------
`username` | The user's username. Required.
`w`        | Profile image width in pixels.
`h`        | Profile image height in pixels.


#### Response

This response includes only the user's publicly-available information. For private details on the current user, use `GET /me`.

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```


```javascript
<%= render "public_user.json" %>
```
**Note:** Supplying the optional `w` or `h` parameters will result
in the custom photo URL being added to the `profile_image` object:

```javascript
{
  // ...
  "profile_image": {
    "small":    ...
    "medium": ...
    "large":   ...
    "custom":  "https://images.unsplash.com/your-custom-image.jpg?q=80&fm=jpg&crop=faces&fit=crop&h=100&w=100"
  }
}
```

### Get a user's portfolio link

Retrieve a single user's portfolio link.

```
GET /users/:username/portfolio
```


#### Parameters

  param    | Description
-----------|----------------------------------
`username` | The user's username. Required.

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
{
  "url": "http://example.com"
}
```

### List a user's photos

Get a list of photos uploaded by a user.

```
GET /users/:username/photos
```

*Note*: See the note on [hotlinking](#hotlinking).

#### Parameters

  param      | Description
-------------|----------------------------------
`username`   | The user's username. Required.
`page`       | Page number to retrieve. (Optional; default: 1)
`per_page`   | Number of items per page. (Optional; default: 10)
`order_by`   | How to sort the photos. Optional. (Valid values: `latest`, `oldest`, `popular`; default: `latest`)
`stats`      | Show the stats for each user's photo. (Optional; default: false)
`resolution` | The frequency of the stats. (Optional; default: "days")
`quantity`   | The amount of for each stat. (Optional; default: 30)

#### Response

The photo objects returned here are abbreviated. For full details use `GET /photos/:id`

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "photo_list.json" %>
```

**Note:** If the optional `stats` param is set to `true`, each photo's stats are included in the response:

```javascript
<%= render "photo_list_with_stats.json" %>
```

### List a user's liked photos

Get a list of photos liked by a user.

```
GET /users/:username/likes
```

*Note*: See the note on [hotlinking](#hotlinking).

#### Parameters

  param    | Description
-----------|----------------------------------
`username` | The user's username. Required.
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)
`order_by` | How to sort the photos. Optional. (Valid values: `latest`, `oldest`, `popular`; default: `latest`)

#### Response

The photo objects returned here are abbreviated. For full details use `GET /photos/:id`

```
200 OK
Link: <https://api.unsplash.com/users/ashbot/likes>; rel="first", <https://api.unsplash.com/photos/users/ashbot/likes?page=1>; rel="prev", <https://api.unsplash.com/photos/users/ashbot/likes?page=5>; rel="last", <https://api.unsplash.com/photos/users/ashbot/likes?page=3>; rel="next"
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "photo_list.json" %>
```

### List a user's collections

Get a list of collections created by the user.

```
GET /users/:username/collections
```

#### Parameters

  param    | Description
-----------|----------------------------------
`username` | The user's username. Required.
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)

#### Response

```
200 OK
Link: <https://api.unsplash.com/users/fableandfolk/collections>; rel="first", <https://api.unsplash.com/photos/users/fableandfolk/collections?page=1>; rel="prev", <https://api.unsplash.com/photos/users/fableandfolk/collections?page=5>; rel="last", <https://api.unsplash.com/photos/users/fableandfolk/collections?page=3>; rel="next"
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "collection_list.json" %>
```

### Get a user's statistics

Retrieve the consolidated number of downloads, views and likes of all user's photos, as well as the historical breakdown and average of these stats in a specific timeframe. The default time is 30 days.

```
GET /users/:username/statistics
```

#### Parameters

  param      | Description
-------------|----------------------------------
`username`   | The user's username. Required.
`resolution` | The frequency of the stats. (Optional; default: "days")
`quantity`   | The amount of for each stat. (Optional; default: 30)

Currently, the only resolution param supported is "days". The quantity param can be any number between 1 and 30.

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "user_statistics.json" %>
```

## Photos

### Link relations

Photos have the following link relations:

  rel      | Description
-----------|----------------------------------
`self`     | API location of this photo.
`html`     | HTML location of this photo.
`download` | Download location of this photo.

### List photos

Get a single page from the list of all photos.

```
GET /photos
```

*Note*: See the note on [hotlinking](#hotlinking).

#### Parameters

  param    | Description
-----------|----------------------------------
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)
`order_by` | How to sort the photos. Optional. (Valid values: `latest`, `oldest`, `popular`; default: `latest`)



#### Response

The photo objects returned here are abbreviated. For full details use `GET /photos/:id`


```
200 OK
Link: <https://api.unsplash.com/photos?page=1>; rel="first", <https://api.unsplash.com/photos?page=1>; rel="prev", <https://api.unsplash.com/photos?page=346>; rel="last", <https://api.unsplash.com/photos?page=3>; rel="next"
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "photo_list.json" %>
```

### List curated photos

Get a single page from the list of the curated photos.

```
GET /photos/curated
```

*Note*: See the note on [hotlinking](#hotlinking).

#### Parameters

  param    | Description
-----------|----------------------------------
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)
`order_by` | How to sort the photos. Optional. (Valid values: `latest`, `oldest`, `popular`; default: `latest`)



#### Response

The photo objects returned here are abbreviated. For full details use `GET /photos/:id`


```
200 OK
Link: <https://api.unsplash.com/photos/curated?page=1>; rel="first", <https://api.unsplash.com/photos/curated?page=1>; rel="prev", <https://api.unsplash.com/photos/curated?page=346>; rel="last", <https://api.unsplash.com/photos/curated?page=3>; rel="next"
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "photo_list.json" %>
```

### Get a photo

Retrieve a single photo.

```
GET /photos/:id
```

*Note*: See the note on [hotlinking](#hotlinking).

#### Parameters

  param    | Description
-----------|----------------------------------
`id`       | The photo's ID. Required.
`w`        | Image width in pixels.
`h`        | Image height in pixels.
`rect`     | 4 comma-separated integers representing x, y, width, height of the cropped rectangle.

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "full_photo.json" %>
```

<%= render "custom_url.json" %>


### Get a random photo

Retrieve a single random photo, given optional filters.

```
GET /photos/random
```

*Note*: See the note on [hotlinking](#hotlinking).

#### Parameters

All parameters are optional, and can be combined to narrow the pool of photos from which a random one will be chosen.

  param       | Description
--------------|----------------------------------
`collections` | Public collection ID('s) to filter selection. If multiple, comma-separated
`featured`    | Limit selection to featured photos.
`username`    | Limit selection to a single user.
`query`       | Limit selection to photos matching a search term.
`w`           | Image width in pixels.
`h`           | Image height in pixels.
`orientation` | Filter search results by photo orientation. Valid values are `landscape`, `portrait`, and `squarish`.
`count`       | The number of photos to return. (Default: 1; max: 30)

*Note*: You can't use the collections and query parameters in the same request

*Note*: When supplying a `count` parameter - and *only* then - the response will be an array of photos, even if the value of `count` is 1.

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

##### Without the `count` parameter:

```javascript
<%= render "full_photo.json" %>
```

##### With the `count` parameter:

```javascript
<%= render "random_multiple.json" %>
```

<%= render "custom_url.json" %>


### Get a photo's statistics

Retrieve total number of downloads, views and likes of a single photo, as well as the historical breakdown of these stats in a specific timeframe (default is 30 days).

```
GET /photos/:id/statistics
```


#### Parameters

  param      | Description
-------------|----------------------------------
`id`         | The public id of the photo. Required.
`resolution` | The frequency of the stats. (Optional; default: "days")
`quantity`   | The amount of for each stat. (Optional; default: 30)

Currently, the only resolution param supported is "days". The quantity param can be any number between 1 and 30.

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "photo_statistics.json" %>
```

### Get a photo's download link

Retrieve a single photo's download link.

Preferably hit this endpoint if a photo is downloaded in your application for use
(example: to be displayed on a blog article, to be shared on social media, to be remixed, etc.).

*Note:* This is different than the concept of a view, which is tracked automatically when you [hotlinking  an image](#hotlinking)

```
GET /photos/:id/download
```


#### Parameters

  param    | Description
-----------|----------------------------------
`id`       | The photo's ID. Required.

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
{
  "url": "https://image.unsplash.com/example"
}
```

### Update a photo

Update a photo on behalf of the logged-in user. This requires the `write_photos` scope.

```
PUT /photos/:id
```

#### Parameters

  param    					| Description
---------------------------|----------------------------------
`id`       					| The photo's ID. Required.
`location[latitude]` 		| The photo location's latitude (Optional)
`location[longitude]` 	| The photo location's longitude (Optional)
`location[name]`			| The photo location's name (Optional)
`location[city]`			| The photo location's city (Optional)
`location[country]`		| The photo location's country (Optional)
`location[confidential]`	| The photo location's confidentiality (Optional)
`exif[make]`				| Camera's brand (Optional)
`exif[model]`				| Camera's model (Optional)
`exif[exposure_time]`		| Camera's exposure time (Optional)
`exif[aperture_value]`	| Camera's aperture value (Optional)
`exif[focal_length]`		| Camera's focal length (Optional)
`exif[iso_speed_ratings]`| Camera's iso (Optional)

#### Response

Responds with the uploaded photo:

```
201 Created
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "full_photo.json" %>
```

### Like a photo

Upvote or like a photo on behalf of the logged-in user. This requires the `write_likes` scope.

*Note*: This action is idempotent; sending the POST request to a single photo multiple times has no additional effect.

```
POST /photos/:id/like
```

#### Parameters

  param    | Description
-----------|----------------------------------
`id`       | The photo's ID. Required.

#### Response

Responds with the abbreviated versions of the user and the liked photo.

```
201 Created
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
{
  "photo": {
    "id": "LF8gK8-HGSg",
    "width": 5245,
    "height": 3497,
    "color": "#60544D",
    "likes": 10,
    "liked_by_user": true,
    "urls": {
      "raw": "https://images.unsplash.com/1/type-away.jpg",
      "full": "https://images.unsplash.com/1/type-away.jpg?q=80&fm=jpg",
      "regular": "https://images.unsplash.com/1/type-away.jpg?q=80&fm=jpg&w=1080&fit=max",
      "small": "https://images.unsplash.com/1/type-away.jpg?q=80&fm=jpg&w=400&fit=max",
      "thumb": "https://images.unsplash.com/1/type-away.jpg?q=80&fm=jpg&w=200&fit=max"
    },
    "links": {
      "self": "http://api.unsplash.com/photos/LF8gK8-HGSg",
      "html": "http://unsplash.com/photos/LF8gK8-HGSg",
      "download": "http://unsplash.com/photos/LF8gK8-HGSg/download"
    }
  },
  "user": {
    "id": "8VpB0GYJMZQ",
    "username": "williamnot",
    "name": "Thomas R.",
    "links": {
      "self": "http://api.unsplash.com/users/williamnot",
      "html": "http://api.unsplash.com/williamnot",
      "photos": "http://api.unsplash.com/users/williamnot/photos",
      "likes": "http://api.unsplash.com/users/williamnot/likes"
    }
  }
}
```

### Unlike a photo

Undo a "like" on a photo. 

*Note*: This action is idempotent; sending the DELETE request to a single photo multiple times has no additional effect.

```
DELETE /photos/:id/like
```

#### Parameters

  param    | Description
-----------|----------------------------------
`id`       | The photo's ID. Required.

#### Response

Responds with a 204 status and an empty body.

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
{
  "photo": {
    "id": "LF8gK8-HGSg",
    "width": 5245,
    "height": 3497,
    "color": "#60544D",
    "likes": 10,
    "liked_by_user": false,
    "urls": {
      "raw": "https://images.unsplash.com/1/type-away.jpg",
      "full": "https://images.unsplash.com/1/type-away.jpg?q=80&fm=jpg",
      "regular": "https://images.unsplash.com/1/type-away.jpg?q=80&fm=jpg&w=1080&fit=max",
      "small": "https://images.unsplash.com/1/type-away.jpg?q=80&fm=jpg&w=400&fit=max",
      "thumb": "https://images.unsplash.com/1/type-away.jpg?q=80&fm=jpg&w=200&fit=max"
    },
    "links": {
      "self": "http://api.unsplash.com/photos/LF8gK8-HGSg",
      "html": "http://unsplash.com/photos/LF8gK8-HGSg",
      "download": "http://unsplash.com/photos/LF8gK8-HGSg/download"
    }
  },
  "user": {
    "id": "8VpB0GYJMZQ",
    "username": "williamnot",
    "name": "Thomas R.",
    "links": {
      "self": "http://api.unsplash.com/users/williamnot",
      "html": "http://api.unsplash.com/williamnot",
      "photos": "http://api.unsplash.com/users/williamnot/photos",
      "likes": "http://api.unsplash.com/users/williamnot/likes"
    }
  }
}
```

## Search

### Search photos

Get a single page of photo results for a query.

```
GET /search/photos
```

*Note*: See the note on [hotlinking](#hotlinking).

#### Parameters

  param    | Description
-----------|----------------------------------
`query`    | Search terms.
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)

#### Response

The photo objects returned here are abbreviated. For full details use `GET /photos/:id`

```
200 OK
Link: <https://api.unsplash.com/search/photos?page=1&query=office>; rel="first", <https://api.unsplash.com/search/photos?page=1&query=office>; rel="prev", <https://api.unsplash.com/search/photos?page=3&query=office>; rel="last", <https://api.unsplash.com/search/photos?page=3&query=office>; rel="next"
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render partial: "documentation/search/photo_list.json" %>
```

### Search collections

Get a single page of collection results for a query.

```
GET /search/collections
```

#### Parameters

  param    | Description
-----------|----------------------------------
`query`    | Search terms.
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)

#### Response

```
200 OK
Link: <https://api.unsplash.com/search/collections?page=1&query=office>; rel="first", <https://api.unsplash.com/search/collections?page=1&query=office>; rel="prev", <https://api.unsplash.com/search/collections?page=3&query=office>; rel="last", <https://api.unsplash.com/search/collections?page=3&query=office>; rel="next"
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render partial: "documentation/search/collection_list.json" %>
```

### Search users

Get a single page of user results for a query.

```
GET /search/users
```

#### Parameters

  param    | Description
-----------|----------------------------------
`query`    | Search terms.
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)

#### Response

```
200 OK
Link: <https://api.unsplash.com/search/users?page=1&query=nas>; rel="first", <https://api.unsplash.com/search/users?page=1&query=nas>; rel="prev", <https://api.unsplash.com/search/users?page=3&query=nas>; rel="last", <https://api.unsplash.com/search/users?page=3&query=nas>; rel="next"
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render partial: "documentation/search/user_list.json" %>
```

## Collections

### Link relations

Collections have the following link relations:

  rel      | Description
-----------|----------------------------------
`self`     | API location of this collection.
`html`     | HTML location of this collection.
`photos`   | API location of this collection's photos.
`related`  | API location of this collection's related collections. (Non-curated collections only)
`download` | Download location of this collection's zip file. (Curated collections only)

### List collections

Get a single page from the list of all collections.

```
GET /collections
```

#### Parameters

  param    | Description
-----------|----------------------------------
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)

#### Response

```
200 OK
Link: <https://api.unsplash.com/collections?page=8>; rel="last", <https://api.unsplash.com/collections?page=2>; rel="next"
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```
```javascript
<%= render "collection_list.json" %>
```

### List featured collections

Get a single page from the list of featured collections.

```
GET /collections/featured
```

#### Parameters

  param    | Description
-----------|----------------------------------
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)

#### Response

```
200 OK
Link: <https://api.unsplash.com/collections/featured?page=8>; rel="last", <https://api.unsplash.com/collections/featured?page=2>; rel="next"
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```
```javascript
<%= render "collection_list.json" %>
```

### List curated collections

Get a single page from the list of curated collections.

```
GET /collections/curated
```

#### Parameters

  param    | Description
-----------|----------------------------------
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)

#### Response

```
200 OK
Link: <https://api.unsplash.com/collections/curated?page=8>; rel="last", <https://api.unsplash.com/collections/curated?page=2>; rel="next"
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```
```javascript
<%= render "collection_list.json" %>
```

### Get a collection

Retrieve a single collection. Viewing a user's private collections requires the `read_collections` scope.

```
GET /collections/:id
```

Or, for a curated collection:

```
GET /collections/curated/:id
```

*Note*: See the note on [hotlinking](#hotlinking).

#### Parameters

  param    | Description
-----------|----------------------------------
`id`       | The collections's ID. Required.

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "collection.json" %>
```

### Get photos in a collection

Use the command below to retrieve a collection's photos.

```
GET /collections/:id/photos
```

Or, for a curated collection:

```
GET /collections/curated/:id/photos
```

*Note*: See the note on [hotlinking](#hotlinking).

#### Parameters

  param    | Description
-----------|----------------------------------
`id`       | The collection's ID. Required.
`page`     | Page number to retrieve. (Optional; default: 1)
`per_page` | Number of items per page. (Optional; default: 10)

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "photo_list.json" %>
```

### List a collection's related collections

Retrieve a list of collections related to this one.

```
GET /collections/:id/related
```

#### Parameters

  param    | Description
-----------|----------------------------------
`id`       | The collection's ID. Required.

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "collection_list.json" %>
```

### Create a new collection

Create a new collection. This requires the `write_collections` scope.

```
POST /collections
```

#### Parameters

 param    | Description
-----------|----------------------------------
`title`     | The title of the collection. (Required.)
`description` | The collection's description. (Optional.)
`private`   | Whether to make this collection private. (Optional; default false).

#### Response

Responds with the new collection:

```
201 Created
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "collection.json" %>
```

### Update an existing collection

Update an existing collection belonging to the logged-in user. This requires the `write_collections` scope.

```
PUT /collections/:id
```

#### Parameters

 param    | Description
-----------|----------------------------------
`title`     | The title of the collection. (Optional.)
`description` | The collection's description. (Optional.)
`private`   | Whether to make this collection private. (Optional.)

#### Response

Responds with the updated collection:

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "collection.json" %>
```

### Delete a collection

Delete a collection belonging to the logged-in user. This requires the `write_collections` scope.

```
DELETE /collections/:id
```

#### Parameters

  param    | Description
-----------|----------------------------------
`id`       | The collection's ID. Required.


#### Response

Responds with a 204 status and an empty body.

```
204 No Content
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

### Add a photo to a collection

Add a photo to one of the logged-in user's collections. Requires the `write_collections` scope.

```
POST /collections/:collection_id/add
```

*Note*: If the photo is already in the collection, this acion has no effect.


#### Parameters

  param    | Description
-----------|----------------------------------
`collection_id`       | The collection's ID. Required.
`photo_id` | The photo's ID. Required.


#### Response

```
201 Created
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "collected_photo.json" %>
```

### Remove a photo from a collection

Remove a photo from one of the logged-in user's collections. Requires the `write_collections` scope.

```
DELETE /collections/:collection_id/remove
```

#### Parameters

  param    | Description
-----------|----------------------------------
`collection_id`       | The collection's ID. Required.
`photo_id` | The photo's ID. Required.


#### Response

```
200 Success
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
<%= render "collected_photo.json" %>
```

## Stats

### Totals

Get a list of counts for all of Unsplash.

```
GET /stats/total
```

#### Response

```
200 OK
X-Ratelimit-Limit: 1000
X-Ratelimit-Remaining: 999
```

```javascript
{
  "total_photos": 88350,
  "photo_downloads": 40775186
}
```


___________
Hopefully you found this guide detailing tha various Unsplash API components to be useful and easy to follow. Please leave all comments and feedback in the discussion section below. Thanks for reading!