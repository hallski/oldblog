---
layout: post
title: Expanding NSOutlineView nodes at application start
comments: true
---
A problem when you want to expand some items in an _NSOutlineView_ programatically at application start is that the _NSTreeController_ prepares it's content after `awakeFromNib` is called on your controller.

To ensure that the data is loaded before you try to expand the items you can observe the `content` key on the tree controller and expand the nodes as you receive the the `observeValueForKeyPath:` message as shown below:

``` objc
- (void)observeValueForKeyPath:(NSString *)keyPath 
                      ofObject:(id)object 
                        change:(NSDictionary *)change 
                       context:(void *)context
{
    if (object == treeController) {
        // Expand the first row (which is our section header)
        [sourceList expandItem:[sourceList itemAtRow:0] 
                expandChildren:NO];
        [treeController removeObserver:self 
                            forKeyPath:@"content"];
    }
}

- (void)awakeFromNib
{
    // Listen on the treeController to expand the root node 
    // when it has prepared it's content.
    [treeController addObserver:self 
                     forKeyPath:@"content" 
                        options:0 
                        context:nil];
}
```
