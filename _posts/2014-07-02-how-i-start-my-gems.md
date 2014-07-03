---
layout: post
title: "How I Start My Gems"
date: 2014-07-02 22:54:00
category: ruby
tags:
- ruby
- gems
- howto
links:
  - "Railscast #245: New Gem with Bundler": http://railscasts.com/episodes/245-new-gem-with-bundler
  - "7 Lines Every Gem's Rakefile Should Have": http://erniemiller.org/2014/02/05/7-lines-every-gems-rakefile-should-have/
  - "Gemspec Reference": http://guides.rubygems.org/specification-reference/
---


## Introduction
I've built quite a few rubygems and I'd like to share how I begin my gems nowadays.

For the purposes of this article, assume that we want to build a gem called ```bad_math```.
This article will not cover how to _build_ a gem, merely how to make it easier on the developer.
If you've not built a gem before, I recommend watching [Railscast #245: New Gem with Bundler](http://railscasts.com/episodes/245-new-gem-with-bundler).

## Prerequisites
1. ```bundler```

## Generate Gem Template
Yes, I _could_ build the gem directory/file structure manually, but that's too slow, and bundler makes it hella easy.
(I do recommend doing it manually at least once to appreciate what bundler does for you.)

Running ```bundle gem bad_math``` will output something similar to the following:

{% highlight bash %}
[~/workspace/gems] $ bundle gem bad_math
     create  bad_math/Gemfile
     create  bad_math/Rakefile
     create  bad_math/LICENSE.txt
     create  bad_math/README.md
     create  bad_math/.gitignore
     create  bad_math/bad_math.gemspec
     create  bad_math/lib/bad_math.rb
     create  bad_math/lib/bad_math/version.rb
Initializing git repo in ~/workspace/gems/bad_math
{% endhighlight %}
Look at that! I've got a template and all I have to do is fill in the blanks and start coding.

## Modifying the gemspec
For additional reading, you can view the gem [specification reference](http://guides.rubygems.org/specification-reference/) to see all of the various
configuration options that are available to you within this file.  For the sake of this article, I'm only concerned with adding
[```yard```](http://rubygems.org/gems/yard) and [```redcarpet```](http://rubygems.org/gems/redcarpet) to my development dependencies.
{% highlight ruby %}
  spec.add_development_dependency "yard"
  spec.add_development_dependency "redcarpet"
{% endhighlight %}
These gems work together to provide a very straightforward way of documenting my gem as I build it.
I've found that _if I have the capability to document from the get go, I'm more likely to actually write documentation_, because nobody
likes an undocumented gem.


##  Rakefile Addition
{% highlight ruby linenos %}
task :console do
  require 'irb'
  require 'irb/completion'
  require 'bad_math'
  ARGV.clear
  IRB.start
end
{% endhighlight %}
After reading the short ["7 Lines Every Gem's Rakefile Should Have"](http://erniemiller.org/2014/02/05/7-lines-every-gems-rakefile-should-have/), I've
been adding this to my gems ever since.  As Ernie states, nothing beats playing around in the console, and no you don't need to use IRB.
You can use the console of your choice.  The point is to start it up a console and load your gem so you can play around with it.


## Configurable From the Start
I've written gems that started off with what I thought would be rock solid functionality, but find out later down the road that it's more flexible than
I'd originally thought.  This is the reason why the first functionality I add is a Configuration class.  With configuration set up from the start, it's
easier to stay flexible with functionality.  Believe me, it's a pain in the ass to add configuration functionality after the fact that your logic has
mostly been written.

### lib/bad\_math/configuration.rb
{% highlight ruby linenos %}
module BadMath
  class Configuration
    # These are our configuration options.
    attr_accessor :nonesuch

    def initialize
      # default the values here
      @nonesuch = nil
    end
  end
end
{% endhighlight %}

### lib/bad\_math.rb
{% highlight ruby linenos %}
require "bad_math/version"
require "bad_math/configuration"

module BadMath
  # Customize configuration for library
  # @yieldparam [Configuration] config
  def self.configure
    yield self.config if block_given?
  end

  # @return [Configuration]
  def self.config
    @@config ||= Configuration.new
  end
end
{% endhighlight %}
So now if I need to use a configurable 'foobar' option, I will:

* Add the option to the Configuration class (if not already) (```attr_accessor :foobar```)
  * I use attr_accessor because I want to be able to modify the value at any point.
  * If I need to keep a value from changing, I'll use attr_reader instead.
* Set a default value (Configuration#initialize: ```@foobar = ""```)
* Use it by calling ```BadMath.config.foobar```
* PROFIT!!!

### User Configuration
{% highlight ruby linenos %}
BadMath.configure do |cfg|
  cfg.nonesuch = "I'm not here"
  cfg.foobar = "bizbang"
end
{% endhighlight %}

## Summary
Boom! Bundler to flesh out the boring templating, yard/redcarpet for simple documentation, and a simple configuration setup.

<hr />

{% include signature.md %}
