---
permalink: /history
title: Revision History
layout: post
---

Version 1 Features
------------------

* Babel attributes are read by the parser but not used for anything.
* Scoped attributes are passed onto the code generation when the scope is enabled on the command-line.
* Only supports JSON. XML is experimental and unlikely to be developed further.
* URL contains service name and method name, and only uses POST.
* "as" keyword has no function (needed for XML).
* Only supports C#, Java, ASP, Go, and Node.js
* Composition supported with "extends" keyword, but polymorphism is not.
* "Abstract" keyword allows generated classes to be abstract and prevents them from being declared in structs or used in methods.
* Error handling is done by standardized Babel exceptions and a standard error JSON format.
* Coding standards will be provided.
* Structure initializers can be used on basic types.
* Parameter initializers should not be used.
* All fields are nullable.
* Test code generation - try me docs
