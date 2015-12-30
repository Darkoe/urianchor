# uriAnchor #

## Summary ##

A jQuery plugin for management of the URI hash component.

Use this plugin to manage dependent and independent variables in the hash
fragment of the URI.  It has been designed and updated over five commercial
SPA projects and is featured in the book
[Single page web applications, JavaScript end-to-end][1]
It provides the capability to make the URI fragment your application state
API. Bookmarks, browser history, the back button, and the forward button
can all be made to act as the user expects while enabling you to update
only the part of the page that has changed.

## Preferred listener ##

The preferred listener to use with URI Anchor is `hashchange`, like so:

    // jQuery
    $(window).bind( 'hashchange', onHashChange );

    // Native
    window.onhashchange( onHashChange );

## Example implementation ##

See a full example, clone the [SPA listings](10),
dereference a copy of the 6.5 listings, and then open the web page.  Like so:

    cd spa/listings/ch05-06
    cp -aL 6.5 6.5dr
    cd 6.5dr
    google-chrome spa.html

You must log in -- any user name will work -- to open and close the chat window and activate the avitars.

## Methods ##

### $.uriAnchor.setAnchor ##

#### Purpose ####

Sets Anchor component of the URI from a Map.
The Anchor component is also known as the 'hash fragment' or 'bookmark component'.

#### Arguments ####

Arguments are positional:

- 1 ( `anchor_map` ) : The map to be encoded to the URI anchor
- 2 ( `option_map` ) : A map of options
- 3 ( `replace_flag` ) : A boolean flag.  When true, the method replaces the URI so that prior URI is not entered into the browser history

#### Environment ####

Expects the document.location browser object

#### Settings ####

none

#### Returns ####

boolean: true on success, false on failure

#### Throws ####

none

#### Details ####

The first positional argument, `anchor_map`, may be a simple map:

    $.uriAnchor.setAnchor({
      page   : 'profile',
      slider : 'confirm',
      color  : 'red'
    });

This changes the URI anchor to:

    #!page=profile&slider=confirm&color=red

All these arguments are independent, that is, they can vary
independent of each other. We also support dependent values -
values that depend on others.

An independent argument key has no `_` prefix.  The same key name,
prefixed by an `_`, holds the arguments that are dependent on
an independent argument. The dependent key always points
to a map. Consider:

    $.uriAnchor.setAnchor({
      page   : 'profile',
      _page  : {
        uname   : 'wendy',
        online  : 'today'
      }
    });

This changes the URI Anchor to:

    #!page=profile:uname,wendy|online,today

Only independent keys and their matching dependent keys are
processed.  All other keys are ignored.  Importantly, this includes
keys of the form `_s_<key>` ( e.g. `_s_page` ) returned by makeAnchorMap

Setting a more complex anchor map is illustrated below:

    $.uriAnchor.setAnchor({
      page : 'profile',
      _page : {
        uname   : 'wendy',
        online  : 'today'
      },
      slider  : 'confirm',
      _slider : {
       text   : 'hello',
       pretty : false
      },
      color : 'red'
    });

This sets the URI Anchor to:

     #!page=profile:uname,wendy|online,today&slider=confirm:text,hello\
       |pretty,false&color=red

Options: The second positional argument tp this method, `option_map`,
provides a number of options for delimiters:

- `delimit_char`   : Delimiter between independent args. Default is `&`.
- `delimit_kv_char`: Delimiter between key and value of independent args.  Default is `=`.
- `sub_delimit_char` : Delimiter between independent and dependent args. Defaults is `:`.
- `dep_delimit_char` : Delimiter between key-value pairs in dependent args. Default is `|`.
- `dep_kv_delimit_char` : Delimiter between key and value of dependent args.  Default is ','

Boolean values ( as part of a key-value pair ) are convert into the stings 'true' or 'false'.

Validation:

As of 1.0, the ability to optionally check the validity of the
Anchor against a schema has been included.  Since we don't expect
the allowable schema to change during run-time, we use a
module configuration to set the schema, like so:

    $.uriAnchor.configModule({
      schema_map : {
        page    : { profile : true, pdf : true },
        _page   : {
          uname   : true,
          online  : [ 'today','yesterday','earlier' ]
        },
        slider  : { confirm : 'deny' },
        _slider : { text : 'goodbye' },
        color   : { red : true, green : true, blue : true }
      }
    });

