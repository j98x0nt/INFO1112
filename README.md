java c
INFO1112 - Assignment 2 - Tic-Tac-ToeTic-Tac-Toe, also known as Noughts and Crosses, is a simple yet engaging two-player game played on a 3x3 board.  The game is typically played by marking the board spaces with  ‘X’ and ‘O’, with players taking turns. The objective of the game is to be the first to align three of your marks—either horizontally, vertically, or diagonally. More about it on Wikipedia.
Since you have found it quite boring to play alone, you came up with the amazing idea of building a server that allows people to connect and play with you online. The server has the following features:
•  Users can log in
•  Users can create game rooms
•  Users can join existing game rooms either as players or as viewers
•  Users can view players’ moves as they are played
The specifications are divided into 3 main sections, which dictate:
•  The protocol used to communicate between the client and the server for running a game.
•  Server-specific runtime details and implementation
•  Client-specific runtime details and implementation
This assignment is due on Sunday, 20 Oct 2024 at 23:59, Sydney local time.
1    Getting StartedTo assist you in the implementation of the assignment, the game logic and input handling for tic-tac-toe has been implemented in 2 files in the scaffold:  game.py, and tictactoe .py. Your task is to extend the logic contained in these files, and create:
•  a server program which is responsible for handling many games of tic-tac-toe being run simultane- ously, and,
•  a client  program, which interacts with a game server (like the one you implement), and allows an end-user to play tic-tac-toe over a network connection.
You are encouraged to run the function tic tac toe in game .py to play the game and understand how it works. Of course, you are free to modify these files as much as you wish for your submission.
2    Sequence DiagramsThroughout these specifications, we will be using simple sequence diagrams  in order to visually depict how protocol messages are sent back and forth between the client and the server.  The shapes below outline the sequence diagrams used in the assignment specifications:
•  Messages from source to recipient are represented as a solid arrow with a solid arrow head. These will be labelled with the protocol message being sent:
EXAMPLE:MESSAGE
−−−−−−−−−−−−−−−−−−−−−−−−−−−−→
•  Return messages from recipient to source are represented as a dashed arrow with a point arrow- head. This is for cases where a recipient is expected to send a message in response back to the source.
They will also be labelled with the protocol message being sent:
EXAMPLE:MESSAGE:RESPONSE
←−−−−−−−−−−−−−−−−−−−−
•  Participants, which in this case, will either be a client or a server, can send and receive messages. These will be depicted as rectangles with an associated lifeline – a dashed vertical line representing the lifetime of the participant:
Participant
3    Protocol
In order to support playing the game over the network, your programs will need to implement a custom application-layer protocol as described here in order to facilitate client and server interactions.All communications will use TCP, and you should use sockets  to communicate over the network.  More details about sockets are available from the documentation of Python’s socket module.  A how-to guide for getting started with using TCP sockets is available on Python’s website, and some material will be included in the lab content to help explain how to create and interact with TCP sockets.
A couple of general notes about the protocol:
•  Like many application-layer protocols (e.g. HTTP, SMTP), the protocol is text based, and specifically, uses ASCII encoding for all data.
• When sending data over the network, you will need to ensure you  encode  data into bytes to send it over a network socket, and when receiving data, you will need to ensure you decode the received data to interpret this as a string.
–  Hint 1: use the str.encode method to encode a string into bytes with a specified encoding.
–  Hint 2: use the bytes.decode method to decode bytes to a string.
• You may assume that no message will ever exceed a buffer size of 8192 bytes.
•  The majority of protocol messages require the user to be authenticated  (after a  LOGIN message) in order for a client to be able to successfully interact with the server.  If a category of messages (one of the subheadings below) requires authentication, the subheading will have (authentication required) written next to it to indicate this.
–  Attempting to send a message from a client requiring authentication without being logged in will result in a BADAUTH message being sent from the server to the client in response to the sent message (see below).
3.1    Authentication-related Messages
3.1.1    LOGIN::
This message is sent from the client to the server, and is used to authenticate a user account in order to be able to play and/or watch games hosted by the server.
The client will send over the username of the user who is attempting to authenticate themselves, and a plaintext password.When the server receives this message, it will inspect the user database to check if the user with the given  exists, and if so, checks that the sent  matches the password hash of the user in the user database, which has been encrypted with the bcrypt algorithm (see Allowed Modules for installing the bcrypt Python library on your local device).
•  If the username exists in the user database, and the password provided matches the password hash in the user database, the user will be considered to be successfully authenticated, meaning:
– the server should associate the client socket object of the user which sent the message with the user account as actively authenticated, meaning the user has permission to interact with all messages requiring authentication.
*  multiple clients logging in to the same user account will not be assessed – you are free to handle this how you see fit.
– the server should respond to the client with a LOGIN:ACKSTATUS:0 message, indicating a suc- cessful authentication.
– the client should print the message Welcome   to stdout after having received the above message from the server.
• If the username cannot be found in the user database:
–  the server should respond to the client with a LOGIN:ACKSTATUS:1 message.
– the client should print Error:    User    not  found message to stderr after having received the message above
• If the username was found in the database, but the password sent did not match the password hash in the database:
–  the server should respond to the client with a LOGIN:ACKSTATUS:2 message.
– the client should print Error:    Wrong  password  for  user   to stderr after hav- ing received the above message from the server.
•  If a LOGIN message was sent with an invalid format, i.e.  0 or 1 arguments:
–  the server should respond to the client with a LOGIN:ACKSTATUS:3 message.
–  NOTE: Since your client program should be correct, it should never send an invalid LOGIN message like this, and so there is no error message the client should print in this case.  However, this error message is included to help ensure robustness for the server handling such messages, such as if you were to use a program such as netcat/nc to directly send messages to the server.
Sequence Diagram

