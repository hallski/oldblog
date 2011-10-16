---
layout: post
title: Source List with Core Data
---
Last week I decided to look into how to create a source list that is backed by Core Data with section headers created from code. New to Cocoa I set out to Google for a solution but surprisingly enough I didn't find a lot about how to achieve this (or I simply googled for the wrong things). After a few days I managed to derive, what I believe, an elegant solution to this and figured it could be useful for others wanting to do this.

![The planned source list](/images/posts/sl-cd-result.png)

Approaches
----------
My first plan was to create some kind of layer in between Core Data and the NSTreeController but that quickly added a lot of complexity and code. The second solution was to create the headers and put them in Core Data the first time you start the application. While this was easily done it felt a bit hackish.

After spending a couple of days with this, trying out various solutions and ideas I and [Richard](http://twitter.com/rhult) discussed the topic, wondering if maybe Core Data had some builtin support for this.

Twenty minutes and some fifty lines of code later I looked at the screen, amazed at how awesome Core Data is. This is what I ended up with, mind though that it's not a full tutorial but only the crucial steps, you still need to know how to connect it all with the NSTreeController and NSOutlineView.

The Solution
------------
The idea is to use two different persistent stores, the regular XML store and an in memory store for the code generated entites.

For the purpose of this example the data model is very simple, two entities called *Section* (for the code generated section headers) and *Item* (for the entities that are read from Core Data). 

*Section* will have a `name` attribute and a one-to-many relationship to *Item* called `children`. *Item* will have a `name` and a reverse to-one relationship back to *Section* called `section`. Make sure you **make this relationship transient** to tell Core Data that this relationship only exists runtime and should not be stored.
 
![The data model](/images/posts/sl-cd-datamodel1.png)

The *NSTreeController* has a property called `childrenKeyPath` that defines the key path to retrieve an array of children from each of the nodes in the tree. All nodes displayed in the tree need to respond to this key path. Since *Item*s don't have children it is not included in the data model (though you could have added it in the model designer but in my opinion isn't as nice), I will instead create a subclass of *NSManagedObject* to represent an *Item*. 

Select the *Item* entity in the model designer and create a new file (File->New File and choose Managed Object Class from the Cocoa section). Simply name it Item.m (and check the *Also create "Item.h"*) box. Add a method `children` to your *Item* class.

{% highlight objc %}
/* In Item.h */
- (id)children;

/* In Item.m */
- (id)children 
{
    return nil;
}
{% endhighlight %}

This method will be called by the NSTreeController when it is populating the tree and since an *Item* doesn't have any children it should return `nil`.

That's it for the model, time to setup the stores.

I extended the method `persistentStoreCoordinator` in the auto generated application delegate to create a second store with the URL _memory://store_. This store should be of the type `NSInMemoryStoreType` which means that any entity tied to it won't be saved to disk. 

{% highlight objc %}
url = [NSURL URLWithString:@"memory://store"];
if (![persistentStoreCoordinator addPersistentStoreWithType:NSInMemoryStoreType
                                              configuration:nil
                                                        URL:url
                                                    options:nil
                                                      error:&error]) {
    [[NSApplication sharedApplication] presentError:error];
}
{% endhighlight %}

Both of these stores act in the same context so the layers on top of Core Data won't know the difference between entites in one store or another.

With the store in place the *Section* entities are created when the application is started, I added the code to the application delegate init method. After I create the *Section* I populate it with the result of fetching all my *Item* entities from Core Data (lines 30-33 below).

{% highlight objc %}
- (NSArray *)fetchAllWithEntity:(NSString *)entity
                          error:(NSError **)error
{
    NSFetchRequest *request;
    NSEntityDescription *desc;

    desc = [NSEntityDescription entityForName:entity
                       inManagedObjectContext:[self managedObjectContext]];

    request = [[[NSFetchRequest alloc] init] autorelease];
    [request setEntity:desc];

    return [[self managedObjectContext] executeFetchRequest:request error:error];	
}

- (id)init
{
    [super init];

    NSError *error;
    NSURL *url = [NSURL URLWithString:@"memory://store"];
    id memoryStore = [[self persistentStoreCoordinator] persistentStoreForURL:url];

    section = [[NSEntityDescription insertNewObjectForEntityForName:@"Section"
                                             inManagedObjectContext:[self managedObjectContext]] retain];
    [section setValue:@"My section" forKey:@"name"];
    [[self managedObjectContext] assignObject:section
                            toPersistentStore:memoryStore];

    NSArray *items = [self fetchAllWithEntity:@"Item" error:&error];
    for (id item in items) {
        [item setValue:section forKey:@"section"];
    }

    return self;
}
{% endhighlight %}

Here the message `assignObject:toPersistentStore:` is sent to the `NSManagedObjectContext` to put the newly created *Section* in the in memory store so that it is not saved to disk with the *Item*s.

I also added an action to my application delegate to create new *Item*s and make them children to the *Section*.

{% highlight objc %}
- (void)addItem:(id)sender
{
    id newObject = [NSEntityDescription insertNewObjectForEntityForName:@"Item"
                                                 inManagedObjectContext:[self managedObjectContext]];
    [newObject setValue:section forKey:@"section"];
}
{% endhighlight %}

That's it! Now a regular `NSTreeController` can be used in entity mode by setting the entity to _Section_ as you would normally do.

I hope you found it useful, if you did, make sure you share it with your friends and subscribe to my [RSS feed](http://blog.m5h.org/feed) or [follow me on twitter](http://twitter.com/m5h) for notifications about future posts.

I'd love to get feedback on better solutions, improvements or just comments on what you think.

