# Realtime-communications-and-distributed-computing
Realtime communications and distributed computing  ,System design of a distributed chat system WebSockets
## System design of a distributed chat system

#### Why WebSockets?

1. Http is great, but doesn't cover a few usercases. <br>
2. The most important use case it doesn't cover is server side events. <br>
3. What if you want to create a realtime application? like
   zerodha or Binance<br>
   
In normal https, server can't directly send you events. client ask the server and server responds, what if your application is realtime. let say you have a webapp and a mobile app. changes in one should be seen imediately in the other. like notion or let say google docs, if you make changes in one file in google docs in one tab and open the same file in another pc it will show the changes in realtime.
#### So to create a realtime application we need a protocol that gives a **<span style="background-color: darkblue;">Persistent</span>**  connection between client and server. and that is websockets. A continuous connection will let the client constantly know the changes in the server using TCP. 
Let say two users are chatting with each other. one user send a message to the other using a htpps request. here you can  say the client does polling after let say every 1 seconds. this is called **<span style="background-color: darkblue;">long polling.</span>** but this is not **<span style="background-color: darkblue">efficient .</span>** because the client is constantly asking the server if there is any new message. Another is **<span style="background-color: darkblue;">Server Side Events</span>** that is SSE. It sends events from server to client. but it is not **<span style="background-color: darkblue;">bidirectional .</span>** So we need a protocol that is bidirectional and that is websockets. Under the hood in http there is handshake between client and server. it works on transport layer and creates a **<span style="background-color: darkblue;">TCP</span>** connection and after the request is completed it **<span style="background-color: darkred;">closes</span>** the TCP connection it is not persistent connection.

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
WebSockets is a communication protocol that provides a **<span style="background-color: darkblue;">full-duplex communication</span>**  channels over a single, long lived connection. they are not shortlived.