3.1.2    REGISTER::
This message is sent from the client to the server, and is used to create a new user account in order to be able to play and view games.
The client will send over the username and password of the user who is attempting to register themselves.
When the server receives this message, it will inspect the user database to ensure the user with the given  exists does not exist, and will then perform. an appropriate action described below:•  If the username does not  exist in the user database, the user may be successfully created, meaning:
– the server should create a new user record in the user database, containing:
* the username of the new user.
*  a password  hash for the password for the new user  account,  hashed  using  the  bcrypt algorithm.
– once the user record is created, the server is required to immediately write to and update the user database file with the new information for the user.
* you should either open and close the file for writing to do this, or altenatively, call the flush method to immediately write the contents of the file to the disk – either is acceptable, as long as no data is lost.
–  the server should respond to the client with a REGISTER:ACKSTATUS:0 message, indicating a successful user account creation.
– the client should print the message  Successfully  created  user  account   to stdout after having received the above message from the server.
–  NOTE: the user will not be logged in immediately after registration – the client needs to send a separate LOGIN message to authenticate the user.
• If the username already exists in the user database:
– the server should respond to the client with a REGISTER:ACKSTATUS:1 message, indicating the user already exists in the user database.
– the client should print the message Error:   User    already  exists to  stderr after having received the above message from the server.
•  If a REGISTER message was sent with an invalid format, i.e.  0 or 1 arguments:
–  the server should respond to the client with a REGISTER:ACKSTATUS:2 message.
–  NOTE: Since your client program should be correct, it should never send an invalid REGISTER message like this, and so there is no error message the client should print in this case.  However, this error message is included to help ensure robustness for the server handling such messages, such as if you were to use a program such as netcat/nc to directly send messages to the server.
Sequence Diagram
3.1.3    BADAUTH
This is a special message which will be sent from the server  to the  client  in  response to  any  protocol message which requires authentication to perform. – so make sure you check for this message when sending



an authenticated message from the client!
When  a client receives this message,  it should output  Error:    You must  be  logged  in  to  perform. this  action to stderr.
Sequence Diagram

3.2    Room-related Messages (authentication required)
3.2.1    ROOMLIST:
This message is sent from the client to the server, and is used to list rooms that are available to be joined in the specified .
 may be either:
•  PLAYER – indicating rooms available to join as a player (and be able to play the game against an opponent).
• VIEWER – indicating rooms available to join as a viewer (and be able to watch a given game).When the server receives this message, if  is formatted correctly, the server will respond with the message ROOMLIST:ACKSTATUS:0:,  where   is a list of comma-separated room names that are available. For instance, if the available room names are:
• Room  1
•  epic  room  2
•  another  epic  room
 will be formatted as Room  1,epic  room  2,another  epic  room, and thus, the acknowledge- ment message sent from the server back to the client in full will be:
ROOMLIST:ACKSTATUS:0:Room  1,epic  room  2,another  epic  room
Upon receiving the server feedback, the client will output ”Room  available  to  join  as  :    ” which in the example will be ”Room  available  to  join  as  :    Room  1,epic  room
2,another  epic  room”
Error handling
•  If a ROOMLIST message was sent with an invalid format, such as hav代 写INFO1112 - Assignment 2 - Tic-Tac-ToePython
代做程序编程语言ing more/less than 1 argument, or  not being PLAYER or VIEWER:
–  the server should respond to the client with a ROOMLIST:ACKSTATUS:1 message.
– the client should rasie the error to stderr”Error:    Please  input  a  valid  mode .”
Sequence Diagram
3.2.2    CREATE:
This message is sent from the client  to the server, and is used to create a room with a specified .
Room names must meet the following criteria:
• they may only contain alphanumeric characters (a-z, A-Z, 0-9), dashes, spaces, and underscores.
• they must be a maximum of 20 characters in length. which is validated on the server side.
The server also only allows a maximum of 256 rooms to be created.  Once a game is complete, the room is deleted.
When the server receives this message, it will perform. an appropriate action described below:
•  If the room named  does  not  already exist,  and  the    is valid  (from the above criteria):
– the user that created the room will automatically join the room
–  the server should respond to the client with a CREATE:ACKSTATUS:0 message.
– the  client should print the message  Successfully  created  room    to  stdout after having received the above message from the server.
–  And the client end will wait until there’s another player joined the room.  During the wait, the client isn’t supposed to do anything else other than printing ”Waiting  for  other player . . .” .
*  Once the client end of the first player in teh room received the BEGIN messsage mentioned below, the game will start.
• If the room name character is invalid,
–  the server should respond to the client with a CREATE:ACKSTATUS:1 message.
– the client should print the message Error:    Room    is  invalid to stderr after having received the above message from the server.
•  If the  received already refers to a created room:
–  the server should respond to the client with a CREATE:ACKSTATUS:2 message.
– the client should print the message Error:    Room    already  exists to stderr after having received the above message from the server.
• If there are already 256 rooms created in the server (meaning no further rooms can be created):
–  the server should respond to the client with a CREATE:ACKSTATUS:3 message.
– the client should print the message Error:    Server  already  contains  a  maximum  of  256 rooms to stderr after having received the above message from the server.
•  If a CREATE message was sent with an invalid format, i.e. 0 or 1 arguments:
–  the server should respond to the client with a CREATE:ACKSTATUS:4 message.
–  NOTE: Since your client program should be correct, it should never send an invalid CREATE message like this, and so there is no error message the client should print in this case.  However, this error message is included to help ensure robustness for the server handling such messages, such as if you were to use a program such as netcat/nc to directly send messages to the server.
Sequence Diagram

3.2.3    JOIN::
This message is sent from the client to the server, and is used to join a room with a specified  in the specified .
 may be either:
•  PLAYER – indicating rooms available to join as a player (and be able to play the game against an opponent).
–  NOTE: There may only be 2 players in 1 given room.  It is invalid for any further players to try to join a room as a player in this case.
• VIEWER – indicating rooms available to join as a viewer (and be able to watch a given game). When the server receives this message, it will perform. an appropriate action described below:
•  If the room named  exists,  and the  provided is a valid mode, and the user can successfully join the room (from the above criteria):
– the server should add the user into the room in the mode specified by the message sent by the client.
– the server should respond to the client with a JOIN:ACKSTATUS:0 message.
– the client should print the message Successfully  joined  room    as  a   to stdout after having received the above message from the server, where mode> is either player or viewer, based on what was sent.
•  If there is no room named  in the server:
– the server should respond to the client with a JOIN:ACKSTATUS:1 message.
– the client should print the message Error:    No  room  named   to sderr after hav- ing received the above message from the server.
•  If the player is attempting to join  as a PLAYER, but the room is already full  (has 2 players):
– the server should respond to the client with a JOIN:ACKSTATUS:2 message.
– the client should print the message Error:    The  room    already  has  2  players to stderr after having received the above message from the server.
•  If a JOIN message was sent with an invalid format, such as having more/less than 2 arguments, or  not being PLAYER or VIEWER:
– the server should respond to the client with a JOIN:ACKSTATUS:3 message.
–  NOTE: Since your client program should be correct,  it should never send an invalid  JOIN message like this, and so there is no error message the client should print in this case.  However, this error message is included to help ensure robustness for the server handling such messages, such as if you were to use a program such as netcat/nc to directly send messages to the server.
Sequence Diagram

