---
layout: post
title: Initial release of MHConvore
comments: true
---
Just pushed the first cut of a small framework to connect with the newly launched chatservice [Convore](http://convore.com/). In short Convore is an attempt to create a better way to chat than IRC and also support a web frontend while making it possible to connect with native clients.

The API will likely change a bit and it's fresh out of the editor so might be a bit flaky still. 

The API makes heavy use of blocks for actions initiated from your code and a client listener for notifications initiated from the server.

A small example of how you use the API, I will add a better example later.
``` objc
@interface AppDelegate : NSObject <NSApplicationDelegate, MHConvoreClientListener> {
@private
    MHConvoreClient *client;
}
```

``` objc
- (void)connectToServer
{
    MHConvoreClient *client = [MHConvoreClient clientWithUsername:@"u" password:@"p"];
    client.listener = self;

    [client verifyAccount:^ (MHConvoreUser *user, NSError *error) {
        if (error == nil) {
            //   [self doSomething];
            [client listen];
        } else {
            NSLog(@"Error while verifying account: %@", error);
        }
    }];
}

/* Called when the client receives a new message */
- (void)newMessage:(MHConvoreMessage *)message
{
    NSLog(@"[%@]: %@", message.user.name, message.message);
}
```

To connect to Convore you first need to create an account through their website.

There still is work that needs to be done to it so feel free to fork and hack on it. For example there is currently no way to cancel the listener.

[The code at Github](https://github.com/m5h/mhconvore)
