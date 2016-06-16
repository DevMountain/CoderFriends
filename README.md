<img src="https://devmounta.in/img/logowhiteblue.png" width="250" align="right">

CoderFriends
============

##Objective
Use a Firebase backend with Passport and Express to show a user's coder friends.

##Resources
* [passport-github] (https://github.com/cfsghost/passport-github)
* [Github API Docs] (https://developer.github.com/v3/)
* [node-github] (https://github.com/mikedeboer/node-github)

##Step 1: Create Skeleton of Angular App
To mix it up, let's create the file structure for the Angular app first.

* Create a `/public` folder
* Create the following files:
  * app.js
  * services/github-service.js
  * index.html
  * templates/home.html
  * templates/friend.html

Let's create routes for our app:

###/
The base route will display the index.html template. Show a "Login with Github" button that will redirect users to `/auth/github`

###/home
The home route will display the current user's GitHub friends via the home.html template

###/friend/:github_username
This route will display a friend's information as well as what they're currently working on.

Create the server.js file and set it up to serve your static files.

##Step 2: Create the auth endpoints

* Install and require your dependencies
  * express
  * express-session
  * passport
  * passport-github2
* [Create a Github app](https://github.com/settings/applications) and then set up the Github Strategy in your server.js with your associated `clientID` and `clientSecret`. Use a callbackURL that will redirect the user to `/auth/github/callback`
* Make sure you use the session, passport.initialize and passport.session middelware
* Set up your auth endpoints:

####/auth/github
Use passport.authenticate with Github

####/auth/github/callback
Use passport.authenticate and upon successful auth, send the user to `/#/home`

##Step 3: Github following Endpoint
Let's link the Angular Github service to our server.js

####GET `/api/github/following`
In server.js, create the above endpoint and have it return the users that currently logged in user follows. You can either use an http request using the `request` module, or you can use the npm module [node-github](https://github.com/mikedeboer/node-github)

Some hints:
* You'll want to make sure that whichever client that requests this endpoint is currently logged in. The best way to do this would be to write a middleware function that runs before the "get followers" logic so that you're sure that the current requesting user is logged in. Your middleware function could look like this:

```
var requireAuth = function(req, res, next) {
  if (!req.isAuthenticated()) {
    return res.status(403).end();
  }
  return next();
}
```

If the client gets a status of 403, it will know that it needs to redirect the user to the `/` page so the user can log in again. **Keep in mind, this will happen every time your server restarts.**

##Step 4: HomeCtrl + Github Service
Now let's connect your Angular app to this setup.

* In GithubService, create a `getFollowing` method that returns the results from the API call we created in Step 3.
* Let's resolve the promise from `getFollowing` into a `friends` variable in the `/home` route. 
* In your HomeCtrl, let's throw friends into the scope and render them in the view.

##Step 5: NG un-authed auto-redirect
We need a way for Angular to detect an un-authed web request (403) so we can redirect them back to the login page. We can do that by injecting a service that acts as an interceptor in Angular's httpProvider. It works sort of like middleware in Node.

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

##Step 5: Friend route
Make it so that when the user clicks on one of the selected friends, it loads in that user's latest activity.

####GET /api/github/:username/activity
Create this endpoint in your server.js that grabs data for the given username at this url:

```
https://api.github.com/users/<username>/events
```

* Create a method in your Github service called `getFriendActivity` and make sure it's passed a username
* Have `eventData` be a resolved variable in the app's routing, then render each of the events in the `/friend/:github_username` route in the Angular app.

## Contributions
If you see a problem or a typo, please fork, make the necessary changes, and create a pull request so we can review your changes and merge them into the master repo and branch.

## Copyright

© DevMountain LLC, 2015. Unauthorized use and/or duplication of this material without express and written permission from DevMountain, LLC is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to DevMountain with appropriate and specific direction to the original content.

<img src="https://devmounta.in/img/logowhiteblue.png" width="250">
