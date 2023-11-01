# Realtime communications and distributed computing

## System design of a distributed chat system

#### Why WebSockets?

1. Http is great, but doesn't cover a few usercases. <br>
2. The most important use case it doesn't cover is server side events. <br>
3. What if you want to create a realtime application? like
   zerodha or Binance<br>
   
In normal https, server can't directly send you events. client ask the server and server responds, what if your application is realtime. let say you have a webapp and a mobile app. changes in one should be seen imediately in the other. like notion or let say google docs, if you make changes in one file in google docs in one tab and open the same file in another pc it will show the changes in realtime.
#### So to create a realtime application we need a protocol that gives a **<span style="background-color: darkblue;">Persistent</span>**  connection between client and server. and that is websockets. A continuous connection will let the client constantly know the changes in the server using TCP. 
Let say two users are chatting with each other. one user send a message to the other using a https request. here you can  say the client does polling after let say every 1 seconds. this is called **<span style="background-color: darkblue;">long polling.</span>** but this is not **<span style="background-color: darkblue">efficient .</span>** because the client is constantly asking the server if there is any new message. Another is **<span style="background-color: darkblue;">Server Side Events</span>** that is SSE. It sends events from server to client. but it is not **<span style="background-color: darkblue;">bidirectional .</span>** So we need a protocol that is bidirectional and that is websockets. Under the hood in http there is handshake between client and server. it works on transport layer and creates a **<span style="background-color: darkblue;">TCP</span>** connection and after the request is completed it **<span style="background-color: darkred;">closes</span>** the TCP connection it is not persistent connection.

Compare to WebSocket Sever the goal is to create a persistent connection between the server and the client. The browser sends request to the server and server sends something or some data to the browser or client.

**<span style="background-color: darkblue;">Both Http and WebSocket are different Protocols/Standards.</span>** 

**<span style="background-color: brown;">#### Interview Question</span>** 
1. Is WebSocket http or not? <br>
answer: It is not http but the first request that you send out( from client or browser) when you want to create a webSocket connection to any open server. that request is a http request.but after that under the hood it gets updated to a WebSocket connection that is persistent connection to the backend.

to check if it is a websocket connection you can inspect in a page and go to network and see the headers the request url is **<span style="background-color: darkblue;">wss://</span>** and a lot of connection stream of data. its not you repeatedly asking the server with request . its the server sending you events pushed down by backend in webSocket.

**<span style="background-color: darkblue;"> ##### WebRTC</span>** also lets you get data its a similar approach but it has its differences.


**<span style="background-color: brown;">#### Interview Question</span>** 
1. Difference between WebRTC and WebSockets and when to use what ? <br>
   answer: WebRTC uses **<span style="background-color: darkblue;">UDP</span>**  while Websockets uses **<span style="background-color: darkblue;">TCP .</span>** </br>
   a. if you need all info you use WebSockets if you are ok with some data loss you can use WebRTC. like live stream, gaming <br>

TCP: Transmission Control Protocol. It is a connection oriented protocol. It is reliable and slower than UDP. It is used for applications that require high reliability, and transmission time is relatively less critical. Ex: HTTP, HTTPS, FTP, SMTP, Telnet etc.   

UDP: User Datagram Protocol. It is a connectionless protocol. It is unreliable and faster than TCP. It is used for applications that require fast transmission, such as games. Ex: DNS, DHCP, TFTP, SNMP, RIP, VOIP etc.

#### HTTP is not enough
When you need realtime updates from the backend
You could
1. Use long polling
2. Use server side events
3. Use websockets

What are WebSockets?
WebSockets is a communication protocol that provides a **<span style="background-color: darkblue;">full-duplex communication</span>** channels over a single, long lived connection. they are not shortlived.

so is there only server where everyone connects to or are there multiple servers? and those servers talk to each other and propogate data or changes. or are there **<span style="background-color: darkblue;">swarm</span>** of servers where each server can connect to x number of users? here is where **<span style="background-color: red;">redis</span>** comes in.

Websocket is node js specific concept? No. it's a protocol it can be written in any language. like there are many libraries in nodejs to create a http server like express just like that there are many npm library to create a websocket server. like **<span style="background-color: darkblue;">socket.io</span>** or **<span style="background-color: darkblue;">ws</span>** or **<span style="background-color: darkblue;">websocket</span>**. socket /io is not completly websocket it is some abstraction on top of websockets. It has a room creation feature out of the box. It is not used in production servers because if you use it, your client also needs to talk to socket.io and it is not the exact websocket protocol you can tweak it though like you can create a normal websocket connection which talks to socket.io but your anroid, ios, golang client they dont understand socket.io they only understand websockets. It became popular because someone wrote socket.io client, c++, android etc but still it is not widely used. **<span style="background-color: darkblue;">Websocket protocol or Raw websocket is widely used even if a interviewer ask what should you use in production socket.io or raw websocket you should say raw websocket why ? because it is proven widely used and someone has written a client for it in every language, android, ios, c++, golang etc .</span>**

