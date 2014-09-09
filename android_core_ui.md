---
layout: page
title: Social Invites SDK for Android (tm) with UI
group: "core_ui"
weight: 4
---

# Social Invites library for Android™ #

This project is an Android library which can be merged with your Android project and enable you to use Infobip Social Invites.

## Requirements ##

- The lowest required Android SDK version is set to 8.

## Step by step integration ##

1. Copy the library file into libs folder.
2. Add dependency to build.gradle file (`compile (name:'social-invites-ui', ext:'aar')`)
3. Add activity declaration and user permissions to application project Manifest.xml file.
4. Rebuild your project.

User permissions:

	<uses-permission android:name="android.permission.READ_CONTACTS" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.INTERACT_ACCROSS_USERS_FULL" />

Activity declaration:

	<activity
    	android:name="com.infobip.socialinvites.android.lib.ui.ContactsActivityLib"
        android:configChanges="orientation|screenSize"
        android:windowSoftInputMode="stateHidden" >
	</activity>

### Application registration ###

In order to use the Social Invites library you have to create an Infobip account and register your mobile application. After registering the application you will obtain the **application key** and the **application secret key**.
The next step is creating the default message for your application after which you will obtain the **default message ID**.

### Initialization ###

Initialization is required for using the main library functionality - sending social invitations.

The first thing you have to do is to create the *SocialInvitesUIManager* object with **context** (type: android.content.Context) as only and mandatory parameter.

	SocialInvitesUIManager socialInvitesManager = new SocialInvitesUIManager(context);

You have to create the *ClientMobileApplication* object with the **application key** (type: String) and the **application secret key** (type: String):

    ClientMobileApplication application = new ClientMobileApplication("application_key_abc123", "application_secret_abc123");

and the *InfobipAccountCredentials* object with your **Infobip account username** (type: String) and **Infobip account password** (type: String):

    InfobipAccountCredentials ibCredentials = new InfobipAccountCredentials("infobip_username", "infobip_password");

Then you should call the method `initializeLibrary(ClientMobileApplication application, String defaultMessageId, InfobipAccountCredentials ibCredentials)` or the method `initializeLibrary(ClientMobileApplication application, String defaultMessageId)` from *SocialInvitesUIManager* class. These methods throw *IllegalArgumentException* if any of the parameters is null or an empty string.

Passing *InfobipAccountCredentials* object is optional, but be aware that, if you choose to initialize library without credentials, you will not be able to use some library functionalities.

    socialInvitesManager.initializeLibrary(application, "default_message_id_abc123", ibCredentials);

or

	socialInvitesManager.initializeLibrary(application, "default_message_id_abc123");

At any moment you can check if the library has been initialized by calling the `isLibraryInitialized()` method. If it is not, the method will initialize it.

Usage example:

	SocialInvitesUIManager socialInvitesManager = new SocialInvitesUIManager(activity.getBaseContext());
    ClientMobileApplication application = new ClientMobileApplication("application_key_abc123", "application_secret_abc123");
    InfobipAccountCredentials ibCredentials = new InfobipAccountCredentials("infobip_username", "infobip_password");
    if (!socialInvitesManager.isLibraryInitialized()) {
        socialInvitesManager.initializeLibrary(application, "default_message_id_abc123", ibCredentials);
    }

#### Sender ID ####

In order to send an invitation you must set the Sender ID.

