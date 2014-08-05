---
layout: post
title: "HTTP 301 redirection with exceptions in lighttpd"
date: 2014-06-06 23:25:46 -0400
comments: true
categories: lighttpd config
---
I've recently decided to make my base domain ```alisonc.net``` redirect to 
this blog's subdomain ```blog.alisonc.net```, with a 
[HTTP 301](https://en.wikipedia.org/wiki/HTTP_301) Moved Permanently 
status code. However, I didn't want to break 
links to my résumé which was hosted at the URL ```http://alisonc.net/resume.pdf```. 
(Why not just stick it in my blog's directory? Well, it has nothing to 
do with my blog.) Also, I did not want userdirs of the format 
```http://alisonc.net/~username``` to redirect to the blog subdomain either. 

Therefore, the redirect setting needed to include exceptions to prevent those patterns 
from being redirected.

<!-- more -->

{% codeblock lang:perl lighttpd.conf snippet %}
# redirecting alisonc.net and www.alisonc.net to blog.alisonc.net
# except for résumé and userdirs
$HTTP["host"] =~ "^(?:www\.)?alisonc\.net" {
    $HTTP["url"] !~ "^/resume\.pdf" {
        $HTTP["url"] !~ "^/\~" {
            url.redirect = (
                 "^/(.*)$" => "http://blog.alisonc.net/$1"
            )
        }
    }
}
{% endcodeblock %}

I found [Regular Expressions 101](http://regex101.com/) to be super helpful 
in understanding and testing the regular expressions in the above snippet. 
Here's some Regex101 permalinks to test out the above:

* [```^(?:www\.)?alisonc\.net```](http://regex101.com/r/iE6jO2)
* [```^/resume\.pdf```](http://regex101.com/r/wD1rV1)
* [```^/\~```](http://regex101.com/r/hU2tM7)
* [```^/(.*)$```](http://regex101.com/r/wS1hB1)

Note that Regex101 wants each forward slash (```/```) escaped with a backslash 
(```\```), whereas forward slashes are _not_ escaped in the lighttpd configuration.

Firefox and Chromium cache HTTP 301 results, so the changes in redirect configuration 
were not reflected, which caused me somewhat headache in debugging.... You just need 
to clear browser history for your site each time you change the redirect configuration. 
Once I did that, I was able to test each configuration change.

Useful resources:

* [lighttpd documentation for mod_redirect](http://redmine.lighttpd.net/projects/1/wiki/docs_modredirect)
* [Permanent redirect (301) with lighttpd](http://charles.lescampeurs.org/2008/06/30/permanent-redirect-301-with-lighttpd)
