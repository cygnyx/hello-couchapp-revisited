# hello-couchapp-revisted

The couchapp version of Hello, World!
written January 2014 by [cygnyx](https://github.com/cygnyx)

# Introduction

This is a step-by-step guide to creating a 'Hello, World!' pure
[CouchDB](http://couchdb.apache.org) application based on version 1.6.1.
This version is a variation of [hello-again-couchapp](https://github.com/cygnyx/hello-again-couchapp).
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
           "to": "_show/field",
           "query": {
               "name": "index.html"
           }
       },
       {
           "from": "/index.html",
           "to": "_show/field",
           "query": {
               "name": "index.html"
           }
       },
       {
           "from": "/style/:name",
           "to": "_show/field",
           "query": {
               "name": ":name",
               "type": "text/css"
           }
       },
       {
           "from": "/script/:name",
           "to": "_show/field",
           "query": {
               "name": ":name",
               "type": "text/javascript"
           }
       },
       {
           "from": "*",
           "to": "_show/field",
           "query": {
               "name": "notfound.html"
           }
       }
   ],
   "shows": {
       "field": "function(doc, req) { var q=req.query; return {body: this[q.name] || '', headers: {'Content-Type': q.type || 'text/html'}}}"
   },
   "index.html": "<!DOCTYPE html>\n<html lang=\"en\" class=\"\">\n<head>\n<meta charset=\"utf-8\">\n<meta http-equiv=\"Content-Language\" content=\"en\">\n<meta name=\"viewport\" content=\"initial-scale=1\">\n<link rel=\"stylesheet\" href=\"style/main.css\" type=\"text/css\">\n<script src=\"https://code.jquery.com/jquery-2.1.3.min.js\"></script>\n<script src=\"script/app.js\"></script>\n</head>\n<body>\n</body>\n</html>\n",
   "notfound.html": "<!DOCTYPE html>\n<html lang=\"en\" class=\"\">\n<head>\n<meta charset=\"utf-8\">\n<meta http-equiv=\"Content-Language\" content=\"en\">\n<meta name=\"viewport\" content=\"initial-scale=1\">\n<link rel=\"stylesheet\" href=\"style/main.css\" type=\"text/css\">\n<title>Not Found</title>\n</head>\n<body>\n<h1>Not Found</h1>\n</body>\n</html>\n",
   "main.css": "body { background-color: ivory; }\n\nh1 { text-align: center; padding: 4em; }\n",
   "app.js": "(function() {\n  document.title = 'Hello, World!'\n  document.body.innerHTML = '<h1>' + document.title + '</h1>'\n})()\n"
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
translating URLs into URIs.
These rules are located in the field named `rewrites`.
The structure of these rules is defined by
`CouchDB`.
The first rules translate `/` and `/index.html` to a reference to
`_show/field` with a name query parameter.
The next 2 rules translate the paths `/style` and `/script` to
`_show/field` with the name parameter and a type of `text/css` and
`text/javascript`, respectively.
The final rule translates all others paths to `_show/field` with
a name query parameter.
All of the rewrites go to the same `show` function called `field`.

We add an anonymous javascript function to process the `_show/field`
command in the field named `shows`.
`field` extracts a single field from the design document and
identifies the appropriate type: style documents are `text/css` and
script documents are `text/javascript`.

That completes the server side logic of this application.
The remainder of the application relies on the browser.

The field `notfound.html` has a complete HTML document for missing pages.

The field `index.html` has the HTML boilerplate for the application itself.
Note that the `head` section doesn't have a `title` and the `body` is empty.
But the document makes 2 requests back to the server for the `main.css` and
`app.js`. The CSS applies some simple formatting and the script adds
the title and fills in the body of the document.

Finally, the app is wired together by adding an entry to the vhosts
configuration. This routes anything with the hostname 'hello' to
/hello/_design/app/_rewrite. The prefix is the fully qualified path
to the design document we created. The suffix _rewrite is a system
identifier that will process URLs and convert them into URIs based on
the rules in the design document.

## Next

These steps build a simple Hello, World! website with an emphasis
on having the browser handle more of the work.
Adding more complex URL rewriting rules can make documents part of
your website. The principal idea of a `CouchApp` is to provide enough
infrastructure to deliver HTML files that are needed by
websites to enable dynamic access to the data documents in `CouchDB`.
Since `CouchDB` uses HTTP protocol and JSON documents, it works well
with Ajax based websites.

In this example, the JSON design document is used for 2 purposes:
interfacing with the `CouchApp` API and packaging source files.
The fields used for the API are: `_id`, `_rev`, `rewrites`, `shows`.
All the other fields are text documents encoded in a JSON object
that package the application. For standalone applications,
a reasonable development method
would be to have these text documents on the local file system
and then cut-and-paste them to deploy them on `CouchDB`.

Futon provides very limited editing capabilities. Nevertheless it
is possible to use it to develop simple applications.
