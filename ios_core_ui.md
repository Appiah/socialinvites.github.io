---
layout: page
title: Social Invites SDK for iOS (tm) with UI
group: "core_ui"
weight: 4
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

At any moment you can check if the library has been initialised by calling the `(BOOL)isLibraryInitialized`  method from the **InfobipSocialInvite** class, which returns **true** value if the library has indeed been initialized or **false** if it hasn’t.

Sender ID
---------

The sender ID could either be a numeric (e.g. user's MSISDN - phone number) or some alphanumeric. If you decide to set the sender ID you have to take care of its length – the max length for alphanumeric sender ID is 11 characters and 16 for numeric senders. The method used for setting the sender ID is `(void)setSenderId:(NSString *)senderId`.

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

You can obtain your default message in any moment using method `(NSString *)defaultMessageText` from **InfobipSocialInvite**.

If you want, it is possible for your users to send personalised messages instead of your default message by calling the `(void)setMessageEditing:(BOOL)edit` method from **InfobipSocialInvite**. In that case, the hyperlink will be added to the end of the message. All you have to do is to enable the sending of a custom message.

If custom message sending is enabled, the message line will be shown in the bottom of contacts table view. That line contains the message text that will be sent and pen icon. Clicking on the line opens the “edit message” dialog where the user can customise his invitation message. The user has option to reset his custom message to the default message. If the user clicks the “save” button his custom message will continue to be sent to his contact until he closes the application. Then, it will be reset to the default message. By default, your default message text is used when sending.

User's contacts list
---------------------
In order to allow your users to send invitations to their contacts you have to include the table view containing all the contacts from the user’s address book. To do that you have to call the following method:

    (void)startSocialInviteView:(UIViewController *)viewContoller
                           block:(void(^)())block;

The contacts table view contains a list of all the contacts from the user's address book. If a contact has his picture stored on the user's device, the same picture will be shown in the list. If he doesn't, that place will be taken by a default image.

<img style="width: 480px;" src="http://www.infobip.com/images/demo/1_ios_contacts_table_view_image_without_any_status_icon_with_contacts_with_image.png" />


For contacts with more than one phone number, the overall number of the phone numbers is presented in the brackets next to the Invite button. When a user clicks on a contact, all numbers are shown in dropdown list. The user can choose if he wants to invite his friends by sending the invitations only to one phone number or to all contact numbers at the same time.

<img style="width: 480px;" src="http://www.infobip.com/images/demo/2_ios_expanded_contact_with_more_than_one_phone_number.png" />

If the user tries to the send the invitation when he does not have an active Internet connection a dialog box warning for a lack of data connection will be displayed.

<img style="width: 480px;" src="http://www.infobip.com/images/demo/3_ios_connect_to_internet_dialog_image.png" />


Sending an invite
------------------------
After you have included the contacts table view, the user simply has to click on the ***invite*** button next to the contact he wants to invite.

In case a contact has more than one number the user can send the invite to all numbers by clicking on the ***invite*** button. If he wants to choose one number, he has to click on the contact and to get a list of the numbers. By clicking on any number from the list, the invitation message will be sent only to that specific number.

<img style="width: 480px;" src="http://www.infobip.com/images/demo/4_ios_list_of_contact_numbers_image.png" />

Message resending
-----------------
You also have a control over the number of invitations that a user can send to one phone number. If you want to let your user send the invitation to one phone number more than once you have to enable invitation resending. You can do that by calling the `[InfobipSocialInvite setSocialInviteResending:true]`method.

If you want your user to only be able to send one invitation to one phone number, you should call the previous method with the ***false*** `[InfobipSocialInvite setSocialInviteResending:false]`parameter. This is disabled by default.

Delivery status
---------------

When a user sends the invitation, he may want to see its delivery status. The delivery status icon is shown in the front of every invited phone number and over the contact's image. This way the user has all the information he needs.

There are three possible states of an invitation message:

- **Delivered** - if the message has been delivered to the invited phone number
- **Failed** - if the message has failed to be delivered to the invited phone number or if the phone number is invalid,
- **Pending** - if the message has no terminal status as of yet.

<img style="width: 480px;" src="http://www.infobip.com/images/demo/5_ios_delivery_status_for_one_or_more_contacts_image.png" />

Every time a user opens the contacts table view, the delivery status checking (for pending phone numbers) starts.

Infobip social invite delegate
------------------------------

The majority of these functionalities uses an http requests such as “GET”, “POST” or “PUT”. This request can return “error” as a response. If you want to handle these errors and be notified when an error occurs, you can use the delegate **InfobipSocialInviteDelegate**.

To use this delegate, add it in your class .h file like this:

    #import "InfobipSocialInvite.h"
    @interface YourClassName: NSObject <InfobipSocialInviteDelegate>

This means that the “YourClassName” class is a subclass of NSObject (or another class) and conforms to the InfobipSocialInviteDelegate protocol.

Delegate protocols exist so that you can make sure that the delegate object has the necessary methods to deal with your messages. Methods in the delegate protocol can be optional or enforced. Any methods that are optional don't have to be defined. All the methods in **InfobipSocialInviteDelegate** are optional so you can freely choose which one to use.

If you want to include your code when sent invites return an error, define the method `-(void)sendInviteDidReceiveErrorResponse:(NSError *)error` in your class include your code in its body. To work with a successful response for sent invites use the `-(void)sendInviteDidReceiveSuccessfulResponse:(SendSmsResponse *)response` delegate method.

For the delivery status error define the `-(void)getDeliveryInfoDidReceiveErrorResponse:(NSError *)error`method. For a successful response use the `-(void)getDeliveryInfoDidReceiveSuccessfulResponse:(SocialInviteDelivery *)response` method.

“Edit message” can create, get or update the message so you can use one of the following methods to get an error: `-(void)createMessageDidReceiveErrorResponse:(NSError *)error`, `-(void)getMessageDidReceiveErrorResponse:(NSError *)error` or `-(void)updateMessageDidReceiveErrorResponse:(NSError *)error`. For a successful response use one of the following methods: `-(void)createMessageDidReceiveSuccessfulResponse:(ClientMobileApplicationMessageResponse *)response`, `-(void)getMessageDidReceiveSuccessfulResponse:(ClientMobileApplicationMessageResponse *)response` or `-(void)updateMessageDidReceiveSuccessfulResponse:(ClientMobileApplicationMessageResponse *)response`.
Owners
------
Framework Integration Team @ Infobip Belgrade, Serbia

*IOS is a trademark of Cisco in the U.S. and other countries and is used under license.*

© 2014, Infobip Ltd.
