---
layout: page
title: Social Invites SDK for iOS (tm)
group: "core"
weight: 3
---

Infobip Social Invites library for iOS
=====================================
This project is an iOS library which can be merged with your iOS project and enables you to use the Infobip Social Invites service.

Requirements
------------
Include the following files to your project:

    libInfobipSocialInvite.a
    InfobipSocialInvite.h
    ResouresCore.bundle

- Check is the `.a` file added to `Build Phases -> Link Binary With Libraries`
- Check is the `.bundle` file added to `Build Phases -> Copy Bundle Resources`

####Include frameworks

Include the following frameworks into your project:

    AddressBook.framework
    UIKit.framework
    SystemConfiguration.framework
    MobileCoreServices.framework
    QuartzCore.framework
    CoreData.framework
    CoreTelephony.framework
    CoreGraphics.framework

####Linking library to the Application

Since Objective-C generates only one symbol per class we must also force the linker to load the members of the class by using the -ObjC flag. We also must to force an inclusion of all our objects from our static library by adding the -all_load linker flag. If you skip these flags (sooner or later) you will run into the unrecognised selector error or get other exceptions such as the one observed here.

In the Application target under `Build setting -> Linking -> Other Linker Flags` add the following flags

- `-ObjC`
- `-all_load`

Application registration
-------------------------
In order to use the Social invites library you first have to create an Infobip account ([http://infobip.com/sign_up/](http://infobip.com/sign_up/ "Infobip sign up")) and register your [Mobile application](http://developer.infobip.com/api#!/SocialInvite/GET_1_social_invite_application). After the application registration you will obtain the **application key** and **application secret key**.

The next step is creating a default message for your application after which you will obtain the **default message ID**.

Initialisation
--------------
Initialisation is required for using the main library functionality - sending social invitations.  
For library initialisation, simply call the following method:

    [InfobipSocialInvite initWithSecretKey:APPLICATION_KEY
                          defaultMessageId:MESSAGE_ID];

This method uses your Infobip application key and the default message id for your application, all method parameters are “strings”.

If you want to get or set application key use the method `+(NSString *)applicationKey` or `+(void)setApplicationKey:(NSString *)appKey` from the **InfobipSocialInvite** class.

To get or set the application secret key use the method `+(NSString *)secretKey` or `+(void)setSecretKey:(NSString *)secretKey` from **InfobipSocialInvite** class.

In every moment you can check if the library is initialized by calling the method `(BOOL)isLibraryInitialized` from the **InfobipSocialInvite** class, which returns **true** value if the library is initialized or **false** if it is not.

Sender ID
---------

The sender ID could either be a numeric (e.g. user's MSISDN - phone number) or some alphanumeric number. If you decide to set the sender ID you have to take care of its length – the max length for alphanumeric sender ID is 11 characters and 16 for numeric senders. The method used for setting the sender ID is `(void)setSenderId:(NSString *)senderId`.

Usage example:

    //numeric sender id
    [InfobipSocialInvite setSenderId:@"your_phone_number"];

    //alphanumeric sender id
    [InfobipSocialInvite setSenderId:@"your_name"];

Once you set the sender ID, you can use the `(NSString *)senderId` method to find out the string representation of the sender ID.

Usage example:

    String senderId = [InfobipSocialInvite senderId];

Invitation message
------------------

As mentioned above, after application registration you have to add the default message for your application. This default message is the invitation message for your application. Every time a user sends the invite, he uses the message ID which you provided. The social invite message text will be the text of your default message with the hyperlink included at the end.
If you want to get or set your default message ID you can use the methods `(NSString *)defaultMessageId` or `(void)setDefaultMessageId:(NSString *)defaultMessageId` from **InfobipSocialInvite**, respectively.

You can also get or set your default message text in any moment using methods `(NSString *)defaultMessageText` or `(void)setDefaultMessageText:(NSString *)defaultMessageText` from **InfobipSocialInvite**.

Besides the default message you can enable your users to send personalised messages. For message editing you can use the  `(void)setMessageEditing:(BOOL)edit` method from **InfobipSocialInvite**. If message editing is enabled (meaning that the user can edit default message), the parameter should be **true** or **false** if disabled.

In order to create custom message for the first time you should use the method `(void)createMessageWithText:(NSString *)text placeholder:(NSString *)placeholder successBlock:(IBSIMessageResponseSuccess)success failureBlock:(IBSIResponseFailure)failure` with success and failure block from **InfobipSocialInvite**. The placeholder will be replaced with the url.

This method uses `typedef void (^IBSIMessageResponseSuccess)(ClientMobileApplicationMessageResponse *messageResponse)` and for the error `typedef void (^IBSIResponseFailure)(NSError *error)`, `ClientMobileApplicationMessageResponse` model is discussed in section **Models**.

Usage example:

    [InfobipSocialInvite createMessageWithText:@"New message text <>" placeholder:@"<>"
            successBlock:^(ClientMobileApplicationMessageResponse *messageResponse) {
                NSLog(@"Created new message with text %@ and message ID%@",messageResponse.text,messageResponse.key);
            } failureBlock:^(NSError *error) {
                NSLog(@"Error: %@",[error localizedDescription]);
            }];

If you want to obtain the message for the specific message ID, use the method `(void)getMessageByMessageId:(NSString *)messageId  withSuccessBlock:(IBSIMessageResponseSuccess)success failureBlock:(IBSIResponseFailure)failure`.

Usage example:

    [InfobipSocialInvite getMessageByMessageId:[InfobipSocialInvite customMessageId]
            withSuccessBlock:^(ClientMobileApplicationMessageResponse *messageResponse) {
                NSLog(@"Message text with placeholder: %@", messageResponse.text);
            } failureBlock:^(NSError *error) {
                NSLog(@"Error: %@", [error localizedDescription]);
            }];  

If you just want to update the message text for a specific message ID use **InfobipSocialInvite** method `(void)updateMessageByMessageId:(NSString *)messageId withText:(NSString *)text placeholder:(NSString *)placeholder withSuccessBlock:(IBSIMessageResponseSuccess)success failureBlock:(IBSIResponseFailure)failure`.


To save (set) or get a custom message id use the methods `(NSString *)customMessageId` or `(void)setCustomMessageId:(NSString *)cusMessageId`, respectively.

To save (set) or get a custom message text use the methods`(NSString *)customMessageText` or `(void)setCustomMessageText:(NSString *)cusMessageText` from **InfobipSocialInvite** class.


Sending invitations
-------------------

In order to send the invitation to one or multiple phone numbers use the `+(void)sendSocialInviteToRecipients:(NSArray *)recipients withMessageId:(NSString *)messageId successBlock:(IBSIResponseSuccess)success failureBlock:(IBSIResponseFailure)failure` method with the success and failure block from the *InfobipSocialInvite* class.

This method uses `typedef void (^IBSIResponseSuccess)(SendSmsResponse *sendResponse)` and for the error `typedef void (^IBSIResponseFailure)(NSError *error)`. The “SendSmsResponse” model is discussed in the section **Models**. “MessageId” is optional and if the nil is passed, a default message will be sent.

Usage example:

    NSMutableArray * recipients= [NSMutableArray new];

    [InfobipSocialInvite sendSocialInviteToRecipients:recipients
                                        withMessageId:messageId
                                         successBlock:^(SendSmsResponse *result){
                                             NSLog(@"bulkId: %@", result.bulkId);
                                             NSLog(@"deliveryInfo: %@", result.deliveryInfoUrl);
                                         }
                                         failureBlock:^(NSError *error){
                                             NSLog(@"Error: %@",[error description]);
                                         }];
Message resending
-----------------

You also have a control over the number of invitations that a user can send to one phone number. If you want to let your user send the invitation to one phone number more than once you have to enable invitation resending. You can do that by calling the `[InfobipSocialInvite setSocialInviteResending:true]`method. If you want your user to only be able to send one invitation to one phone number, you should call the previous method with the ***false*** `[InfobipSocialInvite setSocialInviteResending:false]`parameter. This is disabled by default.


Address Book
------------
For sending social invites you will need a list of all the contacts from the user's mobile phone. In order to obtain the list of contacts with phone numbers you can use ***IBSIAddresBook***.

***IBSIAddresBook*** is a wrapper for the ***RHAddressBook*** and it contains every functionality we need to send social invites.
To get the instance of IBSIAddressBook use `+(IBSIAddressBook *)addressBook` method from the ***InfobipSocialInvite*** class.

If you want to set the instance of IBSIAddressBook use the  `+(void)setAddressBook:(IBSIAddressBook *)addressBook` method.
**IBSIAdressBook** uses **Person** class for contact and **InvitationInfo** class for contacts invitation details.

###Authorisation###

In order to get access to the user's contacts, the first thing you need to do is ask the user for permission. Do this with the `-(void)requestAuthorizationWithCompletion:(void (^)(bool granted, NSError* error))completion` function.
To check the authorisation status use the `+(IBSIAuthorizationStatus)authorizationStatus` method witch can return one of the following **IBSIAuthorizationStatus** statuses:

1. ***IBSIAuthorizationStatusNotDetermined***,

2. ***IBSIAuthorizationStatusRestricted***,

3. ***IBSIAuthorizationStatusDenied***,

4. ***IBSIAuthorizationStatusAuthorized***,

Usage example:

        if(([IBSIAddressBook authorizationStatus] == IBSIAuthorizationStatusNotDetermined)) {
        [[InfobipSocialInvite addressBook] requestAuthorizationWithCompletion:^(bool granted, NSError *error) {
            if (granted) {
                // First time access has been granted, add the contact
                NSLog(@"Access to address book has been granted!");
            } else {
                 NSLog(@"User denied access!");

            }
        }];
    } else if ([IBSIAddressBook authorizationStatus] == IBSIAuthorizationStatusAuthorized) {
        // The user has previously given access, add the contact
    }
    else {
        // The user has previously denied access
    }

###User's contacts###

**IBSIAddressBook** works with the list of persons (Person class instance). In order to get the number of the person in the address book, use the `-(long)numberOfPeople` method.

To get the array of all persons sorted by their first name, use `-(NSArray*)peopleOrderedByFirstName`  or `-(NSArray*)peopleOrderedByLastName` if you want to arrange the array by their last name.
If you want to get the person with the specific personId use the `-(IBSIPerson *)personWithId:(NSInteger)personId` method.

To search through the list of people use the `-(NSMutableArray *)searchThroughPeople:(NSArray *)people withText:(NSString *)searchText` method.

###Delivery report for address book###

If you want to update your delivery status for the all persons in the Address book use the `-(void)requestDeliveryStatusAndUpdatePendingInvitationsWithDelegate:(id)delegate` method. Our database contains the person ID, MSISDN, bulk ID and delivery status for all numbers to which the social invite was sent.

The “Send invite method” uses the POST request and response arrives asynchronously. Because of that, at the beginning of the method, the MSISDN and the person ID for all recipients will be entered into the database. If you get a successful response, the bulk id will be entered into the database for all MSISDNs to which the invite message was sent.

If you geta  failure response you have to delete the MSISDNs without the bulk id from the database. You should use the `-(void)updateDatabaseDeleteItemsWithoutBulkId` method for that purpose.

###Person###

The person class contains information about one contact from the Address book. The class uses following properties:

`@property (nonatomic, copy, readonly) NSString *name; ` - alias for compositeName

`@property (nonatomic, copy) NSString *firstName` - person's first name

`@property (nonatomic, copy) NSString *lastName` - person's last name

`@property (nonatomic, copy) NSArray *phoneNumbers` - person's phone numbers

`@property (nonatomic) NSArray *invitationInfos` - array of ***InvitationInfo*** object, one object for one phone number.

To get the contact’s image use the method `-(UIImage*)thumbnail`. In order to check if certain contacts have an image, use the `-(BOOL)hasImage` method.

If you want to send the invitation to only one person (to the all MSISDNs of that person) use the method `-(void)sendInviteWithDeliveryDelegate:(id)deliveryDelegate messageId:(NSString *)messageId`. This method takes into account the message resending mode and duplicated phone numbers. For example, if you turn off the message resending mode for one phone number, the invitation will be sent just once. Also, if you turn on the message resending your user can send more than one invitation to the one phone number

If the one contact has duplicate phone numbers than he will receive just one invite message.

Usage example:

    IBSIPerson *person =[[InfobipSocialInvite addressBook] personWithId:(ABRecordID)sender.buttonId ];
    if ([InfobipSocialInvite customMessageId] == NULL) {
        [person sendInviteWithDeliveryDelegate:self messageId:[InfobipSocialInvite defaultMessageId]];
    }else{
         [person sendInviteWithDeliveryDelegate:self messageId:[InfobipSocialInvite customMessageId]];
    }

To get the number of MSISDNs that are not invited, use the method `-(NSInteger)numberOfNonInvitedMsisdns`.

If the object is not synchroniased with the database, call `-updateDeliveryStatus` to update the delivery statuses from database for the all MSISDNs.

`-(NSSet *)allBulkIds;`  returns all bulk IDs for current person

To get the delivery status for the person, call `-(void)requestDeliveryStatusInSeparateThreadWithDelegate:(id)delegate;`. This will trigger the asynchronous request that will check the person’s delivery status every 8 seconds (10 times) or until it receives the terminating delivery status.

In order to refresh just one row in table view, use the `-(void)requestDeliveryStatusInSeparateThreadWithDelegate:(id)delegate atIndexPath:(NSIndexPath *)indexPath`  method that passes the parameter - indexPath to the delegate method.

Terminal states are all the states that differ from pending and unknown states.
`-(BOOL)inTerminalState` person is in the terminal state if the all his phone numbers are in terminal state.
`-(BOOL)isInPendingState` person is in the pending state if at least one of his phone numbers is in a pending state.
`-(BOOL)isInvited` person is invited if at least the one invitation was successfully sent to that person.


This method `-(IBSIDeliveryStatus)deliveryStatus` returns the delivery status of the person.



###InvitationInfo###

The ***InvitationInfo*** class contains information about the invitations for one phone number. This class has the following properties:

`@property NSString * msisdn` - phone number,

`@property NSInteger personId` - contact id which contains this phone number,

`@property IBSIDeliveryStatus deliveryStatus` - delivery status for the sent invite,

`@property NSString *bulkId` - information which we use for getting delivery status,

To get invitation info use one of the following methods:

`-(IBSIInvitationInfo *)initWithMsisdn:(NSString *)msisdn personId:(NSInteger)personId`,

`-(IBSIInvitationInfo *)initWithMsisdn:(NSString *)msisdn personId:(NSInteger)personId bulkId:(NSString *)bulkId deliveryStatus:(NSString *)status`.

For sending social invites through the Infobip’s platform, the phone numbers must contain the country code. Because some of the phone numbers don’t, you can use the `-(NSString*)formatedMsisdn` method  to add country code to the phone number.

To send the social invite to one MSISDN use the method
`-(void)sendInviteWithDeliveryDelegate:(id)deliveryDelegate messageId:(NSString *)messageId`.

To get the delivery status for InvitationInfo, call  `-(void)requestDeliveryStatusInSeparateThreadWithDelegate:(id)delegate` or
`-(void)requestDeliveryStatusInSeparateThreadWithDelegate:(id)delegate atIndexPath:(NSIndexPath *)indexPath`  This will trigger the asynchronous
 request that will check the person’s (that have Invitation Info) delivery status every 8 seconds (10 times) or until it receives terminating delivery status.

Call this `-(void)updateDeliveryStatus` to update InvitationInfo delivery status from the database.

`-(BOOL)inTerminalState` Invitation Info is in a terminal state if the all phone numbers are not pending or unknown
`-(BOOL)isInvited` returns if “InvitationInfo” has been invited
`-(BOOL)isPending` returns if “InvitationInfo” is in a pending state
`-(BOOL)isInImpossibleState` returns if “InvitationInfo” is in a “can’t invite” state


Delivery status
--------------

After you send the invitation, you can show the sent invitation delivery status to the user.

If you want to get the current delivery status you should use the `+(void)getDeliveryInfoByBulkId:(NSString *)balkId withSuccessBlock:(IBSIDeliveryResponseSuccess)success failureBlock:(IBSIResponseFailure)failure` method. This method returns
`typedef void (^IBSIDeliveryResponseSuccess)(SocialInviteDelivery *deliveryResponse)`for successful response. *SocialInviteDelivery* object contains all information about the delivery response which is in the database too and it is explained in the **Models** section.

Usage example:

    [InfobipSocialInvite getDeliveryInfoByBulkId:bulkId
        withSuccessBlock:^(SocialInviteDelivery *deliveryResponse) {
            NSLog(@"Delivery info: %@", deliveryResponse.deliveryInfo);
            NSLog(@"Resource URL: %@", deliveryResponse.resourceURL);
        } failureBlock:^(NSError *error) {
            NSLog(@"ERROR: %@", [error localizedDescription]);
        }
To get the delivery status for one MSISDN (phone number) from the database, use `(NSString *)deliveryStatusForMsisdn:(NSString *)msisdn` the method from ***InfobipSocialInvite***.

Usage example:

    NSString * status = [InfobipSocialInvite deliveryStatusForMsisdn:@"5551234"];

You can get one of the following five status:

1. ***DeliveredToTerminal***,

2. ***DeliveryUncertain***,

3. ***DeliveryImpossible***,

4. ***MessageWaiting***,

5. ***DeliveredToNetwork***.



InfobipSocialInviteDelegate
---------------------------

The majority of these functionalities uses an http requests such as “GET”, “POST” or “PUT”. This request can return “error” as a response. If you want to handle these errors and be notified when an error occurs, you can use the delegate **InfobipSocialInviteDelegate**.

To use this delegate add it in your class .h file like this:

 #import "InfobipSocialInvite.h"

 @interface YourClassName: NSObject <InfobipSocialInviteDelegate>

This means that the “YourClassName” class is a subclass of NSObject (or another class) and conforms to the InfobipSocialInviteDelegate protocol.

Delegate protocols exist so that you can make sure that the delegate object has the necessary methods to deal with your messages. Methods in the delegate protocol can be optional or enforced. Any methods that are optional don't have to be defined. All the methods in **InfobipSocialInviteDelegate** are optional so you can freely choose which one to use.

If you want to include your code when sent invites return an error, define the method `-(void)sendInviteDidReceiveErrorResponse:(NSError *)error` in your class include your code in its body. To work with a successful response for sent invites use the `-(void)sendInviteDidReceiveSuccessfulResponse:(SendSmsResponse *)response` delegate method.

For delivery status error define the `-(void)getDeliveryInfoDidReceiveErrorResponse:(NSError *)error` method. For a successful response use the `-(void)getDeliveryInfoDidReceiveSuccessfulResponse:(SocialInviteDelivery *)response` method.

Edit message can create, get or update message so you can use one of the following methods to get error: `-(void)createMessageDidReceiveErrorResponse:(NSError *)error`, `-(void)getMessageDidReceiveErrorResponse:(NSError *)error` or `-(void)updateMessageDidReceiveErrorResponse:(NSError *)error`. For successful response use one of the following methods: `(void)createMessageDidReceiveSuccessfulResponse:(ClientMobileApplicationMessageResponse *)response`,` -(void)getMessageDidReceiveSuccessfulResponse:(ClientMobileApplicationMessageResponse *)response` or `-(void)updateMessageDidReceiveSuccessfulResponse:(ClientMobileApplicationMessageResponse *)response`.

Models
------
###ClientMobileApplicationMessageResponse###

The *ClientMobileApplicationMessageResponse* class contains the information about the application's message and has the following properties:

`@property (nonatomic, strong) NSString* clientMobileApplicationKey` - contains application key,

`@property (nonatomic, strong) NSString* key`- the message id,

`@property (nonatomic, strong) NSString* text;` - the message text with the placeholder,

`@property (nonatomic, strong) NSString* placeholder`- the placeholder which defines a text that will be recognised as a substring of the message text and replaces it with a shortened link of the application's download page.
If you want to work with this information just use the “property getter” like this:

     ClientMobileApplicationMessageResponse *messageResponse = [[ClientMobileApplicationMessageResponse alloc]init];
                NSLog(@"Message text with placeholder: %@", messageResponse.text);
                   NSLog(@"Message key: %@", messageResponse.key);


###SendSmsResponse###

*SendSmsResponse* class contains the response to the social invite and has the following properties:

`@property (nonatomic, strong) NSString* bulkId` - message bulk id,

`@property (nonatomic, strong) NSString* deliveryInfoUrl` - url for getting the delivery info for current message bulk

`@property (nonatomic, strong) NSSet *responses`- set of responses containing information about every sent message (messageId, status and price).

To work with this information just use the “property getter”

###SocialInviteDelivery###

The *SocialInviteDelivery* class contains information about the delivery report for the specific bulk id and has the following properties:

`@property (nonatomic, strong) NSString* resourceURL` -  an URL uniquely identifying this request,

`@property (nonatomic, strong) NSSet *deliveryInfo` - set of objects contains the delivery information, i.e. an address/deliveryStatus pair for each address that you asked to send the message to. More information about **DeliveryInfo** can be found below.

To work with this information just use the “property getter”.

####DeliveryInfo####

*DeliveryInfo* class contains information about the delivery status for one phone number and has the following properties:


`@property (nonatomic, strong) NSString* address`- the address of the recipient (normally, the MSISDN),

`@property (nonatomic, strong) NSString* clientCorrelator`- bulk id for that number,

`@property (nonatomic, strong) NSString* messageId` - the message ID,

`@property (nonatomic, strong) NSNumber* personId` - contact id which contains this phone number,
`@property (nonatomic, strong) NSString* deliveryStatus` - one of five possible statuses.

Delivery status may be one of the following:

***DeliveredToTerminal***, indicating successful delivery to the terminal,

***DeliveryUncertain***, delivery status unknown, e.g. because it was handed off to another network,

***DeliveryImpossible***, unsuccessful delivery - the message could not be delivered before it expired,

***MessageWaiting***, the message is still queued for delivery. This is a temporary state, pending transition to one of the preceding states,

***DeliveredToNetwork***, successful delivery to the network.


To work with this information just use the “property getter”



Owners
------
Framework Integration Team @ Infobip Belgrade, Serbia

*IOS is a trademark of Cisco in the U.S. and other countries and is used under license.*

© 2014, Infobip Ltd.
