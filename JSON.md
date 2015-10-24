![Babel](media/logo.png)

About JSON Serialization
========================

Babel's default protocol is JSON. JSON maps nicely to data structures in a variety of languages.

All types may be `null`. However, Babel-generated code initializes lists and maps, so they will usually not be `null`. Member initializers can also be used to prevent a value from being `null`.

When `struct` members are null, they are not written to the JSON. This behavior should be standard across all supported languages.

Lists and maps can contain `null` values and those will be written to the output.

<table><thead><tr><th>Babel Type</th><th>JSON Representation</th><th>Notes</th></tr></thead>
<tbody>
<tr><td><code>bool</code></td><td><code>null<br/>true<br/>false</code></td><td></td></tr>
<tr><td><code>byte</code></td><td><code>null<br/>8</code></td><td>Bytes are unsigned.</td></tr>
<tr><td><code>int8</code></td><td><code>null<br/>-8<br/>8</code></td><td></td></tr>
<tr><td><code>int16</code></td><td><code>null<br/>-16<br/>16</code></td><td></td></tr>
<tr><td><code>int32</code></td><td><code>null<br/>-32<br/>32</code></td><td></td></tr>
<tr><td><code>int64</code></td><td><code>null<br/>"-64"<br/>"64"</code></td><td>Quoted to make large values safe for JavaScript.</td></tr>
<tr><td><code>float32</code></td><td><code>null<br/>-3.14<br/>3.14</code></td><td></td></tr>
<tr><td><code>float64</code></td><td><code>null<br/>-3.14159<br/>3.14159</code></td><td></td></tr>
<tr><td><code>string</code></td><td><code>null<br/>""<br/>"Hello, world"</code></td><td></td></tr>
<tr><td><code>datetime</code></td><td><code>null<br/>"2013-09-09T13:44:22.341-05:00"<br/>"2013-09-09T18:44:22.341Z"</code></td><td>Standard format string with a required time zone offset. "Z" is allowed to designate UTC time.</td></tr>
<tr><td><code>decimal</code></td><td><code>null<br/>"-99.987"<br/>"99.987"</code></td><td>Quoted to make long values safe for JavaSceript.</td></tr>
<tr><td><code>char</code></td><td><code>null<br/>"A"</code></td><td>Should contain exactly one character.</td></tr>
<tr><td><code>binary</code></td><td><code>null<br/>"YXNhZGFzZAo="</code></td><td>Base64-encoded string containing binary data.</td></tr>
<tr><td><code>list</code></td><td><code>null<br/>[ ]<br/>[ 3, null, 5, 6]</code></td><td>The code makes every attempt to keep lists initialized; you should generally not see null. Lists may contain null values, however. Items in a list all have the same type and can be any type supported by Babel, including lists, maps, and user-defined types.</td></tr>
<tr><td><code>map</code></td><td><code>null<br/>{ }<br>{ "3":"hello", "4":null, "5":"world" }</code></td><td>The code makes every attempt to keep maps initialized; you should generally not see null. Maps can have null values, but keys may not be null. Keys are all of the same type and must be a primitive type. Keys are always quoted regardless of type. Values are all of the same type and can be any type supported by Babel, including lists, maps, and user-defined types.</td></tr>
<tr><td><code>enum</code></td><td><code>null<br/>"Texas"</code></td><td>Enumerations are represented as a string containing the name. The ordinal value is not transmitted but can be used by client code.</td></tr>
<tr><td><code>const</code></td><td></td><td>Consts have no impact on the JSON format.</td></tr>
<tr><td><code>struct</code></td><td><code>null<br/>{ }<br/>{ "X":32, "Y":["a","b"] }</code></td><td>Structures serialize their members like a map of named items. Members with a null value are not serialized. Maps and lists should not be null; they will appear as empty.</td></tr>
<tr><td><code>void</code></td><td></td><td>void is only allowed on method returns. The server should provide an empty response for void methods.</td></tr>
</tbody></table>


Methods
-------

Methods transmit parameters in the HTTP request and receive the return value in the HTTP response.

### Parameters

Parameters are serialized like an anonymous `struct`. If the function is:

	bool CanHazCheeseburger(string nameOfCat, float64 weight);

Then the parameter list would look like:

	{
		"nameOfCat": "Moxy",
		"weight": 13.3
	}

Just like regular structs, a null parameter does not need to be written. Thus, if all parameters were null, the client might send:

	{}

This is actually what a client should send when there are no parameters. However, Babel libraries should also accept empty data in this case.

### Return Values

Return values are sent directly with no wrapper object.

#### Primitive Types

A function returning a primitive type returns it directly in the response:

	int32 GetData();

Returns:

	34

Keep in mind that the service might return `null`.

#### struct

A function returning a `struct`:

	MyStruct GetData();

Returns:

	{ "X": 32 }

It could also return a struct will all `null` members:

	{ }

Keep in mind that the service might return `null`.

#### list

A function returning a `list`:

	list<string> GetData();

Returns:

	[ "Bob", null, "Jane" ]

It could also return an empty list:

	[ ]

Babel libraries should convert a `null` response to an empty list, but this is not a guarantee. Don't forget to check for `null`.

#### map

A function returning a `map`:

	map<string,int32> GetData();

Returns:

	{ "Bob": 12, "Jane": 24 }

It could also return an empty map:

	{ }

Babel libraries should convert a `null` response to an empty map, but this is not a guarantee. Don't forget to check for `null`.

#### void

A function with no return:

	void SayHello();

Should not return any data in the HTTP response.

General Rules
-------------

* Don't write struct members which are `null`.
* Always return a response unless the return type is `void`.
* Minify the JSON response if possible.
* Lists can have `null` entries.
* Maps can have `null` values, not not `null` keys.

