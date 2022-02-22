# dBridge NodeJS Client

This dBridge client library supports Node.js and Electron.

If you're looking for the dBridge server library for Node.js, use
[dBridge Server Library](https://github.com/s) instead.

For tutorials and more in-depth information about dBridge, visit
our [official docs](https://dbridge.com/docs/javascript_quick_start).

----

# Usage Document

- TOC
{:toc}

----

## What can I do with dBridge?

**dBridge|NodeJS** makes **distributed, realtime Web applications easy**: it provides the infrastructure for both **distributing live updates** to all connected clients (using the PubSub messaging pattern) and for **calling remote procedures** in different backend components (using RPC).

It is ideal for distributed, multi-client and server applications, such as multi-user database-drive business applications, real-time charts, sensor networks (IoT), instant messaging or MMOGs (massively multi-player online games).

The protocol that **dBridges|NodeJS** uses, socket.io, enables application architectures with application code **distributed freely across processes and devices** according to functional aspects. All dBridge clients are equal in that they can publish events and subscribe to them, can offer a procedure for remote calling and call remote procedures.

Since dBridge implementations exist for **multiple languages**, this extends beyond JavaScript clients: dBridge applications can be polyglot. Application components can be implemented in a language and run on a device which best fit the particular use case. Applications can span the range from embedded IoT sensors right to mobile clients or the browser - using the same protocol.

## Features

- works both in the browser and Node.js
- provides asynchronous RPC and PubSub messaging patterns
- uses WebSocket or HTTP long-poll as transport
- easy to use Promise-based API
- small size (244kB source, 111kB minified, 33kB compressed)
- Open-Source (MIT License)

## Installation

### Node.js

You can use any NPM-compatible package manager, including NPM itself and Yarn.

```bash
npm install dbridge-client --save
```

Then:

```javascript
const dbridge = require('dbridge-client');
```

## Initialization

```js
const dbridge = new dBridges();
```

## Global Configuration

### Required

The following is the list of required connection properties before connecting to dBridges network.

```javascript
dbridge.auth_url = 'URL';
dbridge.appkey = 'APP_KEY';
```

You need to replace `URL` and `APP_KEY` with the actual URL and Application Key.

| Properties | Description                                                  | Exceptions                                               |
| ---------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| `auth_url` | *(string)* Authentication url from  [dBridge dashboard](https://dashboard.dbridge.com/). | `source: DBLIB_CONNECT` <br />`code: INVALID_URL`        |
| `appkey`   | *(string)* Application Key from  [dBridge dashboard](https://dashboard.dbridge.com/). | `source: DBLIB_CONNECT` <br />`code: INVALID_AUTH_PARAM` |

### Optional

The following is the list of optional connection properties before connecting to dBridges network.

```javascript
dbridge.maxReconnectionRetries = 5;
dbridge.maxReconnectionDelay = 120000; 
dbridge.minReconnectionDelay = 1000 + Math.random() * 4000;
dbridge.reconnectionDelayGrowFactor = 1.3;
dbridge.minUptime = 5000 ; 
dbridge.connectionTimeout = 20000;
dbridge.autoReconnect = true; 
dbridge.cf.enable = false; 
```

| Properties                    | Default                       | Description                                                  |
| ----------------------------- | ----------------------------- | ------------------------------------------------------------ |
| `maxReconnectionDelay`        | `10`                          | *(integer)* The maximum delay between two reconnection attempts in seconds. |
| `minReconnectionDelay`        | `1000 + Math.random() * 4000` | *(integer)* The initial delay before reconnection in milliseconds (affected by the `reconnectionDelayGrowFactor` value). |
| `reconnectionDelayGrowFactor` | `1.3`                         | *(float)* The randomization factor used when reconnecting (so that the clients do not reconnect at the exact same time after a server crash). |
| `minUptime`                   | `5000`                        | *(integer)* Uptime before `connected` event is triggered, value in milliseconds. |
| `connectionTimeout`           | `20000`                       | *(integer)* Number of milliseconds the client will wait for a connection to be established. If it fails it will emit a `connection_error` event. |
| `maxReconnectionRetries`      | `100`                         | *(integer)* The number of reconnection attempts before giving up. |
| `autoReconnect`               | `true`                        | *(boolean*) If false, client will not attempt reconnecting.  |
| `cf.enable`                   | `false`                       | *(boolean)* Enable exposing *client function* for this connection. (Check *Client Function* section for details.) |
| `access_token`                | `function`                    | *(function)* If you need custom authorization behavior for  generating signatures for private/ presence/ system channels/rpc functions, you can provide your own `access_token` function as below: |

#### access_token *(function)*

```javascript
dbridge.access_token(async (channelName, sessionId, action, response) => { 
    response.end({
                    statuscode: 0/1,
                    error_message: '',
                    accesskey: 'computed access token'
                });
});
```

***	TODO : Gyan for Access_Token Generation... JWT etc...  to discuss with @kannan.***

#### `authorizer` (Function)

If you need custom authorization behavior you can provide your own `authorizer` function as follows:

```js
const dbridge = new dbridge(APP_KEY, {
  cluster: APP_CLUSTER,
  authorizer: function (channel, options) {
    return {
      authorize: function (socketId, callback) {
        // Do some ajax to get the auth string
        callback(null, {auth: authString});
      }
    };
  }
})
```

Example: An authorizer which uses the [`fetch`
API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) to make a JSON
request to an auth endpoint

```js
let authorizer = (channel, options) => {
  return {
    authorize: (socketId, callback) => {
      fetch(authUrl, {
        method: "POST",
        headers: new Headers({ "Content-Type": "application/json" }),
        body: JSON.stringify({
          socket_id: socketId,
          channel_name: channel.name
        })
      })
        .then(res => {
          if (!res.ok) {
            throw new Error(`Received ${res.statusCode} from ${authUrl}`);
          }
          return res.json();
        })
        .then(data => {
          callback(null, data);
        })
        .catch(err => {
          callback(new Error(`Error calling auth endpoint: ${err}`), {
            auth: ""
          });
        });
    }
  };
};
const dbridge = new dbridge(APP_KEY, {
  cluster: APP_CLUSTER,
  authorizer: authorizer,
})
```



## Connection

Once the properties are set, use `connect()` function to connect to dBridge Network.

```javascript
dbridge.connect().catch((err) => {
    console.log('dBridge Connection exception..', err.source, err.code, err.message);
});
```

#### Exceptions: 

`Source : DBLIB_CONNECT`

| Code                           | Message                         | Description                                                  |
| ------------------------------ | ------------------------------- | ------------------------------------------------------------ |
| `INVALID_URL`                  | `null`                          | Value of `dbridge.auth_url` is not a valid dBrdige authentication URL. |
| `INVALID_AUTH_PARAM`           | `null`                          | Value of `dbridge.appkey` is not a valid dBrdige application key. |
| `INVALID_ACCESSTOKEN_FUNCTION` | `null`                          | If *"callback function"* is not declared for authentication while using  private, presence or system channel/RPC functions. *(Check Channel or RPC section for details.)* |
| `HTTP_`                        | HTTP protocol reported message. | HTTP Errors returned during authentication process. ***HTTP Error code*** will be concatenated with `HTTP_` in the `err.code`. `eg. HTTP_501` |
| `INVALID_CLIENTFUNCTION`       | `null`                          | If *"callback function"* is not declared for client function **or** `typeof()` variable defined is not a *"function"*. *(Check Client Function section for details.)* |

#### sessionid *(string)*

```javascript
console.log(dbridge.sessionid);
```

Making a connection provides the client with a new `sessionid` that is assigned by the server. This can be used to distinguish the client's own events. A change of state might otherwise be duplicated in the client. It is also stored within the connection, and used as a token for generating signatures for private/presence/system channels/rpc functions.

#### disconnect *(function)*

To **close a connection** use disconnect function. When a connection has been closed explicitly, no automatic reconnection will happen.

```javascript
dbridge.disconnect();
```

## Objects

| Object            | Description                                       |
| ----------------- | ------------------------------------------------- |
| `connectionstate` |                                                   |
| `channel`         |                                                   |
| `rpc`             | RPC functions, properties and events.             |
| `cf`              | Client-function functions, properties and events. |



## object:ConnectionState

Object that expose properties, functions and events to manage/monitor dBridge network connection.

### Properties

The following is the list of connection state properties.

```javascript
console.log(dbridge.connectionstate.state);
console.log(dbridge.connectionstate.isconnected);
console.log(dbridge.connectionstate.rttms);
console.log(dbridge.connectionstate.reconnect_attempt);
```

| Property                            | Description                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| `connectionstate.state`             | *(String)* Current state of dBridge network connection . List of Return Values are detailed below. |
| `connectionstate.isconnected`       | *(Boolean)* Current state of dBridge network connection.     |
| `connectionstate.rttms`             | *(integer)* Latency between your app and dBridge network in milliseconds. |
| `connectionstate.reconnect_attempt` | (integer) Number of reconnection attempted as of now.        |

##### connectionstate.state

| Return Values      | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| *connecting*       | Your app is now attempting to connect to dBridge network.    |
| *connected*        | The connection to dBridge network is open and authenticated with your `appkey`. |
| *connection_break* | Intermediate state between `connecting` and `connect_error`. |
| *connect_error*    | The dBridge network connection was previously connected and has now errored and closed. |
| *disconnected*     | The dBridge network connection was previously connected and has now intentionally been closed. |
| *reconnecting*     | Your app is now attempting to reconnect to dBridge network as per properties set for reconnection. |
| *reconnect_error*  | Reconnection attempt has errored. dBridge network will retry as per properties set for reconnection. |
| *reconnect_failed* | Reconnection attempt has failed. dBridge network is now disconnected, as reconnection attempts has exhuasted. |
| *reconnected*      | The connection to dBridge network is open and reconnected after `connect_error` **or** `reconnect_error`. |

### Functions

```js
dbridge.connectionstate.rttping();

dbridge.connectionstate.bind("rttpong", (payload) => {
    console.log('rttpong', payload, dbridge.connectionstate.rttms);
});
```

#### rttping()

This method is used to check the dBridge network latency. Event `rttpong` is triggered once response  is received from dBridge network.

#### Exceptions: 

| Source        | Code                 | Description                                  |
| ------------- | -------------------- | -------------------------------------------- |
| DBLIB_RTTPING | NETWORK_DISCONNECTED | Connection to dBridge network is not active. |

### Binding To Events

Event binding takes a very similar form to the way events are handled in jQuery. You can use the following methods on connectionstate object to bind to events.

```javascript
dbridge.connectionstate.bind(eventName, () => {});
dbridge.connectionstate.unbind(eventName);
dbridge.connectionstate.unbind();
```

`bind()` on `eventName` has callback functions to be defined where you can write your own code as per requirement.

To stop listening to events use `unbind(eventName)` function.

To stop listening to all events use `unbind()` *[without eventName]* function.

Below are library events which can be bind to receive information about dBridge network.

#### Default Events

```js
dbridge.connectionstate.bind("connecting", () => {
    console.log('connecting');
});
dbridge.connectionstate.bind("reconnecting", () => {
    console.log('reconnecting', "reconnect_attempt=", dbridge.connectionstate.reconnect_attempt);
});
dbridge.connectionstate.bind("disconnected", () => {
    console.log('disconnected');
});
dbridge.connectionstate.bind("reconnected", () => {
    console.log('reconnected');
}); 
dbridge.connectionstate.bind("connection_break", (payload) => {
    console.log('connection_break', payload.source, payload.code, payload.message);
});
dbridge.connectionstate.bind("state_change", (payload) => {
    console.log('state_change', dbridge.connectionstate.state, payload);
});
dbridge.connectionstate.bind("connect_error", (payload) => {
    console.log('connect_error', payload.source, payload.code, payload.message); 
});
dbridge.connectionstate.bind("reconnect_error", (payload) => {
    console.log('reconnect_error', payload.source, payload.code, payload.message);
}); 
dbridge.connectionstate.bind("reconnect_failed", (payload) => {
    console.log('reconnect_failed', payload.source, payload.code, payload.message); 
});
dbridge.connectionstate.bind("rttpong", (payload) => {
    console.log('rttpong', payload, dbridge.connectionstate.rttms);
}); 
dbridge.connectionstate.bind("connected", () => { 
    console.log('dBridge Event Bind', 'connected');
});
```

| Events             | Parameters                                                   | Description                                                  |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `connecting`       |                                                              | This event is triggered when your app is attempting to connect to dBridge network. |
| `connected`        | *payload* with `payload.source, payload.code, payload.message` | This event is triggered when connection to dBridge network is open and authenticated with your `appkey`. |
| `connection_break` | *payload* with `payload.source, payload.code, payload.message` | This event is triggered when state is between between `connecting` and `connect_error`. |
| `connect_error`    | *payload* with `payload.source, payload.code, payload.message` | This event is triggered when the dBridge network connection was previously connected and has now errored and closed. |
| `disconnected`     |                                                              | This event is triggered when the dBridge network connection was previously connected and has now intentionally been closed. |
| `reconnecting`     |                                                              | This event is triggered when  app is now attempting to reconnect to dBridge network as per properties set for reconnection. |
| `reconnect_error`  | *payload* with `payload.source, payload.code, payload.message` | This event is triggered when reconnection attempt has errored. |
| `reconnect_failed` | *payload* with `payload.source, payload.code, payload.message` | This event is triggered when reconnection attempt has failed. |
| `reconnected`      |                                                              | This event is triggered when the connection to dBridge network is open and reconnected after `connect_error` **or** `reconnect_error`. |
| `state_change`     | *payload* with `payload.previous, payload.current`           | This event is triggered whenever there is any state changes in dBridge network connection. Payload will have previous and current state of connection. |
| `rttpong`          | `payload`                                                    | *(integer)* In Response to `rttping()` function call to dBridge network, payload has 2 way latency information in milliseconds from dBridge network. |

#### Exceptions: 

| Source             | Code              | Description                                                  |
| ------------------ | ----------------- | ------------------------------------------------------------ |
| DBLIB_CONNECT_BIND | INVALID_EVENTNAME | Invalid Event name. Not in defined events as above.          |
| DBLIB_CONNECT_BIND | INVALID_CALLBACK  | If *"callback function"* is not declared **or** `typeof()` variable defined is not a *"function"*. |



## object:Channel

dBridge library supports **4** types of channel. The *namespace is the  4 characters* preceding the channelName (`pvt:,prs:,sys:`), identifying which type of channel the application is connecting to. If the channel type is not Public, dBridge library will use the `access_token` function to get the access encrypted token and will use it for all communication with this channel.

| Channel Type | Channel Name Style | Description |
| ------------ | ------------------ | ----------- |
| Public       | channelName        |             |
| Private      | **pvt:**channeName |             |
| Presence     | **prs:**channeName |             |
| System       | **sys:**channeName |             |

dBridge library provides 2 different ways to access any of above channel types based on usage.

| Channel Type         | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| Subscribe to Channel | Application can listen to user-defined events for which it has subscribed. |
| Connect to Channel   | Application can only publish user-defined events for which it has connected to. |

### Subscribe to Channel

#### subscribe()

The default method for subscribing to a channel involves invoking the `channel.subscribe` function of your dbridge object:

```js
try {
    const subscribed_channel = dbridge.channel.subscribe('pvt:mychannel');
} catch (err) {
    console.log(err.source, err.code, err.message);
}
```

| Parameter | Rules                                                        | Description                                     |
| --------- | ------------------------------------------------------------ | ----------------------------------------------- |
| `string`  | *channelName **OR**<br />**pvt:**channelName **OR**<br />**prs:**channelName **OR**<br />**sys:**channelName* | *channel*Name to which subscription to be done. |

| Return Type | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `object`    | *channel* object which events and related functions can be bound to. |

Application can directly work with dbridge object without using Channel object. Using this method application **cannot publish** any user-defined events.

```js
try {
    dbridge.channel.subscribe('pvt:mychannel');
} catch (err) {
    console.log(err.source, err.code, err.message);
}
```

##### Exceptions: 

| Source                  | Code                       | Description                                                  |
| ----------------------- | -------------------------- | ------------------------------------------------------------ |
| DBLIB_CHANNEL_SUBSCRIBE | NETWORK_DISCONNECTED       | Connection to dBrdige network is not active.                 |
| DBLIB_CHANNEL_SUBSCRIBE | INVALID_CHANNELNAME        | *channelName* is not defined.                                |
| DBLIB_CHANNEL_SUBSCRIBE | INVALID_CHANNELNAME        | channelName validation error, `typeof()`  *channelName*  is not type string |
| DBLIB_CHANNEL_SUBSCRIBE | INVALID_CHANNELNAME_LENGTH | *channelName* validation error, length of *channelName*  greater than **64** |
| DBLIB_CHANNEL_SUBSCRIBE | INVALID_CHANNELNAME        | *channelName* validation error, *channelName* fails `a-zA-Z0-9\.:_-` validation. |
| DBNET_CHANNEL_SUBSCRIBE | ERR_FAIL_ERROR             | dBrdige network encountered error when subscribing to the channel. |
| DBNET_CHANNEL_SUBSCRIBE | ERR_ACCESS_DENIED          | dBrdige network reported **access violation** with `access_token` function during subscription of this channel. |
| DBLIB_CHANNEL_SUBSCRIBE | CHANNEL_ALREADY_SUBSCRIBED | *channelName* is already subscribed.                         |
| DBLIB_CHANNEL_SUBSCRIBE | ACCESS_TOKEN               | `dbridge.access_token` function returned error.              |

#### unsubscribe() 

To unsubscribe from a subscribed channel, invoke the `unsubscribe` function of your dbridge object. `unsubscribe` cannot be done on channel object.

```javascript
try {
    dbridge.channel.unsubscribe('pvt:mychannel');
} catch (err) {
    console.log(err.source, err.code, err.message);
}
```

| Parameter | Rules                                                        | Description                                     |
| --------- | ------------------------------------------------------------ | ----------------------------------------------- |
| `string`  | *channelName **OR**<br />**pvt:**channelName **OR**<br />**prs:**channelName **OR**<br />**sys:**channelName* | *channel*Name to which subscription to be done. |

| Return Type | Description |
| ----------- | ----------- |
| `NA`        |             |

##### Exceptions: 

| Source                    | Code                          | Description                                                  |
| ------------------------- | ----------------------------- | ------------------------------------------------------------ |
| DBLIB_CHANNEL_UNSUBSCRIBE | NETWORK_DISCONNECTED          | Connection to dBrdige network is not active.                 |
| DBLIB_CHANNEL_UNSUBSCRIBE | CHANNEL_NOT_SUBSCRIBED        | *channelName* is not subscribed.                             |
| DBLIB_CHANNEL_UNSUBSCRIBE | INVALID_CHANNEL               | *channelName* is not subscribed, but it is in connected state. |
| DBNET_CHANNEL_UNSUBSCRIBE | ERR_FAIL_ERROR                | dBrdige network encountered error when unsubscribing to the channel. |
| DBNET_CHANNEL_UNSUBSCRIBE | ERR_ACCESS_DENIED             | dBrdige network reported **access violation** with `access_token` function during unsubscribing of this channel. |
| DBLIB_CHANNEL_UNSUBSCRIBE | UNSUBSCRIBE_ALREADY_INITIATED | *channelName* is already trying to unsubscribe.              |

### Connect to Channel

#### connect()

The default method for connecting to a channel involves invoking the `channel.connect` function of your dbridge object. Application can only publish user-defined events for which it has connected to. 

```js
try {
    const connected_channel = dbridge.channel.connect('pvt:mychannel');
} catch (err) {
    console.log(err.source, err.code, err.message);
}
```

| Parameter | Rules                                                        | Description                                   |
| --------- | ------------------------------------------------------------ | --------------------------------------------- |
| `string`  | *channelName **OR**<br />**pvt:**channelName **OR**<br />**prs:**channelName **OR**<br />**sys:**channelName* | *channel*Name to which connection to be done. |

| Return Type | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `object`    | *channel* object which events and related functions can be bound to. |

##### Exceptions: 

| Source                | Code                      | Description                                                  |
| --------------------- | ------------------------- | ------------------------------------------------------------ |
| DBLIB_CHANNEL_CONNECT | NETWORK_DISCONNECTED      | Connection to dBrdige network is not active.                 |
| DBNET_CHANNEL_CONNECT | ERR_FAIL_ERROR            | dBrdige network encountered error when subscribing to the channel. |
| DBNET_CHANNEL_CONNECT | ERR_ACCESS_DENIED         | dBrdige network reported **access violation** with `access_token` function during subscription of this channel. |
| DBLIB_CHANNEL_CONNECT | ACCESS_TOKEN              | `dbridge.access_token` function returned error.              |
| DBLIB_CHANNEL_CONNECT | CHANNEL_ALREADY_CONNECTED | *channelName* is already connected.                          |
| DBLIB_CHANNEL_CONNECT | INVALID_CHANNELNAME       | *channelName* is not defined.                                |
| DBLIB_CHANNEL_CONNECT | INVALID_CHANNELNAME       | channelName validation error, `typeof()`  *channelName*  is not type string |
| DBLIB_CHANNEL_CONNECT | INVALID_CHANNELNAME       | *channelName* validation error, length of *channelName*  greater than **64** |
| DBLIB_CHANNEL_CONNECT | INVALID_CHANNELNAME       | *channelName* validation error, *channelName* fails `a-zA-Z0-9\.:_-` validation. |
| DBLIB_CHANNEL_CONNECT | INVALID_CHANNELNAME       | if *channelName* contains `:` and first token is not `pvt,prs,sys` |

#### disconnect() 

To disconnect from a connected channel, invoke the `disconnect` function of your dbridge object. `disconnect` cannot be done on channel object.

```js
try {
    dbridge.channel.disconnect('pvt:mychannel');
} catch (err) {
    console.log(err.source, err.code, err.message);
}
```

| Return Type | Description |
| ----------- | ----------- |
| `NA`        |             |

##### Exceptions: 

| Source                   | Code                         | Description                                                  |
| ------------------------ | ---------------------------- | ------------------------------------------------------------ |
| DBLIB_CHANNEL_DISCONNECT | NETWORK_DISCONNECTED         | Connection to dBrdige network is not active.                 |
| DBLIB_CHANNEL_DISCONNECT | ERR_FAIL_ERROR               | dBrdige network encountered error when unsubscribing to the channel.. |
| DBLIB_CHANNEL_DISCONNECT | ERR_ACCESS_DENIED            | dBrdige network reported **access violation** with `access_token` function during unsubscribing of this channel. |
| DBLIB_CHANNEL_DISCONNECT | DISCONNECT_ALREADY_INITIATED | *channelName* is already trying to disconnect.               |
| DBLIB_CHANNEL_DISCONNECT | INVALID_CHANNEL              | *channelName* is not connected.                              |
| DBLIB_CHANNEL_DISCONNECT | INVALID_TYPE                 | *channelName* is not connected, but it is in subscribed state. |

### Channel Information

#### isOnline()

*<u>dbrdigeObject</u>* as well as *<u>channelObject</u>* provides a function to check if the channel is online. 

```js
const isonline = dbridge.channel.isOnline('pvt:mychannel');
```

```js
const isonline = subscribed_channel.isOnline(); 
```

| Parameter | Rules                                                        | Description                                     |
| --------- | ------------------------------------------------------------ | ----------------------------------------------- |
| `string`  | *channelName **OR**<br />**pvt:**channelName **OR**<br />**prs:**channelName **OR**<br />**sys:**channelName* | *channel*Name to which subscription to be done. |

| Return Values | Description                                         |
| ------------- | --------------------------------------------------- |
| `boolean`     | Is the current status of channel online or offline. |

#### list()

<u>*dbrdigeObject*</u>  provides a function to get list of successfully subscribed or connected channel. 

```javascript
const channels = dbridge.channel.list();
#=> [{"name":  "pvt:mychannel" , "type": "subscribed/connect" ,  "isonline": true/false }];
```

| Return Type     | Description                                |
| --------------- | ------------------------------------------ |
| `array of dict` | Array of channels subscribed or connected. |

Dictionary contains below information.

| Key        | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `name`     | *(string)* *channelName* of subscribed or connected channel. |
| `type`     | *(string)* `subscribed` **or** `connected`                   |
| `isonline` | *(boolean)* Is the current status of channel online or offline. |

#### getChannelName() 

*<u>channelObject</u>* provides a function to get the *channelName*. 

```js
const chName = channelobject.getChannelName() 
```

| Return Type | Description                                       |
| ----------- | ------------------------------------------------- |
| `string`    | *channelName* of subscribed or connected channel. |

#### call() 

*<u>channelObject</u>* provides a function to get the *channelMemberList* and *channelMemberInfo* of subscribed channel. This function is only applicable for *channelType* **presence** (`prs:`) or **system**(`sys:`). 

```javascript
channelObject.call(functionName, parameter, ttlms, (response) => {
    // functionName = '', 'channelMemberInfo'
    console.log('multipart: ', response) 
}).then((response) => {
    console.log('response: ', response);
}).catch((err) {
	console.log(err.source, err.code, err.message);
}); 

```

| Parameter      | Expected Value                                 | Description                                                  |
| -------------- | ---------------------------------------------- | ------------------------------------------------------------ |
| `functionName` | `channelMemberList` **OR** `channelMemberInfo` | *(string)* Get all members who has subscribed to the channel. |
| parameter      | `null`                                         |                                                              |
| ttlms          | `10000`                                        | *(integer)* Time to live in millisecond, timeout value before the library throws error timeout. |

| Return Values | Description                                               |
| ------------- | --------------------------------------------------------- |
| `dict`        | List or details of all members subscribed to the channel. |

Dictionary contains below information.

| Key       | Description |
| --------- | ----------- |
| `members` | *(string)*  |
| `type`    | *(string)*  |

##### Exceptions: 

| Source                   | Code                      | Description                                                  |
| ------------------------ | ------------------------- | ------------------------------------------------------------ |
| DBLIB_CHANNEL_CALL       | NETWORK_DISCONNECTED      | Connection to dBrdige network is not active.                 |
| DBNET_CHANNEL_CALL       | ERR_FAIL_ERROR            | dBrdige network encountered error when unsubscribing to the channel.. |
| DBNET_CHANNEL_CALL       | ERR_ACCESS_DENIED         | dBrdige network reported **access violation** with `access_token` function during unsubscribing of this channel. |
| DBNET_CHANNEL_CALL       | ERR_CALLEE_NOT_REACHABLE  |                                                              |
| DBNET_CHANNEL_CALL       | ERR_CALLER_NOT_REACHABLE  |                                                              |
| DBNET_CHANNEL_CALL       | ERR_ROUTING_EXCEPTIONS    |                                                              |
| DBNET_CHANNEL_CALL       | ERR_CALLER_QUEUE_EXCEEDED |                                                              |
| DBNET_CHANNEL_CALL       | ERR_DUPLICATE_RPCID       |                                                              |
| DBNET_CHANNEL_CALL       | ERR_RATELIMIT_EXCEEDED    |                                                              |
| DBLIB_CHANNEL_CALL       | INVALID_FUNCTION          | *functionName* is not supported.                             |
| DBLIB_CHANNEL_CALL       | INVALID_CHANNELNAME       | *channel* is not connected or is invalid.                    |
| DBRPCCALLEE_CHANNEL_CALL | CALLEE_EXCEPTION          | $CustomMessageSentByCalleeProgram                            |

## Publish to Channel

It's possible to publish user-defined events using the `publish` function on an instance of the `channel` object.

A few gotchas to consider when using user-defined events:

- User-defined events must be enabled in the settings page for your app: `https://dashboard.dbridge.com/apps/$YOUR_APP_ID/settings`

#### publish()

The default method for publishing to a channel involves invoking the `channel.publish` function of your *channelObject*. Application can only publish user-defined events for which it has connected to. 

```javascript
try {
    channelObject.publish(event, payload, seqno);
} catch (err) {
    console.log(err.source, err.code, err.message);
}

```

| Parameter | Description                                                  |
| --------- | ------------------------------------------------------------ |
| `event`   | *(string)* *event* Name to which the message to be sent. *event* Name cannot start with `dbridges:` |
| `payload` | *(string)* Payload to be sent with the event.                |
| `seqno`   | *(string) [optional]* Message sequence number to be sent out. This is optional parameter. |

| Return Values | Description |
| ------------- | ----------- |
| `NA`          |             |

##### Exceptions: 

| Source                | Code                 | Description                                                  |
| --------------------- | -------------------- | ------------------------------------------------------------ |
| DBLIB_CHANNEL_PUBLISH | INVALID_SUBJECT      | *event* validation error, `typeof()`  *event*  is not type string |
| DBLIB_CHANNEL_PUBLISH | INVALID_SUBJECT      | *event* validation error, `typeof()`  *event*  is not defined |
| DBLIB_CHANNEL_PUBLISH | NETWORK_DISCONNECTED | Connection to dBrdige network is not active.                 |
| DBLIB_CHANNEL_PUBLISH | INVALID_CHANNELNAME  | if *channelName* contains `sys:*`                            |

## Binding to events

Event binding takes a very similar form to the way events are handled in jQuery. You can use the following methods either on a *channelObject*, to bind to events on a particular channel; or on the *dbridgeObject*, to bind to events on all subscribed channels simultaneously.

### `bind` and `unbind`
**Bind** to "event" on channel: payload and metadata is received.

```js
//  Binding to channel events on channelObject  
try {
    channelObject.bind('event',  (payload, metadata) => {
        console.log(payload, metadata);
    });
} catch(err){
     console.log(err.source, err.code, err.message);
}

// Binding to channel events on dbridgeObject 

try {
    dbridge.channel.bind('event', (payload, metadata) => {
    	console.log(payload, metadata);
	});
} catch(err){
     console.log(err.source, err.code, err.message);
}
```

Callback out parameter `payload, metadata` details are explained with each event below in this document.

#### Exceptions: 

| Source                | Code              | Description                                                  |
| --------------------- | ----------------- | ------------------------------------------------------------ |
| DBLIB_CONNECT_BIND    | INVALID_EVENTNAME | Invalid Event name. Not in user-defined events or default events. |
| DBLIB_CONNECT_BIND    | INVALID_CALLBACK  | If *"callback function"* is not declared **or** `typeof()` variable defined is not a *"function"*. |
| DBLIB_CHANNEL_CONNECT | INVALID_BINDING   | Invalid Event name. This binding is not allowed in `channel.connect`. |

**Unbind** behavior varies depending on which parameters you provide it with. For example:

```js
// Remove just `handler` of the `event` in the subscribed/connected channel 
channelObject.unbind('event',handler);

// Remove all `handler` of the `event` in the subscribed/connected channel
channelObject.unbind('event');

//Remove all handlers for the all event in the subscribed/connected channel
channelObject.unbind();

// Remove `handler` of the `event` for all events across all subscribed/connected channels
dbridge.channel.unbind('event',handler);

// Remove all handlers of the `event` for all events across all subscribed/connected channels
dbridge.channel.unbind('event');

// Remove all handlers for all events across all subscribed/connected channels
dbridge.channel.unbind();
```

### `bind_all` and `unbind_all`

`bind_all` and `unbind_all` work much like `bind` and `unbind`, but instead of only firing callbacks on a specific event, they fire callbacks on any event, and provide that event in the metadata  to the handler along with the payload. 

```js
//  Binding to all channel events on channelObject  
try {
    channelObject.bind_all((payload, metadata) => {
        console.log(payload, metadata);
    });
} catch(err){
     console.log(err.source, err.code, err.message);
}

// Binding to all channel events on dbridgeObject 

try {
    dbridge.channel.bind_all((payload, metadata) => {
    	console.log(payload, metadata);
	});
} catch(err){
     console.log(err.source, err.code, err.message);
}
```

Callback out parameter `payload, metadata` details are explained with each event below in this document.

#### Exceptions: 

| Source             | Code             | Description                                                  |
| ------------------ | ---------------- | ------------------------------------------------------------ |
| DBLIB_CONNECT_BIND | INVALID_CALLBACK | If *"callback function"* is not declared **or** `typeof()` variable defined is not a *"function"*. |

`unbind_all` works similarly to `unbind`.

```js
// Remove just `handler` across the channel 
channelObject.unbind_all(handler);

//Remove all handlers for the all event in the subscribed/connected channel
channelObject.unbind_all();

// Remove `handler` across the subscribed/connected channels
dbridge.channel.unbind_all(handler);

// Remove all handlers for all events across all subscribed/connected channels
dbridge.channel.unbind_all();
```

### user-defined events

user-defined events are triggered whenever an event is published to the channel from any other application. To listen to the event you need to bind it as below. These events will be triggered in bind_all handlers also if defined.

```javascript
//  Binding to user-defined events on channelObject  
try {
    channelObject.bind('event', (payload, metadata) => {
        console.log( payload, metadata);
    });
} catch(err) {
     console.log(err.source, err.code, err.message);
}
//  Binding to user-defined events on dbridgeObject  
try {
    dbridge.channel.bind('event', (payload, metadata) => {
        console.log( payload, metadata);
    });
} catch(err) {
     console.log(err.source, err.code, err.message);
}
```

| Parameter | Description                                                  |
| --------- | ------------------------------------------------------------ |
| `event`   | *(string)* user-defined *event* Name to which binding to be done. |

##### Callback parameters

###### payload: 

`(string)` Payload data sent by the publisher.

###### metadata `(dict)`:

```json
{
    "channelname": "channelName" , 		// (string) channelName to which subscription is done.
    "eventname": "event",				// (string) eventName 
    "sourcesysid": "", 					// (string) Sender system identity, applicable only for presence or system channel.
    "sqnum": "1",						// (string) user defined, sent during publish function.
    "sessionid": "", 					// (string) Sender sessionid, applicable only for presence or system channel.
    "intime": 1645554960732  			// (string) EPOC time of the sender at time of publish.
}
```

### Default events

There are a number of events which are triggered internally by the library, but can also be of use elsewhere. Below are the list of all events triggered by the library.

Below syntax is same for all system events.

```javascript
//  Binding to systemevent events on channelObject  
try {
    channelObject.bind('dbridges:subscribe.success', (payload, metadata) => {
        console.log( payload, metadata);
    });
} catch(err) {
     console.log(err.source, err.code, err.message);
}
//  Binding to systemevent events on dbridgeObject  
try {
    dbridge.channel.bind('dbridges:subscribe.success', (payload, metadata) => {
        console.log( payload, metadata);
    });
} catch(err) {
     console.log(err.source, err.code, err.message);
}
```

  (:scroll: : Subscribe process, :cl: : Connect Process)

#### dbridges:subscribe.success :scroll:

##### Callback parameters

###### payload: 

`(string)` Payload data sent by the publisher.

###### metadata `(dict)`:

```json
{
    "channelname": "channelName" , 		// (string) channelName to which subscription is done.
    "eventname": "event",				// (string) eventName 
    "sourcesysid": "", 					// (string) Sender system identity, applicable only for presence or system channel.
    "sqnum": "1",						// (string) user defined, sent during publish function.
    "sessionid": "", 					// (string) Sender sessionid, applicable only for presence or system channel.
    "intime": 1645554960732  			// (string) EPOC time of the sender at time of publish.
}
```

<table>
  <tr>
    <td>
           hi
    </td>
  </tr>
</table>







#### dbridges:subscribe.fail

#### dbridges:channel.online

#### dbridges:channel.offline

#### dbridges:channel.removed

#### dbridges:unsubscribe.success

#### dbridges:unsubscribe.fail

#### dbridges:resubscribe.success

#### dbridges:resubscribe.fail

#### dbridges:participant.joined

#### dbridges:participant.left

dbridges:connect.success
dbridges:connect.fail
dbridges:disconnect.success
dbridges:disconnect.fail
dbridges:reconnect.success
dbridges:reconnect.fail



```js
dbridge.connection.bind('state_change', function(states) {
  // states = {previous: 'oldState', current: 'newState'}
  $('div#status').text("Channels current state is " + states.current);
});
```











| Event | public             | pvt: | prs: | sys: |
| ----- | ------------------ | ---- | ---- | ---- |
|       | :heavy_check_mark: | :x:  |      |      |
|       |                    |      |      |      |
|       |                    |      |      |      |



