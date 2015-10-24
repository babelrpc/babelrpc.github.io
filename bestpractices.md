---
title: Best Practices
layout: post
---

![Babel](media/logo.png)

Best Practices
==============

We aim to provide robust web services that are:

* Well documented
* Based on an API contract
* Use coarse-grained APIs
* Have minimal dependencies
* Scale horizontally and are fault-tolerant
* Can be deployed by pod or region
* Hide and encapsulate critical resources like databases
* Have robust security
* Have a robust versioning process to simplify deployment
* Avoid locking into user models and other conventions
* Utilize asynchronous operations where appropriate

Following this best practices guide will help you deliver good web services based on Babel - but you will also need to consider the topics above.

Syntax and Naming Conventions
-----------------------------

### File Naming

Model files should be named after their primary `struct`, `enum`, `const`, or purpose, **without** the word "Model".
Service files should be named after their primary `service` or purpose, ending with the word "Service".

	Discounts.babel
	DiscountService.babel

#### Importing Babel Files

Give some thought to your include structure. Mixing request and response types around different files could lead to
duplicated includes. In general, includes should be in the following layers:

	* Services (topmost)
	* Response structs
	* Request structs
	* Common structs
	* Common enums and consts (lowest)

### Enums

Enumerations should be upper-camel cased and values should be either uppercase or upper-camel cased.

	enum TripState {
		RESERVED = 1,
		TICKETED = 2,
		WITHDRAWN = 3
	}

	enum ButtonState {
		Off = 1,
		On = 2
	}

Although Babel allows multiple separators to be used with enums, we recommend commas for readability.

### Consts

Consts should be upper-camel cased and values should be either uppercase or upper-camel cased.

	const StateNames {
		WA = "Washington";
		VA = "Virginia";
		MN = "Minnesota";
		TX = "Texas";
		GA = "Georgia";
	}

	const UserNames {
		MikeLore = "michaell";
		CraigLenzen = "craigl";
		ViktorStolbovoy = "viktors";
	}

Although Babel allows multiple separators to be used with consts, we recommend semicolons for readability.

### Structs

Structs should be upper-camel cased and fields should be upper-camel cased.

	struct LogEntry {
		datetime Date;
		string Message;
		string UserName = UserNames.MikeLore;
	}

Although Babel allows multiple separators to be used with structs, we recommend semicolons for readability.

### Services

Services should be upper-camel cased and methods should be upper-camel cased. Services should end in the word "Service".
Parameters shold be lower-camel case.

	service LogService {
		bool LogMessage(LogEntry myLogEntry);
		list<LogEntry> Top(int32 numOfResults);
	}

Although Babel allows multiple separators to be used with services, we recommend semicolons for readability.

### Naming Conventions and Babel Output

For some languages babel will modify a field name to be consistent with the syntax of the language. As an example,
in Go a field must start with a capital letter to be public (which is also the Babel best practice). Thus, a lower-cased
field needs to be modified to work with the language.

### Nitpicky

These won't fail code review, but...

* Braces should be C-style with the brace on the same line as the definition.
* Fields and methods should be indented with a single tab.
* Only use tabs.

Comments
--------

Documentation comments should be provided for all elements. The following is an example.

	/// File Header Comment
	/// Often contains your Copyright (C) CoolCompany, Inc.

	/// Valid states of a button
	enum ButtonStates {
		OFF = 0,
		ON = 1
	}

	/// Names of companies
	const CompanyNames {
		Concur = "Concur";
		Outtask = "Outtask";
		TripIt = "TripIt";
		GDSX = "GDSX";
		TRX = "TRX";
	}

	/// System-wide definition of a user
	struct User {
		/// The global user identifier. This never changes.
		int64 ID;

		/// The login identifier
		string LoginID;
	}

	/// The user service provided access to the global user database
	/// and supports multiple data centers.
	service UserService {
		/// Gets a user by ID
		User GetUser(
			/// The user identifier of the user to fetch
			int64 id);

		/// Returns the list of users matching the given criteria.
		list<User> FindUsers(
			/// Specifies the search criteria for users
			SearchCriteria criteria,
			/// Maximum number of results
			int32 maxResults
		)
	}

Note that parameters in methods can (and should) be individually commented.

Any Babel file lacking documentation should be rejected in code review. It is extremely import that service developers
and clients have a common set of documentation to work from. The documentation is automatically generated into the output
code (for supported languages).

Of course, regular comments are also welcome, but these are not added to the output of Babel.

Choosing Data Types
-------------------

After you read the next section on versioning, you will probably understand why it is important to choose your data types correctly the first time. It's very challenging to change types later on, when many callers depend on your service.

### Numbers

When choosing numeric types, think carefully about their size. An `int32` may be enough today, but is very hard to change it later when you scale out your system. On the other hand, choosing a type much larger than need wastes space.

For this reason, many of the data types are not abbreviated. For instance, there is no "int" - only `byte`, `int16`, `int32`, or `int64`.

### Booleans

Avoid using strings, numbers, or enumerations where `bool` is more appropriate. "T" and "F" values get confusing to callers.