The sender ID could either be a certain MSISDN (e.g. user's MSISDN - phone number) or an alphanumeric ID. If you decide to use the alphanumeric ID you have to take care of its length - max length of the alphanumeric sender ID is 11 characters. The method which is used for setting the Sender ID is `setSenderId(String senderId)`. If the Sender ID is null or an empty string, *IllegalArgumentException* will be given.

Usage example:

    socialInvitesManager.setSenderId("sender_id");

If you want to use the user's MSISDN as a sender ID, you can use the *SocialInvitesUIManager* method `fetchMsisdn()` which tries to get MSISDN from the user's mobile phone. Be aware that this method **will not return the MSISDN in every case**. It returns either MSISDN or null if fetching MSISDN is not possible on that device. If you use this method, you will have to handle *CannotFetchUserMSISDNException* if the user's MSISDN is not found on the phone. A recommendation for you is to try to fetch user's MSISDN, but you need to have a mechanism for getting the user's phone number. You can use Infobip’s 2-Factor Authentication solution (2FA).

Usage example:

    String senderId = "";
    try {
        senderId = socialInvitesManager.fetchMSISDN();
    } catch (CannotFetchUserMSISDNException e){
        senderId = "sender_id";
    }

    socialInvitesManager.setSenderId(senderId);

Once you set the Sender ID or you fetch the MSISDN, you can use the method `getSenderId()` to find out the string representation of the Sender ID saved for that specific user.

Usage example:

    String senderId = socialInvitesManager.getSenderId();

#### Invitation message ####

As mentioned above, after registering your application, you have to add the default message for your application. This default message is the invitation message for your application. Every time a user sends the invitation, the text of the message will be the text of the default message with the hyperlink included. You can get your default message at any moment as a *ClientMobileApplicationMessage* object using the `getDefaultMessage()` method from the *SocialInvitesUIManager*.

If you want, you can provide your users with the possibility to send personalized messages instead of your default message by calling the `enableSendingCustomMessage()` method from *SocialInvitesUIManager*. **You can do this only if you have initialized library with Infobip account credentials.** In that case, the hyperlink will be added to the end of the customized message. All you have to do is to enable custom message sending. If custom message sending is enabled, the message line will be shown in the bottom of the contacts layout. That line contains message text which will be sent and the pen icon. Clicking on the **Pen icon** opens the edit message dialog where a user can customize his invitation message. The user has option to reset his custom message to default.

**WARNING:** If you allow users to change the message, remember that a user can send a message with content which you cannot approve or uses this functionality to other purposes (e.g. if both resend mode and edit message are enabled).

<img style="width: 480px;" src="http://www.infobip.com/images/demo/EditMessageLineImage.png" />

<img style="width: 480px;" src="http://www.infobip.com/images/demo/EditMessageDialogImage.png" />


#### User's contacts list ####

Contacts layout contains the list of all contacts from the user's address book. If the contact has a picture stored on the user's device, the same picture will be shown in the list. If the contact doesn't have a  picture, the default picture will be shown instead.

<img style="width: 480px;" src="http://www.infobip.com/images/social_invites/ContactsLayoutImage.png" />

For contacts with more than one phone number, the number of phone numbers is presented in the brackets beside **Invite** button. When the user clicks on the contact, all numbers are shown in the dropdown list. The user can choose whether he will invite his friend by sending invitations only to one phone number or to all numbers at the same time.

<img style="width: 480px;" src="http://www.infobip.com/images/social_invites/ExpandedContactWithMoreThanOnePhoneNumber.png" />

If the user tries to send the invitation while he has no Internet connection (WiFi and 3G are turned off), the dialog for enabling WiFi or 3G network will be displayed. When a user enables his network connection, the invitation will be sent automatically.

<img style="width: 480px;" src="http://www.infobip.com/images/social_invites/ConnectToInternetDialog.png" />

You can also manage and control the number of invitations that a user can send to one phone number. If you want to let the user send invitations to one phone number more than once, you have to enable invitation resending. You can do it by calling the `enableResendingInvitation()` method. And if you want a user to send only one invitation to one phone number, you should call the `disableResendingInvitation()` method. This is disabled by default.

Usage example:

    socialInvitesManager.enableInvitationResending();
    socialInvitesManager.disableInvitationResending();

#### Delivery statuses ####

After a user sends the invitation, he may want to see if it has been delivered. **He will not be able to see delivery status if you have not initialized library with Infobip account credentials.**  Anyway, you will be able to enable getting delivery statuses by calling the method `enableGettingDeliveryStatus()` or disable it by calling the `disableGettingDeliveryStatus()` from *SocialInvitesUIManager* class. If getting delivery status is disabled, status of invitation will be `SENT` and this status will be stored in the database. Getting delivery status will be disabled by default if you initialize library without Infobip account credentials.

	socialInvitesManager.enableGettingDeliveryStatus();
	socialInvitesManager.disableGettingDeliveryStatus();

Delivery status icon is shown in front of every invited phone number and over the contact's image, so the user has the information about which contact and which phone number he has invited.

There are four possible delivery statuses:

- **Delivered** - if the message has been delivered to the invited phone number,
- **Failed** - if the message delivery to the invited phone number has failed or if the phone number is invalid,
- **Pending** - if message has no final status yet,
- **Sent** - if message has been sent and getting delivery status is disabled. In this case, delivery status icon will be the same as delivered icon.

<img style="width: 480px;" src="http://www.infobip.com/images/social_invites/SlikaSaStatusima.png" />

Every time a user opens his contacts activity, checking of the delivery status for pending phone numbers starts.

# CUSTOMIZATION #

### User Interface Customization ###

If you want you can customize your user interface by using your own resources in order to adjust it to your application design.

Create a new *UICustomizationManager* instance in your code (with context as only and mandatory parameter).

    UICustomizationManager uiCustomizationManager = new UICustomizationManager(context);

#### Customization of layouts ####

It is possible to redesign whole contacts activity or just part of it (contact item or contact detail from our list view), or to redesign dialogs (message editing dialog and Internet connection dialog). You have to create the appropriate layout and to put it in your **res/layout** folder for each of these customizations.

##### Contacts activity layout #####

For activity customization where the contacts will be displayed, you have to create your layout with two mandatory elements.

- EditText for searching contacts with id: `inputSearch`,
- ExpandableListView for displaying contacts data with id: `contactsListView`.

If you want to give your users the possibility to read and change the message text, you will have to put three more mandatory elements into your layout.

- LinearLayout for displaying and editing message text with id: `editMessageLinearLayout` and default height set to 0dp,
- TextView for displaying message text inside linear layout with id: `defaultMessageTextView`,
- ImageView or TextView for opening the dialog for message text editing inside a linear layout with id: `editMessageView`.

You can reorder and customize elements as you see fit.

Usage example:

    <EditText
        android:id="@+id/inputSearch"
    ... />

    <ExpandableListView
        android:id="@+id/contactsListView"
        ... />

    <LinearLayout
        android:id="@+id/editMessageLinearLayout"
        android:layout_height="0dp"
        ... >

        <TextView
            android:id="@+id/defaultMessageTextView"
            ... />

        <ImageView
            android:id="@+id/editMessageView"
            ... />

    </LinearLayout>

Or:

    <LinearLayout
        android:id="@+id/editMessageLinearLayout"
        android:layout_height="0dp"
        ... >

        <TextView
            android:id="@+id/editMessageView"
            ... />

        <TextView
            android:id="@+id/defaultMessageTextView"
            ... />

    </LinearLayout

    <ExpandableListView
        android:id="@+id/contactsListView"
        ... />

    <EditText
        android:id="@+id/inputSearch"
    ... />

After finishing your layout file and save it into the **res/layout** folder, you should call the `setContactsActivityLayout(String customContactsLayoutName)` method to apply your contact details theme.

Usage example:

    uiCustomizationManager.setContactsActivityLayout("custom_contacts_layout");

##### Contact item layout #####

For the customization of contact items in the list you have to create your layout with four mandatory elements.

- ImageView for displaying the contact image with id: `contactImage`,
- ImageView or TextView for displaying delivery status with id: `statusView`,
- TextView for displaying the contact name with id: `contactNameTextView`,
- TextView for sending the invitation to all contact numbers with id: `inviteContactTextView`.

You can reorder and customize elements as you see fit.

Usage example:

    <ImageView
       android:id="@+id/contactImage"
       ... />

    <ImageView
       android:id="@+id/statusView"
       ... />

    <TextView
       android:id="@+id/contactNameTextView"
       ... />

    <TextView
       android:id="@+id/inviteContactTextView"
    ... />  

Or:

    <ImageView
       android:id="@+id/contactImage"
       ... />

    <TextView
       android:id="@+id/contactNameTextView"
       ... />

    <TextView
       android:id="@+id/statusView"
       ... />

    <TextView
       android:id="@+id/inviteContactTextView"
    ... />  


After you finish your layout file and save it into **res/layout** folder, you should call `setContactItemLayout(String customContactItemLayoutName)` to apply your contact item theme in the list.

Usage example:

    uiCustomizationManager.setContactItemLayout("custom_contact_item_layout");

You can overlap views as we did in our default design (in order to have the status on top of the contact image).

##### Contact details layout #####

For contact detail customization you have to create your layout with three mandatory elements.

- ImageView or TextView for displaying delivery status with id: `statusView`,
- TextView for displaying the contact's phone number with id: `phoneNumberTextView`,
- TextView for sending invitation with id: `invitePhoneNumberTextView`.

You can reorder and customize elements as you see fit.

    <TextView
        android:id="@+id/phoneNumberTextView"
        ... />

    <ImageView
        android:id="@+id/statusView"
        ... />

    <TextView
        android:id="@+id/invitePhoneNumberTextView"
        ... />

Or:

    <TextView
        android:id="@+id/statusView"
        ... />

    <TextView
        android:id="@+id/phoneNumberTextView"
        ... />

    <TextView
        android:id="@+id/invitePhoneNumberTextView"
        ... />

After finishing your layout file and saving it into **res/layout** folder, you should call `setContactDetailsLayout(String customContactDetailsLayoutName)` to apply your theme for contact details.

Usage example:

    uiCustomizationManager.setContactDetailsLayout("custom_contact_details_layout");

##### Edit message dialog layout #####

For Edit message dialog customization you have to put five mandatory elements on your layout and you can customize them as you want.

- EditText for input of the custom
-  with id: `customMessageEditText`,
- TextView for showing how many characters left to the end of message with id: `messageLengthTextView`,
- TextView for reseting the custom message to default value with id: `resetToDefaultMessageTextView`,
- TextView for saving the custom message with id: `saveCustomMessageTextView`,
- TextView for canceling the dialog with id: `cancelCustomMessageTextView`.

You can reorder elements on dialog as you want.

Usage example:

     <EditText
         android:id="@+id/customMessageEditText"
         ... />

     <TextView
         android:id="@+id/messageLengthTextView"
         ... />

     <TextView
         android:id="@+id/resetToDefaultMessageTextView"
         ... />

     <TextView
         android:id="@+id/saveCustomMessageTextView"
         ... />

     <TextView
         android:id="@+id/cancelCustomMessageTextView"
         ... />

After finishing your layout file and saving it into **res/layout** folder, you should call `setEditMessageDialogLayout(String customEditMessageDialogLayoutName)` to apply your theme for the dialog.

Usage example:

    uiCustomizationManager.setEditMessageDialogLayout("custom_edit_message_dialog_layout");

##### Connect to internet dialog layout #####

For the customization of this dialog you have to put three mandatory elements in your layout and you can customize them as you want.

- TextView for enabling 3G network with id: `enable3GTextView`,
- TextView for enabling WiFi Internet with id: `enableWifiTextView`,
- TextView for the dismissal of the dialog with id: `cancelConnectDialogTextView`.

Your dialog must contain these elements and you can store it however you want and can customize them as you wish:

     <TextView
        android:id="@+id/enableWifiTextView"
        ... />

     <TextView
        android:id="@+id/enable3GTextView"
        ... />

     <TextView
        android:id="@+id/cancelConnectDialogTextView"
        ... />

After finishing your layout file and saving it into **res/layout** folder, you should call `setConnectToInternetDialogLayout(String customConnectToInternetDialogName)` to apply your dialog theme.

Usage example:

    uiCustomizationManager.setConnectToInternetDialogLayout("custom_connect_to_internet_dialog_layout");

#### Customization of images ####

You can use your own images or icons as a part of our user interface.

You can set different images for the invitation status (invitation delivered, invitation not delivered and invitation pending), default user image and edit message image. You have to put your images in your **res/drawable** folder and call the appropriate method from the *UICustomizationManager* class.

- `setDeliveredImage(String customDeliveredImageName)` - sets the name of the ‘delivered’ image,
- `setPendingImage(String customPendingImageName)` - sets the name of ‘pending’ image,
- `setNotDeliveredImage(String customNotDeliveredImageName)` - sets the name of the ‘not delivered’ image,
- `setEditMessageImage(String customEditMessageImageName)` - sets the name of ‘custom message’ image,
- `setUserImage(String customUserImageName)` - sets the name of ‘user’ image.

Usage example:

    uiCustomizationManager.setDeliveredImage("custom_delivered_image");
    uiCustomizationManager.setPendingImage("custom_pending_image");
    uiCustomizationManager.setNotDeliveredImage("custom_not_delivered_image");
    uiCustomizationManager.setEditMessageImage("custom_edit_message_image");
    uiCustomizationManager.setUserImage("custom_default_user_image");

If you decide to use icons (as letters from given font), you can also add your own icons. All you have to do is add your icon codes to strings.xml and call the appropriate method from the *UICustomizationManager* class and pass on the icon id.

- `setDeliveredIcon(int deliveredIconFromR)` - sets the ‘delivered’ icon id from R.java,
- `setPendingIcon(int deliveredIconFromR)` - sets the ‘pending icon’ id from R.java,
- `setNotDeliveredIcon(int deliveredIconFromR)` - sets the ‘not delivered’ icon id from R.java,
- `setEditMessageIcon(int deliveredIconFromR)` - sets ‘edit message’ icon id from R.java.

If you use icons, you will probably want to set color of them. Methods for setting the icon color are:

- `setPendingIconColor(String pendingIconColor)` - sets value of pending icon color,
- `setDeliveredIconColor(String deliveredIconColor)` - sets value of delivered icon color,
- `setNotDeliveredIconColor(String notDeliveredIconColor)` - sets value of not delivered icon color,
- `setEditMessageIconColor(String editMessageIconColor)` -sets value of edit message icon color.

Usage example:

    uiCustomizationManager.setDeliveredIcon(R.string.myDeliveredIcon);
    uiCustomizationManager.setDeliveredIconColor("#008500");
    uiCustomizationManager.setPendingIcon(R.string.myPendingIcon);
    uiCustomizationManager.setDeliveredIconColor("#FF7700");
    uiCustomizationManager.setNotDeliveredIcon(R.string.myNotDeliveredIcon);
    uiCustomizationManager.setDeliveredIconColor("#A60000");
    uiCustomizationManager.setEditMessageIcon(R.string.myEditMessageIcon);
    uiCustomizationManager.setEditMessageIconColor("#000000");

#### Customization of text color ####

You can set your color for invite text on the list view. The only thing you have to do is call the `setInviteColor(String color)` method from the *UICustomization* class.

Usage example:

    uiCustomizationManager.setInviteColor("#FFFFFF");

#### Message layout displaying ####

Set whether you to display the message text on the user interface or not by calling: `displayEditMessageLayout(boolean displayEditMessageLayout)`.

Usage example:

    uiCustomizationManager.setDisplayOfEditMessageLayout(true);

## Models ##

### ClientMobileApplication ###

The *ClientMobileApplication* class contains information about the application and can be instantiated by using the default constructor or the constructor with parameters: **application key** (type: String) and **secret key** (type: String).

Usage example:

    ClientMobileApplication mobileApplication = new ClientMobileApplication();

Or:

    ClientMobileApplication mobileApplication = new ClientMobileApplication("application_key_abc123", "secret_key_abc123");

You can access these fields or change their values by using the following methods:

- `getApplicationKey()` - returns the value of the application key,
- `setApplicationKey(String applicationKey)` - sets the application key value,
- `getSecretKey()` - returns the value of secret key,
- `setSecretKey(String secretKey)` - sets the secret key value.

### InfobipAccountCredentials ###

The *InfobipAccountCredentials* class contains information about your Infobip account: your username and your password. This class can be instantiated by using the default constructor or the constructor with parameters: **Infobip username** (type: String) and **Infobip password** (type: String).

Usage example:

    InfobipAccountCredentials ibCredentials = new InfobipAccountCredentials();

Or:

    InfobipAccountCredentials ibCredentials = new InfobipAccountCredentials("infobip_username", "infobip_password");

You can access these fields or change their values by using the following methods:

- `getInfobipUsername()` - returns the Infobip username,
- `setInfobipUsername(String infobipUsername)` - sets the Infobip username value,
- `getInfobipPassword()` - returns the Infobip password,
- `setInfobipPassword(String infobipPassword)` - sets the Infobip password.

### Owners ###

Framework Integration Team @ Infobip Belgrade, Serbia

Android is a trademark of Google Inc.

© 2014, Infobip Ltd.
