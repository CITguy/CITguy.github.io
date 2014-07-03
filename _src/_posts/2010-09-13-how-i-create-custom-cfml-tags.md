---
layout: post
title: "How I Create Custom CFML Tags"
date: 2010-09-13 12:00:00
category: coldfusion
tags:
- coldfusion
---

While I was cleaning up some code in a portal of ours, I realized that I could condense some of the code by combining a variable assignment with a custom tag attribute assignment.

To make things simple, the tag I am trying to optimize creates a database record with a note assigned to it.
{% highlight cfm linenos %}
  <cfset notes="A ton of html code here.">
  <cf_my_tag notes="#notes#">
{% endhighlight %}

The issue is that this code wouldn't necessarily add a note in the same manner, every time. There might be times where I have to build the notes in a loop before I run the tag. Then there are times where I want to run the tag and NOT add a note. Also, I might want to define the note directly in the tag contents.

So here are the requirements for my new custom tag:

1. The tag must be willing to accept an attribute named "notes"
2. The tag must be willing to run without defining "notes"
3. The tag must be willing to transform its contents into "notes", if it is not empty.

With the above constraints in mind, here's what I came up with.
{% highlight cfm linenos %}
<!--- Content of my_tag.cfm --->
<cfscript>
  param ATTRIBUTES.notes = "";
  if (ThisTag.hasEndTag) { // Closable Tag
    if (ThisTag.ExecutionMode EQ "start") {
      // ThisTag.GeneratedContent is not set yet.
    } else {
      // This is the best place to execute code
      // All possible 'ThisTag' variables are defined.
      if (ThisTag.GeneratedContent NEQ '') {
        // GeneratedContent takes priority over attribute
        ATTRIBUTES.notes = ThisTag.GeneratedContent;
      }
      engage();
    }
  } else {
    // Self-standing Custom Tag
    engage();
  }
</cfscript>

<cffunction name="engage">
  <cfscript>
    // Capture Generated Content
    // (if we need to refer to the original content)
    LOCAL.content = ThisTag.GeneratedContent;

    // Prevent browser display of GeneratedContent
    ThisTag.GeneratedContent = "";
  </cfscript>

  <!--- Do Stuff Here --->
</cffunction>
{% endhighlight %}

This setup fulfills all three requirements:

1. I can refer to the ATTRIBUTES.notes variable to insert my note.
2. I don't need to define ATTRIBUTES.notes as it is param'ed at the top of the tag.
3. If I have a closing tag with a non-empty string for its GeneratedContent, the ATTRIBUTES.notes variable will be overwritten with the GeneratedContent. This works for self-closing tags as well.
4. It also allows me to place my logic into a single function rather than splitting it up in an if..else block.

So there you have it. A simple solution to create a custom tag with flexible closing tags.

{% include signature.md %}
