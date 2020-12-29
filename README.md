## The Idea Exchange App 

This is a stripped down illustration of how to create a multiple user
datastore for your Thunkable app.  This basic concept could be applied to
multi-user app concepts like chat room, task list, project organizer, or as
presented here, an idea trader app.

After reviewing this document and the Thunkable shared project, I'm sure
you'll immediately think of several things that this illustrative app DOESN'T
do; we're purposely paring down  functionality to the very minimum required to
illustrate a multi user Firebase implementation.

###Authentication

Authentication is required in order to distinguish the data entered by
individual users.  Many apps can function as a single user app, for example, a
game that only needs to store a high score across all users.  If you are
building a single user app, you can skim over this section and leave your
Firebase database (at least temporarily) in [test
mode](https://firebase.google.com/docs/rules/basics#realtime-database).

As you look over this shared Thunkable project (link here) you'll note that
scnLogin contains the majority of both the design and code blocks within the
app.   Here's a summary of  the functionality provided by scnLogin:

  * Supports entry of the user's name, email address and password.  Users can "Sign Up" upon first use of the app, and then "Log In" in subsequent usage.

  * "Sign Up" automatically invokes an email verification to the user.

  * Retains the three fields mentioned above as stored variables, and pre-populates these three fields, allowing the user to simply click "Log In" during subsequent usage of the app.

  * Allows a user to log themselves out and Log In as another user or Sign Up as a new user.

  * Creates and maintains a unique userID internal to the app which will allow us to attribute data entered into the app by user.  The userID is additionally saved as a stored variable.

When scnLogin opens, we set our three user editable text fields to their
appropriate stored variables, and we display the userID stored variable
associated with the saved user at the bottom of scnLogin.  Don't forget to set
the password's text field's "secure text" flag which is found beneath the
Advanced Settings of the text field's properties.  

If the user clicks the "Log In" button, we attempt to sign in to our Firebase
database, and if the user exists and has validated their email address, we
open our app to the first screen (scnChallenge) in our bottom tab navigator.
In other words, we allow authenticated users to start using the app and along
with that, we have the means of identifying the user by way of their stored
variable userID.

For brand new users of the app, or users that have elected to Log Out, users
fill in the Your Name, Email, and Password fields on scnLogin and click "Sign
Up".  This creates a new Firebase user account, upon which Firebase will send
an email verification to the email address entered by the user.  The app
issues an alert message to remind the user to respond to the verification
email.  After responding to the email, the user can click the "Log In" button
to access the app.

There are two functions within the scnLogin code; the function
"userIdFromEmail" creates a valid userID by replacing punctuation characters
from the base part of the user's email address.  To keep things simple, we
just replace punctuation characters with a 'X'.  

Function "addUserId" adds the now valid userID (if new) to our Firebase
structure as a new database key beneath the parent key "users".  The new
userID key adds the properties email and name, containing the email address
and name of the user as provided on scnLogin.  Firebase supports a limited
number of punctuation characters as keys, hence the need for the
"userIdFromEmail" function.

###Firebase database structure

We can't go much further without discussing the structure of the Firebase
("Realtime") database, as well as how we go about creating the database and
linking it to our Thunkable app.


First let's discuss database structure.  Firebase uses key / value pairs to
store data, unlike a spreadsheet or SQL database that models data as rows
(records) and columns (fields).  When planning your database structure, try to
keep your structure as "flat" as possible, meaning avoid layer after layer of
nesting.   As an illustration, review the structure shown below.   We could
have elected to store the "slides" (slides are new ideas) as an array beneath
the slides attribute, but Firebase's functionality promotes the structure as
shown below.

![Firebase Realtime DB structure](/ThunkableFirebaseDatabaseStructureImage.png?raw=true "Firebase Structure")

Beneath the root of our app, we have two keys, users and challenges.  A
challenge is an invitation to collect ideas about a specific challenge
concept, like "Help Name our Rescue Dog!" as per our example.  The users
branch maintains the userID, name and email addresses of the users that have
accessed our database.

While our example Thunkable app does not support a means of creating new
"challenges", our database structure would support this functionality if we
eventually added the necessary "create a new challenge" screen to our app.  

Each challenge has properties representing the name of the challenge, a
description of the challenge, and currently unused properties of image and
archive as illustrations of potential future functionality.  A challenge
additionally contains a slides key property, beneath which each of our
individual ideas (i.e. a "slide") are stored.  In this simple example, each
slide consists of just two properties, the content (i.e. the idea the user
entered) and the userID, which associates the content to a specific user.

You'll note the addition of "placeholders" beneath the user and slides keys.
Firebase doesn't have the concept of an empty or null key; you search for a
key and if it isn't found you assume it is empty and you add it.  Adding
placeholders as shown was easier (*cough* lazy) as it avoids adding the
conditional logic to the code required for first time usage.  Once you have
added a user and some slides, you can manually delete the placeholder branches
using Firebase's console.

###Creating the Firebase database and linking it to Thunkable

Thunkable's instructions for this process are quite good and here I'll simply
[point you to those here](https://docs.thunkable.com/realtime-db).  Within the
Firebase console, be sure that you have Enabled Email/Password as the only
Authentication Sign-In method, and that your Realtime database Rules are set
as follows:


    {
    
      "rules": {
    
        ".read": "auth != null", 
    
        ".write": "auth != null",
    
      }
    
    }

… which simply means that only authenticated users can read or write to the
database.

Before linking the Firebase application to your Thunkable application, create
the basic database structure shown above using the Firebase Console.  Use the
Realtime Database / Data screen and click on the + sign to add data keys and
values until your structure and the content matches that shown in the example
above.  Note that this repo contains a JSON copy of this structure / content,
which you can import from this same screen if you'd prefer.

Link the Firebase application to your Thunkable app by copying the Firebase
API and Database URL keys into your app as explained within the link provided
above.  If you are having trouble identifying the Database URL, note that it
is displayed immediately above the Firebase Console's Realtime Database Data editor.

You should now be able to replicate this project's scnLogin Design and Blocks
within your Thunkable app, and by doing so, add your own users to your
Firebase database.

###The Idea Exchange App's Challenge, New and View screens

In this app illustration we've used Thunkable's bottom tab navigator, removing
one of the default screens and renaming the others to scnChallenge, scnNew and
scnView.   For simplicity, we've retained the default bottom tab navigator
icons, but we changed the bottom navigation tab bar labeling to Challenge, New
and View.   Do note that you'll need to order scnLogin ABOVE the bottom tab
navigator screen within your project, else you will experience problems with
Thunkable Live.  Thunkable Live will attempt to load the first screen listed
within your project, and if it isn't scnLogin, the app will hang as we aren't
error checking access to Firebase within our scnChallenge.

Once we have authenticated into the Firebase database using scnLogin, we
navigate to scnChallenge.  scnChallenge simply displays our (single)
challenge's name and content on the screen as a reference to remind us of the
ideas we need to enter.

To enter a new idea, click on New within the bottom tab navigator, which
navigates to scnNew. New ideas are entered as sequentially numbered slides
(see function slideID within scnNew's blocks) and the idea text and the userID
are captured within the database.  

Note that within scnNew's blocks we use a "listener"; a very powerful feature
that supports viewing real time (or transactional) updates to our database.
In this case, we're simply using a listener to determine how many slides we
have within this challenge for the purposes of numbering the next slide we're
about to enter.  With multiple users concurrently using this app, any number
of users could be entering ideas which would then be reflected immediately
within our app variable counter.

Click on the bottom tab navigator icon View to view the ideas currently
entered into the Firebase database.  In this case, we are simply gathering up
the ideas from the database when we open scnView as opposed to using a
listener to continually update our scnView's underlying data.  scnView offers
simple left and right navigation which supports "paging" through each of the
saved ideas.   You'll note that we have added some wait blocks within
scnView's blocks.  Asynchronous calls to databases or APIs often need a little
time to grab the data before executing the remaining code blocks, and one way
of supporting this within Thunkable is to add wait blocks.  Alternatively, you
test for the population of a variable you'd hope to get back from the database
(or API) call within a loop, breaking out of the loop once data was populated.

Thunkable's List Viewer would be an alternative means of displaying the ideas
on scnView, in particular if we elected to enter and store a menu title
property in addition to the content property.  You could display all slides
menu properties within the List Viewer, and clicking on any list item would
then display the content property of the selected slide.

###Conclusion

I hope that this app will be of use to Thunkable users considering how to
manage data from multiple users with their app.  Please feel free to share
comments as repo issues or discussions.

In addition to including the Thunkable project [link here](https://x.thunkable.com/copy/a5b21dfaeade0896a5f64ff1957224e2), note that
this repo includes screenshots of the code blocks underlying scnLogin,
scnChallenge, scnNew, and scnView as mentioned within, in the event that the Thunkable project link has expired.