just like fetch websocket is a browser api. this is client side code.
like you use fetch.

Webhooks: For example you use razorpay which lets you do payments. Razorpay uses http server. razorpay's http server will send request to your http server and this is called webhooks. if your payment fail from a bank let say. users request hits the razorpay endpoint but razorpays endpoint then bank's enpoint has to tell razorpays endpoint about the transactions and then razor pays endpoint has to tell my endpoint about the transaction. so this is called webhooks. so razorpays server talking to my backend through different banks servers or across servers is called webhook. the protocol there is still TCP but this communications between different servers is called webhook.

A Websocket is completely different thing, it is realtime communication between client and server. and completely different from webhooks. where as webhooks is communication across different servers.

import http from "http"; is a native http server that comes with nodejs out of the box. you can create a http server with it we will use express. 

```javascript
const server = http.createServer(app);
``` 
 //http is native nodejs to create a http server of express app.
```javascript
wss = new WebSocketServer({ server }); 
```
 websocketserver needs a parameter that is a http server. so we are passing the express server to it. so that it can create a websocket server on top of it.
**<span style="background-color: darkblue;"> So this has the logic to send the first http request and upgrade it to a persistent long live connection .</span>**


#### Server Side Code

```javascript
wss.on("connection", async (ws, req) => {
    console.log("someone connected");
    ws.on("message", (message) => {
        console.log("received: %s", message);
        ws.send(`Hello, you sent -> ${message}`);
    });
});
```
websocket on connection or if someone connects to the server the control reaches the console log and prints "someone connected".
wss.on message, that means when someone sends a message to the server. you can perform some action like a console log message or do database operation. This above code shows the **<span style="background-color: darkblue;"> Full Duplex </span>** nature of websocket. since client can send a message to the server and server can send a message to the client. This is kind of a simple example of sever side code. in here async (ws, req) ws is the websocket connection for that **<span style="background-color: darkblue;"> particular client/user .</span>**

Websocket connection can also be hit by a client/browser/postman/postwoman(hopscotch.io) etc.

![Alt text](image.png)
<br/><br/>
![Alt text](image-2.png)


#### Client Side Code

```HTML
<html>
    <head>
        <title>Simple WS Client</title>
        <script>
            const webSocket = new WebSocket("ws://localhost:3000");  
            function sendMessage() {
                webSocket.send(document.getElementById("inputbox").value);
            }
            webSocket.onmessage = function (event) {
                document.getElementById("serverMessages").appendChild(document.createTextNode(event.data));
                document.getElementById("serverMessages").appendChild(document.createElement("br"));
            }
        </script>
    </head>
    <body> 
        <input type="text" id="inputbox" />
        <button onclick="sendMessage()">send</button>
        <br/><br/>
        <h2>
            Events from server
        </h2>
        <div id="serverMessages">

        </div>
    </body>
</html>
```

const webSocket = new WebSocket("ws://localhost:3000");  // you can use axios/fetch in react here, we are using Raw WebSocket Client here by passing URL to our backend server. 
webSocket.onmessage = function (event) // when a server sends message to the client using function **<span style="background-color: darkblue;"> webSocket.onmessage = function (event) </span>** is called and it will append the message to the div with id: serverMessages. so this is how you can create a simple chat application. als we have created an input box and a button to send the message to the server using **<span style="background-color: darkblue;"> sendMessage() </span>**. so this is how you can create a simple chat application.
You can either use Liveserver in VsCode extension to run this or **<span style="background-color: darkblue;">npm i -g serve</span>** npm package to run this. because file path is not allowed for this HTML to connect to WebSocket. so to folder with index.html or above file make sure its the only file present and run serve.

### An Actual Chat Application 

![Chat Application](image-4.png)
So people in same room connect to let say connect ws server 1. people all in room 2 connect to ws server 2 and so on. each of those members gets all chats or data/info of that room.

#### Server Side Code

