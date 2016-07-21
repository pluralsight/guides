![Horizon, RethinkDB and Onsen UI 2.0](https://onsen.io/blog/content/images/2016/Jul/onsen_horizon.png)

A lot of modern apps, such as Twitter or Facebook work in real-time: They update themselves as soon as new information is available without having the user refresh manually. With a modern real-time database like [RethinkDB](http://rethinkdb.com) and the JavaScript Framework [Horizon](http://horizon.io/), creating a real-time application is as easy as writing any other JavaScript Application. 

In this tutorial we will build a simple chat app with [MobX](https://github.com/mobxjs/mobx) and the [React Components for Onsen UI](https://onsen.io/v2/).

<!-- more -->

The entire source code of the app is available on [GitHub](https://github.com/philolo1/Horizon-MobX-OnsenUI). Our end result will look like this:

<div style="display: flex; margin: 40px 0;" >
  <div style="flex-grow: 1;"> </div>
  <div style="background-image: url('https://onsen.io/blog/content/images/2016/Feb/iphone6.png'); width: 401px; height: 806px; ">
    <iframe scrolling="yes" style="margin-top: 92px; margin-left: 26px;" src="https://onsenhorizon.herokuapp.com" width="349" height="617" scrolling="no" class="lazy-hidden"></iframe>
  </div>
  <div style="flex-grow: 1;"> </div>
</div>

We will first take a look at RethinkDB and Horizon.js. Then we'll build our app.

## What is RethinkDB and Horizon.js?

RethinkDB is a real-time database. The difference between RethinkDB and a traditional database is the ability to listen to database changes. Continuous listening makes real-time updates very easy. 

Horizon.js is a JavaScript framework that makes it easier to interact with RethinkDB. You can install Horizon by calling the following npm commands:

```
$ npm install -g horizon
```

With the following command, one can start a simple Horizon server in development mode on port 5000:

```
$ hz serve --dev --bind all -p 5000
```

The parameter `--bind all` makes it accessible throughout the entire network. In development mode there is no user authentication and no security rule. For this tutorial we are not going to deal with authentication, but if you are interested to learn about it, please check the [documentation](http://horizon.io/docs/auth/).

To start Horizon in our application we will need to just use the `connect()` function

```
import Horizon from '@horizon/client';
const horizon = Horizon();


horizon.onReady(() => {
  console.log('horizon is ready');

});

console.log('connect horizon');
horizon.connect();
```

For the chat app, we will create two tables, one for the rooms -- this table just contains the room names with their IDs -- and another table for the the `roomID`, the message itself, and the username.

To create a room and messages we can write some simple functions:

```js
createRoom: (roomName) => {
  horizon('chatRooms').store({ name: roomName })
}

createMessage: (authorName, roomID, message) => {
  horizon('messages').store({
    author: authorName,
    date: new Date(),
    message: message,
    roomID: roomID
  });
}
```

The function `createRoom` creates a simple room with the provided room name. `createMessage` creates a message for the room. 

The interesting part is how we can listen to changes of the database. We want to get all the messages from one room ordered by their date:

```
horizon('messages').findAll({roomID: roomID}).order('date').watch().subscribe((data) => {
  // update messages here
});
```

The function in `subscribe()` will be called __every time__ the data is updated. In React, we could just update the state and have updated data with almost no code. 

Now that we can create a chat room and create messages, what's left? Actually, this is almost all of the backend code we will need for our app! 

Now let's start building using MobX and the React Components for Onsen UI.

# Building the components in Onsen UI

The React Components for Onsen UI make it very easy to build simple pages with Navigation without too much code. Our application will contain two screens: The first one will be a simple login screen, where the user can enter the name of the chatroom and the user name. We will render everything in a component called `App` that will contain the children:

```
import React from 'react';
import {render} from 'react-dom';
import AppState from './AppState';
import App from './App';
import Horizon from '@horizon/client';

const horizon = Horizon({host: 'localhost:5000'});
const appState = new AppState(horizon);

horizon.onReady(() => {
  console.log('horizon is ready');

  render(
    <App appState={appState} />,
    document.getElementById('root')
  );
});

console.log('connect horizon');
horizon.connect();
```

Before we start looking at the views, let's look at the application state first. The application state is managed via [MobX](https://github.com/mobxjs/mobx). MobX is a JavaScript library that uses observables to do certain actions automatically once an observed variable changes. In the case of React, MobX calls `setState()` if a variable changes which the state depends on. If you want to learn more about MobX, I highly recommend a look at our [recent tutorial](https://onsen.io/blog/mobx-tutorial-react-stopwatch/) that uses MobX to create a simple stopwatch.

The following code defines the application state. The app state will contain the current username, chatroom, the database connection to Horizon and some additional page-related information. In MobX we mark variables with the decorator `@observable` to indicate that MobX should observe it. If they change, the associated views will automatically call a rerender. Functions marked with `@action` are functions that change observable variables and `@computed` functions getters that depend on observables that will be only updated once an associated variable is changed.

```
import {computed, observable, action} from 'mobx';

class ChatRoom {
  @observable name;
  id;

  constructor(data) {
    this.name = data.name;
    this.id = data.id;
  }
}

export class ChatRoomPageState {
  @observable text = '';

  @action setText(text) {
    this.text = text;
  }

  @action resetText() {
    this.text = '';
  }
}

class AppState {
  horizon;
  @observable userName;
  @observable roomName = 'onsenui';
  @observable chatRooms = [];
  @observable loading = false;
  @observable messages = [];
  @observable newMessage = false;

  constructor(horizon) {
    this.horizon = horizon;
    this.chatRooms = [];
  }

  @computed get messageList() {
    var list = this.messages.map((el) => el);
    for (var i = 1; i < list.length; i++) {
      if (list[i - 1].author === list[i].author) {
        list[i].showAuthor = false;
      } else {
        list[i].showAuthor = true;
      }
    }

    if (list.length > 0) {
      list[0].showAuthor = true;
    }

    return list;
  }

  @computed get lastAuthor() {
    if (this.messages.length === 0) {
      return '';
    }

    return this.messages[this.messages.length - 1].author;
  }

  @action setMessages(data) {
    this.messages = data;
  }

  @action hideMessageNotification() {
    this.newMessage = false;
  }

  @action showMessageNotification() {
    if (this.newMessage) return;
    this.newMessage = true;
    setTimeout(() => this.newMessage = false, 2000);
  }

  @action setChatRoom(data) {
    this.chatRooms = data.map((el) => new ChatRoom(el));
  }
}

export default AppState;
```

# The Login Screen

In the following explanation, we will look at the structure of our application and the basic interactions of the components. The curious reader can look up more details at [GitHub](https://github.com/philolo1/Horizon-MobX-OnsenUI).

For the navigation, we will use the Onsen UI `Navigator`. We provide it an initial route which will contain a component we will render to. In our case, this component will be `LoginPage`.

```
import React, {Component} from 'react';
import ons from 'onsenui';
import {Modal, Page, Col, Row, BottomToolbar, List, ListItem, Button, Navigator, Toolbar, Input} from 'react-onsenui';

// ... LoginPage definition omitted for simplicity
// it would render to something like <Page> .. </Page>

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {};
    this.renderPage = this.renderPage.bind(this);
  }
  renderPage(route, navigator) {
    const props = route.props || {};
    props.navigator = navigator;
    return React.createElement(route.component, route.props);
  }

  render() {
    return (
      <Navigator
        initialRoute={{component: LoginPage, props: {
          appState: this.props.appState
        }}}
        renderPage={this.renderPage}
      />
    );
  }
}

export default App;
```

The structure of the `Login` component will look something like this:

![Login Page](https://onsen.io/blog/content/images/2016/Jul/LoginPage.png)

The `LoginPage` consists of an image, two inputs and a button. The application state is passed down to the component and updating is very simple, as can be seen on the `UserInput`:

```
const UserInput = observer(({appState}) => {
  return (
    <div>
      <Input
        modifier='underbar'
        style={{width: '100%'}}
        onChange={(e) => appState.userName = e.target.value}
        value={appState.userName}
        placeholder='Name'
        float
      />
    </div>
  );
});
```

Once the value changes, we just update the variable, and MobX will automatically rerender the `UserInput` component and only this component, since the username is only displayed there.

After the username and room name are entered, we will either create a room if it does not already exist or load an existing room. Check this out:

```
joinRoom = () => {
    var {userName, roomName} = this.props.appState;

    if (!(userName != null && userName.length > 0)) {
      ons.notification.alert('Please fill in a userName');
      return;
    }

    if (!(roomName != null && roomName.length > 0)) {
      ons.notification.alert('Please fill in a roomName');
      return;
    }

    this.setState({loading: true});

    this.chatRooms.find({name: roomName}).fetch().defaultIfEmpty().subscribe(room => {


     // while loading the page, we will show a modal
      this.setState({loading: true});

      if (room == null) {
        console.log('room does not exist');
        this.chatRooms.store({
          name: roomName
        }).subscribe((el) => {

          this.props.navigator.pushPage({
            component: ChatRoomPage,
            props: {
              appState: this.props.appState,
              title: roomName,
              roomID: el.id,
              author: userName,
              pageState: new ChatRoomPageState()
            }
          });
        });
      } else {
        console.log('joining room ' + roomName);

        this.props.navigator.pushPage({
          component: ChatRoomPage,
          props: {
            appState: this.props.appState,
            title: roomName,
            roomID: room.id,
            author: userName,
            pageState: new ChatRoomPageState()
          }
        });
      }
    });
  }
```

# Chat Room Page

The second page will contain the main chat messages. On the left side your messages will be presented, while on the right side other people's messages will be shown. Again, let's look at a small overview of the messages:

![Login Page](https://onsen.io/blog/content/images/2016/Jul/ChatRoomPage.png)

The `ChatRoomPage` component consists of a toolbar at the top, an empty text box at the bottom, and the messages in the middle. We will use RethinkDB and MobX to update the view: Whenever a new message that's been written by the user is received, the view will scroll down. Whenever a message written by somebody else is received, a small popup message will be shown that the user can click to go to the bottom. 

The code of the main component looks like this:

```
export default class ChatRoomPage extends Component {

  componentDidUpdate() {
    if (this.props.appState.lastAuthor === this.props.author) {
      this.scrollBottom();
    }
  }
  
  /** scroll to the bottom method defined*/
  scrollBottom = () => {
    const page = document.getElementById('page2').querySelector('.page__content');
    page.scrollTop = page.scrollHeight;
  }

  constructor(props) {
    super(props);
    this.initial = true;
    this.state = {
      typedText: '',
      messages: []
    };
  }

  get pageState() {
    return this.props.pageState;
  }

  componentDidMount() {
    this.messages = this.props.appState.horizon('messages');

    this.messages.findAll({roomID: this.props.roomID})
        .watch().subscribe((data) => {
      if (data) {
        const newData = _.sortBy(data, (el) => el.date);
        this.props.appState.setMessages(newData);
        if (this.initial) {
          this.initial = false;
          this.scrollBottom();
        } else if (this.props.appState.lastAuthor !== this.props.author) {
          this.props.appState.showMessageNotification();
        }
      }
    });
  }

  sendText = () => {
    if (this.pageState.text.length === 0) {
      return;
    }
    this.messages.store({
      author: this.props.author,
      date: new Date(),
      message: this.pageState.text,
      roomID: this.props.roomID
    });

    this.pageState.resetText();
  }

  render() {
    return (
      <Page
        id='page2'
        renderBottomToolbar={() =>
          <MessageBar
            sendText={this.sendText}
            appState={this.props.appState}
            pageState={this.pageState}
          />
        }
        renderToolbar={() =>
          <Toolbar>
            <div className='left'><BackButton>Back</BackButton></div>
            <div className='center'> {this.props.title} </div>
          </Toolbar>
        }
      >
        {this.props.appState.messageList.map((data) => {
          return (
            <Message data={data} author={this.props.author} />
          );
        })}

        <div style={{height: 15}} />

        <NewMessage
          onClick={() => {
            this.scrollBottom();
            this.props.appState.hideMessageNotification();
          }}
          show={this.props.appState.newMessage}
        />
      </Page>
    );
  }
}

```

# Conclusion

In sum, Horizon combined with Onsen UI facilitates building a real-time chat application by limiting our backend involvement and expanding our control over real-time UI features. 

In case you need some additional reading, there are many resources and videos that you can use to learn more about RethinkDB. I highly recommend this [video](https://www.youtube.com/watch?v=3BPLsljZVIc) and the RethinkDB [website](http://www.rethinkdb.com/). 

If you have any questions or feedback feel free to ask in our [community](https://community.onsen.io/). We would also appreciate a 🌟  on our [GitHub repo](https://github.com/OnsenUI/OnsenUI) and a favorite on this guide!

Thank you for reading!