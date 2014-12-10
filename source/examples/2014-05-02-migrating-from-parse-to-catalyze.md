Title: Migrating from Parse to Catalyze for HIPAA Compliance

----

Author: Alex Foran

----

Date: 05/02/2014

----

Post: [Parse](http://www.parse.com) is a popular platform, allowing for data to be easily stored and retrieved. But, what happens if you need to store PHI (protected health information)? Bad news - [Parse isn't HIPAA-compliant](https://www.parse.com/questions/hipaa-compliance). Storing such data in the Parse platform would put you in hot water. Good news - [Catalyze is a fully compliant environment](https://catalyze.io/compliance/), and migrating from Parse (or even starting fresh) isn't hard. This post will go over migrating a small app.

Parse modified the backbone [TodoMVC](http://todomvc.com/) example to show how to use their Javascript SDK - we'll base our example migration off of that. Their code for that example is [here](https://github.com/ParsePlatform/Todo).

# Models
The Todo app makes use of two Parse data types - Users and Collections. We need to decide what [Catalyze models](http://docs.catalyze.io/api/latest/) we're going to replace those with before we start editing code.

## Users
([Parse's User Model](https://parse.com/docs/js_guide#users))

A Parse user is pretty basic - it requires a username and password to sign up, and allows for an optional email address - which this app doesn't use. The choice of corresponding Catalyze model here is pretty easy - the [User model](http://docs.catalyze.io/api/latest/#users) even has the same name. Let's map out the required fields for both:

<table>
    <tr>
        <th>Parse.User</th>
        <th>Catalyze.User</th>
    </tr>
    <tr>
        <td>username</td>
        <td>username</td>
    </tr>
    <tr>
        <td>password</td>
        <td>password</td>
    </tr>
    <tr>
        <td>email (optional)</td>
        <td>email.primary</td>
    </tr>
    <tr>
        <td>-</td>
        <td>name.firstName</td>
    </tr>
    <tr>
        <td>-</td>
        <td>name.lastName</td>
    </tr>
</table>

Uh oh, a few problems here. First off - this app doesn't use the email property. Catalyze needs this to activate users (they'll receive an email with an activation link to be clicked), there's no way around that - so we'll have to add it to the application. Inconvenient, but necessary.

Second, a Catalyze User requires a first and last name. This information doesn't need to be correct, however - you can put in whatever you want. We'll revisit this in a bit.

## Collections
([Parse's Collections Model](https://parse.com/docs/js_guide#collections))

A Parse collection holds data based off a custom object model, and allows it to be queried. This app goes a bit further, using an ACL to restrict the created collection data to the created user. This mapping choice is a bit harder - there are two potential matches.

### 1. User.extras
The user model offers a property called `extras` - just a miscellaneous object in which to store data in any format. This is very basic - it doesn't adhere to any schema, and doesn't allow for server-side querying, like Parse collections.

### 2. Custom classes
Catalyze's [Custom Class](http://docs.catalyze.io/api/latest/#custom-classes) model allows you to create a schema and have users add data to it. Users can then query their own data. This is a perfect match for Parse Collections, but one problem for this demo - a Custom Class schema requires administrator privileges to create or modify. It can be done from the Catalyze developer portal, but for this example, it's simpler to just use User.extras.

# Code
(some of this is redundant from our [Quick Start](http://docs.catalyze.io/#quick-start) section)

## Setup
Like Parse, Catalyze also requires an API key. You can grab one for a free plan in the developer portal ([http://devportal.catalyze.io/](http://devportal.catalyze.io/)). You'll need to:

1. Create an account
2. Create an organization
3. Create an application
4. Create an API key for that application.

The API key is three items in the form `<type> <identifier> <key>`. Hold onto this - you'll need it (but you can always access it in the developer portal later).

(take note - the free plan [has some limits](https://catalyze.io/backend-as-a-service/) - this should be fine for this demo)

## Framework
Parse's demo uses their Javascript SDK to access their API. That by itself isn't hard to adapt from, but they've also based it on backbone and added some custom things. That is hard. In order to deal with these, we've converted the application to use Angular. This post won't go over that part of the conversion (it's not terribly complicated in terms of Angular) - instead, we'll cover the trouble spots.

## Code
### A Catalyze Wrapper
Currently, Catalyze doesn't offer a Javascript SDK (if you'd like to see one, please [let us know](mailto:support@catalyze.io)). Fortunately, the Catalyze v2 API is designed to be easy to call - it's a standard REST API - JSON requests and JSON responses, and semantic HTTP verbs. So, for this conversion, we've created a small custom wrapper as an angular service.

#### Configuration
The Parse code (`todos.js`) has configuration to be set at the top of the file - to simplify this a bit, we've made a `Config` global object contained in `config.js`, for you to edit easily.

```
Config = {
    host: "https://apiv2.catalyze.io",
    apikey: "your-API-key-goes-here"
};
```

You'll need to edit `apikey` above with the API key you got from the developer portal.

#### Custom Headers
Every request to the Catalyze v2 API needs to have the `X-Api-Key` header set to your application's API key - this is how we know what application this is for.

```
var buildHeaders = function() {
    return {
        "X-Api-Key": self.apikey
    };
};
```

(The `Content-type` and `Accept` headers, should be set, as well as CORS headers - but Angular handles those for us)

For requests requiring a session, the `Authorization` header is also required, holding the `Bearer` token received from signin.

```
var buildAuthedHeaders = function(token) {
    var headers = buildHeaders();
    headers.Authorization = "Bearer " + token;
    return headers;
};
```

#### Signing Up
([Catalyze Docs - Creating a User](http://docs.catalyze.io/api/latest/#create-a-user-with-pii))

Creating a User is done via a POST request to /v2/users, with the user data as a JSON object for the request body. No `Authorization` header required for this.

```
this.signUp = function(username, password, email) {
    var user = {username: username, password: password, email: {primary: email}};
    user.name = {
        firstName: username,
        lastName: "-"
    };
    return $http.post(this.host + "/v2/users",
        user,
        {headers: buildHeaders()});
};
```

Note the `name` property - we mentioned that in the User model discussion above. Catalyze requires it to exist, but not to be correct. It's used in the activation email, which looks something like this:

<a href="http://catalyzeio.github.io/Todo/images/activation.png"><img src="http://catalyzeio.github.io/Todo/images/activation.png" style="height:200px;width:auto;"/></a>

That email was generated using the code above - "not-my-real-name" was the username registered.

#### Sign In and Sign Out
([Catalyze Docs - Authentication](http://docs.catalyze.io/api/latest/#authentication))

Signing in is done by making a POST request to /v2/auth/signin, passing `username` and `password` in the request body JSON.

```
this.signIn = function(username, password) {
    return $http.post(this.host + "/v2/auth/signin",
            {username: username, password: password},
            {headers: buildHeaders()});
};
```

The response for that POST request is the JSON for the user object, including one very important field - `usersId`. This is the ID of the user - you'll need this for subsequent requests.

Signing out is a GET to /v2/auth/signout - no request body required, since the user information is passed in the `Authorization` header.

```
this.signOut = function(token) {
    return $http.get(this.host + "/v2/auth/signout",
            {headers: buildAuthedHeaders(token)});
};
```

#### Retrieving the User
([Catalyze Docs - Retrieve a User](http://docs.catalyze.io/#retrieve-a-user's-pii))
Retrieving a user is done with a GET request, using the route /v2/users/{usersId}, where `{usersId}` is the ID of the user to be retrieved.

```
this.getUser = function(token, userId) {
    return $http.get(this.host + "/v2/users/" + userId,
        {headers: buildAuthedHeaders(token)});
};
```

#### Updating the User
([Catalyze Docs - Update a User](http://docs.catalyze.io/#update-a-user's-pii))

Updating a user is similar to creating one, with two key differences. First is the route - /v2/users/{usersId} (just like when retrieving the user). Second, the method is PUT, not POST. You still pass the (updated) user object as the request body. Any added or changed fields will be updated, and any omitted fields will be removed.

```
this.updateUser = function(token, userId, user) {
    return $http.put(this.host + "/v2/users/" + userId,
        user,
        {headers: buildAuthedHeaders(token)});
};
```

### The App
#### Sign In and Sign Up

<table>
    <tr>
        <td>
            <a href="http://catalyzeio.github.io/Todo/images/parse-signin.png"><img src="http://catalyzeio.github.io/Todo/images/parse-signin.png" style="height:200px;width:auto;"/></a><br/>
            <center>Parse</center>
        </td>
        <td>
            <a href="http://catalyzeio.github.io/Todo/images/catalyze-signin.png"><img src="http://catalyzeio.github.io/Todo/images/catalyze-signin.png" style="height:200px;width:auto;"/></a><br/>
            <center>Catalyze</center>
        </td>
    </tr>
</table>

Aside from changing the title and footer, you can see that we've also added an email field. As discussed in the User model, Catalyze requires an email address for user activation, so we need a place to enter that.

These forms are hooked up to the `signIn` and `signUp` functions from the angular service described above.

#### Managing Todos

<a href="http://catalyzeio.github.io/Todo/images/todos.png"><img src="http://catalyzeio.github.io/Todo/images/todos.png" style="height:200px;width:auto;" /></a>

Since we're using User.extras to contain the todos, working with them is a matter of manipulating an array - we won't talk about that part. The extras object for the the above would look something like:

```
extras: {
    todos: [
        {id: 123, desc: "Take screenshots", done: true},
        {id: 456, desc: "Finish Blog Post", done: false},
        {id: 789, desc: "Nothing", done: true}
    ]
}
```

To retrieve the list when the user logs in, we need the user object itself, since that's where the `extras` are (`user.extras`). This is returned from the signin request, so that's convenient for us. In the event that the user loads the page and already has a valid session, we retrieve the ID (stored in their cookies) using their session token (also stored in their cookies). This will return the user object if all is well (as in, if both the session token is valid and the user is accessible) with HTTP 200, or a different status if something is wrong (at which point we take them back to sign in / sign up).

```
if ($cookies.session && $cookies.userId) {
    catalyze.getUser($cookies.session, $cookies.userId).success(function (data) {
        ...
    }).error(...)
```

When a todo is added, we add it to the array in user.extras, and call `updateUser` from the service we created, passing back the user object with the updated extras. Same when it's marked done or undone, or when one's deleted.

# And that's it!
There's some extra angular fiddling done for the filters, and some additional CSS. You can explore all of that for yourself, and run this app on your own, from the repo on github.

[https://github.com/catalyzeio/Todo](https://github.com/catalyzeio/Todo)

Thanks for reading! We hope you enjoy working with the Catalyze v2 API. If you have any problems, please contact us at [support@catalyze.io](mailto:support@catalyze.io).

----

Tags: code, tutorial, example, javascript, api, compliance, hipaa, guide