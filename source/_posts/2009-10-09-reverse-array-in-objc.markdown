---
layout: post
title: Reversing an array in Objective C
---
Was looking for the best way to reverse an array in Objective C and stumbled over this solution that might not be obvious to everyone (wasn't to me at least).
{% highlight objc %}
    NSArray *reversedArray = [[array reverseObjectEnumerator] allObjects];
{% endhighlight %}