This check occurs only during setting of the Anchor, not
during its parsing ( See makeAnchorMap )

The `replace_flag` instructs the routine to replace the URI,
discarding browser history

### $.uriAnchor.makeAnchorMap

#### Purpose

Parses URI anchor and returns as map

#### Arguments

none

#### Environment

Expects the document.location browser object

#### Settings

none

#### Returns

Map

#### Throws

none


#### Details

Parses the browser URI anchor into a map using the same
rules used to set the anchor in the method setAnchor
( see above ).

This method creates an additional key type, `_s_<independent_arg>`
for each independent argument with dependent arguments.

These keys point to a string representation of the independent
argument along with all its dependent arguments.

These values are ignored by setAnchor, but they are useful
for routines using setAnchor to check if a part of the anchor
has changed.

#### Example

If the browser URI Anchor looks like this:

    #!page=profile:uname,wendy|online,true&slider=confirm:text,hello\
      |pretty,false&color=red

Then calling $.uriAnchor.makeAnchorMap(); will return a map that looks like so:

    { page : 'profile',
      _page : {
        uname   : 'wendy',
        online  : 'true'
      },
      _s_page : 'profile:uname,wendy|online,true',
      slider  : 'confirm',
      _slider : {
       text   : 'hello',
       pretty : false
      },
      _s_slider : 'confirm:text,hello|pretty,false',
      color : 'red'
    };

## Avoid complex 'SPA framework' libraries ##

jQuery used with this and a few other well-chosen tools forms
a fantastic basis for a lean, easy to use SPA architecture
as detailed in [Single page web applications, JavaScript end-to-end][1].
Here are the recommended tools:

| Capability   | Tool                | Notes                             |
| ------------ | ------------------- | ----------------------------------|
| Websockets   | [Socket.io][6]      | Prefer websockets over AJAX.      |
| AJAX         | jQuery native       | Use jQuery AJAX methods.          |
| Promises     | jQuery native       | Use jQuery promise methods.       |
| Model Events | [Global Events][2]  | jQuery plugin eliminates having   |
|              |                     | to manage multiple event types.   |
| Touch        | [Unified events][3] | Unify desktop and touch events.   |
| Routing      | [uriAnchor][4]      | jQuery plugin for robust routing. |
|              |                     | Includes support for dependent    |
|              |                     | and independent query arguments.  |
| Data Model   | [taffyDB][5]        | A powerful and flexible SQL-like  |
|              |                     | client data management tool.      |
| SVG          | [D3][7]             | Great for easy graphs and charts  |
|              | [SVG][8]            | Low-level jQuery plugin           |
| Templates    | [Dust][9]           | Uses a powerful template DSL that |
|              |                     | minimizes chances to intemingle   |
|              |                     | business and display logic.       |

This suite of tools has all the capabilities of a bleeding-edge
SPA "framework" library within the reliable and mature jQuery ecosystem.
It can provide an application that is significantly more flexible and
testable since display logic can easily be decoupled from business logic.
Finally, it leverages jQuery's maturity, performance, and excellent
tools instead of competing with them.

## Release Notes ##

### Copyright (c)###
2013 Michael S. Mikowski (mike[dot]mikowski[at]gmail[dotcom])

### License ###
Dual licensed under the MIT or GPL Version 2
http://jquery.org/license

### Versions 1.1.0-3 ###
These are the first releases registered with jQuery plugins.

### Versions 1.2.0-2, 1.3.0-1 ###
Updated documentation, fixed minor bug.

### Version 1.3.2 ###
Added example to show use of hashchange listener.

### Version 1.3.3 ###
Added changeAnchorPart example code

## TODO ##

- Maybe reconsider the structure of dependent and indepent variables

## Contribute! ##

If you want to help out, like all jQuery plugins this is hosted at
GitHub. Any improvements or suggestions are welcome!
You can reach me at mike[dot]mikowski[at]gmail[dotcom].

## END ##

[1]:http://manning.com/mikowski
[2]:https://github.com/mmikowski/jquery.event.gevent
[3]:https://github.com/mmikowski/jquery.event.ue
[4]:https://github.com/mmikowski/urianchor
[5]:https://github.com/typicaljoe/taffydb
[6]:http://socket.io
[7]:https://github.com/mbostock/d3
[8]:http://keith-wood.name/svg.html
[9]:http://linkedin.github.io/dustjs
[10]:https://github.com/mmikowski/spa
