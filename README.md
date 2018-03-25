# iocp_tcp_server
A single threaded Windows IOCP TCP echo server.

## How it works?
We wait for events and when events occur we take of them one by one. 

## What events are we waiting for?

 1. Connection has been accepted event.
 2. Data has been read from the socket event.
 3. Data has been wrote to the socket event.

## How it doesn't work 

 - We keep an array of listening and connected sockets
 - Pass this array to a system function which will notify us whenever some events on these sockets occur. Ex - New connection to be accepted, data to be read etc.
 - When a new connection event occurs we accept the connection and add it to our socket list.
 - When there is data to be read we do that and write something back to the socket.

## How this really works
We get notifications when an event is completed rather than when it occurs. 
 - We do not get a notification when there is a new connection to be accepted, but after a connection has been accepted.
 - We do not get a notification when there is data to be read, but when we have finished reading the data to a buffer.
 We can do this with windows events, but we can wait for events only from 64 sockets. So we use IO completion ports which will notify us when a task is completed and has no limits.

## So does the kernel automatically accept connections and reads data from it?
Not really.  We call the accept and recv functions ourselves. But the functions doesnt block and returns immediately. Notification arrives at the completion port when the task has been completed.

## So, how to you put this all together?

 - We call the accept function (AcceptEx) at the beginning, and start waiting for completions in an infinite loop.
 - When a connection arrives and it has been accepted, our waiting function returns and our wait stops. The waiting function blocks until any completion occurs.
 - We add the accepted socket to our completion port
 - We call recv (WSARecv) function on this socket which will notify us whenever data has been read. (AcceptEx function can read initial data along with accepting the connection)
 - We call the accept (AcceptEx) function again
 - The loop continues.

## Further

 - We can wait for completion on any file descriptor, not only sockets. 
 - Can create a worker thread pool for any other IO operations like reading and writing to file or database. 
 - Can keep connections alive and not close it after data has been written to it

I read that Node.js uses IOCP in windows for it's event loop. So why not use it directly
