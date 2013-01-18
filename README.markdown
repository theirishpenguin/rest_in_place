Fork's Purpose
==============
* To work around https://github.com/janv/rest_in_place/issues/33 ("After R.I.P. submission the updated field is injected into the html without any escaping")
* To work around https://github.com/janv/rest_in_place/issues/34 ("Truncation on editing a string containing a double quote")
* Fixes issue https://github.com/janv/rest_in_place/issues/36 ("Doesn't save input fields on tabbing out of input box")
* Note: No jasmine tests added for these issues

REST in Place
===========
                                  _______
                                 /       \
                                 | R.I.P.|
                                 |       |
                                 |       |
                               -------------

REST in Place is an AJAX Inplace-Editor that talks to RESTful controllers.
It requires absolutely no additional server-side code if your controller
fulfills the following REST preconditions:

-   It uses the HTTP PUT method to update a record
-   It delivers an object in JSON form for requests with
    "Accept: application/json" headers

The editor works by PUTting the updated value to the server and GETting the
updated record afterwards to display the updated value.
That way any authentication methods or otherwise funky workflows in your
controllers are used for the inplace-editors requests.

To save the additional GET request, you can take the shortcut of returning the
updated record in the response to the PUT request. See the testapp for an
example.

URL:         <http://github.com/janv/rest_in_place/>  
REPOSITORY:  git://github.com/janv/rest_in_place.git  
BLOG:        <http://jan.varwig.org/projects/rest-in-place>

If you like REST in Place, you can flattr me: <a href="http://flattr.com/thing/1984/REST-in-Place" target="_blank">
<img src="http://api.flattr.com/button/flattr-badge-large.png" alt="Flattr this" title="Flattr this" border="0" /></a>

Requirements
============

Rails
-----

Since I guess most people use REST in Place in Rails apps, I turned this
entire thing into a gem that you can require in your Gemfile. It requires
jQuery, but it will NOT install a `jquery-rails` dependency. This is done so
you aren't forced to use `jquery-rails` if you want to run a more up-to-date
version of jQuery. Just make sure that jQuery is there.

REST in Place requires Rails >= 3.1 as a dependency since it loads through the
asset pipeline.

CoffeeScript
------------

The CoffeeScript code (`lib/assets/javascripts/rest_in_place/rest_in_place.coffee.erb`)
only relies on the presence of jQuery. You can extract just that file and use
it with whatever framework in whatever server-side language you want, given
that you follow the coventions described later in this document.

Even though this is processed by ERB to sniff out some relevant Rails settings,
you can use it as a CoffeeScript file without modification. (This feature
might vanish at any time in the future, tying RIP closer to Rails).

JavaScript only
---------------

If you're still using JavaScript, give [CoffeeScript](http://jashkenas.github.com/coffee-script/)
a try. It's a preprocessor/different syntax, that makes writing JavaScript
bearable. If you absolutely don't want to learn anything new, just convert
REST in Place to JavaScript using http://js2coffee.org/ before including it in
your project.

Installation
============

Just add

    gem 'rest_in_place'

to your Gemfile.

Then load the JavaScript by adding <%= javascript_include_tag "rest_in_place" %>
into your layout. Alternatively you can require 'rest_in_place' in your
JavaScript files in `app/assets`, for example in your application.js:

    //= require 'rest_in_place'

In both cases, make sure you load REST in Place __after__ jQuery.

Rails Request Forgery Protection
================================

For REST in Place to work with Rails request forgery protection, you need to
have the Rails CSRF meta tags in your HTML head:

    <%= csrf_meta_tags %>

Usage Instructions
==================

To make a piece of Text inplace-editable, wrap it into an element (a span
usually) with class "rest-in-place". The editor needs 3 pieces of information
to work: a URL, an object name and the attribute name. These are provided as
follows:

-   put attributes into the element, like this:
    
        <span class="rest-in-place" data-url="/users/1" data-object="user" data-attribute="name">
          <%= @user.name %>
        </span>
  
-   if any of these attributes is missing, DOM parents of the element are searched
    for them. That means you can write something like:
    
        <div data-object="user" data-url="/users/1">
          Name:  <span class="rest-in-place" data-attribute="name" ><%= @user.name %></span><br/>
          eMail: <span class="rest-in-place" data-attribute="email"><%= @user.email %></span>
        </div>
    
-   You can completely omit the url to use the current document's url.
    With proper RESTful controllers this should always work, the explicit
    url-attribute is for cases when you want to edit a resource that is
    displayed as part of a non-RESTful webpage.
  
