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
 * An **OSC Query** is an OSC message with zero values whose OSC address path is the address of the node being queried appended by a number sign ("#") and the name of the query.  Examples:
  * "/foo/bar#INFO"  this address will dispatch the "INFO" query to the OSC destination "/foo/bar"
  * "/foo/bar#CONTENTS"  dispatch the "CONTENTS" query to the OSC destination "/foo/bar"
 * If the query type is unrecognized (for example, "/foo/bar#GABBAGABBAHEY"), the server should return an error of type 400 (bad request/syntax error) so the client knows that this query isn't supported.  This allows specific applications to develop more specialized query systems without causing significant conflicts with other software that supports this basic query protocol.
 * The response to an OSC query is either a **REPLY** (if the query was successful) or an **ERROR** (if something went wrong).
  * **REPLIES**    If the response is a reply, the query executed successfully and the server needs to send some data back to the client which issued the query.  This response is an OSC message, and the address of this message contains the address of the query as well as the query itself, separated by two number signs ("##").  The values in this message depend on the reply, and it's assumed that the client will be able to locate the original query from the reply's address (which contains the address and type of the original query).
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
-> | /foo/bar2#VAL |   |   |   
<- | /foo/bar2##VAL | i | : | 1
-> | /foo/bar3#VAL |   |   |   
<- | /foo/bar3#!VAL | i | : | 404

 Exploring the OSC address space (the reply indicates one container and several methods)  
~~~
->  /foo/bar#CONTENTS
<-  /foo/bar##CONTENTS [s][ssss]:"containerNameA" "methodName1" "methodName2" "methodName3" "methodName4";
~~~
 OSC method with inaccessible values
~~~
->  /foo/bar/methodName#ACCESS
<-  /foo/bar/methodName##ACCESS  i : 0
~~~
 OSC method that is write-only
~~~
->  /foo/bar/methodName#ACCESS
<-  /foo/bar/methodName##ACCESS  i : 2
->  /foo/bar/methodName#VAL
<-  /foo/bar/methodName#!VAL  i : 204
~~~
 OSC method that is read-only
~~~
->  /foo/bar/methodName#ACCESS
<-  /foo/bar/methodName##ACCESS  i : 1
->  /foo/bar/methodName  f : 0.5
<-  <the server might not respond at all>, or /foo/bar/methodName#!  i : 204
~~~
 OSC methods with fully read/writable values
~~~
->  /foo/bar/methodName#ACCESS
<-  /foo/bar/methodName##ACCESS  i : 3
~~~
 OSC Method that returns/expects a single float value, ranged 0.0-1.0
~~~
->  /foo/bar/methodName#TYPE
<-  /foo/bar/methodName##TYPE  s : "f"
->  /foo/bar/methodName#RANGE
<-  /foo/bar/methodName##RANGE  [ffN] : 0.0 1.0
->  /foo/bar/methodName#VAL
<-  /foo/bar/methodName##VAL  f : 1.0
->  /foo/bar/methodName  f : 0.5
~~~
 OSC Method that returns/expects two float values (not an array, just two float values in the OSC message), ranged 0.0-1.0
~~~
->  /foo/bar/methodName#TYPE
<-  /foo/bar/methodName##TYPE  s : "ff"
->  /foo/bar/methodName#RANGE
<-  /foo/bar/methodName##RANGE  [ffN][ffN] : 0.0 1.0 0.0 1.0
->  /foo/bar/methodName#VAL
<-  /foo/bar/methodName##VAL  ff : 1.0 1.0
->  /foo/bar/methodName  ff : 0.5 0.5
~~~
 OSC Method that returns/expects two float values ranged 0.0-1.0 as an array
~~~
->  /foo/bar/methodName#TYPE
<-  /foo/bar/methodName##TYPE  s : "[ff]"
->  /foo/bar/methodName#RANGE
<-  /foo/bar/methodName##RANGE  [ffN][ffN] : 0.0 1.0 0.0 1.0
->  /foo/bar/methodName#VAL
<-  /foo/bar/methodName##VAL  [ff] : 1.0 1.0
->  /foo/bar/methodName  [ff] : 0.5 0.5
~~~
 OSC method that returns/expects an OSC string (any OSC string):
~~~
->  /foo/bar/methodName#TYPE
<-  /foo/bar/methodName##TYPE  s : "s"
->  /foo/bar/methodName#RANGE
<-  /foo/bar/methodName##RANGE [NNN]:
->  /foo/bar/methodName#VAL
<-  /foo/bar/methodName##VAL  s : "default string"
->  /foo/bar/methodName s : "newString"
->  /foo/bar/methodName#VAL
<-  /foo/bar/methodName##VAL  s : "newString"
~~~
 OSC method that returns/expects an OSC string from a list of possible choices (the strings "one", "two", and "three")
