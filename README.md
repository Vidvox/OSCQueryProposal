#OSC Query Proposal

###BACKGROUND
 * Here are some existing / proposed specs for OSC querying:  
[Schmeder/Wright proposed spec](http://opensoundcontrol.org/files/osc-query-system.pdf)  
[OSNIP](https://github.com/jamoma/osnip/wiki)  
[Minuit](https://github.com/Minuit/minuit)  
[QLAB](http://figure53.com/qlab/docs/osc-api/)  
 * Some definitions copied from the [OSC spec](http://opensoundcontrol.org/spec-1_0):  
  * An **OSC server** has a set of OSC Methods.  
  * **OSC methods** are the potential destinations of OSC messages received by the OSC server and correspond to each of the points of control that the application makes available.  "Invoking" an OSC method is analgous to a procedure call; it means supplying the method with arguments and causing the method's effect to take place.  
  * An OSC server's OSC Methods are arranged in a tree structure called an **OSC Address Space**.  The leaves of this tree are the **OSC Methods** and the branch nodes are called **OSC Containers**.

###GOAL
To create a minimal query protocol for OSC that allows a client to browse and interact with a remote server's OSC address space.  The protocol should be capable of both describing the address space's structure/layout as well as the types of messages that destinations in the OSC address space are capable of both sending and receiving.  The protocol should be easily human-readable to facilitate debugging, and should have a standard set of errors to describe commonly-encountered problems.

The intent of this goal is to provide baseline functionality that other developers may take advantage of to construct impromptu or improvisational interfaces for dynamic environments.

###PROPOSAL
 * An **OSC Query** is an OSC message with zero values whose OSC address path is the address of the node being queried appended by a number sign and question mark ("#?") and the name of the query.  Examples:
  * "/foo/bar#?INFO"  this address will dispatch the "INFO" query to the OSC destination "/foo/bar"
  * "/foo/bar#?CONTENTS"  dispatch the "CONTENTS" query to the OSC destination "/foo/bar"
 * All OSC destinations (both methods and containers) respond to a series of queries.  These queries allow the address space to be explored and describe how to interact with the OSC methods.  The possible defined queries are:
  * **INFO**    Returns an OSC message with a single OSC string containing a human-readable description of this container/method
  * **CONTENTS**    Returns an OSC message with two arrays.  The first array contains OSC strings with the names of the OSC containers (branches) inside this node, the second array contains OSC strings with the names of the OSC methods (leaves) inside this node.
  * **ACCESS**    Returns an integer that represents a binary mask.  Returns 0 if there is no value associated with this OSC destination, 1 if the value may only be retrieved, 2 if the value may only be set, or 3 if the value may be both retrieved and set
  * **TYPE**    Returns an OSC message with a single OSC string that contains an OSC type tag string.  If you query the value of this OSC destination, your reply will have this type tag string.  Correspondingly, if you send an OSC message to this destination, the message is required to have this type tag string.  It is presumed that an OSC method will only have a single type of value (an OSC method with the declared type "f" should not be expected to respond to OSC messages with integer values).  If an OSC method doesn't have a value and doesn't require a value to be sent to it, the "TYPE" query should return an OSC "nil" value.
  * **VAL**    Returns an OSC message with the current value of this OSC destination.  The OSC type tag of this message will be the OSC string returned by the "TYPE" query.
  * **RANGE**    Returns an OSC message with a number of arrays- there should be one array for each value it returns.  In other words, there should be one array for each value in the OSC type tag string returned by the TYPE query.
    * The first item in each array is the minimum value of the range (or the OSC value "nil" if there's no minimum or the value isn't ranged)
    * The second item in the array is the maximum value of the range (or the OSC value "nil" if there's no max range or the value isn't ranged).
    * If the value has a minimum or maximum range, the third value in the array should be "nil" (likely the most common occurrence).  If the value doesn't have a range- if the value is expressed as an item chosen from a list of possible values- the third value in the array should be another array containing the list of available choices.
 * If the query type is unrecognized (for example, "/foo/bar#?GABBAGABBAHEY"), the server should return an error of type 400 (bad request/syntax error) so the client knows that this query isn't supported.  This allows specific applications to develop more specialized query systems without causing significant conflicts with other software that supports this basic query protocol.
 * The response to an OSC query is either a **REPLY** (if the query was successful) or an **ERROR** (if something went wrong).
  * **REPLIES**    If the response is a reply, the query executed successfully and the server needs to send some data back to the client which issued the query.  This response is an OSC message, and the address of this message contains the address of the query as well as the query itself, separated by a number sign and a less-than sign("#<").  The values in this message depend on the reply, and it's assumed that the client will be able to locate the original query from the reply's address (which contains the address and type of the original query).
  * **ERRORS**    If the response is an error, there was a problem with the query and the client needs to be notified.  Like replies, errors are just OSC messages, and the address of the message contains the address of the query as well as the query itself- unlike replies, these are separated by a number sign and an exclamation point ("#!").  Also unlike replies, error messages will always have exactly one value, and that value will either be an integer or a string: if it's an integer the value describes one of the defined error values listed below, if it's a string the error is user-defined somehow and the string describes it in greater detail.  the defined errors are:
    * **204**  (no content- server received request, but the request is inappropriate in some way- querying VAL or RANGE when ACCESS returned 0, for example)
    * **400**  (bad request/syntax error- corrupt message/packet, unrecognized query, etc)
    * **404**  (no container or method was found at the supplied OSC address)
    * **406**  (not acceptable- the OSC message type doesn't match the expected type tag string)
    * **408**  (request time-out- this error should be generated by the client if it doesn't hear back from the server in a specified time period)

###EXAMPLES
In the below examples, messages from client -> server (queries) are denoted with "->", the reverse indicates a message from server to client (replies/errors).  The message formatting is as follows:

Direction | Message Address | Message Type Tag String | : | Message values: space-delineated, strings in quotes
--- | --- | --- | --- | ---
-> | /foo/bar | ifs | : | 1 2.0 "three"
-> | /foo/bar2#?VAL |   |   |   
<- | /foo/bar2#<VAL | i | : | 1
-> | /foo/bar3#?VAL |   |   |   
<- | /foo/bar3#!VAL | i | : | 404

 Exploring the OSC address space (the reply indicates one container and several methods)  
~~~
->  /foo/bar#?CONTENTS
<-  /foo/bar#<CONTENTS [s][ssss]:"containerNameA" "methodName1" "methodName2" "methodName3" "methodName4";
~~~
 OSC method with inaccessible values
~~~
->  /foo/bar/methodName#?ACCESS
<-  /foo/bar/methodName#<ACCESS  i : 0
~~~
 OSC method that is write-only
~~~
->  /foo/bar/methodName#?ACCESS
<-  /foo/bar/methodName#<ACCESS  i : 2
->  /foo/bar/methodName#?VAL
<-  /foo/bar/methodName#!VAL  i : 204
~~~
 OSC method that is read-only
~~~
->  /foo/bar/methodName#?ACCESS
<-  /foo/bar/methodName#<ACCESS  i : 1
->  /foo/bar/methodName  f : 0.5
<-  <the server might not respond at all>, or /foo/bar/methodName#!  i : 204
~~~
 OSC methods with fully read/writable values
~~~
->  /foo/bar/methodName#?ACCESS
<-  /foo/bar/methodName#<ACCESS  i : 3
~~~
 OSC Method that returns/expects a single float value, ranged 0.0-1.0
~~~
->  /foo/bar/methodName#?TYPE
<-  /foo/bar/methodName#<TYPE  s : "f"
->  /foo/bar/methodName#?RANGE
<-  /foo/bar/methodName#<RANGE  [ffN] : 0.0 1.0
->  /foo/bar/methodName#?VAL
<-  /foo/bar/methodName#<VAL  f : 1.0
->  /foo/bar/methodName  f : 0.5
~~~
 OSC Method that returns/expects two float values (not an array, just two float values in the OSC message), ranged 0.0-1.0
~~~
->  /foo/bar/methodName#?TYPE
<-  /foo/bar/methodName#<TYPE  s : "ff"
->  /foo/bar/methodName#?RANGE
<-  /foo/bar/methodName#<RANGE  [ffN][ffN] : 0.0 1.0 0.0 1.0
->  /foo/bar/methodName#?VAL
<-  /foo/bar/methodName#<VAL  ff : 1.0 1.0
->  /foo/bar/methodName  ff : 0.5 0.5
~~~
 OSC Method that returns/expects two float values ranged 0.0-1.0 as an array
~~~
->  /foo/bar/methodName#?TYPE
<-  /foo/bar/methodName#<TYPE  s : "[ff]"
->  /foo/bar/methodName#?RANGE
<-  /foo/bar/methodName#<RANGE  [ffN][ffN] : 0.0 1.0 0.0 1.0
->  /foo/bar/methodName#?VAL
<-  /foo/bar/methodName#<VAL  [ff] : 1.0 1.0
->  /foo/bar/methodName  [ff] : 0.5 0.5
~~~
 OSC method that returns/expects an OSC string (any OSC string):
~~~
->  /foo/bar/methodName#?TYPE
<-  /foo/bar/methodName#<TYPE  s : "s"
->  /foo/bar/methodName#?RANGE
<-  /foo/bar/methodName#<RANGE [NNN]:
->  /foo/bar/methodName#?VAL
<-  /foo/bar/methodName#<VAL  s : "default string"
->  /foo/bar/methodName s : "newString"
->  /foo/bar/methodName#?VAL
<-  /foo/bar/methodName#<VAL  s : "newString"
~~~
 OSC method that returns/expects an OSC string from a list of possible choices (the strings "one", "two", and "three")
~~~
->  /foo/bar/methodName#?TYPE
<-  /foo/bar/methodName#<TYPE  s : "s"
->  /foo/bar/methodName#?RANGE
<-  /foo/bar/methodName#<RANGE  [NN[sss]] : "one" "two" "three"
->  /foo/bar/methodName#?VAL
<-  /foo/bar/methodName#<VAL  s : "one"
->  /foo/bar/methodName  s : "two"
->  /foo/bar/methodName#?VAL
<-  /foo/bar/methodName#<VAL  s : "two"
~~~
 OSC method that doesn't have a value, doesn't require a value, and will respond to any method sent to it
~~~
->  /foo/bar/methodName#?TYPE
<-  /foo/bar/methodName#<TYPE  N :
~~~


****
****


###PROBLEMS / QUESTIONS / DISCUSSION POINTS

have been split up and moved to the "Issues" section of GitHub, where they can be discussed comfortably.  If you'd like to discuss a change or propose an addition, please open an issue in the GitHub interface!