```javascript
import express from "express";
import http from "http";
import { WebSocketServer } from "ws";

const app = express();
const port = 3000;

const server = http.createServer(app);

const wss = new WebSocketServer({ server });

const users: { [key: string]: {
    room: string;
    ws: any;
} } = {};
{/*.. An oject of users with keys as string and value as object with 
property room as string and ws or websocket server.*/}

let counter = 0;

wss.on("connection", async (ws, req) => {
   {/*.. When a connection happens it will increase the wsId by 1 using counter.*/}
    const wsId = counter++;
    ws.on("message", (message: string) => {    {/*first step */}

        const data = JSON.parse(message.toString());   {/*second step */}
        {/*.. Now user will send an object(json string) so you need to parse
         it since it will still be text as you get it from a browser you need to parse to get a js object.
         message.toString() converts the message into a string. JSON.parse() takes this string and converts it into a JavaScript object which is then stored in the data constant.*/}

        if (data.type === "join") {   {/*third step*/}
            users[wsId] = {   {/*fourth step will set the id increased during conenction above for the new user */}
                room: data.payload.roomId,   {/*room id from the message*/}
                ws                            {/*websocket connection for that particular user*/}
            };
            {/*.. if the type of message is join then it will create a new user with wsId as key and value as an object with room as roomId and ws as websocket server.*/}
        }

        if (data.type === "message") {
            const roomId = users[wsId].room;
            const message = data.payload.message;

            Object.keys(users).forEach((wsId) => {
                if (users[wsId].room === roomId) {
                    users[wsId].ws.send(JSON.stringify({
                        type: "message",
                        payload: {
                            message
                        }
                    }));
                }else {
                    //
                }
            })
        }
    });
});

server.listen(port);

```
 * Websocket can only send a **<span style="background-color: darkblue;">string</span>**  or a **<span style="background-color: darkblue;">binary data</span>** hence they are parsed to convert into a Javascript object.
  <br/>
 * Types of message a user can send two types of message everything else will be ignored if not json and these types. do put try catch method to catch error if it is not a json string.
  <br/>

 ```javascript
   {
      type: "join",
      payload: {
          roomId: "room1"
      }
   }

   or

   {
      type: "message",
      payload: {
          message: "Hello"
      }
   }
 ``` 

 **<span style="background-color: red;">## VVIMP</span>** </br>
 We cache the user in a in-memory object, shouldn't this be stored in a database ? <br/>
  **<span style="background-color: darkblue;">answer:</span>** no because these are realtime systems, what if database goes down?. **<span style="background-color: darkblue;"> Then everyone on the client connects to a new backend server this is called**<span style="background-color: red;"> resilience </span>**</span>**.

 ```javascript
 
 const users: { [key: string]: {
    room: string;
    ws: any;
} } = {};

 ```
  resilience refers to the ability of the system to handle and recover from failures or disruptions in the connection. This is particularly important in real-time applications where continuous and reliable data exchange is crucial. <br/>

 ![when backend goes down](image-6.png)

 Users gets a message that disconnected from the server and everyone connects to different server as shown above now the new server has the users array all client resend the message that i want to join so and so room_id and redo everything from scratch on a new web-server this is how resilience works in WebSockets servers in companies at large scale.

 if you are going to iterate over an array it is simply. 
 ```javascript
 arr.forEach(x => console.log(x));
 ```
 But since Users is an in-memory Object, to iterate over an Object we use the inbuilt javascript function **<span style="background-color: darkblue;">Object.keys()</span>** which returns an array of keys of the object. so we can iterate over the array keys using foreach **<span style="background-color: darkblue;">forEach()</span>** function.

 ```javascript
 Object.keys(users).forEach((wsId))
 ```
 so arrays of keys of Users is returned by Object.keys and then foreach on keys which are wsId currently in this case and access individual user using users[wsId] and . then their properties. Object.keys() is a slower function though. so when server recieves message from a particular user we store the roomId and message in a variable.
 ```javascript
  const roomId = users[wsId].room;
  const message = data.payload.message;
 ```
now we will search for all users with same room id so will iterate in Users as shown above using Objects.keys() and if the room id of the users is same as roomId so individually send the message from payload to their websocket connection so all including the sender will get the message. so this is how you can create a simple chat application.

#### Client Side Code