~~~
->  /foo/bar/methodName#TYPE
<-  /foo/bar/methodName##TYPE  s : "s"
->  /foo/bar/methodName#RANGE
<-  /foo/bar/methodName##RANGE  [NN[sss]] : "one" "two" "three"
->  /foo/bar/methodName#VAL
<-  /foo/bar/methodName##VAL  s : "one"
->  /foo/bar/methodName  s : "two"
->  /foo/bar/methodName#VAL
<-  /foo/bar/methodName##VAL  s : "two"
~~~
 OSC method that doesn't have a value, doesn't require a value, and will respond to any method sent to it
~~~
->  /foo/bar/methodName#TYPE
<-  /foo/bar/methodName##TYPE  N :
~~~


****
****


###PROBLEMS / QUESTIONS / DISCUSSION POINTS
 * Encoding the query type in the OSC address path is convenient, but what character should be used to separate them?  A question mark ("?") seems like the obvious choice, but question marks are defined in the OSC spec as being wildcard characters for address paths, which could present a problem (particularly if the spec explicitly allows queries to be dispatched to wildcard addresses).  For now, I'm using the number sign ("#") as a separator, as that isn't defined as performing any special function in address paths (which is probably why Schmeder/Wright used it)
 * Should software supporting this protocol explicitly advertise its services via zeroconf?  If so, should it use the standard OSC label ("\_osc.\_udp") or should it use something else (perhaps "\_oscQuery.\_udp", or something like that?)
 * Explicit description of how pattern expansion is handled/whether wildcards are supported for queries might be nice.  On the one hand, wildcard dispatch for queries could be awesome: on the other hand, it could be a major pain point and source of confusion.  Are the benefits sufficient to justify the added complication and confusion?
 * Queries and replies/errors must be matched up on the client end (a given reply/error must be correctly reunited with the query which prompted it, presumably because the client software wants to do something with the reply/error).  At present, this is accomplished by associating the query address with the specific query type, which is echoed back as the address of the reply/error.  this assumes that these two pieces of information are sufficient to uniquely identify a query.  Is this sufficient for everybody's needs?  If not, perhaps appending an OSC timetag to the query (and echoing the timetag back with the reply) will work?  If we start including timetags to uniquely identify queries, perhaps the replies should just have the OSC address of "#reply" and echo back the timetag (to save bandwidth)?  On the other hand, this is significantly less readable, which is one of the goals of the spec...
 * To my mind, the main benefit of JSON support would be the addition of dictionaries to OSC (JSON refers to them as "objects": unordered string/value pairs).  OSC has a basic data type for arrays (the characters "[" and "]" are used in the type tag string of an OSC message to demarcate the values in the array)- perhaps we could use the characters "{" and "}" in the type tag string to contain a number of string+value pairs to be treated as an "object"?  This seems to be a sufficiently useful and general-purpose addition that it could certainly stand on its own as a potential addition to the base OSC spec...
 * If dicts/objects become part of the spec, perhaps we could add an "ATTRIBS" query that returns a single OSC message structured as a dict, containing the data returned by the "ACCESS", "TYPE", "RANGE", and "VAL" queries.  This would be a convenience: one query would return a dict with the above values, eliminating the need for a number of other queries.
 * I'm comfortable with adding JSON support to OSC, but i think that would best be accomplished by adding it as a new data type (rather than just returning JSON strings under the OSC type tag "s" or "S").  maybe type tag "J" for JSON strings?  I don't think JSON objects should be transmitted as OSC strings.
 * I noticed that the OSNIP protocol mentions something about a class-based structure.  I wasn't sure what this was referring to or how you envisioned it working, so this concept is completely absent from the above proposal.
 * The above proposal is based on the assumption that OSC methods will only have a single "TYPE" of value- in other words, each OSC method will only respond to OSC messages with a single, specific type tag string.  This is convenient and simple, but it might also be nice if OSC methods could respond to OSC messages with several different type tag strings (it might be nice if an OSC method could respond to either an "int" *or* a "float" message, for example).  The following alternate "TYPE" and "VAL" query definitions explore this notation- I'm not sure if the additional functionality is worth the additional complexity, and would like some feedback from other people: do you think we should keep things simple, or do you think we should consider using this alternate query type?
  * **ALTERNATE "TYPE" QUERY**  A reply to a TYPE query will contain a number of OSC strings- each of these OSC strings is a type tag string describing the types of value/values that can be retrieved from or sent to this method.  Most OSC methods will only have a single value type ("float", or "int", etc)- while this is convenient (and probably the most common) use, a significant number of existing OSC implementations work with ad-hoc data types consisting of an OSC array with multiple values, or even just OSC messages with more than one value.  It would also be nice if OSC methods could respond to different types of OSC values (an OSC method for setting the value of a pop-up menu could respond to either integer values for choosing by index, or a string value for choosing by name).  The TYPE query is capable of describing all of these, as demonstrated with the following examples:
  OSC Method that returns/expects a single float value
