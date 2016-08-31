

## Introduction
First of all sorry for my bad english, because I'm from Italy.<br>
In this tutorial I'll show you how I built Nubchat, an Adroid app that let you communicate with other android smartphones, or dev boards like Arduino, RaspberryPi and all the devices that support PubNub API's.<br>
To use this app you'll need a subcribe key and publish key from  pubnub.com. It's available on Google play store [here]: https://play.google.com/store/apps/details?id=com.mohamedfadiga.nubchat
The source code is available on Github [here]:https://github.com/Momy93/NubChat

## Working logic
For the first usage, you have to chose a username and insert your pubnub keys.<br>
Of course these keys are personal, and should not share them with other people.<br>

<i>"So, what if i want to chat with my friend Mark?"</i>

This app is designed for communicating with your devices, and an Android smartphone is a device, but used by another person.<br>
So if you want to chat with <i>Mark</i> you'll have to provide him your keys.<br>
Otherwise if you want to use NubChat as a normal chat app for smartphones, hard code your keys in the app souce.<br>

The username you chose, it's  not your pubnub email or username, it's just a name you chose to receive  messages on the current device.<br>
Once you make this first setup, you can start using tha app.<br>
You can edit your username and PubNub keys when you want on Menu->Seetings.<br>



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/897bb1b9-3278-4fe9-91f1-12b920fdf176.jpg)



With the "+" Button you can add new channel. You'll be prompt to chose the channel name and set if it's a public or private. Public and private channel are not functions from Pubnub. This logic is created inside the app using pubnub API's.
When you add a new channel, its informations are store on a local database on your smartphone.<br>
To explain you well how the app works, let say you installed it and you choosed Bob as username.<br>
Let's add a private channel called RaspberryPi.<br> 




![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/af6f8f58-7ea1-43a5-a6bf-91d6d0f714fb.jpg)




As already said data about this new channel are stored in the local database. The databasse's channels table structure is something like this:<br>

<table >
  <tr>
    <th>name</th>
    <th>channel_type</th> 
    <th>channel_status</th>
    <th>last_rec_id</th>
    <th>last_add_id</th>
  </tr>
  <tr>
    <td>RaspberryPi</td>
    <td>0</td> 
    <td>0</td>
    <td>-1</td>
    <td>-1</td>
  </tr>
</table>

name: the name of the channel. <br>
channel_type: 0 for private channel and 1 for a public one. <br>
channel_status: not used in this first release of the app.<br>
last_rec_id: the id of the last received message (from our Raspeberry)<br>
last_add_id: the id of the last message added to this channel, either sent or received.<br>

last_rec_id and last_add_id are -1 because the channel is empty yet.
when we send a message the app use pubnub publish function.

    public void  sendMessage(final Message message)
    {
        pubNub.publish()
            .message(new SerializableMessage(message))
            .channel(message.getChannelName()).shouldStore(true).async(new PNCallback<PNPublishResult>(){
                @Override
                public void onResponse(PNPublishResult result, PNStatus status){
                    if(!status.isError()){
                        message.setStatus(Message.SENT);
                        db.saveMessage(message);
                        if(serviceCallbacks != null) serviceCallbacks.update(message);
                    }
                }
            });
    }