```HTML
<html>
    <head>
        <title>Simple WS Client</title>
        <script>
            const webSocket = new WebSocket("ws://localhost:3000");
            function sendMessage() {
                webSocket.send(JSON.stringify({
                    type: "message",
                    payload: {
                        message: document.getElementById("inputbox").value
                    }
                }));
            }
            webSocket.onmessage = function (event) {
                const data = JSON.parse(event.data);
                if (data.type === "message") {
                    document.getElementById("serverMessages" ).appendChild(document.createTextNode(data.payload.message));
                    document.getElementById("serverMessages").appendChild(document.createElement("br"));
                }
            }
            {/*.. When the connection is open extract the roomid from the URL  through window.location.search get the roomid and send to backend*/}
            webSocket.onopen = () => {
                const urlParams = new URLSearchParams(window.location.search);
                const roomId = urlParams.get('roomId');  
                webSocket.send(JSON.stringify({    {/*Once you get the room ID you send the first message that of type join then as previous stores in an inmemory object user and creates a user with userId and a object with roomid and ws server*/}
                    type: 'join',
                    payload: {
                        roomId: roomId
                    }
                }));
            }

        </script>
    </head>
    <body>
        <input type="text" id="inputbox" />
        <button onclick="sendMessage()">send</button>
        <br/><br/>
        <h2>
            Events from server
        </h2>
        <div id="serverMessages">

        </div>
    </body>
</html>
```

first message automatically goes out with type: join and payload with roomid from the URL.
<br/><br/>
![first request](image-3.png)
<br/><br/>
Two clients connected to the same room with realtime chat application with roomId = 123 if a new user joins another room then this user will not get the message of the other room.
<br/><br/>
![chat](image-5.png)

* When we refresh the data is lost. how to fix it ? <br/>
  answer: So the first request can be a http request to a database to get the stored chat or previous chat info's hence the first request is an Http request. the next request will be a realtime chat with Websockets so initial request is http to get back previous chats.

  ### Large scale Distributed Computing

  Now let say a company has 20 thousand users and a constrain that users with same room id should be on the same websocket server but the problem is servers has a limit of 10 thousand users. so what to do ? and most of the time it is relayed by a third server to which the first http request is sent let say client wants to connect to roomId = 123 it is send from the url as params to the server and server responds ok join roomId = 123 and another user joins the same room but what if server crash then again the client calls the third server send a roomId as a params url through http the same process again so this is no efficent since there is a cap on number of users.<br/>

* ### Using a Load-balancer
  let say the client randomly connects to a websocket server (ws1) through a load balancer. if user 1 sends a message to room 1 and since websocket server is radomly allocated how will user 2 get the message sent by user1 since it is connected to different websocket server let say ws3. how will the message of user 1 get to user 2? so ws1 somehow has to forward the message to ws3 and ws3 has to send message or data to the user2.
<br/><br/>
![with loadbalancer](image-8.png)
<br/><br/>

```javascript
    users[wsId] = {
        room: data.payload.roomId,
        ws
    };
```

You can't store the ws server in a database since this is an object .**<span style="background-color: red;"> Wrong solution </span>**. and let the user2 get acess to the ws server and get data from database on where to connect because its a realtime connection you can't put it in a database and also they are in memory objects.<br/>

* ### Solution: Pub subs
* Disadvantage of using a pub subs or not putting on same server, **<span style="background-color: red;"> Latency increases </span>** 
<br/><br/>
![using pub sub](image-7.png)
<br/><br/>

Pub subs means publish and subscribe. so ws1 will publish the message(hi i am swaraj) to the pub sub and ws3 will subscribe to the pub sub and get the message. <br>
Then why do we even use websocket server ? why not just use pub sub ? <br>
1. Because first you shouldn't expose pub subs to the client they are meant to communicate with backend servers so that backends can talk to eachother.
2. You can't directly make browsers and pubsubs lie redis or kafka talk to each other.
<br/><br/>
![publish and subscribe](image-9.png)
<br/><br/>


#### Interview Question
### Stateless and Stateful Servers
What are stateless server ? <br/>

There are many http servers, they dont have inmemory object to store data they rely on databases to store data they are just authentication layer in the middle and if one goes down your client can connect to a different one this is called a stateless architecture. keep you architecture stateless since it is easy to scale this. there shouldn't be a constrain on user on connecting to a particular server only, you dont need to your connection to be sticky. Stateful servers are sticky so your http servers should always be sticky. But a Websocket server can't be stateless because websocket means a persistent connection here we iterate over users in that particular room they sought have to be sticky that means on same server(without pubsubs) to get the shared messages and also to keep a track of users a inmemory object if connection goes down they reconnect to same server. So to make websocket stateless we use pubsub or centralized space which can talk among websocket servers.another not so good approach will be any server when gets data it broadcast same to all the other servers.<br/>

