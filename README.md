# hello-again-couchapp

The couchapp version of Hello, World!
written January 2014 by [cygnyx](https://github.com/cygnyx)

# Introduction

This is a step-by-step guide to creating a 'Hello, World!' pure
[CouchDB](http://couchdb.apache.org) application based on version 1.6.1.
This version is a variation of [hello-couchapp](https://github.com/cygnyx/hello-couchapp).
The guide is followed by a general explanation of what is going on.

# Preliminaries

Get `CouchDB` running on your local system and verify that it is operating properly.

Edit the file '/etc/hosts' and append 'hello' to the line starting
with: '127.0.0.1'. Initially it might look like:

```
127.0.0.1 localhost  
```

And it should be changed to something like:

```
127.0.0.1 localhost hello
```

Make sure the leave a space character between the words. You may need
administrator permission to modify this file.

## Configuration

Using your web browser go to: <http://localhost:5984/_utils>.
This is the couchdb administration page called Futon.
If necessary, sign in with administrator permissions in the lower
right corner of the page.

Near the top left of the page, select 'Create Database ...' and a
modal dialog box appears with a prompt for the database name.
Enter 'hello' for its name and select 'Create'

Futon will display (the empty) contents of the newly created database.

Near the top left of the page, select 'New Document'.
Near the top right of the page, there is a 'Field' and a 'Source' tab,
select the 'Source' tab instead of 'Field'.
Double click on the text to enable editting.

Replace the default text with:

```
{
   "_id": "_design/app",
   "rewrites": [
       {
           "from": "/",
           "to": "_show/hello"
       },
       {
           "from": "/index.html",
           "to": "_show/hello"
       },
       {
           "from": "*",
           "to": "_show/notfound"
       }
   ],
   "shows": {
       "hello": "function(doc, req) { return require('main').show.call(this, req) }",
       "notfound": "function(doc, req) { return require('main').show.call(this, req) }"
   },
   "main": "!(function() {\n\n  var main = {\n    cache: {},\n    factory: function(ref, option) {\n       if (typeof ref == \"undefined\" || ref == '') return ''\n       if (main.cache[ref]) return main.cache[ref].call(this, option)\n       var str = this[ref] || option[ref] || ''\n       if (str == '') return ''\n       str = str.replace(/[\\t\\r\\n]/g, \" \")\n         .replace(/<%=/g, \"'+options.\")\n         .replace(/%>/g, \"+'\")\n       main.cache[ref] = new Function(\"options\", \"return '\" + str + \"'\")\n       return main.cache[ref].call(this, option)\n     },\n     hello: function() {\n       return main.format.call(this, this.hello)\n     },\n     notfound: function() {\n       return main.format.call(this, this.notfound)\n     },\n     format: function(title) {\n       return main.factory.call(this, 'document_template', {title: title})\n     },\n     show: function(req) {\n       return {\n         body: main[req.path[4]].call(this),\n         headers: this.responseheader\n       }\n     }\n  }\n\n  module.exports = main\n}).call(this)\n",
   "hello": "Hello, World!",
   "notfound": "404 - Document not found",
   "responseheader": {
       "Content-Type": "text/html"
   },
   "document_template": "<!DOCTYPE html>\n<html lang=\"en\" class=\"\">\n<head>\n<meta charset=\"utf-8\">\n<meta http-equiv=\"Content-Language\" content=\"en\">\n<meta name=\"viewport\" content=\"initial-scale=1\">\n<style>\nbody {\n  background-color: ivory;\n}\nh1 {\n  text-align: center;\n  padding: 4em;\n}\n</style>\n<title><%=title%></title>\n</head>\n<body>\n<h1><%=title%></h1>\n</body>\n</html>\n"
}
```

Near the top left of the page, select 'Save Document'
Futon will redisplay this document with some additional information.

On the right side of the page, select 'Configuration' from the menu.

Scroll to the bottom and on the bottom left of the page, select 'Add a
new section' and a modal dialog box appears.
Enter 'vhosts', 'hello', '/hello/_design/app/_rewrite' for section,
option, value, respectively. Select 'Create'

## Verify

In a web browser, goto: <http://hello:5984> to see the 'Hello,
World!' website.

## Explanation

To set up an effective `CouchApp`, or `CouchDB Application`, a scheme is
needed to adjust nice looking 'user' URLs into the internally
consistent couchdb URIs. The mechanism used is based on the host name of
the URL. Thereby many hosts can be located on a single `CouchDB`.

The 'hosts' file is a system file that is used to create
translations from system names to IP addresses. The line modified is
the 'loopback' entry that systems use to translate 'localhost' to your
own local system. The value 127.0.0.1 is a special IP value to
always refers to your local system. You can add many aliases for your
local system. These names do not need to have full internet domain
names like 'example.com'. In this case, we use 'hello' as an alias for
the local system. Add an additional alias for each `CouchApp`.
In Unix-like systems the file is located in the '/etc' directory.
In MS Windows the location may depend on your version.

The `CouchDB` includes a system management interface called Futon.
The URI is <http://localhost:5984/_utils>.
Since the hostname 'hello' will be re-routed, it is important to use
'localhost' here.

Initially `CouchDB` is in 'admin party' mode. That is, everyone is an
administrator and can modify the system. You will likely want to
change this behaviour and create an administrator account. If you have
created an administrator account, then you must login with it in order
to make these configuration changes. The account will logout after a
period of time and it may be necessary to login again.

We create a
database called 'hello' to hold all the data needed for the app.
The data for this app consists of the single [JSON](http://www.json.org) document.
This document contains all the elements of the app.

Near the top right of the page, there is a 'Field' and a 'Source' tab,
select the 'Field' tab instead of 'Source' to view the document fields
whiling reading this explanation.

`CouchDB` manages document oriented databases. The documents are
primarily `JSON` data with some extensions for handling 'attachments'. 
When Futon creates a new document it fills it in with some minimal
`JSON` which has the _id field. All documents must have this
field. The `_` identifies it as a system field. The value of the `_id`
is a UUID (Universally Unique IDentifier). This default `JSON` is not
appropriate this part of the app configuration.

`CouchDB` has a number of conventions on how to structure data within
itself.
At some level a database in `CouchDB` is only a set of `JSON`
documents.
For the developer, the reality is that documents are partitioned into data documents and
design documents. Data documents are the normal run-of-the-mill
documents that contain data. The design documents are important
meta-data documents. Design documents must have ids prefixed with
`_design/`. Note the `_` to identify a system feature and the trailing
`/` to identify a hierarchical structure. 

In this case, we create a design document called 'app'. Any other name
could have been used. The fully qualified path to this design document
is  `/hello/_design/app`. A single database can have many design documents.
The document also has a `_rev` field which is a monotonically increasing
number, a hypen, and a hash value combined.

We added rules for
translating URLs into URIs. The structure of these rules is defined by
`CouchDB`. These rules translate `/` and `/index.html` to a reference to
`_show/hello` and all other URLs go to `_show/notfound`.
These rules are located in the field named `rewrites`.

We added anonymous javascript functions to process the two `_show`
commands in the field named `shows`. In this case, both functions
delegrate their implementation to the `show` function in the
`CommonJS` module in the field named `main`.

`CouchDB` supports `CommonJS` modules. When you inspect the `main`
field in Futon, you will see that it is much more readable than the
original `JSON`. Futon converts between multi-line embedded strings
multi-line text fields. The `main` module includes a factory for
building template functions. The templates are scanned for the
`<%=value%>` pattern. It generates the function to look up values
in the options argument.

The `hello` and `notfound` functions delegate to `format` passing
in the text from their respective fields. `format` applies the
template to the title.
Note how text fields can be located in the
`JSON` object. 

The `show` function calls the appropriate function based on the
request path element.
Note how the entire response header field is located in the
`JSON` object.

Finally, the app is wired together by adding an entry to the vhosts
configuration. This routes anything with the hostname 'hello' to
/hello/_design/app/_rewrite. The prefix is the fully qualified path
to the design document we created. The suffix _rewrite is a system
identifier that will process URLs and convert them into URIs based on
the rules in the design document.

## Next

These steps build a simple Hello, World! website.
Adding more complex URL rewriting rules can make documents part of
your website. The principal idea of a `CouchApp` is to provide enough
infrastructure to deliver HTML files that are needed by
websites to enable dynamic access to the data documents in `CouchDB`.
Since `CouchDB` uses HTTP protocol and JSON documents, it works well
with Ajax based websites.

Since `CouchDB` supports `CommonJS` modules, entire libraries can be
included in the design document. For example a more complex application
can might use the [Mustache](https://mustache.github.io) for a template
engine by copying the mustache.min.js into the `mustache` field. Then
the template library can be used by `require('mustache')` to access it.

Special care must be used when making function calls. In the `shows`
functions, `this` refers to the design document. The `doc` parameter
refers to the `JSON` document uploaded (if any).

Futon provides very limited editting capabilities. Nevertheless it
is possible to use it to develop simple applications. Including
`CommonJS` modules is also relatively straight forward which can be
used to expand application functionality.
