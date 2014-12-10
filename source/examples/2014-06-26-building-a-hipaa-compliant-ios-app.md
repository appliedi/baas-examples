Title: Building a HIPAA Compliant iOS App

----

Author: Josh Ault

----

Date: 06/26/2014

----

Post: There are great backend services, APIs, and SDKs that make it easy to build mobile apps. The problem is they aren't made for healthcare, and don't provide you with what you need to be HIPAA compliant, either in terms of technology or with a business associate agreement (BAA). If you want to store health information, or maybe transmit health information to a healthcare provider through something like HealthKit from Apple, and sell your app to healthcare organizations, you can't use these non-health backend services.

We created the Catalyze HIPAA compliant API to make it simpler, faster, and cheaper for developers to build compliant apps without having to become HIPAA experts, spend $1000s a month for dedicated infrastructure, and spend tons of time setting up a compliant hosted environment.

This post is an example of how you can use our healthcare API to build a compliant mobile app in under a day, without touching a server or database, and without dealing with the myriad of technology [requirements](https://catalyze.io/hipaa/) that exist within HIPAA.

##<span id="Getting_Started">Getting Started</span>

- To start, [sign up](https://dashboard.catalyze.io/) for an account on Catalyze. It's free to get setup and no credit card is required.
- Check out our [API documentation](https://docs.catalyze.io/). You may want to leave it open for reference.
- Download or clone the app for this [guide](https://github.com/catalyzeio/HealthTracker).
- For reference, you may want to look over our [iOS SDK Guide](https://catalyzeio.zendesk.com/hc/en-us/articles/200020688-iOS-Step-by-Step-Guide).
- **Please use the 4-inch iPhone simulator when running this example application. Some elements on screen may not be shown correctly if a different size is used.**

##Using the Dashboard

This section describes first time setup for new Catalyze users. If you already have an account with Catalyze and an application for this guide, you can skip this section and move on to <a href="#Xcode_Setup">Setting up your project in Xcode</a>. You can also read more about organizations, applications, and api keys [here](https://docs.catalyze.io/guides/api/latest/) as this guide simply walks through the steps with sparse explanations for these terms.

Before we start writing some code, we need to setup a few things in the dashboard. After you sign up and verify your email, login and choose the `Mobile Backend as a Service` option. We are shown a list of all our apps (which will be empty if you just signed up). However, before we can create an app we are going to need an organization. Click the `create an organization` link in the displayed message (or go to your account page) and click `Create New Organization`. Fill in the information and save your changes.

###Creating your first Application

Now that we have an organization we can create an application underneath that org. Head back over to the application portal by clicking your email and choosing `Application Portal`. The message telling us we need an organization should disappear and we can now create an application. Fill in the form and save your changes.

###Adding an API Key

We need to add an API Key to our application to use it so click the `API Keys` on your application and create a new API Key. It is common for iOS applications to use their bundle identifier as the API Key `identifier` so for Catalyze, it would be 

	io.catalyze.HealthTracker

Fill in the bundle identifier you will be using and save. Don't worry, we will set your bundle identifier in the Xcode project in the next section.

##<span id="Xcode_Setup">Setting up your project in Xcode</span>

After you've cloned the project from the <a href="#Getting_Started">Getting Started</a> section, double click the Xcode project file. First thing we need to do is change the bundle identifier. Click on the project icon and give the app a new bundle identifier.

###Adding the Catalyze iOS SDK

There are two ways to include the Catalyze iOS SDK into your project: through cocoapods or including the files directly.

###Integrating CocoaPods

If you do not use CocoaPods or do not want to, please skip to the <a href="#Including_Catalyze_SDK">Including the Catalyze SDK Directly</a> section. To learn about CocoaPods, see this [guide](http://guides.cocoapods.org/using/getting-started.html). Before you can use CocoaPods you must create a file named `Podfile` in the root directory of the project. Open it with a text editor and add the following lines

<script src="https://gist.github.com/jault3/ddf2e58581e5ca4c5719.js"></script>

Open a terminal and navigate to the root directory of the project. Type `pod install` to download and install the SDK. You should get output similar to the following

	Analyzing dependencies
	Downloading dependencies
	Installing AFNetworkActivityLogger (2.0.2)
	Installing AFNetworking (2.3.1)
	Installing catalyze-ios-sdk (2.3)
	Generating Pods project
	Integrating client project

	[!] From now on use `HealthTracker.xcworkspace`.

Close the HealthTracker Xcode project if it is open and as stated in the last line of output, double click the HealthTracker.xcworkspace file. When using CocoaPods you will not use the HealthTracker.xcodeproj file.

###<span id="Including_Catalyze_SDK">Including the Catalyze SDK Directly</span>

Although CocoaPods is the preferred way of using the Catalyze iOS SDK, you can still use the files directly. To do this, clone the github [repo](https://github.com/catalyzeio/catalyze-ios-sdk). Copy the `catalyze-ios-sdk` directory from within the root of the project into the HealthTracker project. Do not copy the root SDK folder containing the xcode project file, but the inner folder named `catalyze-ios-sdk` that contains `.h` and `.m` files. Open the HealthTracker Xcode project if not already opened, right click on the folder you wish to add the sdk to, and choose `Add Files to "HealthTracker"...`. Find the `catalyze-ios-sdk` folder you copied into your project and click `Add`.

###Setting your API Key and App ID

Before you can use the Catalyze iOS SDK you need to give your API Key and App ID. This is most commonly done in the AppDelegate.m file. In the `- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions` method, import Catalyze

<script src="https://gist.github.com/jault3/efb6336fa3e2029ef8ad.js"></script>

and add the following line

<script src="https://gist.github.com/jault3/e0e701fab0d649a4b2c8.js"></script>

replacing the values with those you see on the dashboard. It should look something like the following

<script src="https://gist.github.com/jault3/de8e1a8551d63efb64d0.js"></script>

Now you're ready to use the SDK.

##Adding users to your app

If you poke around the project you'll see that there are 5 screens. Login, sign up, Health Tracker, survey, and a profile screen. The idea of HealthTracker is a simple 5 question survey to track your overall well-being over time. After you take the survey you are given a score and you can try and improve your score each time you take it. If you run the application right now, you can navigate through the screens, push all the buttons, and figure out the flow. The entry point of this application is the login screen. Open `LoginViewController.m` and locate the `- (IBAction)login:(id)sender` method. What we need to do here is make the call to login the user with the username and password in the text fields. Import Catalyze at the top of the file.

<script src="https://gist.github.com/jault3/efb6336fa3e2029ef8ad.js"></script>

Now make the login call

<script src="https://gist.github.com/jault3/f65ecc69603394170643.js"></script>

Thats it! You've logged in you're user and they can be accessed by calling 

<script src="https://gist.github.com/jault3/b98e6af8c6a903aba85d.js"></script>

But what if they don't have an account? Open up `SignUpViewController.m` and let's fill in that code. Locate the `- (IBAction)signUp:(id)sender` method and change it to match

<script src="https://gist.github.com/jault3/59b8027b7600c7fe0a54.js"></script>

This assumes that any errors that occur are because of duplicate usernames. No two users signing up for the same app can have the same username. In a production application, you would want to check the error message returned form the API and respond appropriately. Verify your account by clicking the link in the email you should have received and then login.

##Logging Out

After a successful login, you land on the Health Tracker screen. The four elements on the screen are the logout button, profile button, survey button, and a table. The survey and profile buttons don't need any work and are just hooked up to show new screens. We need to fix the logout button and populate the table (discussed in the next section). Open `HealthTrackerViewController.m` and locate the `- (void)logout` method. Update it to match

<script src="https://gist.github.com/jault3/eb1eca0d026666b22dcf.js"></script>

This logs out the currently logged in user.

##Storing and Retrieving data

Now before we go much further, we are going to need a custom class. You can read more about custom classes [here](https://docs.catalyze.io/#custom-classes). Lets go back to the dashboard and click on the `Custom Classes` tab under our app and then add a custom class. Lets name it `surveys`. We are going to need 11 fields. `question_1`, `question_1_answer`, `question_2`, `question_2_answer`, `question_3`, `question_3_answer`, `question_4`, `question_4_answer`, `question_5`, `question_5_answer`, and `score`. All of the questions are `string`s and all of the answers and `score` are `integer`s. Check the `phi` box and click `Add`. Now we are ready to query this class in our app. 

Go back to Xcode and open `HealthTrackerViewController.m`. At the end of the `- (void)viewDidAppear:animated` method you should see a comment `//TODO query the custom class for surveys`. Here we need to query for all available surveys.

<script src="https://gist.github.com/jault3/fa6e79f1aeb57be599b7.js"></script>

This bit of code is going to give us the first 20 entries in our freshly created `surveys` custom class and add them to our global list of surveys. If we run the app now, there shouldn't be any rows in the table because it is a new custom class and there are no entries. So lets fix that!

If we click the `Take survey` button we are given a 5 question survey. Open the `SurveyViewController.m` file and locate the end of the `-(void)save` method with the comment `//TODO save the new survey as a custom class entry`. This is where we are going to create a custom class entry and save it on the Catalyze API. For every one of these we create, we should get a new row in our table.

<script src="https://gist.github.com/jault3/bb10d4be882bc1cbef05.js"></script>

Here we are creating a CatalyzeObject with the proper class name, setting our 11 fields we created in the dashboard, and sending it on its way. Before we are able to see the row in our table, we need to set the text of the cell. Head back to `HealthTrackerViewController.m` and locate the `- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath` method. Replace the comment with

<script src="https://gist.github.com/jault3/ffd3e23a3574c7b1baec.js"></script>

Run the app, fill out the survey if you haven't already and confirm you have a new row in your table.

##ACLs

A question that should be on your mind now is what happens when other users create entries in this custom class? Will my query return these as well and allow me to see that data? At Catalyze we take a deny by default approach. Unless you are explicitly given permission, you cannot see any data that does not belong to you. However, we haven't forgotten the importance of being able to share data. For this we have created ACLs. These can be seen in our API [guide](https://docs.catalyze.io/guides/api/latest/permissions_and_acls/README.html).

Perhaps you want to follow that guide, give your user Read permissions on the `surveys` custom class and create more users to see what happens. Or maybe you wanted to test out the [user-guardian relationship](https://docs.catalyze.io/#update-a-user's-pii). In either case, the scope of this article does not cover ACLs, but we are always writing more guides and documentation. Keep your eye out for new guides or feel free to [email us](mailto:hello@catalyze.io) with any questions you might have.

This example app does filter out survey entries that do not belong to you and puts them in a separate section of the table.

##Putting it all together

The last part we need to fix up is the profile screen. If we open up `ProfileViewController.m` and look for the `- (void)save` method. We can see that this is where we are going to start using the Catalyze User model. When we created the User model, we thought about the most important information you might want to collect about your users. We feel that the User model had to be secure but extensible enough to cover a variety of use cases. So we integrated the [18 elements of PHI](https://catalyze.io/learn/hipaa/what-is-protected-health-information-or-phi/) outlined by HIPAA. Lets take a look at how we can modify these with the iOS SDK

<script src="https://gist.github.com/jault3/54d2a55331da652a2ef0.js"></script>

This gives us some experience dealing with the specialized User model and some of the supporting models it includes such as the `Name` class and the `Address` class. You can now run the fully functional and HIPAA compliant app, congratulations! You can always check out the `finishedVersion` branch on the github repo as well. Just insert your API Key and App ID and start up the simulator.

##What's next

That's it. You're done, at least with this app. You've now got an app running on our compliant API, a backend, and associated organization, that has been certified as HIPAA Compliant by an independent 3rd party auditor.

Our free plan lets you get started. We sign BAAs once you upgrade to a [paid plan](https://catalyze.io/backend-as-a-service/). With a paid plan you inherit all these [policies](https://catalyze.io/policy/) and can use this [page](https://catalyze.io/hipaa/) to show prospects how your technology complies with HIPAA.

We hope this was helpful. Please [email us](mailto:hello@catalyze.io) if you have any questions about the guide or about getting started with Catalyze.

----

Tags: hipaa, baas, tutorial, iOS, compliance