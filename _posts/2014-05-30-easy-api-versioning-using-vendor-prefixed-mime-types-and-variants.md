---
layout: post
title: "Easy API Versioning using Vendor-Prefixed MIME Types and Rails Action Pack Variants"
date: 2014-05-30 23:37:00
categories: apis
tags:
- ruby
- rails
- api
---

Sick of having API versioned paths in your rails app? Try using Action Pack Variants instead.


The following is a simple example on how to implement this functionality with a Rails 4.1 application.

## Set up your routes
Configure your routes as you would for a JSON endpoint.
{% highlight ruby linenos %}
# config/routes.rb
Rails.application.routes.draw do
  scope(format: :json) do
    get :hello, controller: :sandbox
  end
end
{% endhighlight %}

## Define your custom Vendor MIME type
Try to use a definition that has very little chance of interfering with other possible API services.  Here, I use the java naming notation (com.mydomain.api) to associate the MIME type specifically with my domain.

{% highlight ruby linenos %}
# config/initializers/mime_types.rb
# Define a custom Vendor MIME type
Mime::Type.register "application/com.mydomain.api-v1+json", :v1json
{% endhighlight %}

## request.variant
It is now a simple matter of setting the request.variant to the proper variant name for the new MIME format (lines 6-8).

{% highlight ruby linenos %}
# app/controllers/sandbox_controller.rb
class SandboxController < ApplicationController
  def hello
    respond_to do |format|
      format.json # hello.jbuilder
      format.v1json do
        request.variant = :v1 # hello+v1.jbuilder
      end
    end
  end
end
{% endhighlight %}

## Define your variant views
Since we used :v1 as our variant name, we will need a hello+v1.jbuilder variant to go along with it.

{% highlight ruby linenos %}
# app/views/sandbox/hello.jbuilder
json.hello "from default json"

# app/views/sandbox/hello+v1.jbuilder
json.hello "from v1 json variant"
{% endhighlight %}

## Setup your requests
Finally, your requests will need to send the proper Content-Type header to request the correct response variant.
{% highlight javascript linenos %}
// GET [Content-Type:"application/json"] /sandbox/hello
{ "hello":"from default json" }

// GET [Content-Type:"application/com.mydomain.api-v1+json"] /sandbox/hello
{ "hello":"from v1 json variant" }
{% endhighlight %}

## Benefits
A huge benefit of doing this way would be the lack of needing to maintain multiple nested, versioned routes and their associated controllers and views.  This reduces your code footprint as well as routing complexity.  Simply modify your output based on the versioned MIME requested.

{% include signature.md %}