~~~
->  /foo/bar/methodName1#TYPE
<-  /foo/bar/methodName1##TYPE s : "f"
~~~
  OSC Method that returns/expects two float values (not an array, just two float values in the OSC message)
~~~
->  /foo/bar/methodName2#TYPE
<-  /foo/bar/methodName2##TYPE  s : "ff"
~~~
  OSC Method that returns/expects two values as an array
~~~
->  /foo/bar/methodName3#TYPE
<-  /foo/bar/methodName3##TYPE  s : "[ff]"
~~~
  OSC Method that returns/expects either an int or a float
~~~
->  /foo/bar/methodName4#TYPE
<-  /foo/bar/methodName4##TYPE  ss : "i" "f"
~~~
  OSC Method that returns/expects either an int or a symbol
~~~
->  /foo/bar/methodName5#TYPE
<-  /foo/bar/methodName5##TYPE  ss : "i" "s"
~~~
  OSC Method that returns/expects either an int or two floats
~~~
->  /foo/bar/methodName6#TYPE
<-  /foo/bar/methodName6##TYPE  ss : "i" "ff"
~~~
  OSC Method that returns/expects either an int or an array with two floats
~~~
->  /foo/bar/methodName7#TYPE
<-  /foo/bar/methodName7##TYPE  ss : "i" "[ff]"
~~~

  * **ALTERNATE "VAL" QUERY**  The VAL query is the only type of query message that should ever have a value with it.  The value in your query message should be an OSC string which contains the type tag string of the value/values you'd like to receive in reply to your query.  This value is optional: if your VAL query doesn't include a type tag string and the OSC method is capable of responding to multiple types, its reply will have the first type tag string returned by a TYPE query.  If your VAL query requests a type tag string which is unsupported by the destination OSC method, the server should return a 406 error.
  OSC Method that returns/expects a single float value
~~~
->  /foo/bar/methodName1#VAL
<-  /foo/bar/methodName1##VAL f : 1.0
~~~
  OSC Method that returns/expects two float values (not an array, just two float values in the OSC message)
~~~
->  /foo/bar/methodName2#VAL
<-  /foo/bar/methodName2##VAL ff : 1.0 1.0
~~~
  OSC Method that returns/expects two values as an array
~~~
->  /foo/bar/methodName3#VAL
<-  /foo/bar/methodName3##VAL [ff] : 1.0 1.0
~~~
  OSC Method that returns/expects either an int or a float
~~~
->  /foo/bar/methodName4#VAL
<-  /foo/bar/methodName4##VAL i : 1
->  /foo/bar/methodName4#VAL  s : "i"
<-  /foo/bar/methodName4##VAL i : 1
->  /foo/bar/methodName4#VAL  s : "f"
<-  /foo/bar/methodName4##VAL f : 1.0
~~~
  OSC Method that returns/expects either an int or a symbol
~~~
->  /foo/bar/methodName5#VAL
<-  /foo/bar/methodName5##VAL i : 1
->  /foo/bar/methodName5#VAL  s : "i"
<-  /foo/bar/methodName5##VAL i : 1
->  /foo/bar/methodName5#VAL  s : "s"
<-  /foo/bar/methodName5##VAL s : "one"
~~~
  OSC Method that returns/expects either an int or two floats
~~~
->  /foo/bar/methodName6#VAL
<-  /foo/bar/methodName6##VAL i : 1
->  /foo/bar/methodName6#VAL  s : "i"
<-  /foo/bar/methodName6##VAL i : 1
->  /foo/bar/methodName6#VAL  s : "ff"
<-  /foo/bar/methodName6##VAL ff : 1.0 1.0
~~~
  OSC Method that returns/expects either an int or an array with two floats
~~~
->  /foo/bar/methodName7#VAL
<-  /foo/bar/methodName7##VAL i : 1
->  /foo/bar/methodName7#VAL  s : "i"
<-  /foo/bar/methodName7##VAL i : 1
->  /foo/bar/methodName7#VAL  s : "[ff]"
<-  /foo/bar/methodName7##VAL [ff] : 1.0 1.0
~~~
