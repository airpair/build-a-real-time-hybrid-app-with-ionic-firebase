## 1 Introduction
With mobile internet traffic rocketing every day, web developers are extremely interested in converting their Web App into a Mobile App. But which Mobile Platform to choose? iOS, Android, or Windows? Or one of the platforms invented more recently, like FirefoxOS, WebOS, or Tizen?

Developing native apps for all of these platforms is impossible, which is why Cordova/Phonegap was created: to enable web developers to build a single app codebase that can be deployed for all of these platforms.

In this tutorial, we will learn how to build a Hybrid Mobile app using the Ionic framework with Firebase as it's real time database. We will create a native application for iOS & Android using Cordova/Phonegap. 

### 1.1 What is Ionic?
Ionic is the buzzing hybrid app development framework built using the popular AngularJS library. AngularJS is a javascript MV* library used for creating single page applications for web and mobile. Ionic is a product of [Drifty](http://drifty.com), a silicon valley startup.

### 1.2 What is Firebase ?
Gone are the days when you were required to buy a server, spend time installing it and spend long hours programming to build a simple API. Firebase is a cloud based platform which can be integrated into real time apps on any web and mobile platform. It enables apps to write data, and get updates about changes to the database, instantly. Firebase also supports offline capability and your data is synchronized instantly as soon as the app regains network connectivity. Firebase stores your data as JSON documents so that you are free to choose the schema of your data.

### 1.3 What are we building in this tutorial?
We will be building a multi person chat app with pre-defined rooms for chatting. We will make final apps for iOS & Android using Phonegap/Cordova. 

## 2 Setting Up Development Environment

### 2.1 Pre requisites 
   - Mac to build the iOS App
   - If you don't have access to a mac, you can build the Android app on a Windows machine
   - [NodeJS](http://nodejs.org/) installed on your machine as we require NPM - Node Package Manager for installing cordova and Ionic
   - Local Dev environment for [Android](http://appsonmob.com/kickstart-android-development-part-1/) & [ iOS ](http://docwiki.embarcadero.com/RADStudio/XE6/en/Mobile_Tutorial:_Set_Up_Your_Development_Environment_on_the_Mac_(iOS)) should be setup already

### 2.2 Installing Ionic & Cordova

In order to install Ionic & Cordova, you have to install NPM - Node Package Manager - as it's the default package manager for cordova. NPM is used to install a CLI for cordova which enables developers to create a new project, add/remove platforms and add/remove plugins. Ionic has built some extended functionality over Cordova CLI and provides a NPM package.

To install both, run this command in your Terminal/Command Prompt after installing NodeJS.

<!--code lang=coffeescript linenums=true-->

    $ npm install -g ionic cordova

You can check the version of both to see if it has installed the latest version. The latest versions at the time of writing this article are: for Ionic - [v1.0.0-beta.14](https://github.com/driftyco/ionic) and for Cordova [v4.0.0](http://cordova.apache.org/#download).

<!--code lang=bash linenums=true-->

    $ cordova -v
    $ ionic -v

### 2.3 Setting up a Firebase Account
In order to user Firebase, you need to have an account with them. You can sign up on the Hacker Plan for 50 Max Connections, 5 GB Data Transfer, 100 MB Data Storage, 1 GB Hosting Storage and 100 GB Hosting Transfer from this url:
[https://www.firebase.com/signup/](https://www.firebase.com/signup/)

After you signup, a new app will be created for you by default, shown on the dashboard. You can click on `Manage Your App` to go the Forge, which is a single dashboard to manage your app settings and the data. The link for your Firebase db will be [https://your-app-name.firebaseio.com](https://your-app-name.firebaseio.com). 

## 3 App Development 

### 3.1 Scope of the App
In the interest of time, we will keep the scope of the app very simple. We will restrict it to just 3 screens.
   - Login Screen to login/signup using simple Email Address & Firebase
   - Chat Rooms List View to display the complete list of rooms available for chat
   - Room Chat Window where multi person chat will be facilitated

We will be using Firebase Simple login to enable authentication with email and password. We will have a list of static rooms based on pre-defined categories where users can chat. The users can select a particular room and exchange text messages through the room specific chat window.

### 3.2 Creating a Basic Ionic App 
Ionic CLI provides a method to create an app based on different starter templates to bootstrap your projects. We will be creating the app using 'tabs' starter template with tabs for Rooms, Chat & Logout.

<!--code lang=bash linenums=true-->

    $ ionic start MyChat tabs

A new ionic (underlying framework being Angularjs) app will be created with some pre-defined templates and code for a tab based app.

### 3.3 Configuring the router for our app

In Ionic apps, the router is different from the default AngularJS router. Ionic uses [ui-router](http://angular-ui.github.io/ui-router/site/#/api/ui.router)  which helps in managing complex nested states in the application. Ionic has a built in navigation stack which intelligently controls the display of back button on the top bar.

We will update the router configuration with 4 states - one for login, another abstract state for tabs layout and nested views for which two more states need to be defined to show list of rooms & the chat window. The javascript code for router resides in app.js. 

<!--code lang=javascript linenums=true-->

         // MyChat App - Ionic & Firebase Demo

         // 'mychat.services' is found in services.js
          // 'mychat.controllers' is found in controllers.js
         angular.module('mychat', ['ionic', 'mychat.controllers', 'mychat.services'])

         .run(function ($ionicPlatform, $rootScope) {
         $ionicPlatform.ready(function () {
        // Hide the accessory bar by default (remove this to show the accessory bar          above the keyboard
        // for form inputs)
        if(window.cordova && window.cordova.plugins.Keyboard) {
            cordova.plugins.Keyboard.hideKeyboardAccessoryBar(true);
        }
                 if(window.StatusBar) {
                        // org.apache.cordova.statusbar required
                        StatusBar.styleDefault();
                 }

        $rootScope.logout = function () {
            console.log("Logging out from the app");
        }
    }); }).config(function ($stateProvider, $urlRouterProvider) {

    // Ionic uses AngularUI Router which uses the concept of states
    // Learn more here: https://github.com/angular-ui/ui-router
    // Set up the various states which the app can be in.
    // Each state's controller can be found in controllers.js
    $stateProvider

    // State to represent Login View
    .state('login', {
        url: "/login",
        templateUrl: "templates/login.html",
        controller: 'LoginCtrl'
    })

    // setup an abstract state for the tabs directive
    .state('tab', {
        url: "/tab",
        abstract: true,
        templateUrl: "templates/tabs.html"
    })

    // Each tab has its own nav history stack:

    .state('tab.rooms', {
        url: '/rooms',
        views: {
            'tab-rooms': {
                templateUrl: 'templates/tab-rooms.html',
                controller: 'RoomsCtrl'
            }
        }
    })

    .state('tab.chat', {
        url: '/chat',
        views: {
            'tab-chat': {
                templateUrl: 'templates/tab-chat.html',
                controller: 'ChatCtrl'
            }
        }
    })

           // if none of the above states are matched, use this as the fallback
           $urlRouterProvider.otherwise('/login');

         });

We have defined the different html templates associated with each state and the specific controller which will manage the state. In AngularJS, controllers are used to initialize the `$scope` for the view and adding any behavior to the `$scope` object.` $scope` acts as container for Models and is based on POJO i.e. plain old javascript objects. 

We should strictly abstain from using controllers to write logic for DOM manipulation as it should be handled using AngularJS directives. For now, we will create empty controllers to work upon.

<!--code lang=javascript  linenums=true -->

         angular.module('mychat.controllers', [])

              .controller('LoginCtrl', function ($scope, $ionicModal, $state) {
    console.log('Login Controller Initialized');

    $ionicModal.fromTemplateUrl('templates/signup.html', {
        scope: $scope
    }).then(function (modal) {
        $scope.modal = modal;
    });

    $scope.createUser = function (user) {

    }

    $scope.signIn = function () {
        $state.go('tab.rooms');
    } })
    .controller('ChatCtrl', function ($scope, Chats) {
       console.log("Chat Controller initialized");
       $scope.chats = Chats.all();
    })  .controller('RoomsCtrl', function ($scope, Rooms) {
    console.log("Rooms Controller initialized");
    $scope.rooms = Rooms.all(); });

### 3.4 HTML Templates & Styling

Now we will code the views of the app. We should create static views with some dummy data so that the styling and html code are done completely. We will create the templates for Login, Sign Up, Tabs Layout, Chat Room & Rooms List.

Ionic has multiple UI components to help you create an app easily. There are many components that even imitate native UI features and give your app a native user experience. We have used Ion List, Ion Modals and Ion Tabs in our template views. Ionic provides CSS components as well as Javascript components which are actually AngularJS directives.

Screenshots for different static views are given below and the code can be found at the [github repo](https://github.com/mappmechanic/ionic-firebase).

<table> <tr> <td>
<a href='http://postimage.org/' target='_blank'><img src='http://s23.postimg.org/s31tnx5cb/Login_Screen.png' border='0' alt="Login Screen" style="height:300px;border:1px solid;margin:10px;" /></a>
</td><td>
<a href='http://postimage.org/' target='_blank'><img src='http://s23.postimg.org/crs0nb80b/Rooms_List.png' border='0' alt="Rooms List" style="height:300px;border:1px solid;margin:10px;" /></a>
</td><td>
<a href='http://postimage.org/' target='_blank'><img src='http://s23.postimg.org/99g0qx74b/Chat_Window.png' border='0' alt="Chat Window" style="height:300px;border:1px solid;margin:10px;" /></a>
</td></tr></table>

### 3.5 Integrating Firebase to Ionic App

In order to use Firebase in your app, you need to signup for an account with Firebase and create a new app which generates a unique url for you. AngularFire is an open source library maintained by the Firebase team which provides a binding for Firebase in AngularJS projects. As Ionic is based on AngularJS, we will be using this library. We have to include these scripts in our index.html.

<!--code lang=markup linenums=true -->

          <!-- firebase & angularfire libraries -->
          <script src="lib/firebase/firebase.js"></script>
          <script src="lib/angular-fire/angularfire.min.js"></script>

Now we will inject the 'firebase' module to our main 'mychat' module.

<!--code lang=javascript linenums=false -->

         angular.module('mychat', ['ionic', 'mychat.controllers', 'mychat.services', 'firebase'])

Now we can use `$firebase` service to sync data with Firebase and `$firebaseAuth` for Authentication helper functions. 

#### 3.5.1 Implementing SignUp & Login

Firebase has an excellent feature with out of the box authentication providers. It enables your app to use providers like simple login (email & password) or any other OAuth API of Facebook, Twitter, Google, LinkedIn etc. We will be choosing the simple email & password based authentication provider for our app.

The first step is to enable Email & Password based authentication from Firebase's forge control panel for your app. Open Forge and click on the Login & Auth tab on the left to enable the appropriate authentication mechanism.

[![User Authentication](http://s17.postimg.org/c37ewsglr/User_Authentication.png)](http://postimage.org/)

Now we can write the business logic to implement required functionality. We will update the `app.js` with a global variable model under `$rootScope` for firebaseUrl so that it is available to every `$scope`. We will also make a factory named `Auth` for the `$firebaseAuth` object so that we do not have to instantiate it every time it is required. 

<!--code lang=javascript linenums=true-->

        angular.module('mychat.services', ['firebase'])
             .factory("Auth", ["$firebaseAuth", "$rootScope",
             function ($firebaseAuth, $rootScope) {
                var ref = new Firebase(firebaseUrl);
                return $firebaseAuth(ref);
            }]);

We will use the Auth factory in the app.js run method to listen to event `$onAuth` so that we can navigate the user back to the login page whenever a user logs out using method `$unAuth`. The following link provides an exhaustive list of API methods available in the AngularFire library. [https://www.firebase.com/docs/web/libraries/angular/api.html](https://www.firebase.com/docs/web/libraries/angular/api.html)

We will also add code to restrict unauthenticated users to view internal views for Rooms List & Chat window. We will use AngularFire promises `$waitForAuth()` & `$requireAuth()` in resolve this property for different states.

For Login State we will use the following code:

<!--code lang=javascript linenums=true-->

    resolve: {
            // controller will not be loaded until $waitForAuth resolves
            // Auth refers to our $firebaseAuth wrapper in the example above
            "currentAuth": ["Auth",
                function (Auth) {
                // $waitForAuth returns a promise so the resolve waits for it to complete
                    return Auth.$waitForAuth();
        }] }

For other secure states we will use the following code:

<!--code lang=javascript linenums=true-->

    resolve: {
            // controller will not be loaded until $requireAuth resolves
            // Auth refers to our $firebaseAuth wrapper in the example above
            "currentAuth": ["Auth",
                function (Auth) {
             // $requireAuth returns a promise so the resolve waits for it to complete
             // If the promise is rejected, it will throw a $stateChangeError (see above)
                    return Auth.$requireAuth();
      }]
        }

We have to update LoginCtrl controller with 2 methods for signing in & creating a new User. Actually you can integrate Firebase to Angular or Ionic app in two ways, either by using the javascript APIs directly, or by using AngularFire API methods which are exposed as AngularJS services. I would normally recommend using the second method, but the first method can be used in advanced use cases. 

In this tutorial and in our demo app, I have demonstrated both methods for integration. We have used direct Javascript functions in Login & Create User functionalities. The code for LoginCtrl is as follows:

<!--code lang=javascript linenums=true-->

      angular.module('mychat.controllers', [])

     .controller('LoginCtrl', function ($scope, $ionicModal, $state, $firebaseAuth, $ionicLoading, $rootScope) {
    console.log('Login Controller Initialized');

    var ref = new Firebase($scope.firebaseUrl);
    var auth = $firebaseAuth(ref);

    $ionicModal.fromTemplateUrl('templates/signup.html', {
        scope: $scope
    }).then(function (modal) {
        $scope.modal = modal;
    });

    $scope.createUser = function (user) {
        console.log("Create User Function called");
        if (user && user.email && user.password && user.displayname) {
            $ionicLoading.show({
                template: 'Signing Up...'
            });

            auth.$createUser({
                email: user.email,
                password: user.password
            }).then(function (userData) {
                alert("User created successfully!");
                ref.child("users").child(userData.uid).set({
                    email: user.email,
                    displayName: user.displayname
                });
                $ionicLoading.hide();
                $scope.modal.hide();
            }).catch(function (error) {
                alert("Error: " + error);
                $ionicLoading.hide();
            });
        } else
            alert("Please fill all details");
    }

    $scope.signIn = function (user) {

        if (user && user.email && user.pwdForLogin) {
            $ionicLoading.show({
                template: 'Signing In...'
            });
            auth.$authWithPassword({
                email: user.email,
                password: user.pwdForLogin
            }).then(function (authData) {
                console.log("Logged in as:" + authData.uid);
                ref.child("users").child(authData.uid).once('value', function (snapshot) {
                    var val = snapshot.val();
                    // To Update AngularJS $scope either use $apply or $timeout
                    $scope.$apply(function () {
                        $rootScope.displayName = val;
                    });
                });
                $ionicLoading.hide();
                $state.go('tab.rooms');
            }).catch(function (error) {
                alert("Authentication failed:" + error.message);
                $ionicLoading.hide();
            });
        } else
            alert("Please enter email and password both");
    }
    });

We have used the `set` method to create a new object in Firebase. You can think about your Firebase data as a large JSON object - you can manipulate it on the go with the benefit of all clients being notified upon any modifications in the data. We have used `once` method to get the display name of the signed in user.

#### 3.5.2 Showing Rooms List 

We have already created the template for the Rooms view to show list of rooms with their icons. We need to fetch this data from Firebase directly and update to the `$scope` of the RoomsCtrl. In AngularJS it is recommended to write data integration logic as services/factory so we will write `Rooms` factory for the same logic. We have used AngularFire to fetch the data to a local variable and return it using the `all()` method of `Rooms` factory.

<!--code lang=javascript linenums=true-->

      /**
       * Simple Service which returns Rooms collection as Array from Salesforce 
       * & binds to the Scope in Controller
       */
     .factory('Rooms', function ($firebase) {
         // Might use a resource here that returns a JSON array
      var ref = new Firebase(firebaseUrl);
      var rooms = $firebase(ref.child('rooms')).$asArray();

    return {
        all: function () {
            return rooms;
        },
        get: function (roomId) {
            // Simple index lookup
            return rooms.$getRecord(roomId);
        }
    }
    });

We have written code in Controller to glue the `$scope.rooms` model to the data returned by services method `Rooms.all()`. We have also added a method for opening a specific chat room with a specific roomId. It changes the state passing the roomId as a state parameter.

<!--code lang=javascript linenums=true-->

    .controller('RoomsCtrl', function ($scope, Rooms, Chats, $state) {
       console.log("Rooms Controller initialized");
       $scope.rooms = Rooms.all();

      $scope.openChatRoom = function (roomId) {
        $state.go('tab.chat', {
            roomId: roomId
        });
    }
    });

####3.5.3 Implementing Chat Room Functionality

In order to make our chatting work for multiple users, we need to have a chats array under every room object. We will push new chat messages with `from`, `message` and `createdAt` properties. We implement the factory named `Chats` to code the logic for selecting a room, get selected room name to be displayed and then sending the message. The messages will automatically get updated to the chats Firebase object which will be glued to the `$scope.chats` model, thereby updating the DOM.

<!--code lang=javascript linenums=true-->

    .factory('Chats', function ($firebase, Rooms) {

    var selectedRoomId;

    var ref = new Firebase(firebaseUrl);
    var chats;

    return {
        all: function () {
            return chats;
        },
        remove: function (chat) {
            chats.$remove(chat).then(function (ref) {
                ref.key() === chat.$id; // true item has been removed
            });
        },
        get: function (chatId) {
            for (var i = 0; i < chats.length; i++) {
                if (chats[i].id === parseInt(chatId)) {
                    return chats[i];
                }
            }
            return null;
        },
        getSelectedRoomName: function () {
            var selectedRoom;
            if (selectedRoomId && selectedRoomId != null) {
                selectedRoom = Rooms.get(selectedRoomId);
                if (selectedRoom)
                    return selectedRoom.name;
                else
                    return null;
            } else
                return null;
        },
        selectRoom: function (roomId) {
            console.log("selecting the room with id: " + roomId);
            selectedRoomId = roomId;
            if (!isNaN(roomId)) {
                chats = $firebase(ref.child('rooms').child(selectedRoomId).child('chats')).$asArray();
            }
        },
        send: function (from, message) {
            console.log("sending message from :" + from.displayName + " & message is " + message);
            if (from && message) {
                var chatMessage = {
                    from: from.displayName,
                    message: message,
                    createdAt: Firebase.ServerValue.TIMESTAMP
                };
                chats.$add(chatMessage).then(function (data) {
                    console.log("message added");
                });
            }
        }
    }
    });

We will update the ChatCtrl controller to have logic for selecting a room, then fetching room name and expose methods for sending the message and removing it. 

<!--code lang=javascript linenums=true-->

    .controller('ChatCtrl', function ($scope, Chats, $state) {
      console.log("Chat Controller initialized");

    $scope.IM = {
        textMessage: ""
    };

    Chats.selectRoom($state.params.roomId);

    var roomName = Chats.getSelectedRoomName();

    // Fetching Chat Records only if a Room is Selected
    if (roomName) {
        $scope.roomName = " - " + roomName;
        $scope.chats = Chats.all();
    }

    $scope.sendMessage = function (msg) {
        console.log(msg);
        Chats.send($scope.displayName, msg);
        $scope.IM.textMessage = "";
    }

    $scope.remove = function (chat) {
        Chats.remove(chat);
    }
    });

##4 Build Ionic App

As we have completed programming the app, now we can create a build for this app. We will be creating iOS & Android builds but you can only deploy an android build to devices without a developer account. For deploying an iOS App on a device, you need to have an Apple Developer account so you can create the certificates required for building. Without it, you are only able to test the app locally on your iOS simulator.

<!--code lang=bash linenums=false-->

    $ ionic platform add ios android
    $ cordova build
    $ cordova run

I have tested the builds on an iOS simulator and on an Android device and it runs smoothly on a good network connection.

## 5 Conclusion

Firebase is an excellent service for creating real time apps. It eliminates any effort required in developing server side code and managing your server instances. Front end developers looking to create amazing apps without having to learn backend programming should use this service. A large amount of simple to medium use cases can be implemented using Firebase only. 

Ionic also provides nearly same performance as native apps if you use the pre-built components in Ionic. They have been thoroughly tested by the community and drifty team has fixed most of the bugs. I would recommend Ionic for apps which are intended to be delivered on multiple platforms.

In our demo chat app, we have written simple functionality without considering complex edge cases like loading delays or network issues. If you are developing a proper app, you would need to think about your requirements extensively and implement accordingly.

GitHub URL for the Project: [https://github.com/mappmechanic/ionic-firebase](https://github.com/mappmechanic/ionic-firebase)

If you have any questions or suggestions, just tweet me at [@mappmechanic](https://twitter.com/mappmechanic) or send me a message on my [LinkedIn Profile](https://linkedin.com/in/rahatkh).