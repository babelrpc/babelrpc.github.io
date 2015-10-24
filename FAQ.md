![Babel](media/logo.png)

FAQ
===

General
-------

### It seems to me that Babel should support...

The answer is almost always "no, it shouldn't." Babel is completely generic - not specific to a product or language. Really good libraries that are product-specific can be built on top of the Babel foundation.

### What is the difference between `byte` and `int8`?

In languages that support it, `byte` is unsigned and `int8` is signed.

### How should I do namespaces?

Excellent question! There are many ways to do namespaces. First off, you almost never need a language-specific namespace, because Babel's namespace format generates reasonable namespaces for all languages. A namespace like:

	namespace company.com/Tower/Babel

Would result in the C# namespace:

	Company.Tower.Babel

and the Java namespace:

	com.company.Tower.Babel

If needed, you can override it with a language-specific namespace:

	namespace asp "TowerBabel"

Organizing your data models around namespaces is really important. You will be living with these choices for a very long time. Another consideration is that models may be shared by multiple services. We would recommend against designating the midtier or servie name in the namespace. Instead, use product areas.

	namespace company.com/ProductArea/CodeArea

This makes them better for general use.

### How do I include other Babel files?

Use the `import` statement to include other files. We highly recommend using relative paths.

	import "shared/cool.babel"

Multiply-included files don't hurt anything.

Command-Line
------------

### Can Babel process more than one file at a time?

Yes! Babel supports standard file patterns for files and directories, and you can specify multiple files and patterns in a single command-line.

A nice approach for a script is to use `pushd` and `popd`:

	pushd ..\..\..\Babel\Project
	babel -lang asp -output ..\..\webserver\scripts\project\gen-asp -inc -model -client product.babel service.babel
	popd 

### Can Babel generate code for all the dependencies too?

Yes, try the `-inc` command-line switch.

### How do I find out what additional options are available for each language?

We will add this to the command-line help (`-help` switch), but it can be found in the documenation for each language.

