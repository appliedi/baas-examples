Title: A Simple HIPAA Compliant Web Application Using the Catalyze v2 API

----

Author: Alex Foran

----

Date: 06/30/2014

----

Post: When you create a web application, you generally end up writing two parts - the web frontend, and a backend to power it. The backend is responsible for storing data and authenticating users, and can be quite a pain to put together and maintain. We have good news - **the Catalyze HIPAA compliant API can serve as your entire backend**. It's a REST API, meaning that it's quite painless for you to use.

To show that off, I created a quick, simple web application - an interactive version of the [PHQ-9 Depression Questionnaire](http://www.integration.samhsa.gov/images/res/PHQ%20-%20Questions.pdf), designed to help detect and track depressive disorders.


## Accessing the API
We've put together a [Getting Started guide](https://docs.catalyze.io/guides/api/latest/getting_started/README.html) to help you get rolling. For this application, we'll need an API key, which that guide describes how to get.

## The App
This app will do the following:

<ol>
<li>Step the user through all 10 questions of the PHQ-9, and show them their score at the end.</li>
<li>Allow the user to log in and save their scores, and be presented with all of their scores graphed over time.</li>
<li>Allow the user to search for medical professionals in their area.</li>
</ol>

Let's take this one at a time.

## 0. Stack

I used [Angular](https://angularjs.org/) (which also happens to take care of CORS headers for us)  with the [ngRoute](https://docs.angularjs.org/api/ngRoute) extension to create this as a single-page app. I also used [ChartJS](http://chartjs.org/) to display historical results.

## 1. Test Questions
This is just a matter of angular presentation and tallying - we used an array, and presented those sequentially to the user.

<script src="https://gist.github.com/forana/24d2f7d1a3df9bbf5ec8.js"></script>

<img src="http://i.imgur.com/vyfRTPq.png" width="614" />

I won't describe this too much here, but it'll be in the source at the end.

## 2. User Data Management
This is where our backend comes in. We need it for two things - user signup/authentication, and score (data) storage.

### Users
The [User model](https://docs.catalyze.io/#users) is for this very purpose. A user can sign up via an HTTP POST to `/v2/users`, sign in via a POST to `/v2/auth/signin`, and sign out via a GET to `/v2/auth/signout`. To work with this more easily, I created a short angular service for working with the Catalyze API (this isn't intended to serve as an SDK, though one of those may be coming soon - stay tuned!).

<script src="https://gist.github.com/forana/73f091c7df60cee07916.js"></script>

### Storage
Catalyze offers a number of structured clinical data models for storage, but all of them do more than what we really need here. That's fine - we can create a [Custom Class](https://docs.catalyze.io/#custom-classes) to do what we need.

#### Initializing
For this application, I created a custom class named `history` (this can be done from the [dashboard](https://dashboard.catalyze.io/) - see our [Getting Started guide](https://docs.catalyze.io/guides/api/latest/getting_started/README.html)).

<img src="http://i.imgur.com/1kmLQcp.png" width="614" />

A custom class needs a schema to be created - think of it like a custom SQL table. All created entries come with a `createdAt` field, which gets auto-populated with the timestamp at which the entry was created. So, all we need is a `score` field, which I gave the `number` type.

#### Adding Entries
Adding a custom class entry is as easy as making a POST to `/v2/classes/$classname/entry`, where `$classname` is the name of the class you want to add to - in this app's case, `history`. You can see this done in the Angular service above.

#### Retrieving Entries
For this application, in order to graph out results, we're going to want all of them for the current authenticated user, for all time. The Catalyze API offers a querying route - `/v2/classes/$classname/entry/$usersId` (where `$usersId` is the ID for the user, returned from user signin as part of the full user object) - for retrieving all entries that belong to a given user (entries that a user creates via the above route always belong to them). You can pass filtering parameters to this route, but if you don't, you'll just get all of the entries - which is exactly what we want. One thing we do have to pass, though, is the maximum number of results we want. Because this is just a sample application, I picked an arbitrary large number (1000), but for serious applications you'll want to look into the pagination options for that route.

## 3. Medical Professional Search
Catalyze doesn't offer anything like this as part of our backend, currently (though you could create one yourself with a bit of scraping and the [Provider](https://docs.catalyze.io/#providers) model), but [medl.io](http://medl.io) does. I created an Angular service to work with that, as well (part of the source for this application, not posted here) - and it sits next to the service I created for Catalyze just fine.

## Putting it all together
After the 10 questions, the app presents the user with their score (and a recommendation based on that).

<img src="http://i.imgur.com/yrBKozd.png" width="555"/>

One form allows the user to search for medical professionals in their area.

<img src="http://i.imgur.com/TAMgKNb.png" width="555"/>

The other prompts the user to either sign in or register for a new account.

<img src="http://i.imgur.com/m82wMHL.png" width="588"/>

To create a user in the Catalyze backend, the minimum attributes required are `username`, `password`, and `email.primary` - like so:

```
{
    "username": "bob",
    "password": "bobspassword",
    "email": {
        "primary": "bob@example.com"
    },
    "name": {
    }
}
```

For this, though, I wanted the user to just sign in via email, so I fudged things a bit and used the user's email for their username.

```
{
    "username": "bob@example.com",
    "password": "bobspassword",
    "email": {
        "primary": "bob@example.com"
    },
    "name": {
    }
}
```

When a user registers, they're sent a link in their email that they need to follow to activate their account. After they do this, they can click the Sign In button. In the handler for that button, their score is saved and the user is sent to the graph of their scores.

<img src="http://i.imgur.com/lMBMfNW.png" width="606"/>

And we're done! We've created a HIPAA compliant web app to collect, store, and retrieve information from patients.

# Links
If you'd like to use the application I built here and try it out, check it out at [http://catalyzeio.github.com/PHQ9](http://catalyzeio.github.io/PHQ9).

The source for all of the above (with instructions for running it yourself) can be found at [https://github.com/catalyzeio/PHQ9](https://github.com/catalyzeio/PHQ9). If you encounter any problems or have any questions, please don't hesitate to let us know.

----

Tags: hipaa, api, example, sample, tutorial, integration, javascript, code