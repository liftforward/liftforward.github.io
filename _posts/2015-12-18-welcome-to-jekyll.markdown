---
layout: post
title:  "Authentication via Google Oauth2 with Ember-Simple-Auth and Torii"
date:   2015-12-18 14:05:01
categories: jekyll update
---

How to setup and use ember-simple-auth 1.0+ with torii and google oauth2 authentication. 

 1.  Getting Google OAUTH Keys
 
 This page (https://developers.google.com/identity/protocols/OAuth2) gives a overview of the process but I'll walk you through the basic steps here. I'll walk you through it here step by step. (if you have already completed this you can skip to step TBD).
 
 First you need create your account by visiting here: https://console.developers.google.com and signup with your gmail account. (I used my private account for this walkthough and I don't believe I had registered it before. Hopefully your experience is the same.) Once you're signed in you'll want to enable apis for your project. In my case this also required creating the application. 

 1. Create your ember project
{% highlight bash %}
ember init ember-simple-auth-torii-google
{% endhighlight %}
 
 1. Install simple auth and tori
{% highlight bash %}
ember install ember-simple-auth
ember install torii
{% endhighlight %}
 
 1.  Setup application controller
{% highlight javascript %}
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
{% endhighlight %}
 
 1. Setup application template
{% highlight handlebars %}
{% raw %}
  <h2 id="title">Ember Simple Auth with Torii and Google</h2>

  <div>
    {{#if session.isAuthenticated}}
      <a {{action 'invalidateSession'}}>Logout</a>
    {{else}}
      <a {{action 'authenticateSession'}}>Login</a>
    {{/if}}
  </div>
{% endraw %}
{% endhighlight %}

 1. Configuring Torii with your keys from google
 Now is when you'll use the keys and callback url you created when setting up your google app and api client. You'll want to add the following hash to the ```//config/environment.js```. With these settings Torii will know what key to pass to Google when it visits the Google oauth2 authentication page (check name) and where to redirect to after the user has authenticated. 
{% highlight javascript %}
  ENV.torii = {
    providers: {
      'google-oauth2': {
        apiKey: "161580081432-v9mnjust9nhchrf30ri81mtbr8mecje3.apps.googleusercontent.com",
        redirectUri: "http://localhost:4200/oauth2callback"
      }
    }
  };
{% endhighlight %}

 1. Create a custom torii-google authenticator
 Up till now most of what you've built has been boilerplate ember-simple-auth code taken right from the ember-simple-auth readme (https://github.com/simplabs/ember-simple-auth#readme). Now you'll create a custom authenticator which will complete the authentication code returned by Google and allow us to use it get the user's email address, name and google profile. 
{% highlight javascript %}
  //app/authenticators/torii.js
  import Ember from 'ember';
  import Torii from 'ember-simple-auth/authenticators/torii';

  const { service } = Ember.inject;

  export default Torii.extend({
    torii: service('torii'),
    authenticate(options) {
      this._super(options).then(function (data) {
        console.log(options, data);
      });
    }
  });
{% endhighlight %}
