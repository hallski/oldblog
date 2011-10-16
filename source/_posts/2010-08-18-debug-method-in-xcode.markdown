---
layout: post
title: Debug Method in Xcode
---
I often find myself add a bunch of <tt>#ifdef</tt>s for code that I only want executed during the development but not in the shipped products. In order to make the code a bit easier to read I decided to make a small method that takes a C block (available in iOS 4.0 and Mac OS X 10.6) instead.

You would use it like this:

{% highlight objc %}
/*
 * Simple example of usage
 */

#include "debug.h"

int
main(int argc, **argv)
{
    debug(^ {
        // Your debug code here
    });

    return 0;
}
{% endhighlight %}

You can download <tt>debug.[ch]</tt> from <a href="http://gist.github.com/534487">GitHub</a> and simply add them to your project. You also need to edit your project settings for the Debug configuration to set the <tt>DEBUG</tt> macro.

![Project Settings Dialog](/images/posts/project-settings-debug.png)

[http://gist.github.com/534487](http://gist.github.com/534487)
