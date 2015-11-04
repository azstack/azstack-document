# 1. Create application
AZStack will create & provide an application ID (appID) for your application and a RSA public key (appKey); appID will be stored inside your app (client), and public key will be stored in your server.

azStackUserID: Each unique user of your application need 01 unqiue identifier (azStackUserID) in AZStack server. Its format type is string and called azStackUserID. 

For example, if your user use email to identify user, and you have 2 different users: user1@email.com, user2@abc.com then they need 2 different azStackUserID, like: user1_email_com, user2_abc_com. Or can use their emails (user1@email.com, user2@email.com as azStackUserID).

Another example, if your system use mobile number, username, ... to identify user then can use it as azStackUserID.

To avoid complexity, please use same user id in your database (username, email, phone number, ...) as in AZStack (azStackUserID).
(Two different apps can have 2 users with same azStackUserID)


# 2. Add the SDK to your Xcode Project
### 2.1. Download AZStack Framework t?i:

https://www.dropbox.com/s/7giv1isvzjp3m9y/AzStack_SDK.zip?dl=0

>a. Unzip the file: AzStack.framework, AzStackRes.bundle


>b. Drag the AzStack.framework and AzStackRes.bundle to Frameworks in Project Navigator. Create a new group Frameworks if it does not exist.

>c. Choose Create groups for any added folders.

>d. Deselect Copy items into destination group's folder. This references the SDK where you installed it rather than copying the SDK into your app.


