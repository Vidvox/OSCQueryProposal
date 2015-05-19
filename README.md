#OSC Query Proposal

###BACKGROUND
 * Here are some existing / proposed specs for OSC querying:  
[Schmeder/Wright proposed spec](http://opensoundcontrol.org/files/osc-query-system.pdf)  
[OSNIP](https://github.com/jamoma/osnip/wiki)  
[Minuit](https://github.com/Minuit/minuit)  
[QLAB](http://figure53.com/qlab/docs/osc-api/)  
[libmapper](http://libmapper.github.io/about.html)  
[oscit](http://lubyk.org/en/software/oscit)  
[AVBC](http://grouper.ieee.org/groups/1722/1/contributions/2010/1722.1-koftinoff-AVBC-2010-04-07-v1.pdf)  
 * UBI-OSC (presented at ICMC/SMC/2014) is an interesting look at some of the possibilities that can be realized with a shared OSC interface that is standardized to allow interoperation between a number of independent devices : [paper](http://mobilesound.org/ubi-osc.pdf) - [poster](http://mobilesound.org/ubi/paper/ubiosc-poster.pdf) - [mailing list](http://mobilesound.org/ubi)
 * Some definitions copied from the [OSC spec](http://opensoundcontrol.org/spec-1_0):  
  * An **OSC server** has a set of OSC Methods.  
  * **OSC methods** are the potential destinations of OSC messages received by the OSC server and correspond to each of the points of control that the application makes available.  "Invoking" an OSC method is analgous to a procedure call; it means supplying the method with arguments and causing the method's effect to take place.  
  * An OSC server's OSC Methods are arranged in a tree structure called an **OSC Address Space**.  The leaves of this tree are the **OSC Methods** and the branch nodes are called **OSC Containers**.
 * This is the second version of this proposal, and is a substantial departure from the first version because of the feedback I got from other devs.  The initial version of this proposal described a query protocol based on a modified/expanded version of the OSC specification- other devs weren't happy about having to modify their OSC libs (or switch to modified libs), so i scrapped this approach.  OSC is good at sending simple control data over a UDP connection, and let's leave it at that: this proposal explores the OSC address space on a side channel as a wholly separate service.  This proposal is also less complete than the initial version- I've been encouraged to share this proposal sooner rather than later, so please chime in with any suggestions or corrections you have!

###GOAL
To establish a protocol that allows a client to browse and interact with a remote server's OSC address space.  The protocol should be capable of both describing the address space's structure/layout as well as the types of messages that destinations in the OSC address space are capable of both sending and receiving.  The protocol should be easily human-readable to facilitate debugging, and should have a standard set of errors to describe commonly-encountered problems.

The intent of this goal is to provide baseline functionality that other developers may take advantage of to construct impromptu or improvisational interfaces for dynamic environments.

###IMPLEMENTATIONS
[@jcelerier](https://github.com/jcelerier) has started work on the first implementation of this protocol.  It's called coppa (common protocol for parameters) and it [can be found here](https://github.com/jcelerier/coppa).

###PROPOSAL
 * Unlike the majority of OSC libraries, this "protocol" should probably be implemented using TCP connections to transfer data in the interest of ensuring complete delivery regardless as to the size of the payload.  If we're going to be using TCP-based connections to transfer text-based representations of data structures and we want the end product to be widely compatible, HTTP seems like the natural starting point for a basic communication protocol (reminder: we can't use OSC here because it would require using a modified version of the OSC spec).
  * Information about which ports clients and servers are listening to for OSC/UDP data can be exchanged as request/reply headers within HTTP transactions- the lib will need to be made aware of which ports you're listening to, but once the lib knows which ports you're using the protocol can ensure that any clients of your address space will have all the information they need to send OSC/UDP data directly to it (and vice versa).  This is useful if we want to use OSC/UDP to stream values between address spaces, reserving HTTP/TCP for sharing info about the structure and attributes of the address spaces.
  * The server should probably be an HTTP/1.1 server (capable of pipelining/maintaining a persistent connection/chunked transfer encoding), but i don't have any experience worth mentioning with these tools so if your experience means you have advice/cautionary tales, please let me know!  Barring that, this will be a "learn-as-you-go" experience...
 * If the goal of this server is to provide a data description of an OSC address space, it should consistently reply to queries/messages with one or more commonly recognizable data container formats.  I'm writing this proposal under the assumption that the server replies will be JSON blobs (presumably content-type "application/json") because that's what everybody who got in touch with me said they wanted to work with.
 * Instances of this server should advertise their presence (and look for other servers) on the local network using bonjour/zeroconf- i suggest using the service type "_oscjson._tcp.", but any service name will suffice as long as we're all using the same one.  Connections to clients on remote networks can be established by the traditional means (manually entering an IP address/joining a VPN/etc).
 * HTTP is fundamentally a request/reply protocol- a client is requesting a piece of information from a server, and a TCP connection is maintained at least until a reply is issued (or much longer- WebSockets can be used to "push" server notifications to clients).  HTTP requests are specified using URLs, the format of which is likely familiar:
 
 ~~~
         foo://example.com:8042/over/there?name=ferret#nose
         \_/   \______________/\_________/ \_________/ \__/
          |           |            |            |        |
       scheme     authority       path        query   fragment
 ~~~
 
 (ref: http://tools.ietf.org/html/rfc3986)
 
 In addition to providing a mechanism for specifying the destination of the request ('path' describes an OSC method or container in the address space), this format also provides a simple, open-ended mechanism for packing additional parameters or values into the URL (via 'query'), and explicitly requesting that the reply be limited to a single attribute ('fragment') for the sake of brevity.
 * Error handling is relatively simple- the initial response line to any HTTP query includes a standard response code and a reason phrase.  Examples of errors could include:
  * **204**  (no content- server received request, but the request is inappropriate in some way- querying VALUE or RANGE when ACCESS returned 0, for example)
  * **400**  (bad request/syntax error- corrupt message/packet, unrecognized query, etc)
  * **404**  (no container or method was found at the supplied OSC address)
  * **408**  (request time-out- this error should be generated by the client if it doesn't hear back from the server in a specified time period)
 * Retrieving the "VALUE" of a node- or the value of a particular attribute of the node- in a remote OSC address space was one of the initial goals of this project, and we can use the "fragment" portion of a URL to quickly and intuitively request the attribute of a particular node ( http://ip:port/abs/path#VALUE ).
 * Assuming the HTTP server is returning a JSON blob, these are some of the expected string:value pairs found in the returned blob.  Establishing a minimum set of attributes to be implemented should be a goal with the first implementation of this spec- please open an issue to discuss any attributes I may have overlooked/changes you'd like to see made to these attributes.  At present, the required key/value pairs are: 
  * **FULL_PATH**    The value stored with this string is the full OSC address path of the OSC node described by this object.
  * **CONTENTS**    The value stored with this string is a JSON object containing string:value pairs.  the strings correspond to the names of sub-nodes, and the values stored with them are JSON objects that describe the sub-nodes.  If the "CONTENTS" attribute is used, a single JSON object can be used to fully describe the attributes and hierarchy of every OSC method and container in an address space.  This attribute will only be used within a JSON object that describes an OSC container.  If this string:value pair is missing, the corresponding node should be assumed to be an OSC method (rather than a container).
  * **DESCRIPTION**    The value stored with this string is a string containing a human-readable description of this container/method.  While not all OSC nodes are required to return a human-readable description, this attribute is listed as "required" because every implementation of this protocol should be able to recognize it.
 * In addition to the minimum/required string:value pairs described above, implementations of this protocol can return pairs which describe additional attributes that, while not strictly necessary to describing the layout of an address space, are useful for working with OSC nodes that accept/expect typed values.  Here are some suggested string:value pairs you can return to describe in greater detail OSC methods which expect number values:
  * **TYPE**    The value stored with this string is a string- this is the OSC type tag string used by the OSC node at this address.  The number of elements returned in the "VALUE", "RANGE", and "UNIT" attribute arrays- as well as their type- is determined by this string.  Likewise, any OSC messages sent to this method should send data of the type and quantity described in this type tag string.  It is presumed that an OSC method will only have a single type of value (an OSC method with the declared type "f" should not be expected to respond to OSC messages with integer values).  If an OSC method doesn't have a value and doesn't require a value to be sent to it (usually an indicator that this node is purely a container), this attribute will be omitted.
  * **ACCESS**    The value stored with this string is an integer that represents a binary mask.  Returns 0 if there is no value associated with this OSC destination, 1 if the value may only be retrieved, 2 if the value may only be set, or 3 if the value may be both retrieved and set.
  * **VALUE**    The value stored with this string is always an array- this array contains one or more values (the number and type of values is described by the "TYPE" attribute).
  * **RANGE**    The value stored at this string is an array, which in turn contains a number of sub-arrays.  There should be one sub-array for each value returned (or expected) by this OSC method (this attribute is supposed to describe the range of expected values- as such, there needs to be a description of the range for each value).  In other words, there should be one sub-array for each of the types described in the "TYPES" attribute.  The "RANGE" attribute may be omitted if the OSC node doesn't have a value.
    * The first item in each sub-array is the minimum value of the range (or the "null" value if there's no minimum or the value isn't ranged).
    * The second item in each sub-array is the maximum value of the range (or the "null" value if there's no maximum or the value isn't ranged).
    * If the value has a min or max range, the third value in the array should be "null" (likely the most common occurrence).  If the value doesn't have a range- if the value is expressed as an item chosen from a list of possible values- the third value in the array should be another array containing the list of available value choices.
  * **TAGS**    The value stored at this string is an array of strings describing the OSC node- these tags are intended to serve an identifying role, making it possible to search or filter OSC nodes.
  * **CLIPMODE**    If provided, the value stored with this string is always an array- this array contains one string per value returned (or expected) by this OSC method (this attribute is supposed to describe the clipping mode of expected values, so there needs to be a description of the mode for each value).  The string should be either "none", "low", "high", or "both".  The CLIPMODE attribute acts as a "hint" to how the OSC method handles values outside the indicated RANGE- "none" indicates that no clipping is performed/the OSC method will try to use any value you send it, "low" indicates that values below the min range will be clipped to the min range, "high" indicates that values above the max range will be clipped to the max range, and "both" is self-explanatory.  This attribute is optional, and if it doesn't exist, software that expects it should assume that no clipping will be performed.
  * **UNIT**    If provided, the value stored with this string is always an array- this array contains one string per value returned (or expected) by this OSC method (this attribute is supposed to describe the unit of the expected values, so there needs to be one unit per value).  The string should describe the units of the value, from a list of commonly-accepted values (currently being assembled).  examples are assumed to be "hz", "meters", "seconds", etc.
 * In addition to describing attributes of remote OSC methods, JSON string:value pairs can be used to convey messages about specific events that clients need to respond to, or any other data that can be conveyed by serializing to JSON.
  * **PATH_CHANGED**    The value stored with this string is the OSC address path string of the OSC node on the server whose contents have changed.  If the node "/foo/bar" on a remote address space gets deleted, clients will receive a JSON object with PATH_CHANGED: "/foo".
 * Notifying clients of changes to the server's underlying address space is an important task.  Like many aspects of this proposal, I'm unsure as to the best way to implement this (and will remain unsure until I've done some experimenting), but it seems like this would be fairly easy to implement with WebSockets (the client would establish a WebSocket connection with the server, and from that point onwards the server can "push" data to the client).  While useful, this is also optional- minimal implementations of this protocol (perhaps running on embedded hardware) can skip this functionality to save resources without affecting the server's ability to present its basic attributes/address space.
 * The ability to "stream" values from a remote address space is something that devs from the Jamoma project have requested.  I'm still unsure of the details/best way to implement this, but here are a couple ideas in no particular order:
  * As mentioned earlier, the "query string" part of a URL can be used pass variables/parameters from client to server- a client could use this to request that the server start/stop streaming values to it (http://ip:port/abs/path?listen=true).  I'm not sure if this is "good design" or not, but it would be easy to use keywords to add functionality in this manner to the server's GET method...
  * Streaming the actual values back could be performed with simple OSC/UDP messages- this proposal (and presumably the lib implementation of it) is built to accompany an existing OSC service.  As such, it will know what port you're listening for OSC traffic on, and it can reasonably expect to be able to query this information from other instances.  Implementation note: the server should probably have an explicit TTL on these streams so if a client "forgets" to terminate it the stream will time out.
  * Streaming the actual values back could also be performed over WebSockets- if we're going to establish a connection for the purpose of notifying clients of address space changes, that connection can also be used to stream values.  While the values would suffer from the same latency that is inherent to TCP, their delivery would be guaranteed and less likely to run into routing issues caused by network middleware (which presumably is easier to configure and operate with traffic on port 80, but is realistically only a concern when sharing data that has to cross network boundaries).  This should probably be benchmarked and given consideration based on the results!


###EXAMPLES
 * Basic "GET" query, returns a two-level address space with four OSC nodes, one of which is a container (/baz) and three of which are methods (/foo, /bar, /baz/qux).  Note that the default behavior is to return a full description of the destination node and all of its contents.
 
 **http://ip:port/**
 ~~~
	{
		"DESCRIPTION": "root node",
		"FULL_PATH": "/",
		"ACCESS": 0,
		"CONTENTS": {
			"foo": {
				"DESCRIPTION": "demonstrates a read-only OSC node- single float value ranged 0-100",
				"FULL_PATH": "/foo",
				"ACCESS": 1,
				"TYPE": "f",
				"VALUE": [
					0.5
				],
				"RANGE": [
					[0.0, 100.0, null]
				]
			},
			"bar": {
				"DESCRIPTION": "demonstrates a read/write OSC node- two ints with different ranges",
				"FULL_PATH": "/bar",
				"ACCESS": 3,
				"TYPE": "ii",
				"VALUE": [
					4,
					51
				],
				"RANGE": [
					[0, 50, null],
					[51, 100, null]
				]
			},
			"baz": {
				"DESCRIPTION": "simple container node, with one method- qux",
				"FULL_PATH": "/baz",
				"ACCESS": 0,
				"CONTENTS": {
					"qux":	{
						"DESCRIPTION": "read/write OSC node- accepts one of several string-type inputs",
						"FULL_PATH": "/baz/qux",
						"ACCESS": 3,
						"TYPE": "s",
						"VALUE": [
							"half-full"
						],
						"RANGE": [
							[null, null, ["empty", "half-full", "full"]]
						]
					}
				}
			}
		}
	}
 ~~~
 
 * Same address space, but this is what you'd see if the GET query's address was slightly different (this time we're just trying to GET one specific container node and its contents).  Again, the default is to fetch the full "tree" of the specified node and all its sub-nodes.
 
 **http://ip:port/baz**
 
 ~~~
	{
		"DESCRIPTION": "simple container node, with one method- qux",
		"FULL_PATH": "/baz",
		"ACCESS": 0,
		"CONTENTS": {
			"qux":	{
				"DESCRIPTION": "read/write OSC node- accepts one of several string-type inputs",
				"FULL_PATH": "/baz/qux",
				"ACCESS": 3,
				"TYPE": "s",
				"VALUE": [
					"half-full"
				],
				"RANGE": [
					[null, null, ["empty", "half-full", "full"]]
				]
			}
		}
	}
 ~~~
 
 * This is one way the "fragment" portion of a URL can be used to request a single attribute from a node in a remote address space:
 
 **http://ip:port/foo#VALUE**
 
 ~~~
 	{
 		"VALUE": [
 			0.5
 		]
 	}
 ~~~
 
 * The same technique can be used to request any attribute from a given node:
 
 **http://ip:port/baz/qux#RANGE**
 
 ~~~
 	{
 		"RANGE": [
 			[null, null, ["empty", "half-full", "full"]]
 		]
 	}
 ~~~
 
 * Assuming a persistent HTTP connection, this is what the client would receive if the node "/baz/qux" is deleted from the remote address space:
 
 ~~~
 	{
 		"PATH_CHANGED": "/baz"
 	}
 ~~~
 
 * This is how you would request that the remote server begin streaming any updates to the remote address space's "/foo" node back to the client server:
 
 **http://ip:port/foo?LISTEN=true**
 
 ...at this point, the remote server would start sending values from "/foo" back to the client, though it is yet to be determined exactly how those values should be sent back.  The easiest and most "natural" solution to my mind would be to send a stream of OSC messages via UDP from server to client (as mentioned earlier, OSC listening port info can be exchanged in request/reply headers), but it would also be possible to implement this functionality over a persistent HTTP connection.
 
 Regardless as to implementation, here's how you would request that the remote server stop streaming any updates to the client:
 
 **http://ip:port/foo?LISTEN=false**
 
 * This pattern can be extended further, and more keywords can be adopted to add more functionality.  For example, if we want to use OSC messages to enable value streaming, it would be nice if the OSC address paths of those messages could be customized (instead of just echoing the address from the remote server).  This sort of behavior could be enabled using a "TARGET_ADDRESS" keyword in the URL's query string:
 
 **http://ip:port/foo?LISTEN=true&TARGET_ADDRESS=/dest/address/in/client**
 
 ...at this point, the remote server would start sending values from "/foo" back to the client as OSC messages- and those OSC messages would be addressed to "/dest/address/in/client".

****
****


###PROBLEMS / QUESTIONS / DISCUSSION POINTS

have been split up and moved to the "Issues" section of GitHub, where they can be discussed comfortably.  If you'd like to discuss a change or propose an addition, please open an issue in the GitHub interface!
