---
layout: post
comments: true
title:  "Authentication via Google Oauth2 with Ember-Simple-Auth and Torii - Part 1"
date:   2016-01-19 14:05:01
categories:
---

In this post I'm going to walk though the process of setting up and using ember-simple-auth 1.0+ and Torii with Google Oauth2 authentication. When LiftForward first implemented this in our backoffice tool it was with simple-auth 0.X and things were much different. As we upgraded to 1.0 a few weeks ago I noticed there were little to no examples available using Google's Oauth authentication.

I will explain some details of how Oauth works however is you're interested in an overview from Google I recommend [this page](https://developers.google.com/identity/protocols/OAuth2).

## Step 1. Getting Your Google Oauth Keys

If you have already completed this and have a Client ID and Client Secret you can skip to Creating your ember project.

1. First you need create your account by visiting here: [https://console.developers.google.com](https://console.developers.google.com) and signup with your gmail account. (I used my private account for this walkthough and I don't believe I had registered it before. I'm pretty sure your experience will be the same.
![My helpful screenshot](/assets/ember-simple-auth-torii-google-oauth2/google-oauth2-01.png){: .screenshot}
1. Once you've logged in click the *Enable and manage APIs* link.
![My helpful screenshot](/assets/ember-simple-auth-torii-google-oauth2/google-oauth2-02.png){: .screenshot}
1. This should open the new project modal which you'll need to complete. If you already have a project this will take you to the overview page shown in next.
![My helpful screenshot](/assets/ember-simple-auth-torii-google-oauth2/google-oauth2-03.png){: .screenshot}
1. From the overview page select *Credentials* from the left menu.
![My helpful screenshot](/assets/ember-simple-auth-torii-google-oauth2/google-oauth2-04.png){: .screenshot}
1. From here you can create multiple Oauth 2.0 client IDs for your app. For now you'll just create one for development. Eventually you'll need to created one for each environment you've deployed your app to. Select *Oauth consent screen* in that top menu as we need to configure that first.
![My helpful screenshot](/assets/ember-simple-auth-torii-google-oauth2/google-oauth2-05.png){: .screenshot}
1. On this screen you'll configure what's shown to users in the Google's Oauth consent screen. For now just enter your app's name. The rest can be blank for our purposes. Once you're finished click save and then *Credentials* in that top menu.  
![My helpful screenshot](/assets/ember-simple-auth-torii-google-oauth2/google-oauth2-06.png){: .screenshot}
1. Now we'll create your apps first Client ID. Select *Oauth Client ID* from the New Credentials menu to start the process.
![My helpful screenshot](/assets/ember-simple-auth-torii-google-oauth2/google-oauth2-07.png){: .screenshot}
1. First select *Web application* as that's what we are setting up the IDs for. Next enter a name for this ID, I named these after the environments we have at LiftForward but you can use anything you want. For the *Authorized Javascript origins* use `http://localhost:4200`. (If you plan to run your EmberCLI application on a different port enter that port here.) For the *Authorized redirect URIs* enter the same hostname you first entered with a path of `/oauth2callback`.

    These are very important becuase once you get your application running and attempt to display the google Oauth screen it will not work if the request doesn't come from the origin you list here and once the user completes the screen she will be redirected to the provided redirect URI with an auth token parameter that your app must consume.
![My helpful screenshot](/assets/ember-simple-auth-torii-google-oauth2/google-oauth2-08.png){: .screenshot}
1. Click create and you'll get both your *Client ID* and client secret. Save both of these somewhere as you'll need the later on in this tutorial.

That's it for setting up your OAUTH keys. Now onto building your ember app.

## Step 2. Create your ember project

Here I'm assuming you've already installed ember and are familiar with it. If not [click here](http://lmgtfy.com/?q=installing+ember)

```
ember init ember-simple-auth-torii-google
```

### Install simple auth and tori

```
cd ember-simple-auth-torii-google
ember install ember-simple-auth
ember install torii
```

### Setup the application controller

There are many ways to setup simple-auth to handle the different session actions used for authentiation. For the purposes of this tutorial I'll keep my app simple and just use the application controller for everything. Here we are injecting the simple-auth session and handling the actions authenticateSession and invalidateSession to perform the login and logout actions respectively. In both cases the action just calls an appropriate method on the session.

In the case of ```authenticate``` I'm passing the authenticator and 'google-oauth2' as parameters. Simple-auth comes with a bunch of builtin authenticators but in this case I need to use a custom one to handle the Oauth callback and serverside call required for authentication. By passing in "authenticator:torii" I'm telling simple-auth to look for the ember object defined in ```/app/authenticator/torii.js```. The second parameter is for Torii and it tells Torii what Oauth2 system we are using.

In the case of ```invalidate``` there are no parameters it simply calls the invalidate() method on session to logout the current user.

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
Here I've created a custom authenticator which will handle authentication token returned by Google. Note that it extends the Torii authenticator built into simple-auth and overrides its authenticate method. When this is run it will call `this._super(options)` which will cause Torii to open a popup showing the familiar Google Oauth2 authentication interface and return a promise. When the user completes this screen and is successful the popup will close and the promise will resolve by calling the function supplied in the `then()` with Oauth token data as the argument. Later I will send this token to our serverside to be resolved into the user's email and google profile via google's APIs. For now I just display it in an alert.

```javascript
  //app/authenticators/torii.js

  import Ember from 'ember';
  import Torii from 'ember-simple-auth/authenticators/torii';

  const { service } = Ember.inject;

  export default Torii.extend({
    torii: service('torii'),
    authenticate(options) {
      return this._super(options).then(function (data) {
        alert(`authorizationCode:\n${data.authorizationCode}\nprovider: ${data.provider}\nredirectUri: ${data.redirectUri}`);
      });
    }
  });
```

### Setup application template

I needed a way to trigger the ```authenticateSession``` action so I created a simple ```appplication.hbs``` with just a single link to trigger the action.

```handlebars {% raw %}
  <!-- //app/template/application.hbs -->

  <h2 id="title">Ember Simple Auth with Torii and Google</h2>

  <div>
    <a {{action 'authenticateSession'}}>Login</a>
  </div>
{% endraw %}```

### Configuring Torii with your keys from google

Here is where the keys and callback url created when setting up the google app and api client come into play. I added the following hash to the ```//config/environment.js```. With these settings Torii will know what key to pass to Google when it visits the Google Oauth2 authentication page and where to redirect to after the user has authenticated.

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

Run `ember server` and hit [http://localhost:4200](http://localhost:4200) your browser. You should be shown the application route's tempate with a login link which when clicked will open the google Oauth consent screen. Once you click allow the promise created in the `authentiate` action will resolve and display the auth token returned by google in an alert box.

In Part 2 I'll walkthrough how to combine the authorization code with the secret key and resolve them into the user's name and email.

All the source code I used in this post is available here: [https://github.com/drewnichols/ember-simple-auth-torii-google](https://github.com/drewnichols/ember-simple-auth-torii-google)