![Add the SDK 1](http://azstack.com/docs/static/AddTheSDK1.png "Add the SDK 1")

![Add the SDK 2](http://azstack.com/docs/static/AddTheSDK2.png "Add the SDK 2")


### 2.2. Configure Xcode Project
> a. Add Linker Flag
Open the "Build Settings" tab, in the "Linking" section, locate the "Other Linker Flags" setting and add the "-ObjC" flag:

![Add Linker Flag](http://azstack.com/docs/static/ConfigOtherLinkerFlags.png "Add Linker Flag")

Note:
This step is required, otherwise crash will happen:
```objective-c
[AzFMDatabase columnExists:inTableWithName:]: unrecognized selector sent to instance 0x...
```
> b. Add other frameworks and libraries
Open the "Build Phases" tab, in the "Link Binary With Libraries" section, add frameworks and libraries:

- CoreGraphics
- CoreLocation
- libxml2
- MobileCoreServices
- CoreMedia
- QuartzCore
- AssetsLibrary
- CoreTelephony
- CoreText
- MapKit
- libz
- SystemConfiguration
- AVFoundation
- ImageIO
- MessageUI
- Security
- CFNetwork
- AudioToolbox
- MediaPlayer
- libsqlite3.0.dylib

![Add other frameworks and libraries](http://azstack.com/docs/static/Libraries.png "Add other frameworks and libraries")

# 3. Concepts and flow

Before ingtegration and users can call & chat, SDK and authentication process needed to initialized by 3 parties: client (use AZStack SDK), AZStack server v� your server. This is to make sure you authorize/ not authorize any unwanted user to make a chat/ call.

Process is described by a model behind:

![AZStack init and authentication](http://azstack.com/docs/static/IosAuthentication.png "AZStack init and authentication")

# 4. SDK initialization
AZStack SDK initialization should be done at the start of a function:
```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
```
### 4.1. Setup AppID
```objective-c
[[AzStackManager instance] setAppId:@"YOUR_APP_ID_HERE"];
```

### 4.2. Setup Server
```objective-c
[[AzStackManager instance] setServerType: AZSERVER_PRODUCTION];
```
- AZSERVER_PRODUCTION: Server for real operation, used when your system & app is stable and completed.
- AZSERVER_TEST: Server for test purpose, used for development phase.

### 4.3. Setup delegates of AZStack:
- AzAuthenticationDelegate
- AzUserInfoDelegate
- AzChatDelegate
- AzCallDelegate

We will explain those delegates [at step 5]. Please see sample code [here].

### 4.4. Setting some parameters:
- Title color, button on navigation bar to match with your app screen:

  Notes: This setup only impact UIViewController of SDK, not impacted on other UIViewController inside your app

```objective-c
[[AzStackManager instance] setTintColorNavigationBar:[UIColor whiteColor]];
```
- Set language displayed in AzStack SDK

  By default, SDK will check system language to see if it support system language or not. If not, SDK will use English as default. 

```objective-c
[[AzStackManager instance] setLanguage:@"vi"];
```
  Input parameter is language code. Example: English: �en�, Vietnamese: �vi� 

- Debug log display:

  Allow to display debug log of SDK or not?

```objective-c
[[AzStackManager instance] setDebugLog:YES];
```

### 4.5. K?t n?i v� x�c th?c v�o AZStack Server
```objective-c
//connect AZ
[[AzStackManager instance] connectWithCompletion:^(NSString * authenticatedAzStackUserID, NSError *error, BOOL successful) {
        if (successful) {
            AzStackLog(@"Connect AzStack Server Successful: authenticatedAzStackUserID : %@", authenticatedAzStackUserID);
        }
        else{
            AzStackLog(@"Connect AzStack Server Fail! ResponseCode: %d, Error Message: %@", error.code, [error description]);
        }
    }];
```

This function should be called right after user is authorized with your server.

Authorization process between your application (AZStack SDK), AZStack server and your server is described in step 3.

# 5. Process delegates of AZStack SDK
### 5.1. AzAuthenticationDelegate:
```objective-c
- (void) azNonceReceived:(NSString *)nonce
```

After clients connect successfully to AZStack by calling function [[AzStackManager instance] connectWithCompletion:] at step 4.5 then AZStack will return [none] to client.

This delegate is called by AZStack SDK after receive [none] response from AZStack server, this function need to send azStackUserID, none to your seerver in order to get 1 authenToken.

At your server, authenToken must be generated by string encryption:
```objective-c
{"azStackUserID":"user_1", "nonce":"none_1"}
```
by publicKey generated in step 1. Where as, user_1 and none_1 is sent by client. See sample PHP code here.

After client get authenToken from your server, you need to send this authenToken so inorder to authenticate at AzStack server to complete authentication process and connect to Server AzStack

```objective-c
[[AzStackManager instance] authenticateWithIdentityToken:authenToken];
```
### 5.2. AzUserInfoDelegate
> a. Request information of 1 user
```objective-c
- (void) azRequestUserInfo: (NSArray *) azStackUserIds withTarget: (int) target;
```
This function is caleld by AZStack SDK in order to inform SDK need to collect user information in table: listAzStackUserIDs.


Now, you can get information from user at client (if stored) or from your server, then pass information to AZStack SDK by calling function: 
```objective-c
[[AzStackManager instance] sendUserInfoToAzStack:listUserInfo withTarget:purpose.intValue];
```
See sample code here.

> b. Request your user's friend list
```objective-c
- (NSArray *) azRequestListUser;
```

AZStack SDK will call this function in order to call this function in order to get user's friendlist (when creating new group for example, ...)

See sample code here.

> c. Need 1 controller to display user information
```objective-c
- (UIViewController *) azRequestUserInfoController: (AzUser *) user withAppUserId: (NSString *) appUserId;
```

AZStack SDK will call this function to retrieve UIViewController in order to display user information.

See sample code here.

### 5.3. AzCallDelegate
```objective-c
- (void) azJustFinishCall: (NSDictionary *) callInfo;
```
This function AZStack SDK call to inform the call is over.

### 5.4. AzChatDelegate

> a. Request navigation controller
```objective-c
- (UINavigationController *) azRequestNavigationToPushChatController;
```

THis function AZStack SDK call to retrieve UINavigationController in order to push ChatController when user clicks on In-app Notification 

![In-app Notification](http://azstack.com/docs/static/FakeNotification.png "In-app Notification")

or after makeing a group.

See sample code here.

> b. Display unread message changed
```objective-c
- (void) azUpdateUnreadMessageCount: (int) unreadCount;
```
AZStack SDK call this function to inform when unread messages changed.


# 6. Create chat window (ChatController)
```objective-c
UIViewController* chatController =  [[AzStackManager instance] chatWithUser:self.contact.username withUserInfo:@{@"name": self.contact.fullname}];
```

```objective-c
- (UIViewController *) chatWithUser: (NSString *) azStackUserId;
```
Call this feature to create chat controller (chat window with 1 user).

In case, you want to call Controller to select user then call:
```objective-c
[[AzStackManager instance] createChat11];
```
This Controller will retriev user lise from AzUserInfoDelegate 

# 7. Call to single user
```objective-c
[[AzStackManager instance] callWithUser:self.contact.username withUserInfo:@{@"name": self.contact.fullname}];
```
```objective-c
[[[AzStackManager instance] callWithUser: azStackUserId];
```

Use this function to make a call to a user from current user.

# 8. Create group chat
```objective-c
- (UIViewController *) chatWithGroup: (NSArray *) azStackUserIds withGroupInfo: (NSDictionary *) groupInfo;
```

azStackUserIds: Array of AZStackUserIds 

In case you want to call controller to get azStackUserIds easily, you can call this function:
```objective-c
[[AzStackManager instance] createChatGroup];
```

This will get list of users from t? AzUserInfoDelegate 

# 9. Push notification
### 9.1. Register for push notification

Firstly, you have to register push notification for your app, in function:
```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions 
```

add this code:
```objective-c
if([UIDevice currentDevice].systemVersion.floatValue >= 8.0) {
    UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:(UIRemoteNotificationTypeBadge|UIRemoteNotificationTypeSound|UIRemoteNotificationTypeAlert) categories:nil];
    [[UIApplication sharedApplication] registerUserNotificationSettings:settings];
} else {
    [[UIApplication sharedApplication] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeAlert)];
}
```

### 9.2. Send Device Token to AZStack server and process push notification / local notification
See sample code:
```objective-c
- (void)application:(UIApplication*)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData*)deviceToken {
    [[AzStackManager instance] registerForRemoteNotificationsWithDeviceToken:deviceToken];
}
- (void)application:(UIApplication*)application didFailToRegisterForRemoteNotificationsWithError:(NSError*)error {
    NSLog(@"didFailToRegisterForRemoteNotificationsWithError: %@", error);
}
- (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings {
    [application registerForRemoteNotifications];
}
- (void)application:(UIApplication *)application handleActionWithIdentifier:(NSString *)identifier forRemoteNotification:(NSDictionary *)userInfo completionHandler:(void(^)())completionHandler {
    if ([identifier isEqualToString:@"declineAction"]){
    } else if ([identifier isEqualToString:@"answerAction"]){
    }
}
- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notif {
    [[AzStackManager instance] processLocalNotify:notif];
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    [[AzStackManager instance] processRemoteNotify:userInfo];
}
```
### 9.3. Process when user click on local notification / push notification:
```objective-c
[[AzStackManager instance] processRemoteNotify:launchOptions[UIApplicationLaunchOptionsRemoteNotificationKey]];
UILocalNotification *locationNotification = [launchOptions objectForKey:UIApplicationLaunchOptionsLocalNotificationKey];
if (locationNotification) {
    [[AzStackManager instance] processLocalNotify:locationNotification];
}
```
this code has to be called at the end of the function:
```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
```
(Before function didFinishLaunchingWithOptions return)

 