What if redis/kafka or centralized space goes down ? <br/>
Then you can have resiliency so if one goes down you can connect to another one but it can be a signle point of failure if there is only one.<br/>

Code to run redis: docker run -p 6379:6379 redis:latest

#### Interview Question
## What is Redis ?

Redis gives out of the box a lot of distributed things.it can handle a lot of load on common use cases in distributed systems. like 
1. In-memory Storage
2. Pub/Subs
3. persistence
4. Message Queues
5. Cluster mode: multiple redis servers can talk to each other and also with resiliency. if one goes down other can take over.and distributs data among them. </br>
   
Goal is to get minimum latency instead of using a database.which takes time to read and write so redis provides it in-memory so read and writes are faster and also if redis goes down it tries best to recover data but there might be case where it gets stored in redis but didn't reach the file system and if we close it then data might get lost but most cases in production yes it  backups in file system and gurantees persistence.
Redis is an open-source, in-memory data structure store that can be used as a database, cache, and message broker

Usually used for caching data before the server for faster lookups.It is not used as a primary data source. it is used as a cache between the server and the database. If data is not present then we hit the database show it to client and store it in cache for future request. What is someone puts an update on database and redis cache has different value then? so whenever we insert into database first we remove it from the redis cache. So even though there might be different servers redis cache is only one. Sometime people do have individual cache for each server but then cache resets after sometime. redis can be used for advertisment.
</br>
</br>
![redis requests flow](image-10.png)
</br>
</br>

#### commands
1. docker run -p 6379:6379 redis
2. docker exec -it 3347db15f01e<containerId> /bin/bash
3. redis-cli
Redis by default gives you an client, you can have nodejs client and other client.
4. SET USER_METADATA_1 Swaraj
   USER_METADATA_1 is the key used for caching and Swaraj is the value. key is required by user to get their metadata.
5. GET USER_METADATA_1   
6. SET USER_METADATA_1 Swaraj EX 10
   EX 10 means expire in 10 seconds.afetr 10 seconds it will be deleted from redis cache and will give null and hit the database again.

</br>
</br>

![redis ssh](image-13.png)

</br>
</br>

#### In a Nodejs Process

```javascript
import express from 'express';
import { createClient } from "redis";

const app = express();
const PORT = 3000;

const redisClient = createClient();
redisClient.connect();
{/*.. We can put await here ideally to connect and wait for it */}

app.use(express.json());

app.get('/uncached', async (req, res) => {
    const data = await expensiveOperation();
    res.json(data);
});

app.get('/cached', async (req, res) => {
    const cachedData = await redisClient.get('data');
    if (cachedData) {
        return res.json(JSON.parse(cachedData));
    }
    const data = await expensiveOperation();
    await redisClient.set('data', JSON.stringify(data));

    res.json(data);
});

app.listen(PORT, () => {
    console.log(`Server started on http://localhost:${PORT}`);
});

async function expensiveOperation() {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    return {
        username: "swaraj",
        email: "swaraj@gmail.com"
    }
}