### Strings

A `string` doesn't have a specific limit in size, but your database and service likely does. The `char` type should only be a single character, and should not be used where an enumeration is more appropriate.

### Enumerations

Enumerations are serialized to JSON using their string representations. In some cases, versioning can be a challenge. Use enumerations for relatively short lists of things that are not very dynamic.

### Date-time

Be very careful about time zones when transmitting times. Many developers commonly ignore the issue, assuming either GMT or local time. As the environment grows, however, we end up with calls from remote data centers or other users in a different time zone.

The time libraries in most programming languages typically store time relative to either GMT or the system's local time. Because of this, the time zone offset provided is usually not retained. A time transmitted as 4pm from a client in eastern time correctly shows up as 1pm at the server in pacific time.

Some programming problems, like travel segments, use time values that are always relative to the city or time zone they belong to.

The `datetime` type will transmit the time with a time zone offset indicator. The client or server, when receiving a time, will parse that value and, in most cases, convert it to either GMT or system local time.

Use the `datetime` type for specific "points in time" that can be safely made relative to GMT time. For instance, the time of a log entry is a specific moment in time no matter what time zone is used.

You should probably avoid the `datetime` type when you have a time that is relative to a specific location. For instance, a bank only accepts funds until 5pm Eastern time, or a travel agency tickets until 10pm in Seattle. These use cases cannot be safely converted to GMT time and stored because they are affected by daylight savings time.

If you can't use the `datetime` type, you should send your data in a `string` formatted like "2013-09-17T16:15:02.312" or use a `struct`.

#### Date-Only

Dates without times can be represented as a `string` formatted like "2013-09-17" or in a structure.

#### Time-Only

Time fields can be represented as a `string` formatted like "16:15:02.312" or in a structure.

### Binary

The `binary` type is serialized in JSON using a base64 string. This should be transparent to the server and client; however, keep in mind that the data passed over the wire is somewhat larger than raw binary data. Client and server can compress very large data if needed.

### Using defaults (initializers)

Babel does not guarantee that nulling out a field with an initializer will work. The JSON protocol typically omits null fields, so it won't send the nulled out value. When the server parses the incoming JSON, its structure is already initialized and the value is never reassigned. You should think of these as defaults.

Because of this, we recommend using initializers only when `null` is never an expected or acceptable value for the field. If `null` is perfectly valid, then there usually no need for an initializer.

Every field in Babel is nullable, so use initializers only when `null` is invalid. Set the initializer to the most typically expected value and don't change it later on (see the section on versioning).

In the following example, a field can be `ON` or `OFF` - the server will generate an exception is null is used.

	enum State {
		OFF = 0,
		ON = 1
	}

	struct Engine {
		State Running = State.OFF; // null doesn't mean anything
	}

#### Additional thoughts

To reduce versioning changes, allow things that work like search parameters to be `null` rather than initialized. This allows the server to determine the appropriate ranges rather than the client.

For example:

	struct Query {
		int32 minEntry = 1;
		int32 maxEntry = 16;
	}

	service DataStore {
		Results RunQuery(Query q);
	}

In theory this looks fine - the query defaults to the entire acceptable range, returning everything by default.

Now we upgrade our server and the whole range is 1-32. Oops, every client needs to be rebuilt. We've just created a problem.

A better solution is to avoid the initializers and have the service understand that a `minEntry` of `null` means 1, and a `maxEntry` of `null` means 16.

### Return Values

When returning lists or maps, it's better to return an empty `list` or empty `map` rather than retuning a `null`. This helps reduce bugs where clients don't check for the nulls, and it's more consistent with how lists and maps are initialized in a `struct`.

Passing Authentication Information
----------------------------------

All Babel client libraries support the ability to add headers. When passing authentication data (for instance, OAuth or other credentials), use the client library's methods to add headers. Do *not* put the authentication details into your `struct` or `service` defintions.

We do encourage you to document the authentication headers that are needed in the documenation comments for your `service`.

Versioning and Compatibility
----------------------------

Careful service design will lead to fewer versioning headaches. Babel's design attempts to make it easy to deal with
versioning, but no system can completey isolate a developer from the issues. It is desirable for all of our services
to have the following properties:

* An existing (old) client will continue to work when a new version of a service is deployed. The client does **not** need to be rebuilt.
* The old client code will still compile as-is when a new Babel file is released.
* It is **not** necessary for a new client to be able to call an old service deployment - although in many cases it would be possible when following the rules.

Changes to Babel files can be classified as **breaking changes** and **non-breaking changes**.

* **Breaking changes** are changes that will cause an existing client to fail or not compile.
* **Non-breaking changes** are all changes that allow existing clients to continue to work.

When breaking changes are introduced, it is import to create new versions of service methods, or even entirely new services
if the changes are severe enough to warrant it.

### General changes

Breaking changes:

* Renaming an `enum`, `const`, `struct`, or `service`.
* Removing an `enum`, `const`, `struct`, or `service`.
* Changing the namespace entries.

Non-breaking changes:

* Adding an `enum`, `const`, `struct`, or `service`.
* Adding a namespace of a new language not previously supported.

### Changing enumerations

Breaking changes:

* Removing an entry.
* Renaming an entry.
* Changing the integer value of an entry.

Non-breaking changes:

* Adding an entry. (However, your service should not return new values to old clients.)

### Changing constants

Breaking changes:

* Removing a constant.
* Renaming a constant.

Non-breaking changes:

* Adding a constant.

Potentially problematic changes:

* Changing the value of a constant - if the server code relies on the value for any logic, it can break. In general, client and server code should not make flow decisions based on a constant value (enumerations are better for that). Also, constants can be used as initializers, so changing the value of a constant can have a bigger impact than you might think.

It's safer to avoid changing the values of constants once defined.

### Changing structures

Breaking changes:

* Removing a field.
* Renaming a field.
* Changing the type of a field. (Even changing int32 to int64 is a breaking change.)

Non-breaking changes:

* Adding fields to a structure. (However, the service must expect that those new fields will be null when called from an old client.)
* Changing the documentation comment for a field.

Potentially problematic changes:

* Changing the initialzer of a field. If the service behavior is dependent on the field, you could break clients who relied on the old default value.

For instance:

	struct Operation {
		OpType Type = OpType.Add;	// changing this to OpType.Delete would give unexpected results!
	}

Also, keep in mind that client and server code both use the initialers when creating structures. If they were compiled at different times and you changed the initializer, you then have an inconsistency between server and client.

It's generally safer to avoid changing initializers once defined. Also, avoid using initializers that prescribe a certain behavior - instead force the client to specify what they want by assigning the value.

### Changing services

Breaking changes:

* Removing a method.
* Renaming a method.
* Changing the return type of a method.
* Changing the type of any parameter to the method. (Even changing int32 to int64 is a breaking change.)
* Renaming any parameter to the method.
* Removing an initializer for a parameter.
* Adding a parameter without an initializer.
* Changing the order of the paramters in any way.

Non-breaking changes:

* Adding a method.
* Changing the documentationc comments.
* Adding a parameter with an initializer **at the end** of the parameter list. (Otherwise old code won't compile.)

Potentially problematic changes:

* Changing the initialzer of a method parameter. If the service behavior is dependent on the parameter, you could break clients who relied on the old default value. (NOTE: Parameter initializers are not supported in version 1.)

For instance:

	service Foo {
		void DoSomething(OpType Type = OpType.Add);	// changing this to OpType.Delete would give unexpected results!
	}

If the service and client were compiled at different times and you changed the initializer, you then have an inconsistency between server and client.

It's generally safer to avoid changing initializers once defined. Also, avoid using initializers that prescribe a certain behavior - instead force the client to specify what they want by assigning the value.

Other Recommendations
---------------------

There are numerous service design patterns, but some up-front thinking often reduces the need to version a service method or structure.

* Rather than having 3, 5 or more paramters to a method, consider passing a single structure. This is a good pattern for "search" functions that have a number of search criteria. It's highly likely that you'll want to add additional search criteria in the future.

Example:

	service Foo {
		Result Search(int32 domain, string name, string companyRegex, string state);
		Result NewSearch(int32 domain, string name, string companyRegex, string state, string country = "US"); // initializer not supported in version 1 of babel
		Result BetterSearch(SearchCriteria criteria); // easy to add to!
	}

* Consider returning a result wrapper as opposed to a single value. It's likely that you will want to add things to the response in the future, and you'll have to version the service unless there is a wrapper to put things into.

Example:

	struct User {}
	struct UserResult {
		User User;
		string ShardID;
	}
	struct UserSearchResult {
		list<User> Users;
		string ShardID;
	}
	service Foo {
		bool IsUserActive(int64 userId);	// nice, but now I need the login ID - that's another service call
		User GetUser(int64 userId);			// better, I get all the user details in one call
		UserResult GetUser2(int64 userId);	// potentially even better, now I can include logging info, shard ID, or other interesting things.
		UserSearchResult FindUsers(SearchCriteria criteria);	// different result type which holds lists of users
	}

### Resource-oriented Design

As an RPC technology, Babel rejects RESTful services and HATEOAS in favor of the simpler and faster JSON over HTTP. However, this does not mean that resource-oriented design is bad or that Babel can't do it. Designing resource-oriented APIs often results in clean, coarse-grained APIs that scale well and don't need to version rapidly.

	struct Resource {
		/// ID of resource
		string ID;
		/// Name
		string Name;
	}

	struct Response {
		/// ID of resource
		string ID;
		/// Number of resources affected
		int32 Count;
	}

	service ResourceOriented {
		/// Get list of resources
		list<Resource> List();
		/// Get a specific resource
		Resource Get(string id);
		/// Add a new resource
		Response Post(Resource r);
		/// Replace a resource
		Response Put(string id, Resource r);
		/// Delete a resource
		Response Delete(string id);
	}

This design is a lot like message-passing. Combined with REST annotations and `babelproxy`, you can actually serve it out as a RESTful service. But even if you don't, it can be a good design.
