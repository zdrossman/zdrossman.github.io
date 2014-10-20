---
layout: post
title: "How to build your own completion handler method"
date: 2014-06-24 23:45:58 -0400
comments: true
categories: 
---

What is a completion handler method?

So you're finally at the point in your app where you have to make a network call. Let's say your app downloads pictures of cats from the interwebs. A network call is the process of asking the web for something and then waiting for it to come back to you with what you asked for. Or not. What if the internet has no pictures of cats on it anywhere? In that case, your request will come back with nothing at all.

So, to prepare for these possibilities, we use completion handlers. Completion handlers tell us whether or not a certain process completed with the results we expected. 


Step 1: Create a CompletionBlock typedef.

This step isn't crucial, but it will make your code look neater, so I recommend it.

You can think of a typedef as an alias. I'm not sure that's a perfect analogy, but it'll do for now if this is your first typedef.

An example of a basic typedef:
```
typedef unsigned char BYTE;
```

This typedef lets us use the word BYTE in place of "unisigned char" anytime we declare a variable.

For a block, our typedef might look something like this:

```
typedef void(^CompletionBlock)(NSArray *queryResult, NSError *error);
```
NB: You might think you need a name for your completion block at the end of the typedef. I'm not actually sure why you do not, but this will name it CompletionBlock, just as if you were writing a standard function.

Step2: Create a method that takes a Completion block as an argument, e.g.

```
-(void) getObjectsInBackgroundWithBlock:(CompletionBlock)completionBlock;
```

How we define this method is as follows (using pseudocode, since we don't have a specific use case in mind):

```
-(void) getObjectsInBackgroundWithBlock:(CompletionBlock)completionBlock
{
	// Run a request for information from a web source

	dispatch_queue_t downloadQueue = dispatch_queue_create("Get our data", NULL);
	dispatch_async(downloadQueue, ^{
		NSError *error;
		completionBlock(arrayOfDataFromDownload, error);
		});
}
```

Step3: Now use the method!
```
-(void)modifyWebData
{
	//assumes you have a shared data store, but if you don't you might have written the "getObjectsInBackgroundWithBlock:" method in the same file, in which case you can just call [self getObjects...]
	
	SharedDataStore *dataStore = [SharedDataStore sharedInstance]; //uses a singleton method in a class called SharedDataStore where the "getObjectsInBackgroundWithBlock:" method will be found.

	[dataStore getObjectsInBackgroundWithBlock:^(NSArray *queryResult, NSError *error) {
		//now do something with the queryResult data. If it were an image, maybe we'd resize it here, for instance.
		//or get an error back and deal with that!
	}
}
```
All that said, I wrote this blog to open up a discussion about good use of completion blocks, something I am still getting my head around. It sure does seem like modifyWebData does 2 things, and we all know that a method should only do one thing according to the single responsibility principle. So what now? For discussion.