```

const redisClient = createClient(); in here we can pass options like global redis server if nothing then it takes
localhost 6379

It has both cached and uncached(everytiem going to database) enpoint, expensive operation would be a database call.

#### In cached

```javascript
app.get('/cached', async (req, res) => {
    const cachedData = await redisClient.get('data');
    {/*.. if data is present in redis cache then return it 
    along with that you hace to send a key like user_id or some indetifier */}
    if (cachedData) {  // if cached data found
        return res.json(JSON.parse(cachedData));
    }
    //else if cached data not found then hit the database
    const data = await expensiveOperation();
    // set the data in redis cache
    await redisClient.set('data', JSON.stringify(data));
    // send the data to the client
    res.json(data);
});
```

A key would be function of what inputs you are hitting the databse with.
for example userr_id to get all data of that user.
To check how many user joined between certain period of time then key would time period will be a part of cache key. else another request comes it will go to database.
Key would be complex.So cached enpoint will be slow for the first time but after that it will be fast compared to uncached enpoint.
</br></br>

![uncached endpoint](image-12.png)

It takes around 1.01 seconds to get the data from database.

![cached enpoint](image-11.png)

it took around 0.14 seconds to get the data from redis cache.
</br>

To check if there is data in redis

#### command in CLI
1. GET data
127.0.0.1:6379> GET data
"{\"username\":\"swaraj\",\"email\":\"swaraj@gmail.com\"}"

#### Use Case:
1. Leetcode during a contest gets a lot of load at the same time so there are not multiple servers running and if they do it will be very expensive, and spinning off a new instance takes time so they use something called as asynchronous processing, they put the submissions in a queue and sends to limited number of workers they in their own time pick a problem from the queue , figure out a solution and let the backend know it and pick another problem from the queue. so submission will be missed since it is in a queue and also wont't overwhlem the server untill then client till it doesn't get a response we can show a loading screen.

2. Notifications sending Emails , here to we put million users in a queue and workers will send emails to them and keep on picking from the queue and send emails to them. it can retry sending emails if it fails. this is a persistent property of redis. so if a worker goes down it can pick up from where it left off. if we directly send emails from the server it will be slow and also if server goes down it will be lost event will be lost and we can't start where we left from and recover.

#### Popular ways to do Queues
1. RabbitMQ
2. Redis

Redis can be used for notifications. </br>
Redis lets you push with following commands:
</br></br>

![Alt text](image-15.png)

LPUSH 1: Left push into the queue </br>
LPUSH 2: Left push into the queue </br>
LPUSH 3: Left push into the queue </br>
RPOP: Right pop from the queue (removes the last element from the queue) here it is 1

Whenever you put LPUSH that is reverse order so to get the order you have to use RPOP. since it is a queue 
if we do LPOP it will behave like a stack and will remove the first element. which we dont want. in real world number would be JSON objects.

LPUSH problems_low_priority here key could be low_priority example that means let say user is a free user so we can send this to a queue which has 3 workers (that means a bit slow) or high_priority which has 10 workers (that means fast) so we can send this to a queue which has 10 workers. so we can have multiple queues for free or premium users (low latency for premium users and high latency for free users).

So Nodejs processes has to PUSH and POP and process if they miss any or server goes down they push it to another server which is a backup_queue and checks backup_queue if there is any data and process it or is there any corrupt data that failed to process.

#### Interview Question
Javascript is single threaded so how can we do multiple things at the same time ? <br/>
Yes javascript is single threaded but we can use multi cores using **<span style="background-color: darkblue;">Clusters</span>** module.
If we buy a ec3 instance with multiple cores we want to it to use all the cores. but javascript is single threaded so what we can do is to use Clusters module.that means a single node js command can run multiple processes.
through node js we can spawn a go process or a python process or a java process and communicate with it. so we can use multiple cores of a machine.
and similarly when spawn a node js process that is we can distribute the load among multiple node js processes. so we can use all the cores of a machine.
**<span style="background-color: darkblue;">Also It understands eventhough there are multiple cores all run on a single port so if someone hits a server and there are multiple processes running in a distributed fashion they will point to the same port or server or localhost for example</span>** . there is no port conflict unlike if we do normally. we get (port already in use). It takes it as a child process and parent routes it to same port hence we get multiple machines.

cluster.isMaster is the first process that starts. we run a forloop to run number of cpus like count = os.cpus().length - 1;. cluster.fork to fork a new process. Master cluster is the parent the first process rest others are child processes. and master cluster runs spwan to spawn a new process. and we can communicate between master and child process.

So on a 8 core machine there could be 20 processes using Interleaving in OS. that goes back and forth between processes. so if one process is blocked it goes to another process, just like that it runs multiple processes.

#### Interview Question

* ### Sharding
![Alt text](image-16.png)

In the above process scaling on the left side that is websocket servers, this can be done easily by increasing the number of websocket servers when load increases. but there is only one pub/sub so there is a chance it gets overwhelm in such a case we need to perform sharding. </br>
We can create a fleet of pub/subs and it has it sets of websocket servers like 50 and half the load of people let say. sharding is basically cutting down and making independent parts that works on their own. in such a case if anything wrong happens only 50 users will get affected not all (lesser blast radius). Load on pub/subs is reduced. but this will work unless there is only a single room which has a lot of users because all of them has to go to a particular fleet and then that fleet pub/sub may get overwhlem and then blast radius.if there are not much user 10^5, 10^6 then sharding can be done.

### Publishing and Subscribing in cli through redis
![Alt text](image-18.png)
</br></br>

In the above messsage we are publishing to user_messages_room1 from one terminal and sending a message while in another terminal we are subscribing to user_messages_room1 and getting the message it is reading messages and waiting for a message to come. so this is how pub/sub works. we can add multiple subscribers and publisher as long as the room is same it will reached by all in that room.
</br>

![Alt text](image-17.png)
</br>

So there could be different web servers which are subscribing to the same room and they will get the message. so this is how pub/sub works.
