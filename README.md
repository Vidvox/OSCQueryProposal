# OSC Query Proposal

## Background

Some definitions copied from the [OSC spec](http://opensoundcontrol.org/spec-1_0):  

* An **OSC server** has a set of OSC Methods.  
* **OSC methods** are the potential destinations of OSC messages received by the OSC server and correspond to each of the points of control that the application makes available.  "Invoking" an OSC method is analgous to a procedure call; it means supplying the method with arguments and causing the method's effect to take place.  
* An OSC server's OSC Methods are arranged in a tree structure called an **OSC Address Space**.  The leaves of this tree are the **OSC Methods** and the branch nodes are called **OSC Containers**.

Here are some existing / proposed specs for OSC querying:

* [Schmeder/Wright proposed spec](http://opensoundcontrol.org/files/osc-query-system.pdf)  
* [OSNIP](https://github.com/jamoma/osnip/wiki)  
* [Minuit](https://github.com/Minuit/minuit)  
* [QLAB](http://figure53.com/qlab/docs/osc-api/)  
* [libmapper](http://libmapper.github.io/about.html)  
* [oscit](http://lubyk.org/en/software/oscit)
* [AVBC](http://grouper.ieee.org/groups/1722/1/contributions/2010/1722.1-koftinoff-AVBC-2010-04-07-v1.pdf)  
* UBI-OSC (presented at ICMC/SMC/2014) is an interesting look at some of the possibilities that can be realized with a shared OSC interface that is standardized to allow interoperation between a number of independent devices : [paper](http://mobilesound.org/ubi-osc.pdf) - [poster](http://mobilesound.org/ubi/paper/ubiosc-poster.pdf) - [mailing list](http://mobilesound.org/ubi)

This is the third revision of this proposal, which is in its later stages and probably won't change much between now and whenever we remove the "proposal" language and flag it as a 1.0 release.  It is mostly similar to the second revision- the main difference is the addition of details about bi-directional communication, and clarification of some important details that weren't emphasized significantly in the prior draft.

## Goal

To establish a protocol that allows a client to browse and interact with a remote server's OSC address space.  The protocol should be capable of both describing the address space's structure/layout as well as the types of messages that destinations in the OSC address space are capable of both sending and receiving.  The protocol should be easily human-readable to facilitate debugging, and should have a standard set of errors to describe commonly-encountered problems.

The intent of this goal is to provide baseline functionality that other developers may take advantage of to construct impromptu or improvisational interfaces for dynamic environments.

## Implementations

* [@jcelerier](https://github.com/jcelerier) started work on the first implementation of this protocol called coppa (common protocol for parameters). The orignal repository is [here](https://github.com/jcelerier/coppa), but it has been since been merged into the [ossia library](https://github.com/OSSIA/libossia).
* [@lov](https://github.com/lov) has started work on another implementation of this protocol, which [can be found here](https://github.com/lov/OSCQuery).
* I've also put together a Cocoa framework that implements this query proposal, along with a simple browser for viewing and interacting with remote OSC query servers, which [can be found here](https://github.com/mrRay/VVOSCQueryProtocol).

## Proposal

This protocol is intended to be minimal and extensible- at its heart is a simple HTTP server that provides structured data (JSON) describing methods in an OSC address space.  This is referred to as the "**core functionality**", and it consists of a standard set of attributes that every server is required to provide about every OSC method it wants to expose for querying via this protocol.  Clients can then use this information to determine how they want to send values to the server.

In addition to the "core functionality", this protocol establishes a set of **optional attributes** that clients and servers are *strongly encouraged* to support- these optional attributes provide a substantial amount of contextual information about OSC methods.  They have been categorized as "optional" because if this information is not provided for any reason, clients will still be able to browse the address space of a remote server and interact with its methods- that interaction just won't be as rich or well-informed.  Clients can query servers for their support for these optional attributes on a case-by-case basis by checking the "EXTENSIONS" object found in the server's "HOST_INFO" (more on this later).

This protocol also defines a simple way to establish **bi-directional communication**, which is used to both send commands or notifications and stream data between clients and servers.  Like the optional attributes, this streaming functionality is part of the OSC query protocol and clients and servers are *strongly encouraged* to support it- but it isn't required.  Clients can determine if a server supports this at runtime by checking the "EXTENSIONS" object found in the server's "HOST_INFO" (more on this later).

Modifying this proposal to support custom attributes specific to your software or situation is intentionally trivial- you just need your server to populate the JSON objects it returns with whatever extra data you want to share, and your client to look for these extra attributes.  If you want to share your custom attributes in this document so that other people may build support for them, get in touch with me and I'll add them below.

## Core Functionality

* The HTTP server should be an HTTP 1.1 server.  If you want to be able to stream values from the server to its clients, the server needs to be capable of supporting websockets.  The protocol also allows for running the two servers on different ports if there is a technical or deployment issue (HTTP server on one port, websocket server on another).
* HTTP is a request/reply protocol- a client is requesting a piece of information from a server, and a TCP connection is maintained until a reply is issued.  HTTP requests are specified using URLs, the format of which is likely familiar:
 
	~~~
			foo://example.com:8042/over/there?name=ferret#nose
			\_/   \______________/\_________/ \_________/ \__/
			|           |            |            |        |
			scheme   authority       path        query   fragment
	~~~
 
	 (ref: http://tools.ietf.org/html/rfc3986)
	* The "path" portion of the HTTP request's URL should be parsed and used to determine which method in the OSC address space is being requested.  The URL http://localhost:1234/foo/bar refers to the method "/foo/bar" in the OSC address space- further examples are provided below.
	* The "query" portion of the HTTP request's URL should be parsed and used to indicate which attributes are to be returned.  The URL http://localhost:1234/foo/bar?VALUE refers to the "VALUE" attribute of the method "/foo/bar" found in the OSC address space.  If a query is made for an attribute that the server doesn't support, it should return an error of type 204.  Further examples are provided below.
	* The "fragment" portion of the request's URL should be stripped and disregarded.
* The server provides JSON objects (content-type "application/json") in response to HTTP requests.  Each JSON object describes the OSC method at the requested URL, which will either correspond directly to an OSC method in the address space (the "path" component of the URL corresponds to the OSC address path), or to some specific attribute of the OSC method exposed by the protocol (in the case of URLs that contain query components).
* Instances of this server should advertise their presence (and look for other servers) on the local network using bonjour/zeroconf using the service type "_oscjson._tcp.".  Connections to clients on remote networks can be established by the traditional means (manually entering an IP address/joining a VPN/etc).

* Errors should be handled by returning the HTTP error that most closely corresponds to the nature of the error you encountered.  Examples of errors could include:
	* **204**  (no content- server received request, but the request is inappropriate in some way- querying VALUE when ACCESS returned 0, for example)
	* **400**  (bad request/syntax error- corrupt message/packet, querying an unrecognized attribute or a recognized attribute that isn't supported by the server, etc)
	* **404**  (no container or method was found at the supplied OSC address)
	* **408**  (request time-out- this error should be generated by the client if it doesn't hear back from the server in a specified time period)

### Core Functionality- Required Attributes

When the HTTP server receives a request for a URL, it returns a JSON object describing the OSC method referred to by the URL.  These JSON objects consist of key:value pairs- following is a list of the minimum attributes that clients/servers must support to implement the core functionality of this proposal.  These attributes are categorized as "required" because if they're missing- or inaccurate- then this protocol won't work.

* **FULL_PATH**    The value stored with this string is the full OSC address path of the OSC node described by this object.  Every OSC method is required to return a non-null value for this attribute.
* **CONTENTS**    The value stored with this string is a JSON object containing string:value pairs.  The strings correspond to the names of sub-nodes, and the values stored with them are JSON objects that describe the sub-nodes.  If the "CONTENTS" attribute is used, a single JSON object can be used to fully describe the attributes and hierarchy of every OSC method and container in an address space.  This attribute will only be used within a JSON object that describes an OSC container.  If this string:value pair is missing, the corresponding node should be assumed to be an OSC method (rather than a container).
* **TYPE**    The value stored with this string is a string- this is the OSC type tag string used by the OSC node at this address.  The number of elements returned in the "VALUE", "RANGE", and "UNIT" attribute arrays- as well as their type- is determined by this string.  Likewise, any OSC messages sent to this method should send data of the type and quantity described in this type tag string.  It is presumed that an OSC method will only have a single type of value (an OSC method with the declared type "f" should not be expected to respond to OSC messages with integer values).  If an OSC method doesn't have a value and doesn't require a value to be sent to it (usually an indicator that this node is purely a container), this attribute may be omitted from the returned object.
* **HOST_INFO**    This attribute will never be provided by the server by default- it will only be provided if the client software explicitly queries this attribute.  If this attribute is queried, the JSON object that is returned will *only* consist of this object- the path in the URL is functionally irrelevant.  The value returned will always be a JSON object, with zero or more of the following keys:
    * **NAME**    The value stored with this key is a human-readable string describing the name of the host.  OSC query servers are not required to provide this value- it may be omitted.
    * **EXTENSIONS**    The value stored with this string is a JSON object describing the various optional attributes listed below.  The keys for this object are the various attribute keys, and the values are boolean values indicating whether or not the host supports the attribute (if an optional attribute is not listed, the client should assume that it isn't supported).
    * **OSC_IP**    The value stored with this string is a string describing the IP address at which the OSC server corresponding to this OSC query server can be reached.  If there is no value stored at this string, it should be assumed that the OSC server can be reached at the same network address as the OSC query server which is serving this request.
    * **OSC_PORT**    The value stored with this string is an integer describing the port at which this OSC address can be reached.  If there is no value stored at this string, it should be assumed that the OSC server can be reached at the same network address as the OSC query server which is serving this request.
    * **OSC_TRANSPORT**    The value stored with this string is a string describing how to reach the OSC server associated with this query server.  The string is expected to be either "**TCP**" or "**UDP**".  If there is no value stored at this string, it should be assumed that the OSC server expects to receive UDP messages.
    * **WS_IP**    The value stored with this string is a string describing the IP address at which the WebSocket server corresponding to this OSC query server can be reached.  If there is no value stored at this string, it should be assumed that the WebSocket server can be reached at the same network address as the OSC query server which is serving this request.
    * **WS_PORT**    The value stored with this string is an integer describing the port at which the WebSocket server can be reached.  If there is no value stored at this string, it should be assumed that the WebSocket server corresponding to this OSC query server will use the same port as the OSC query server which is serving this request.

## Optional Attributes

In addition to the required attributes above, this protocol defines a set of optional attributes which describe further properties of the OSC method to aid in determining how to best interact with it.  Support for these attributes is not mandatory- but you should strive to provide as much of this information as possible in your server implementation.  Your server indicates its support for these optional attributes on a case-by-case basis by including them in the EXTENSIONS object returned by a HOST_INFO query.

If an optional attribute is supported by a server, but that attribute is not returned in a JSON object returned by the server, the client should behave as if the attribute is not relevant to the OSC method described by that JSON object.

Some optional attributes need to include values that correspond directly to OSC values and OSC data types.  The following table lists JSON equivalents to OSC data types:

| OSC Type Tag | JSON equivalent |
| --- | --- |
| i, f, h, d, t | JSON number |
| s, S, c | JSON string |
| r <img width=300/> | JSON string containing hex color notation for an RGBA color (four pairs of hexadecimal values), preceded by a hashtag/pound sign.  Opaque white is "#FFFFFFFF", opaque red is "#FF0000FF", transparent blue is "#0000FF00", etc. |
| [, ] | JSON array |
| b, m | JSON nil.  OSC blobs and MIDI-type OSC values don't have the sort of defined types that allows OSCQuery to publish VALUE or RANGE information about them, so use a JSON "nil" as a placeholder where necessary. |
| T, F, N, I | JSON nil.  These types are values without having a value- they don't have or need a JSON value, so use a JSON "nil" as a placeholder where necessary. |

* **ACCESS**    The value stored with this string is an integer that represents a binary mask.  Returns 0 if there is no value associated with this OSC destination, 1 if the value may only be retrieved, 2 if the value may only be set, or 3 if the value may be both retrieved and set.
	* If the value returned by ACCESS indicates that the method's value may be retrieved, but the "VALUE" attribute is not supported, any queries for the VALUE attribute should return a 204 (no content/inappropriate request).
	* If the value returned by ACCESS may not be retrieved, but the "VALUE" attribute is supported, any queries for the VALUE attribute should return a 204 (no content/inappropriate request).
	* If the ACCESS attribute is not supported, or it is supported but a value is not returned for it, then the assumption should be that the value is always writable, and also readable if the "VALUE" attribute is supported.
* **VALUE**    The value stored with this string is always an array that should contain one JSON item for each of the types described in the "TYPES" attribute (the structure of this array is described in greater detail in ["Other notes on attributes"](#other-notes-on-attributes)).  Each item in the array should be a JSON representation of the corresponding OSC value type (a table is provided above with JSON equivalents for OSC data types).
	* If a query is performed for the "VALUE" attribute, but the "ACCESS" attribute indicates that the OSC method isn't readable, the query should return a 204 (no content/inappropriate request).
	* If a query is performed for the "VALUE" attribute, but the OSC method does not yet have a value, the query should return an empty JSON object.
* **RANGE**    The value stored at this string is an array that should contain one JSON object for each of the types described in the "TYPES" attribute (the structure of this array is described in greater detail in ["Other notes on attributes"](#other-notes-on-attributes)).  Each JSON object in the array describes the range of the corresponding value from "TYPES", using one or more of the following keys:
    * If the JSON object in the "RANGE" array has a value for the key "**MIN**", the accompanying value is the minimum expected value.  If no object is specified for the "MIN" key then the host hasn't supplied a minimum value.
    * If the JSON object in the "RANGE" array has a value for the key "**MAX**", the accompanying value is the maximum expected value.  If no object is specified for the "MAX" key then the host hasn't supplised a maximum value.
    * If the JSON object in the "RANGE" array has a value for the key "**VALS**", the accompanying value is an array listing the only acceptable values to send to the method.  This is typically used to describe a series of menu options in a pop-up button, but can also be applied to numeric values.
* **DESCRIPTION**    The value stored with this string is a string containing a human-readable description of this container/method.
* **TAGS**    The value stored at this string is an array of strings describing the OSC node- these tags are intended to serve an identifying role, making it possible to search or filter OSC nodes.
* **EXTENDED_TYPE**    If provided, the value stored with this string is always an array- this array contains one string per value returned (or expected) by this OSC method (this attribute provides meaningful semantic information about the expected values, so there needs to be one extended type for each value).  The string should describe what the value means/what the value is/what the value does, and should be as brief as possible (ideally only a single word).  For example, if your address space has a method that accepts a filepath as a string, you could set the "EXTENDED_TYPE" to "filepath", and other software that inspects this could provide a file picker UI instead of just a blank text field.
* **UNIT**    If provided, the value stored with this string is an array- this array contains one string per value returned (or expected) by this OSC method (this attribute is supposed to describe the unit of the expected values, so there needs to be one unit per value).  The string should describe the units of the value, from a list of commonly-accepted values [currently being assembled here](units.txt).
* **CRITICAL**    If provided, the value stored with this string is expected to be a boolean value (true/false) used to indicate that the messages sent to this address are of particular importance, and their delivery needs to be guaranteed.  If both the host and client support it (this attribute is optional), they should use a TCP connection of some sort to guarantee delivery of the message.  The "streaming" portion of this protocol describes a simple way of passing binary OSC packets over a (TCP) websocket connection, which would make this very easy.
* **CLIPMODE**    If provided, the value stored with this string is an array- this array contains one string per value returned (or expected) by this OSC method (this attribute is supposed to describe the clipping mode of expected values, so there needs to be a description of the mode for each value).  The string should be either "none", "low", "high", or "both".  The CLIPMODE attribute acts as a "hint" to how the OSC method handles values outside the indicated RANGE- "none" indicates that no clipping is performed/the OSC method will try to use any value you send it, "low" indicates that values below the min range will be clipped to the min range, "high" indicates that values above the max range will be clipped to the max range, and "both" is self-explanatory.  This attribute is optional, and if it doesn't exist, software that expects it should assume that no clipping will be performed.
* **OVERLOADS**    The goal of this attribute is to communicate to clients that this OSC method can respond to OSC messages with type tag strings that differ from the value associated with the TYPE attribute.  If provided, the value stored with this string is always an array- the array contains JSON objects, each of which describes this OSC method with a different OSC type tag string.  These JSON objects should not contain any values for the CONTENTS key- but aside from that exception these JSON objects can use all of the other attributes in this spec.
* **HTML**    This attribute will never be provided by the server- instead, the presence of this attribute in a query indicates that the client is requesting an HTML resource.  The server should try to find a local resource corresponding to the resource path in the requested URL, and return it instead of a JSON object.  For example, this attribute could be used to provide an HTML page and assorted accompanying media that renders either a simple, nicely-formatted description of an OSC node.

### Other notes on attributes
* Many of the optional attributes (**VALUE**, **RANGE**, **EXTENDED_TYPE**, **UNIT**, and **CLIPMODE**) convey information about the individual values an OSC method expects to receive- the JSON object stored with each of these attributes is a JSON array that contains one item per OSC type in the OSC method's type tag string (the "TYPE" attribute).
	* If the OSC method's type tag string includes an explicit array designator using the "[" and "]" characters, this should be reflected in the JSON array by inserting another JSON array to match the basic structure of the OSC method's type tag string.
	*  If all of the elements of one of these optional attribute share the same value, it's acceptable to only list a single value (which clients will assume corresponds to all of the elements) for the sake of brevity.
	* Examples are provided at the end for clarity.

## Optional Bi-Directional Communication

Bi-directional communication between a server and each of its clients is an optional part of this protocol.  If you're writing a client or server- particularly on an embedded device- then this may be impractical or pointless and you can skip it.  Technical limitations aside, devs are strongly encouraged to support this portion of the protocol.  Following is a list of the basic functionality required by clients/servers that choose to implement this:

* Bi-directional communication is achieved by a persistent websocket connection between client and server- this connection is initiated by the client at its convenience.  It is assumed that the websocket server is running on the same IP/port as the HTTP server, but this can be verified by retrieving the server's HOST_INFO object and checking its contents for "WS_IP" and "WS_PORT".
* Once the websocket connection has been established, the client and server communicate with one another by sending websocket messages using the "text" opcode.  The messages are always JSON objects that must contain the following key:value pairs
	* **COMMAND**    The value stored with this key will always be a string describing the command you want the message to perform.  There are only a couple predefined COMMANDs, which will be discussed shortly.
	* **DATA**    The value stored with this key will depend on the specific COMMAND it accompanies, and is defined on an individual basis later.
* In addition to sending JSON objects, raw OSC packets are also sent over the websocket connection using the "binary" opcode of the websocket protocol- the format of these packets is identical to the format of OSC packets sent over the network.  This allows values to be streamed between the server and its clients, and also makes it substantially easier to build browser-based client apps.

### Client -> Server Communication Attributes

* **LISTEN**    If the server receives a websocket message with "LISTEN" as the provided "COMMAND", the value associated with "DATA" must be a string describing an OSC method in the server's address space.  LISTEN messages indicate that the client wants to "listen" to any OSCS messages sent to a given method in the server's address space.  After receiving this message, the server will immediately begin passing all OSC messages sent to that method to every client that requested to "LISTEN" to it.
	* The OSC messages streamed from the server to its clients will be passed as raw binary data using the "binary" opcode of websockets- the format of this raw binary data is identical to the packet format described by the OSC specification.
	* The OSC address specified by a "LISTEN" command is specific: if a client asks to "LISTEN" to /foo, it will only receive messages sent to /foo- it will not receive messages sent to /foo/bar.
	* If at any point the websocket connection between the server and one of its clients is broken, the server will stop streaming any values to it.  At this point, it becomes the client's responsibility to re-establish the websocket connection, and then explicitly request to "LISTEN" to any OSC methods it wants to receive data from again.
	* Clients that register to "LISTEN" to a server's OSC method are responsible for instructing the server to stop sending values to the client by sending the server an "IGNORE" message.  If a client register to LISTEN but fails to issue a corresponding IGNORE, the server will continue streaming values to the client until either the websocket connection is broken, or the server's address space changes and the method no longer exists.
* **IGNORE**    The inverse of "LISTEN"- instructs a server that the client from which this message originated no longer wants to "LISTEN" to the provided path.  The value stored with "DATA" is expected to be a string describing the OSC method the client no longer wants to listen to.  It is not necessary to list the "IGNORE" attribute in the server's EXTENSIONS object if "LISTEN" is already listed (the presence of "LISTEN" implies support for "IGNORE").

### Server -> Client Communication Attributes

* **PATH_CHANGED**    If the client receives a websocket message with "PATH_CHANGED" as the provided "COMMAND", the server is indicating that a method in the OSC address space has changed and the client should request its structure as soon as possible.  The value stored with "DATA" is a string describing the path in the OSC address space that has changed- the method corresponding to this path (and any methods it contained) must all be "reloaded" by the client.
	* This message is sent after any action that would result in a change to the structure of the OSC address space- adding OSC methods, removing OSC methods, and renaming OSC methods would all result in a "PATH_CHANGED" message.
	* If the server has to send multiple "PATH_CHANGED" messages to each client, it is acceptable- but not required- to coalesce these messages, sending a single "PATH_CHANGED" message that encompasses all of the individual "PATH_CHANGED" messages by passing the highest-level common path shared by the messages.
* **PATH_RENAMED**    If the client receives a websocket message with "PATH_RENAMED" as the provided "COMMAND", the server is indicating that a method in the OSC address space has been renamed.  The value stored with "DATA" is a JSON object with two keys: "**OLD**" and "**NEW**".
	* **OLD**    The value stored with this key is a string describing the OSC address of the method that is being renamed.
	* **NEW**    The value stored with this key is a string describing the new OSC address of the OSC method described by **OLD**.
	* Clients that are receiving values from a server because they made a LISTEN request are responsible for updating any local records they maintain with the new address of any OSC methods that might be affected by this rename.  Clients that were LISTENing to the old address on the server should continue receiving values sent to the new address on the server.
	* Servers that are streaming values because a client made a LISTEN request are responsible for updating any local records they maintain with the new address of any OSC methods that might be affected by this rename.  Any clients that were receiving values from the old address will be reconfigured to receive values sent to the new address.
* **PATH_REMOVED**    If the client receives a websocket message with "PATH_REMOVED" as the provided "COMMAND", the server is indicating that a method in the OSC address space has been deleted.  The value stored with "DATA" is a string describing the OSC method that has been removed from the server's address space.
* **PATH_ADDED**    If the client receives a websocket message with "PATH_ADDED" as the provided "COMMAND", the server is indicating that a method in the OSC address space has been created.  The value stored with "DATA" is a string describing the OSC method that has been added to the server's address space.

## Examples

* Basic "GET" query, returns a two-level address space with four OSC nodes, one of which is a container (/baz) and three of which are methods (/foo, /bar, /baz/qux).  Note that the default behavior is to return a full description of the destination node and all of its contents.
	* "/foo" is read-only, so any OSC messages you send to it will be ignored- but you can query its value, and if the server supports the LISTEN attribute then its value can be streamed to clients
	* "/bar" can be read and written (and LISTENed to, if the server supports it), and it requires/returns two integer values per OSC message.
	* "/baz/qux" can be read and written, and it requires an OSC message with one of only three possible string value ("empty", "half-full", or "full").
 
	`http://ip:port/`

	~~~json
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
					{
						"MIN": 0.0,
						"MAX": 100.0
					}
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
					{
						"MIN": 0,
						"MAX": 50
					},
					{
						"MIN": 51,
						"MAX": 100
					}
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
							{
								"VALS": [ "empty", "half-full", "full" ]
							}
						]
					}
				}
			}
		}
	}
	~~~

* Same address space, but this is what you'd see if the GET query's address was slightly different (this time we're just trying to GET one specific OSC method and its contents).  Again, the default is to fetch the full "tree" of the specified node and all its sub-nodes.

	`http://ip:port/baz`
 
	~~~json
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
					{
						"VALS": [ "empty", "half-full", "full" ]
					}
				]
			}
		}
	}
	~~~

* This is an example of using the "query" portion of a URL to request a single attribute from a node in a remote address space:

	`http://ip:port/foo?VALUE`
 
	~~~json
	{
		"VALUE": [
			0.5
		]
	}
	~~~
 
* The same technique can be used to request any attribute supported by the server from a given node:
 
	`http://ip:port/baz/qux?RANGE`
 
	~~~json
	{
		"RANGE": [
			{
				"VALS": [ "empty", "half-full", "full" ]
			}
		]
	}
	~~~

* Some queries may return empty objects- "/baz" is a pure container object- it doesn't accept any OSC messages, so it doesn't have a type:

	`http://ip:port/baz?TYPE`
 
	~~~json
	{
	
	}
	~~~

* Some queries may return HTTP errors- the following query would return a 404 error:

	`http://ip:port/bazzzzz?TYPE`

* This is an example of a "HOST_INFO" query- the "path" component of the URL is ignored, and the server returns the HOST_INFO object:
	* The HOST_INFO object doesn't have any entries for "OSC_IP" or "OSC_PORT", so the OSC server is running on the same IP/port as the OSC query server.  There isn't an entry for "OSC_TRANSPORT", so the OSC server is probably listening for UDP messages.
	* The HOST_INFO object doesn't have any entries for "WS_IP" or "WS_PORT", so the websocket server (if there is one) s running on the same IP/port as the OSC query server.
	* The "EXTENSIONS" object includes an affirmative entry for "LISTEN"- this server supports value streaming
	* The "EXTENSIONS" object includes an affirmative entry for "PATH_CHANGED"- this server will send "PATH_CHANGED" messages to its clients when its address space changes.
 
	`http://ip:port/foo?HOST_INFO`
 
	~~~json
	{
		"NAME": "My Special Server",
		"EXTENSIONS" : {
			"ACCESS": true,
			"VALUE": true,
			"RANGE": true,
			"DESCRIPTION": true,
			"TAGS": true,
			"EXTENDED_TYPE": true,
			"UNIT": true,
			"CRITICAL": true,
			"CLIPMODE": true,
			"LISTEN": true,
			"PATH_CHANGED": true
		}
	}
	~~~

* This is what the client would receive over its websocket connection if the node "/baz/qux" is deleted from the remote address space:
 
	~~~json
	{
		"COMMAND": "PATH_CHANGED",
		"DATA": "/baz"
	}
	~~~

* If a client wants to register to receive values sent to an address space in the server, it would send the server a message over its websocket connection that looks like this:

	~~~json
	{
		"COMMAND": "LISTEN",
		"DATA": "/foo"
	}
	~~~

* If a client registered to LISTEN to an OSC method on the server, it has to tell the server to stop sending values to it at some point- this is done by sending a message over its websocket connection to the server that looks like this:

	~~~json
	{
		"COMMAND": "IGNORE",
		"DATA": "/baz"
	}
	~~~

* These are some examples of JSON objects that illustrate various features or edge cases of the spec:
	* These examples illustrate how the structure of the VALUE, RANGE, and UNIT attributes all directly reflect the structure of its TYPE attribute.  EXTENDED_TYPE and CLIPMODE have the same structure, but aren't demonstrated here because that would be needlessly redundant.
	~~~json
	{
		"TYPE": "ff",
		"VALUE": [
			1.23,
			456.78
		],
		"RANGE": [
			{	"MIN": 0.0, "MAX": 10.0		},
			{	"MIN": 0.0, "MAX": 987.0	}
		],
		"UNIT": [
			"distance.miles",
			"distance.m"
		]
	}
	~~~
	~~~json
	{
		"TYPE": "i[ff]i",
		"VALUE": [
			100,
			[
				1.0,
				1.0
			],
			100
		],
		"RANGE": [
			{	"MIN": 0, "MAX": 100	},
			[
				{	"MIN": 0.0, "MAX": 1.0	},
				{	"MIN": 0.0, "MAX": 1.0	}
			],
			{	"MIN": 0, "MAX": 100	}
		],
		"UNIT": [
			"distance.mm",
			[
				"distance.miles",
				"distance.miles"
			],
			"distance.mm"
		],
	}
	~~~
	* This example demonstrates the use of OVERLOADS to specify additional type tag strings the OSC method will respond to.  The default OSC method requires an OSC color, but OVERLOADS allows it to accept a message with four ints, and also a message with four floats for higher-accuracy colors.  This example also illustrates the use of the hex notation for RGBA colors- the color "#FA6432FF" is (250, 100, 50, 255), or (0.980392156862745, 0.392156862745098, 0.196078431372549, 1.0).
	~~~json
	{
		"TYPE": "r",
		"VALUE": [
			"#FA6432FF"
		],
		"OVERLOADS": [
			{
				"TYPE": "ffff",
				"VALUE": [
					0.980392156862745,
					0.392156862745098,
					0.196078431372549,
					1.0
				],
				"RANGE": [
					{	"MIN": 0.0, "MAX": 1.0	},
					{	"MIN": 0.0, "MAX": 1.0	},
					{	"MIN": 0.0, "MAX": 1.0	},
					{	"MIN": 0.0, "MAX": 1.0	},
				]
			},
			{
				"TYPE": "iiii",
				"VALUE": [
					250,
					100,
					50,
					255
				],
				"RANGE": [
					{	"MIN": 0, "MAX": 255	},
					{	"MIN": 0, "MAX": 255	},
					{	"MIN": 0, "MAX": 255	},
					{	"MIN": 0, "MAX": 255	},
				]
			}
		]
	}
	~~~