3.3    Game-related messages (authentication required)
These messages describe interactions between an active game of tic-tac-toe between 2 players.
For any messages which a client sends to the server, the client is required to be authenticated, otherwise, a BADAUTH message is sent in response by the server.For these messages, both client and server programs may assume that messages will always be sent and received in a valid format, hence there is no need to check that messages constructed are of a valid format in for the below messages.
If, however, a client sends a message specified in this category, but they are not currently in a room (as
either a player or a viewer), the server should respond with a NOROOM message (see below).
3.3.1    BEGIN::
This message is sent from the server to a client, used to inform. clients about the players which will be versing each other when commencing a game of tic-tac-toe.
 is the username of the player who places the first marker  (an ’X’), while  is the username of the player who places the second marker (an ’‘’O’).
When a second player joins a room, the server will send this message to all players and viewers in the current room, informing them that a game is to begin:
• the client who is logged in as player  will be prompted to begin the game and place their first marker.
• the client who is logged in as player  will be prompted to wait until  has placed their first marker (which means that they need to wait until a BOARDSTATUS message has been sent by the server (see below)).
•  any room member’s client end should output the message ”match between  and  will commence, it is currently ’s turn. ’”
This message does not require the client to send anything in response.
Sequence Diagram


3.3.2    INPROGRESS::
This message is sent from the server to a client who joins as a viewer to an in-progress  game, informing them:
• the username of the player who’s turn it currently is ().
• the username of the opposing player who is awaiting their turn ().
When the viewing client receives this message, it should output the message ”Match between  and  is currently in progress, it is ’s turn”
This message does not require the client to send anything in response and players’ client ends are not supposed to receive this message.
Sequence Diagram

3.3.3    BOARDSTATUS:
This message is sent from the server to a client to inform. them of the current status of the game board after a move has been made by a player.
Every player and viewer in a room must receive this message after a move has been made.  explainer
 is a 9-character text string which represents the current state of the 3x3 tic-tac-toe board. There are 3 characters used to represent a space on the tic-tac-toe board:
•  0 – empty space
•  1 – ’X’ marker
•  2 – ’O’ marker
The string is essentially a 1D mapping of the 2D array for tic-tac-toe:  i.e.  each space in the tic-tac-toe board numbered like so:

is mapped to the  string 123456789. For instance, a board of the current status:

would have  equal to 012010020.
Client actions
Any time a client receives this message, they should print the board’s current status to stdout.
Depending on whether the client in the game is a player or viewer, other specific actions will be performed:
•  if the client is a player, and:
–  has just placed their own marker (having just sent PLACE:: to the server), the client will recognise that it is the opposing player’s turn (who’s username is known from the BEGIN message sent when the game has started), and will output that ”It  is  the  opposing  player’s  turn”, and wait for the opposing player to complete their turn.
–  has been waiting for an opposing player – if this message is received by the player, it means that the opposing player has placed their marker, and hence, the client should output ”‘It is the current player’s turn’”, and ask the user to input the x-y coordinates of the marker they
wish to place.
•  if the client is a viewer, after having printed the board to stdout, the client should print that it is the next corresponding player’s turn (the client will have the names of all players from either a BEGIN or INPROGRESS message sent prior).
This message does not require the client to send anything in response.
Sequence Diagram

3.3.4    PLACE::
This message is sent from the client to the server, and is used by a player to place a marker on the board when it is the player’s current turn.
 and  refer to the coordinates on where the marker should be placed on the game board, using 0-based indexing for the spot.The server makes the assumption that the client sending this message has already validated that  and  is valid on the client side  (i.e.  valid coordinates, not currently occupied, etc.),  and that the  PLACE message will always contain 2 arguments, and hence will simply process the request as is without needing to do any error handling.  (As a rationale for this, imagine that this protocol will only ever by used by our own program implementations)Once a client has sent a PLACE message to the server, the server will place the marker corresponding to the player appropriately, and send either a BOARDSTATUS message informing the player of the current board’s status as an acknowledgement, or a GAMEEND message (see below), indicating the game has finished.
Sequence Diagram

3.3.5    FORFEIT
This message is sent from the client to the server, and is used by a player when it is their current turn to indicate that they wish to forfeit the current game.
Once this message is sent, the game will end, and the server will send a GAMEEND message with a  of 2 (indicating a forfeitted win) for the opposing player (see below) to all players and viewers.
Sequence Diagram


         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