-   Rails provides the dom_id helper that constructs a dom id out of an
    ActiveRecord for you. So, your HTML page may look like this:
    
        <div id="<%= dom_id @user # == "user_1" %>">
          Name:  <span class="rest-in-place" data-attribute="name" ><%= @user.name %></span><br/>
          eMail: <span class="rest-in-place" data-attribute="email"><%= @user.email %></span>
        </div>
    
    REST in Place recognizes dom_ids of this form and derives the object parameter
    from them, so that (with the current documents url used) you really only need
    to provide the attributes name in most cases.
   
    **Note that a manually defined (in the element or in one of the parents)
    object always overrides dom_id recognition.**

-   REST in Place supports multiple form types. The default type is a input
    field, a textarea is also supported. To select a form type use the
    `data-formtype` attribute in the element and set it to the name of your
    formtype ( `input`, or `textarea` ).
    
    To write your own form types, just extend the `RestInPlace.forms` object
    and select your new form type throught the `data-formtype` attribute.

Elements with the class `rest-in-place` are picked up automatically upon
`document.ready`. For other elements, grab them via jQuery and call
restInPlace() on the jQuery object.

    $('.my-custom-class').restInPlace()

Events
------

A REST in Place instance triggers four different events on the element that
it's associated with:

- `activate.rest-in-place` when starting the editing of the element.  
  Triggering the event is the first thing that happens, before any processing
  and form building takes place. That means uou can use this event to modify
  the content of the element (for example to remove number/date formatting).
- `success.rest-in-place` with the data retrieved from the server as an
  extra parameter after a successful save on the server.  
  This event is triggered at the very latest moment, after the element has
  been restored with the data from the server. This means you can use the
  event handler to further modify the data and overwrite the displayed value
  (useful for number/date formatting for example).
- `failure.rest-in-place` after an error occured
- `update.rest-in-place` immediately before sending the update to the server
- `abort.rest-in-place` when the user aborts the editing process.

Bind to these events through the jQuery event mechanisms:

    $('#my-editable-element').bind('success.rest-in-place', function(event, data){
      console.log("Yay it worked! The new value is", data.whatever);
    });
    

Example
=======

Your routes.rb:

    resources :users
  
Your app/controllers/users_controller.rb:

    class UsersController < ApplicationController
      def show
        @user = User.find params[:id]
        respond_to do |type|
          type.html
          type.json {render :json => @user}
        end
      end

      def update
        @user = User.find params[:id]
        if @user.update_attributes!(params[:user])
          respond_to do |format|
            format.html { redirect_to( @person )}
            format.json { render :json => @user }
          end
        else
          respond_to do |format|
            format.html { render :action  => :edit } # edit.html.erb
            format.json { render :nothing =>  true }
          end
        end
      end
    end

Your app/views/users/show.html.erb:

    <div id="<%= dom_id @user %>">
      ID: <%= @user.id %><br />
      Name: <span class="rest-in-place" data-formtype="input" data-attribute="name"><%= @user.name %></span><br/><br/>
      Hobbies: <span class="rest-in-place" data-formtype="textarea" data-attribute="hobbies"><%= @user.hobbies %></span>
    </div>

You can run this example by running to the testapp included in this
plugin with script/server (sqlite3 required) and visiting
localhost:3000/users/

Hint:  
you might need to set up the database first.
Copy and edit `testapp/config/database.yml.sample` accordingly.
If you don't want to use the included sqlite3 database, run `rake db:create`
and `rake db:schema:load`.

Troubleshooting
===============

REST in Place is very picky about correct headers and formatting.
If you experience errors, please make sure your controller sends responses as
properly formatted JSON with the correct MIME-Type "application/json".

Testing
=======

The repository contains a `testapp` directory with a rails app that can be
used to test REST in Place. Just head to `http://localhost:3000` to see it in
action.

There's also an automated Jasmine spec suite running at
`http://localhost:3000/jasmine`. You can find the sources for the specs at
`testapp/app/assets/javascripts/spec.coffee`

Participation
=============

I'd love to get comments, bug reports (or better, pull-requests) about REST in
Place. For this, you can either fork the project to send a pull request, or
submit a bug in the tracker at github:
<http://github.com/janv/rest_in_place/issues>

For general comments and questions, please use the comment function on my blog:
<http://jan.varwig.org/projects/rest-in-place>

If you send pull requests be sure to also add tests and make sure the existing
tests pass.


---
Copyright (c) 2012 [Jan Varwig], released under the MIT license
