---
layout: post
title:  "Authentication via Google Oauth2 with Ember-Simple-Auth and Torii"
date:   2015-12-18 14:05:01
categories: jekyll update
---

How to setup and use ember-simple-auth 1.0+ with torii and google oauth2 authentication. 

###  Getting Google OAUTH Keys
 
 [This page](https://developers.google.com/identity/protocols/OAuth2) gives a overview of the process but I'll walk you through the basic steps here. (if you have already completed this you can skip to Creating your ember project).
 
1. First you need create your account by visiting here: https://console.developers.google.com and signup with your gmail account. (I used my private account for this walkthough and I don't believe I had registered it before. Hopefully your experience is the same.)
2. Once you're signed in you'll want to enable apis for your project. In my case this also required creating the application.
TODO add the rest with screen shots

### Create your ember project

Here I'm assuming you've already installed ember and are familiar with it. If not [click here](http://lmgtfy.com/?q=installing+ember)

```
ember init ember-simple-auth-torii-google
```

### Install simple auth and tori

```
ember install ember-simple-auth
ember install torii
```

###   Setup application controller

There are many ways to setup simple-auth to handle the different session actions used for authentiation. For the purposes of this tutorial I'll keep my app simple and just use the application controller for everything. Here we are injecting the simple-auth session and handling the actions authenticateSession and invalidateSession to perform the login and logout actions respectively. In both cases the action just calls an appropriate method on the session. 

In the case of ```authenticate``` I'm passing the authenticator and TBD as parameters. Simple-auth comes with a bunch of builtin authenticators but in this case I need to use a custom one to handle the Oauth callback and serverside call required for authentication. By passing in "authenticator:torii" I'm telling simple-auth to look for the ember object defined in ```/app/authenticator/torii.js```. The second parameter is for Torii but fuck all if i know what it does. 

```javascript
  import Ember from 'ember';

  export default Ember.Controller.extend({
    session: Ember.inject.service('session'),
    actions: {
      authenticateSession() {
        this.get('session').authenticate('authenticator:torii', 'google-oauth2');
      },
      invalidateSession() {
        this.get('session').invalidate();
      }
    }
  });
```

### Create a custom torii-google authenticator
Here I've created a custom authenticator which will handle authentication token returned by Google. Note that it extends the Torii authenticator built into simple-auth and overrides its authenticate method. When this is run it will cause Torii to open a popup showing the familiar Google authenticate user interface. When that's complete and successful the popup will close and the token argument to the promise resolve will contain the Google Oauth token returned by google. Later we will send this token to our serverside to be resolved into the user's email and google profile via google's APIs. 

```javascript
  //app/authenticators/torii.js

  import Ember from 'ember';
  import Torii from 'ember-simple-auth/authenticators/torii';

  const { service } = Ember.inject;

  export default Torii.extend({
    torii: service('torii'),
    authenticate(options) {
      this._super(options).then(function (token) {
        console.log(options, data);
      });
    }
  });
```

### Setup application template

Now I need a way to trigger the ```authenticateSession``` action so I created a super simple ```appplication.hbs``` with just a single link to trigger the action.

```handlebars {% raw %}
  <!-- //app/template/application.hbs -->

  <h2 id="title">Ember Simple Auth with Torii and Google</h2>

  <div>
    <a {{action 'authenticateSession'}}>Login</a>
  </div>
{% endraw %}```

### Configuring Torii with your keys from google

Now is when you'll use the keys and callback url you created when setting up your google app and api client. You'll want to add the following hash to the ```//config/environment.js```. With these settings Torii will know what key to pass to Google when it visits the Google oauth2 authentication page (check name) and where to redirect to after the user has authenticated. 

```javascript
  //config/environment.js

  ENV.torii = {
    providers: {
      'google-oauth2': {
        apiKey: "161580081432-v9mnjust9nhchrf30ri81mtbr8mecje3.apps.googleusercontent.com",
        redirectUri: "http://localhost:4200/oauth2callback"
      }
    }
  };
```

### Run it

Once you have this all together you should be all set to test out the app.