* [Java](https://github.com/babelrpc/lib-java)
* [C#](https://github.com/babelrpc/lib-csharp)
* [ASP](https://github.com/babelrpc/lib-asp)
* [Go](https://github.com/babelrpc/lib-go)
* [Node.js](https://github.com/babelrpc/lib-js)

JSON
----

### Why are some integers or numbers quoted?

Some languages cannot process numbers as big as a `decimal` or `int64` value. This includes JavaScript. To be safe, these values are quoted and can be treated as strings.

### Why do dates always have a time zone indicator?

Babel is designed to pass data between servers. We are surely not unique in having an environment that operates across multiple time zones. The `datetime` type is always passed as a formatted string including either `Z` for UTC or a time zone offset value. This allows the recipient to convert the time into its own time zone.

To pass a date without a time zone, we recommend that you format it the same way, minus the time zone indicator, and place it into a `string`. In this was neither server or client interprets the time, and it's up the code how to handle it.

### Why can't I just return the binary data as-is?

We are considering other content-types for image-like solutions in a future version. In the current version, binary data is encoded as base64 and returned in a string.

XML
---

### Why doesn't Babel support XML (yet)?

We designed Babel to support multiple encoding types, called "protocols". We felt that XML is too large, not modern enough, and introduces a lot of serialization ambiguity. It is on the list to be considered in a future version of Babel, but unlikely.

ASP
---

### ASP has a ToJSON() method in models - is it stable?

Yes, ToJSON generates the exact same JSON used by Babel internally.

### ASP has a ToXML() method in models - is it stable?

No, ToXML is not stable because Babel's XML format has not been decided. Use it at your own risk.

C# (.Net)
---------

### How do I use dates in .Net?

.Net's `DateTime` class may or may not have time zone information in it. Depending on how a `DateTime` is created, it will have a different setting.

	DateTime d1 = DateTime.Now;                                    // DateTimeKind.Local
	DateTime d2 = DateTime.UtcNow;                                 // DateTimeKind.Utc
	DateTime d3 = dbreader["DATETIME_COLUMN"];                     // DateTimeKind.Unspecified
	DateTime d4 = DateTime.Parse("2013-09-17T17:01:02.003-04:00"); // DateTimeKind.Local
	DateTime d5 = DateTime.Parse("2013-09-17T17:01:02.003Z");      // DateTimeKind.Local (yes, local!)
	DateTime d6 = DateTime.Parse("2013-09-17T17:01:02.003");       // DateTimeKind.Unspecified

Generally, a `DateTime` object created from a database column or by parsing any date without a time zone indicator results in a having an unspecified kind. To use such a date with Babel, you must use `DateTime.SpecifyKind` to indicate whether the date is local or UTC.

### What is DateTime.SpecifyKind?

`DateTime.SpecifyKind` is a .Net fuction used to specify what kind of time zone offset is associated with a `DateTime` object. Options are `DateTimeKind.Unspecified`, `DateTimeKind.Utc`, and `DateTimeKind.Local`. The function **does not** change the date value or perform any kind of time zone conversion.

### Can I use DateTimeOffset in C#?

`DateTimeOffset` is a new .Net type that specifies a time and the time zone offset. This corresponds to a new SQL Server type. You can use this type, but Babel does not generate models using it.

When assigning to a Babel model, use one of these properties of `DateTimeOffset`:

	DateTimeOffset X = ...;        // populate it somehow
	model.Date1 = X.LocalDateTime; // DateTimeKind.Local
	model.Date2 = X.UtcDateTime;   // DateTimeKind.Utc

Those properties properly populate the `Kind` member of `DateTime`.

### Can I use built-in enumerations?

No. The enumerations available to C# are not available in other languages. We want to build a service foundation that doesn't depend on specific language features. Who knows, we might rewrite your service in a better language someday!

Java
----

### Why does Babel generate those enumeration classes?

Java enumerations don't have an ordinal value associated with them. The generate class is a standard Java pattern for dealing with this.

Service Design
--------------

### Why should I use headers?

All Babel clients and server code support the concept of headers, which are string key/value pairs. When using the HTTP transport, headers are implimented simply as native HTTP headers. All Babel transports will support headers.

Headers are ideal for metadata about messages or authorization information:

* Information to control logging and metrics
* [OAuth](http://oauth.net/) tokens or [JWTs](http://jwt.io/)
* [OpenID](http://openid.net/) information
* Caller or user information

Headers should usually not be used to transmit required components of the message itself.

### Where can I learn more about versioning?

The [best practices guide](bestpractices.md) discusses versioning issues in depth. It's worth a read.

### Should I create request and response wrappers for all my service methods?

The [best practices guide](bestpractices.md) discusses response and request wrappers.

"Wrapper structs" make sense when you have a number of parameters to pass, and expect to be adding to it continually. For example, _search parameters_ could include a variety of things, and it's likely that you'll add more. Rather than introducing new methods or changing a method signature, it's cleaner to use a `struct`.

Creating a massive, single request wrapper struct could be done, but isn't that clean without polymorphism. Another option is to have a context parameter in your methods, rather than a request wrapper. You'd include the context as the first parameter of every method. It makes the method parameters much easier to understand, and ensures that people are passing the right things to the right endpoints. (Embedding message data in a wrapper would make it hard to understand what the method expects you to provide.) If your context is very small, you could also use headers.

Response wrappers are often a good idea. Often, we want to add information to data we return. Returning singular values like strings means you cannot decide to add another value to the response - you'd have to version your service method. If you had returned a struct, it's pretty easy to add something new.

### Why doesn't polymorphism work?

The main [Babel documentation](babel.md) discusses it at length. Basically, Polymorphism is not supported by all languages and is difficult to implement in JSON. It's also not present in other toolkits like Thrift or Protocol Buffers - we learned from others. There are a handful of cases where it seems like polymorphism is a good thing, but generally exposing that much of your service implementation details probably causes lock-in that you don't want.

### Can I use inheritance?

Yes, Babel supports the `extends` and `abstract` keywords. But, polymorphism is not supported - a struct member or method parameter can only be a single type.
