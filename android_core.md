---
layout: page
title: Social Invites SDK for Android (tm)
group: "core"
weight: 3
---

# Social Invites library for Android™

This project is an Android library which can be merged with your Android project and enable you to use Infobip Social Invites. 

You can download **aar** file <a href="social-invites-core-aar.zip">here</a>.

## Requirements

- The lowest required Android SDK version is set to 8.

## Step by step integration

1. Copy the library file into the libs folder,
2. Add dependency to build.gradle file (`compile (name:'social-invites-core', ext:'aar')`),
3. Add user permissions and receiver declaration (as one of the application's components) to application project Manifest.xml file.
4. Rebuild your project.

User permissions:

	<uses-permission android:name="android.permission.READ_CONTACTS" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.INTERACT_ACCROSS_USERS_FULL" />
    
Receiver declaration:

	<receiver android:name="com.infobip.socialinvites.android.lib.connectivity.ConnectivityReceiver" >
    	<intent-filter>
        	<action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
        </intent-filter>
    </receiver>

### Application registration

In order to use the Social Invites library you have to create an Infobip account and register your mobile application. After registering the application you will obtain the **application key** and the **application secret key**.
The next step is creating default message for your application after which you will obtain the **default message ID**.

### Initialization

Initialization is required for using the main library functionality - sending social invitations.

The first thing you have to do is to create the *SocialInvitesCoreManager* object with **context** (type: android.content.Context) as only and mandatory parameter.

	SocialInvitesCoreManager socialInvitesManager = new SocialInvitesCoreManager(context);

You have to create the *ClientMobileApplication* object with the **application key** (type: String) and the **application secret key** (type: String):

    ClientMobileApplication application = new ClientMobileApplication("application_key_abc123", "application_secret_abc123");
    
If message has client placeholders, you have to create list of String objects which represents what value should be assigned to each placeholder respectively. It is possible to use predefined values from *MessagePlaceholders* enum class:

- **RECEIVER_NAME**: Assigns name of contact, to which user sends social invitation, fetched from user's phonebook. 
- **SENDER_NAME**: Assigns defined sender id for application.
- **CUSTOM_TEXT**: With this placeholder defined in a list you give possibilty to end users to write their own part of message. 
- **END\_USER_MSISDN**: Assigns end user phone number to placeholder. Note that you have to provide end users phone number after initialization of library. Provide phone number by calling of method `setEndUserMSISDNPlaceholder(endUserMSISDNPlaceholderValue);`
- **END\_USER_USERNAME**: Assigns end user username to placeholder.  You have to provide end user username by calling method `setEndUserUsernamePlaceholder(endUserUsernamePlaceholderValue);` after initialization of library.

Each of these placeholders is optional in use. Also it is possible to add custom placeholder but value for such placeholders should be provided from users of this library. Values should be known before sending invitation (see *Sending invitations* section of this file).  

	List<String> messagePlaceholders = new ArrayList<String>();
    messagePlaceholders.add(MessagePlaceholders.RECEIVER_NAME.toString());
    messagePlaceholders.add(MessagePlaceholders.SENDER_NAME.toString());
    messagePlaceholders.add("Hardcoded text");
    messagePlaceholders.add(MessagePlaceholders.CUSTOM_TEXT.toString());
    messagePlaceholders.add(MessagePlaceholders.END_USER_MSISDN.toString());
    messagePlaceholders.add(MessagePlaceholders.END_USER_USERNAME.toString());
    messagePlaceholders.add("customPlaceholder");

Then you should call the method `initializeLibrary` from *SocialInvitesCoreManager* class with two or three parameters: `initializeLibrary(application, messageId)` or `initializeLibrary(application, messageId, messagePlaceholders)`. These methods throw *IllegalArgumentException* if any of the parameters is null or an empty string.

    socialInvitesManager.initializeLibrary(application, "default_message_id_abc123");

or

	socialInvitesManager.initializeLibrary(application, "default_message_id_abc123", messagePlaceholders);

At any moment you can check if the library has been initialized by calling the `isLibraryInitialized()` method. If it is not, the method will initialize it.

Usage example:
	
	// Example 1
	SocialInvitesCoreManager socialInvitesManager = new SocialInvitesCoreManager(activity.getBaseContext());
    ClientMobileApplication application = new ClientMobileApplication("application_key_abc123", "application_secret_abc123");
    if (!socialInvitesManager.isLibraryInitialized()) {
        socialInvitesManager.initializeLibrary(application, "default_message_id_abc123");    
    }
    
    // Example 2
    List<String> messagePlaceholders = new ArrayList<String>();
    messagePlaceholders.add(MessagePlaceholders.RECEIVER_NAME.toString()); //optional
    messagePlaceholders.add(MessagePlaceholders.SENDER_NAME.toString());
    messagePlaceholders.add(MessagePlaceholders.CUSTOM_TEXT.toString());
    messagePlaceholders.add(MessagePlaceholders.END_USER_MSISDN.toString());
    messagePlaceholders.add(MessagePlaceholders.END_USER_USERNAME.toString());
    
    SocialInvitesCoreManager socialInvitesManager = new SocialInvitesCoreManager(activity.getBaseContext());
    ClientMobileApplication application = new ClientMobileApplication("application_key_abc123", "application_secret_abc123");
    if (!socialInvitesManager.isLibraryInitialized()) {
        socialInvitesManager.initializeLibrary(application, "default_message_id_abc123", messagePlaceholders);    
    }
    
    socialInvitesManager.setEndUserMSISDNPlaceholder("endUserMSISDNPlaceholder");
  	socialInvitesManager.setEndUserUsernamePlaceholder("endUserUsernamePlaceholder");

#### Sender ID

In order to send the invitation you must set the Sender ID.

The Sender ID could either be an MSISDN (e.g. user's MSISDN - phone number) or an alphanumeric ID. If you decide to use the alphanumeric ID you have to take care of its length – the max length of an alphanumeric ID is 11 characters. The method which is used for setting the Sender ID is `setSenderId(String senderId)`. If senderId is null or an empty string, *IllegalArgumentException* will be given.

Usage example:

    socialInvitesManager.setSenderId("sender_id");

If you want to use user's MSISDN as the Sender ID, you can use *SocialInvitesCoreManager* method `fetchMsisdn()` which tries to obtain the MSISDN from the user's mobile phone. Be aware that this method **will not return the MSISDN in every case**. It returns either the MSISDN or null if fetching the MSISDN is not possible on that device. By using this method, you will have to handle *CannotFetchUserMSISDNException* if the user's MSISDN is not found on phone. Recommendation for you is to try to fetch the user's MSISDN and to have some mechanism for obtaining the user's phone number. You can use Infobip’s 2-Factor Authentication (2FA).

Usage example:

    String senderId = "";
    try {
        senderId = socialInvitesManager.fetchMSISDN();
    } catch (CannotFetchUserMSISDNException e){
        senderId = "sender_id";
    }
    
    socialInvitesManager.setSenderId(senderId);
    
Once you set the Sender ID or you fetch the MSISDN, you can use the method `getSenderId()` to find out the string representation of the Sender ID that has been saved for that specific user.

Usage example:

    String senderId = socialInvitesManager.getSenderId();

#### Invitation message

As mentioned above, after registering the application, you have to add default message for your application. This default message is the invitation message for your application. Every time user sends the invitation, the message text will represent the text of the default message with the hyperlink included. You can get your default message at any moment as the *ClientMobileApplicationMessage* object using the `getDefaultMessage()` method from the *SocialInvitesCoreManager*. 

Usage example:
    
    ClientMobileApplicationMessage defaultMessage = socialInvitesManager.getDefaultMessage();

This method throws *ClassNotFoundException*, *IOException* and *UserNotConnectedToInternetException* if the user is not connected to Internet (WiFi and 3G are turned off) and he tries to send the invitation.


#### User's contacts

To fetch the user's contacts from his device by using our solution, you must implement the *ContactsListener* interface from library.

Usage example:

    public class ContactsActivity extends Activity implements ContactsListener {
        ....
    }

Methods that have to be implemented are: `onContactsRetrieved(Context context, List<Contact> contacts)` and `onError(Context context, int errorCode)`. 

Usage example: 

    @Override
    public void onContactsRetreived(Context context, List<Contact> contacts) {
        ...
    }
    
    @Override
    public void onError(Context context, int errorCode) {
        ...
    }
    
The `onContactsRetrieved(Context context, List<Contact> contacts)` method receives list of contacts as a parameter and it is invoked when the `getContacts(Context context, ContactsListener contactsListener)` method from *SocialInvitesCoreManager* finishes getting contacts from the user's phone. If the contact has a picture stored on the user's device, value of photoUri field will be URI of that photo. If the contact doesn't have a  picture, value of photoUri will be null.

Usage example:

    protected void onCreate(Bundle savedInstanceState) {
        socialInvitesManager.getContacts(this.getContext(), this);
    }

If you need to get previously invited contacts, you can get them by calling the `getInvitedContacts()` method on the *SocialInvitesCoreManager*. This method returns all contacts invited from the user's phone. 

Usage example:

    socialInvitesManager.getInvitedContacts();

#### Sending invitations

If one contact has multiple phone numbers, the invitation can be sent either to one phone number or to all numbers. If you want to do this, use the following methods from the *SocialInvitesCoreManager* class:

- `sendInvitation(Contact contact, List<String> clientData)` - sends invitations to all of the contact’s MSISDNs 
- `sendInvitation(Contact contact, InvitationListener listener, List<String> clientData)` - sends invitations to all of the contact’s MSISDNs and registers an instance of class implementing *InvitationListener*
- `sendInvitation(InvitationInfo phoneNumber, List<String> clientData)` - sends the invitation to a given phone number,
- `sendInvitation(InvitationInfo phoneNumber, InvitationListener listener, List<String> clientData)` - sends the invitation to the phone number and a status update upon delivery - calls `onDeliveryStatusUpdated(String messageId, final String address, final int newStatus)` method on *listener*.

If message text contains client placeholders created by client (placeholders which are not defined in *MessagePlaceholder* class), values for that placeholders should be in **clientData** list of Strings. If message doesn't contain such placeholders **null** value should be forwarded in *sendInvitation* method instead of *clientData*. 

If you use any of the methods given, you will have to handle *IllegalArgumentException* if the *senderId* is null or an empty string. You will also have to handle *UserNotConnectedToInternetException* if the user is not connected to the Internet; which means that invitations can't be sent.

You can control the number of invitations that one user can send to a phone number. You can enable invitation resending to a single phone number by calling the `enableResendingInvitation()` method or simply disable it by calling `disableResendingInvitation()`. This is disabled by default.

Usage example:

    socialInvitesManager.enableInvitationResending();
    socialInvitesManager.disableInvitationResending();

##### InvitationListener

The invitation listener is used to keep track of sent invitations. It has one method that you have to override. That method is `onDeliveryStatusUpdated(String messageId, final String address, final int newStatus)`. After each update of the delivery status for an invitation sent, this method will be called with the appropriate parameters.

Usage example:

    InvitationListener listener = new InvitationListener() {
        @Override
        public void onDeliveryStatusUpdated(String messageId, final String address, final int newStatus) {
            ...
        }
    }

#### Delivery statuses

After you send the invitation, you may want to show to the user the delivery status of a sent invitation. You will be able to enable getting delivery statuses by calling the method `enableGettingDeliveryStatus()` or disable it by calling the `disableGettingDeliveryStatus()` from *SocialInvitesCoreManager* class. If getting delivery status is disabled, status of invitation will be `SENT` and this status will be stored in the database. Getting delivery status will be disabled by default if you initialize library without Infobip account credentials.

	socialInvitesManager.enableGettingDeliveryStatus();
	socialInvitesManager.disableGettingDeliveryStatus();

There are two ways for getting that delivery info. If you want to get the current delivery status you should use the `getDeliveryInfo(String bulkID)`method. The result of this method execution is the *DeliveryInfoResponse* object.

Usage example:

    String bulkId = socialInvitesManager.getBulkId(contactId, "MSISDN");
    DeliveryInfoResponse deliveryInfo = socialInvitesManager.getDeliveryInfo(bulkId);

Another way for obtaining the delivery info is by calling the `tryGettingDeliveryInfo(String bulkId, InvitationListener listener)` method. This method will be getting delivery statuses until the delivery status is “delivered” or “not delivered” or until the number of attempts reaches a predefined value. Every time the delivery status changes, the *InvitationListener* method `onDeliveryStatusChanged()` is called.

Usage example:

    String bulkId = socialInvitesManager.getBulkId(contactId, "MSISDN");
    socialInvitesManager.tryGettingDeliveryInfo("bulk_id", invitationListener);

If you want to check (manually force check) the delivery status for the pending MSISDN at any moment, you just have to call `checkDeliveryStatusForPendingMSISDNs()` method.

Usage example:

    socialInvitesManager.checkDeliveryStatusForPendingMSISDNs();

## Models

### ClientMobileApplication

*ClientMobileApplication* class contains information about the application and can be instantiated by using the default constructor or the constructor with parameters: **application key** (type: String) and **secret key** (type: String).

Usage example:

    ClientMobileApplication mobileApplication = new ClientMobileApplication();

Or:

    ClientMobileApplication mobileApplication = new ClientMobileApplication("application_key_abc123", "secret_key_abc123");

You can access these fields or change their values by using the following methods:

- `getApplicationKey()` - returns the application key value,
- `setApplicationKey(String applicationKey)` - sets the application key value,
- `getSecretKey()` - returns the secret key value,
- `setSecretKey(String secretKey)` - sets the secret key value.


### ClientMobileApplicationMessage

The *ClientMobileApplicationMessage* class contains information about the application message and can be instantiated by using the default constructor or the constructor with parameters: **text** (type: String), **placeholder** (type: String), **clientPlaceholder** (type: String). The placeholder's purpose is to define a text that will be recognized as a substring of the message's text and replacing it with a shortened link to the application's download page.

Usage example:
 
    ClientMobileApplicationMessage message = new ClientMobileApplicationMessage();

Or:

    ClientMobileApplicationMessage message = new ClientMobileApplicationMessage("message_text", "placeholder_for_url");
    
Or:

    ClientMobileApplicationMessage message = new ClientMobileApplicationMessage("client_mobile_applicationKey", "message_text", "placeholder", "client_placeholder", "key");
    
Example of placeholder & message:

- ***placeholder***: "*!@#$*", 
- ***client_placeholder***: "*cp*", 
- ***message***: "*Give our new application a try (!@#$). Have a nice day! cp*".

You can access the class fields or change their values by using the following methods:

- `getText()` - returns the message text value,
- `setText(String text)` - sets the text value,
- `getPlaceholder()` - returns the placeholder value,
- `setPlaceholder(String placeholder)` - sets the placeholder value,
- `getClientPlaceholder()` - returns the client placeholder value,
- `setClientPlaceholder(String clientPlaceholder)` - sets the client placeholder value,
- `getKey()` - returns the message key value,
- `setKey(String key)` - sets the message key,
- `getApplicationKey()` - returns the application key value,
- `setApplicationKey(String applicationKey)` - sets the application key value.

### Contact

The *Contact* class contains information about the contact. This class can be instantiated by using following constructors:

- `Contact()`,
- `Contact(int id)` - this constructor has the contact ID as a parameter,
- `Contact(int id, String name)` - this constructor has the contact ID and the contact name as parameters
- `Contact(int id, String name, Uri photoUri, List<InvitationInfo> phoneNumbers)` - this constructor has the contact ID, the contact name and the contact phone numbers as parameters

You can access class fields or change their values by using the following methods:

- `getId()` - returns the contact ID value,
- `setId(int id)` - sets the contact ID value,
- `getName()` - returns the contact name value,
- `setName(String name)` - sets the contact name value,
- `getPhoneNumbers()` - returns the list of the invitation info,
- `setPhoneNumbers(List<InvitationInfo> invitationInfos)` - sets the phone numbers value.

### InvitationInfo

The *InvitationInfo* class contains information about the invitation and can be instantiated by using the default constructor, `InvitationInfo(int contactId, String msisdn)` constructor or `InvitationInfo(int contactId, String msisdn, int deliveryStatus, long lastStatusChangeTime)`.

You can access these fields or change their values by using the following methods:

- `getContactId()` - returns contact ID value,
- `setContactId(int contactId)` - sets the contact ID value,
- `getMsisdn()` - returns the MSISDN value,
- `setMsisdn(String msisdn)` - sets the MSISDN value,
- `getInvitationDeliveryStatus()` - returns the invitation delivery status value,
- `setInvitationDeliveryStatus(int deliveryStatus)` - sets the delivery status value,
- `getLastStatusChangeTime()` - returns the last status change value (in milliseconds),
- `setLastStatusChangeTime(long lastStatusChangeTime)` - sets the last status change time value (in milliseconds),
- `getBulkId()` - returns the message bulk ID value,
- `setBulkId(String key)` - sets the message bulk ID value.

### DeliveryInfoResponse

The *DeliveryInfoResponse* class contains information about delivery info response and can be instantiated by using the default constructor.

You can access the class field or change its value by using methods:

- `getDeliveryInfoList()` - returns the value of the delivery info list (*DeliveryInfoList* object),
- `setDeliveryInfoList(DeliveryInfoList deliveryInfoList)` - sets the delivery info list value.

### DeliveryInfoList

The *DeliveryInfoList* class contains information about “delivery infos” and can be instantiated by using the default constructor. 

Methods for accessing the class field or changing its value are:

- `getDeliveryInfosList()` - returns the value of deliveryInfos field (array of *DeliveryInfo* objects),
- `setDeliveryInfosList()` - sets the value of deliveryInfos field.

### DeliveryInfo

The *DeliveryInfo* class contains information about delivery info and can be instantiated by using the default constructor

Methods for accessing the class fields or changing their values are:

- `getAddress()` - returns the address (msisdn) value,
- `setAddress(String address)` - sets the address (msisdn) value,
- `getMessageId()` - returns the invitation message ID value,
- `setMessageId(String messageId)` - sets the invitation message ID value,
- `getDeliveryStatus()` - returns the delivery status value,
- `setDeliveryStatus(String deliveryStatus)` - sets the delivery status value,
- `getClientCorrelator()` - returns the client correlator (same as bulk ID value) value,
- `setClientCorrelator(String clientCorrelator)` - sets the client correlator value,
- `getPrice()` - returns the price of sent invitation value,
- `setPrice(double price)` - sets the sent invitation price value.

This class also contains the method that can be used for mapping the delivery status to delivery status code - `getDeliveryStatusCode()` method. 

Delivery status mapping:

- `INVITATION_STATUS_UNKNOWN` - delivery status is null or an empty string,
- `INVITATION_PENDING` - if the delivery status is "MessageWaiting" (the message is still queued for delivery. This is a temporary state, pending transition to one of the preceding states) or "DeliveredToNetwork" (successful delivery to the network enabler responsible for routing the message),
- `INVITATION_NOT_DELIVERED` - if the delivery status is "DeliveryImpossible" (unsuccessful delivery - the message could not be delivered before it expired),
- `INVITATION_DELIVERED` - if the delivery status is "DeliveredToTerminal" (indicating successful delivery to the terminal) or "DeliveryUncertain" (delivery status unknown, e.g. because it was handed off to another network).

### Owners

Framework Integration Team @ Infobip Belgrade, Serbia

Android is a trademark of Google Inc.

© 2014, Infobip Ltd.