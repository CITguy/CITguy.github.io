---
layout: post
title: "cfsavecontent and cfoutput gotchas"
date: 2010-08-29 12:00:00
categories: coldfusion
---

I was playing around with cfsavecontent to generate dynamic HTML variable values for use as an argument to a function when I came across an interesting behavior regarding the tag. The issue I ran was that Coldfusion was giving me an error that the variable saved by cfsavecontent was not defined when I was trying to output the value. Take a look at the code below:

This code will produce a "Variable not Defined" error in ColdFusion at line #5.
{% highlight cfm linenos %}
<cfoutput>
  <cfsavecontent variable="foo">
    Gozirra!
  </cfsavecontent>
  #MyFunction(foo)#
</cfoutput>
{% endhighlight %}

The reason for this is because the cfsavecontent isn't set until the cfoutput block closes.
Here's the ugly fix.
{% highlight cfm linenos %}
<cfoutput>
  <cfsavecontent variable="foo">
    Gozirra!
  </cfsavecontent>
</cfoutput>
<cfoutput>
  #MyFunction(foo)#
</cfoutput>
{% endhighlight %}

{% include signature.md %}