The parameter of .message() can be any Java Serializable Object. The method will serialize it for you before sending it. 
In this app I have a Message class, but I don't use it because it has some methods, and also has other data that don't need to be sent.
That's why I created this simple class called SerializableMessage, which holds only the informations that need to be sent.


    public class SerializableMessage{
        private int type;
        private String text, sender, url, latLon;

        SerializableMessage(Message message){
            this.sender = message.getSender();
            this.text = message.getText();
            this.url = message.getUrl();
            this.type = message.getType();
            this.latLon = message.getLatLon();
        }

For this first release, only three parameters are used: sender, type and text.
Sender always correspond to your username, so in this case it will be Bob.
Type for now is always 0 (pure text message).
in the message class you can see future types that will be available:

```public static final int TEXT = 0, IMAGE = 1, FILE =2 , GMAPS = 3;```

The process of sending messages is the same for both public and private channel. The difference comes when we want to receive messages.
If the channel is private, all I have to do is to subscribe to my own channel (in this case Bob). When RaspberyPi wants to send me a message, it will send it to Bob, and as a subscriber, I will receive it.
Even if I didn't added the private Channel, if someone publish a message direclty on my channe, I receive it. If it's a new channel, I store in the local database
For public channel, it is not efficient to subscribe to all the channels. Pubnub guidlines says:

> Currently, we recommend you use channel multiplexing for up to 10 (ten) channels. If your app requires a single client connection to subscribe to more than 10 channels, we recommend using channel groups. But you can subscribe to 50 channels from a single client.

So I'm using a channel group. 
Lets say we added two new public channels: Group1 and Group 2:

<table >
  <tr>
    <th>name</th>
    <th>channel_type</th> 
    <th>channel_status</th>
    <th>last_rec_id</th>
    <th>last_add_id</th>
  </tr>
    <tr>
    <td>RaspberryPi</td>
    <td>0</td> 
    <td>0</td>
    <td>7</td>
    <td>5</td>
  </tr>
  <tr>
    <td>Group1</td>
    <td>1</td> 
    <td>0</td>
    <td>-1</td>
    <td>-1</td>
  </tr>
    <tr>
    <td>Group2</td>
    <td>1</td> 
    <td>0</td>
    <td>-1</td>
    <td>-1</td>
  </tr>
</table>

Now what I do is to add Group1 and Group2 to a channel group, so I just need to subscribe to it, in order to receive messages from the these channels.
Adding channels to a channel group does't mean that they are logically inserted in the group. You can still subscribe and publish to this channel.
What the channel group does is something like subscribe to all the channel you added, and send you back the messages he receive from them.
In my case I add public channels to a channel group that has the same name of my channel (Bob).

    pubNub.subscribe()
                .channelGroups(Collections.singletonList(username))
                .channels(Collections.singletonList(username))
                .execute();
 
As you can see subscribing to channels and to channel group has two separated methods, and I'm giving them always "Bob" as a parameter.
When the subscribe method calls my calback, it tells me if someone publish directly on channel "Bob", or someone published on a channel added to the channel group called "Bob".
This is the callback called when there is a new message:

    @Override
    public void message(PubNub pubnub, PNMessageResult r){
        DatabaseHelper db = DatabaseHelper.getInstance(service);
        String sender = r.getMessage().get("sender").asText();
        if(sender.equals(username))return;
        JsonNode n = r.getMessage();
        String channelName = r.getActualChannel();
        if(channelName == null)channelName = n.get("sender").asText();
        service.newMessage(db.saveMessage(n, channelName, r.getTimetoken()));
    }
  
r.getSubscribedChannel() (not used in this code) will alway return "Bob" because I gave the same name to both my cannel and the channel group.
If a message has been sent to a public channel, r.getActualChannel() value could be for example "Group1" or "Group2".
So, when r.getActualChannel() give me a null pointer, I know that the message has been published directly on Bob's channel and not on a public channel.

When you device goes online after being offline, the app uses history funcition to get messages you missed while you were offline.  
This time I have to call the history function for all the public channels because channel group hasn't history capability.
Maybe there is a better way to do this anyway but I don't get by now.
For private channels instead, I just need to call history on my own channel (Bob).
 
 ## PubNub SDK 4
 
For this app I used PubNub Android (Java) SDK 4.0.8 . To install it in your android studio project, just add this to your dependencies:
 
compile group: 'com.pubnub', name: 'pubnub', version: '4.0.8'
 
Pubnub functions I used are methods of the class PubNub from com.pubnub.api.PubNub.

    PNConfiguration pnc = new PNConfiguration();
    pnc.setSubscribeKey(subKey);
    pnc.setPublishKey(pubKey);
    pnc.setUuid(username);
    pubNub = new PubNub(pnc);
                
I also set the UUID for future presence funcion implementation, but it was also usefull while debugging.
The subscribe method, is different from the previous SDK. In the previous one you had to provide a callback for all calls to functions that need a callback.
in this SDK you set a general callback, which will be call for functions like presence, subsscribe/unsubscribe ecc...  

    pubNub.addListener(new MySubscribeCallback(this, username));
       


MySubscribeCallback:

    package com.mohamedfadiga.nubchat.pubnubcallbacks;

    import com.fasterxml.jackson.databind.JsonNode;
    import com.mohamedfadiga.nubchat.services.BackgroundService;
    import com.mohamedfadiga.nubchat.utils.DatabaseHelper;
    import com.pubnub.api.PubNub;
    import com.pubnub.api.callbacks.SubscribeCallback;
    import com.pubnub.api.models.consumer.PNStatus;
    import com.pubnub.api.models.consumer.pubsub.PNMessageResult;
    import com.pubnub.api.models.consumer.pubsub.PNPresenceEventResult;

    public class MySubscribeCallback extends SubscribeCallback
    {
        private BackgroundService service;
        private  String username;
        public MySubscribeCallback(BackgroundService service,  String username){
            this.service = service;
            this.username = username;
        }

        @Override
        public void status(PubNub pubnub, PNStatus status){}

        @Override
        public void message(PubNub pubnub, PNMessageResult r){
            DatabaseHelper db = DatabaseHelper.getInstance(service);
            String sender = r.getMessage().get("sender").asText();
            if(sender.equals(username))return;
            JsonNode n = r.getMessage();
            String channelName = r.getActualChannel();
            if(channelName == null)channelName = n.get("sender").asText();
            service.newMessage(db.saveMessage(n, channelName, r.getTimetoken()));
        }
 
        @Override
        public void presence(PubNub pubnub, PNPresenceEventResult presence) {}
    }
 
 
For the history function instead, we need to provide a callback each time we call it.


    package com.mohamedfadiga.nubchat.pubnubcallbacks;

    import com.fasterxml.jackson.databind.JsonNode;
    import com.mohamedfadiga.nubchat.services.BackgroundService;
    import com.mohamedfadiga.nubchat.utils.DatabaseHelper;
    import com.pubnub.api.callbacks.PNCallback;
    import com.pubnub.api.endpoints.History;
    import com.pubnub.api.models.consumer.PNStatus;
    import com.pubnub.api.models.consumer.history.PNHistoryItemResult;
    import com.pubnub.api.models.consumer.history.PNHistoryResult;
    import java.util.List;

    public class HistoryCallback extends PNCallback<PNHistoryResult>
    {
        private History history;
        private BackgroundService service;
        private String username, channelName;

        public HistoryCallback(History history, BackgroundService service, String username, String channelName)
        {
            this.history = history;
            this.service = service;
            this.username = username;
           this.channelName = channelName;
       }

      @Override
      public void onResponse(PNHistoryResult result, PNStatus status)
      {
          if (!status.isError())
          {
                DatabaseHelper db = DatabaseHelper.getInstance(service);
                List<PNHistoryItemResult> messages = result.getMessages();
                for (PNHistoryItemResult iR:messages)
                {
                    try 
                    {
                        JsonNode n = iR.getEntry();
                        String sender = n.get("sender").asText();
                        if (sender.equals(username)) continue;
                        service.newMessage(db.saveMessage(n,channelName.equals(username)?sender:channelName, iR.getTimetoken()));
                    }
                    catch (Exception e){e.printStackTrace();}
                }
                if(messages.size() == 100)history.start(result.getEndTimetoken()+1).async(this);
            }
        }
    }  

When I call the history function, I provide a timetoken since when to start retrieving messages. if it's a public channel, it's the timetoken of the last message received in that channel +1.
For the others, the timetoken is the timetoken of the last message received on my own channel.


    ArrayList<Channel> channels = db.getChannels();
    ArrayList<Message> messages = db.getUnsentMessages();
    for(Message message: messages)sendMessage(message);
    for(Channel channel: channels)
    {
        if(channel.getType() == 1)
        {
            Message message = db.getMessage(channel.getLastReceivedId());
            History history = pubNub.history();
            history.channel(channel.getName()).start(message == null ? 0 : message.getTimetoken() + 1).reverse(true)
                           .includeTimetoken(true).async(new HistoryCallback(history,BackgroundService.this, username, channel.getName()));
        }
    }
    History history = pubNub.history()
        .channel(username)
        .start(db.getLastTimetoken()+1)
        .reverse(true)
        .includeTimetoken(true);
    history.async(new HistoryCallback(history, BackgroundService.this, username, username));
    pubNub.subscribe()
        .channelGroups(Collections.singletonList(username))
        .channels(Collections.singletonList(username))
        .execute();
             

## Usage

This is a tutotial on how I implemented Pubnub functions  in this app. There are many things you can do with it. 
The main purpose is that you have a devices that will subscribe for example to "Bob" channel, and do action or give you informations about themeselves when yu write to them.
I will soon make tutorial of the usage on the board side.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f40d498e-9a95-410e-b4ab-d82c2c38f4d5.jpg)


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/1d94a6ce-b693-4248-a9e3-4f413cbac804.jpg)


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/79d625a5-3520-4677-a8bb-f0dd0b23ed49.jpg)




