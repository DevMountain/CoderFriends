<img src="https://s3.amazonaws.com/devmountain/readme-logo.png" width="250" align="right">

CoderFriends
============

## Objective

Use a Node backend with Passport and Express to show a user's coder friends.

## Resources

* [passport-github2] (https://github.com/cfsghost/passport-github)
* [Github API Docs] (https://developer.github.com/v3/)
* [axios] (https://github.com/mzabriskie/axios)

## Step 1: Create Skeleton of Angular App

To mix it up, let's create the file structure for the Angular app first.

* Create a `/public` folder
* Create the following files:
  * app.js
  * services/github-service.js
  * index.html
  * templates/home.html
  * templates/friend.html

Let's create routes for our app:

#### /

The base route should display a "Login with Github" button that will redirect users to `/auth/github`. You may accomplish this by either 1) making a login.html templateUrl for this route, or 2) using an inline template in the route configuration (rather than a templateUrl).

#### /home

The home route will display the current user's GitHub friends via the home.html template

#### /friend/:github_username

This route will display a friend's information as well as what they're currently working on.

Create the `index.js` file and set it up to serve your static files.

## Step 2: Create the auth endpoints

* Install and require your dependencies
  * express
  * express-session
  * passport
  * passport-github2
* [Create a Github app](https://github.com/settings/applications) and then set up the Github Strategy in your `index.js` with your associated `clientID` and `clientSecret`. Use a callbackURL that will redirect the user to `/auth/github/callback`
* Make sure you use the session, passport.initialize and passport.session middelware
* Set up your auth endpoints:

#### /auth/github

Use passport.authenticate with Github

#### /auth/github/callback

Use passport.authenticate and upon successful auth, send the user to `/#/home`

## Step 3: Github following Endpoint

Let's link the Angular Github service to our `index.js`

#### GET `/api/github/following`

In `index.js`, create a `/api/github/following` endpoint and have it return the users that the currently logged in user follows. To do this, you will need to make an API call directly from your `index.js` file. Do do this we will are going to use a package called [axios](https://github.com/mzabriskie/axios) which makes it easy to send requests from our server.

* With the username of the user currently logged in, use `axios` to send a request to  
  `https://api.github.com/users/`username`/followers`

#### Require Auth With Middleware
* Let's make sure that whichever client that requests this endpoint is currently logged in. The best way to do this would be to write a middleware function that runs before the "get followers" logic so that you're sure that the current requesting user is logged in. Your middleware function could look like this:

```
var requireAuth = function(req, res, next) {
  if (!req.isAuthenticated()) {
    return res.status(403).end();
  }
  return next();
}
```

* Add this new middleware function to your route
```
app.get('/api/github/following', requireAuth, __your_other_fn_here__)
```

## Step 4: homeCtrl + Github Service

Now let's connect your Angular app to this setup.

* In GithubService, create a `getFollowing` method that returns the results from the API call we created in Step 3.
* Let's resolve the promise from `getFollowing` into a `friends` variable in the `/home` route before it loads.
* In your homeCtrl (create this file, or do an inline controller in the home route in `app.js`), let's throw friends into the scope and render them in the view (home.html).

## Step 5: NG un-authed auto-redirect

Earlier we created a middleware function that responsed with a 403 if a user was not authorized. Let's make our angular app watch for a 403 status on every response and, when a response does have a 403 status, redirect our browser to our login page.

We can do that by injecting a service that acts as an interceptor in Angular's $httpProvider. Add this chunk of code to your `app.js` file:

```
app.config(function($httpProvider) {
    $httpProvider.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
    $httpProvider.interceptors.push('myHttpInterceptor');
});

// register the interceptor as a service
app.factory('myHttpInterceptor', function($q) {
    return {
        // optional method
        'responseError': function(rejection) {
            if (rejection.status == 403) {
                document.location = '/';
                return;
            }
            return $q.reject(rejection);
        }
    };
});
```

## Step 5: Friend route

Make it so that when the user clicks on one of the selected friends, it loads in that user's latest activity.

#### GET /api/github/:username/activity

Create this endpoint in your `index.js` that grabs data for the given username.
* Using `axios` send a request to:  
  `https://api.github.com/users/`username`/events`
* Create a method in your Github service called `getFriendActivity` and make sure it's passed a username
* Have `eventData` be a resolved variable in the app's routing, then render each of the events in the `/friend/:github_username` route in `friend.html`.

## Contributions

If you see a problem or a typo, please fork, make the necessary changes, and create a pull request so we can review your changes and merge them into the master repo and branch.

## Copyright

© DevMountain LLC, 2017. Unauthorized use and/or duplication of this material without express and written permission from DevMountain, LLC is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to DevMountain with appropriate and specific direction to the original content.

<p align="center">
<img src="https://s3.amazonaws.com/devmountain/readme-logo.png" width="250">
</p>

