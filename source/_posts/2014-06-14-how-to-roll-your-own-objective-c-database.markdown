---
layout: post
title: "How to: Roll your own Objective-C Database"
date: 2014-06-14 22:19:48 -0400
comments: true
categories: [Database,External Files]
published: false
---

First, a disclaimer: This is my first attempt at blogging in Octopress. Enough said. Surf my site at your own risk of getting frustrated. This disclaimer may have to appear on every blog post I create for the next few months too, but today's is particularly tenuous.

Second, another disclaimer: You will not see all of the code you would need to write in this post just yet. Though I plan to augment this post with additional code in the future. I have just learned how challenging it can be to write objective-c code without the support of Xcode's autocomplete feature! (It was always there to give me a push when I needed it.)

So, why would you roll your own Objective-C database? The Core Data API is built so you don't have to do this, but let's assume for a moment that you haven't gotten your head around Core Data yet and/or you don't wish to implement such a bulky framework to do a simple task. For example, let us say you need to be able to search the English dictionary (and don't wish to use Apple's autocorrect dictionary due to the many non-English words included in it.) Maybe you are building a scrabble game, for instance. For the sake of this discussion, assume you already have a text file that contains every word in the English language on its own line.

The quickest code you could write to search such a file might look like this (assuming you have already specified your filePath/fileEncoding):

```objective-C

- (void)createArrayOfEnglishWordsFromDictionaryFile
{
	NSMutableArray *arrayOfAllWordsInTheEnglishLanguage = [NSMutableArray alloc] init];
	NSString *allWordsInTheEnglishLanguage = [NSString stringWithContentsOfFile:filePath encoding:fileEncoding error:NULL];
	for (NSString *aWord in [allWordsInTheEnglishLanguage componentsSeparatedByString:@"\n"])
	{
		[arrayOfAllWordsInTheEnglishLanguage addObject:aWord];
	}

}

//then go on to check the array for the word using NSPredicate or other method.

```

Objective-C is blazingly fast when it searches an array, but loading a dataset that spans into the hundreds of thousands of objects on an iPhone in this fashion might take a little while, and would have to be done every time the game was loaded (if not every time the dictionary was to be searched.) And if you just want to load it once per game, you have to keep it all in memory to avoid the constant re-building of the array. Instead, imagine you could just search the file directly and speedily to see if the word being suggested on the board was an actual word.

Final(?) disclaimer: I cannot say with what speed iOS can access and interpret an external file. This may impede the value of my theory here. Feel free to comment on this if you are more familiar!

With that disclaimer out of the way you must reorganize the data in your file. And for this, you must read the entire file in the way I described in code above once, but only at the time of development (not while a user is accessing the program.) But given you will be storing a local copy of this dictionary on each user's phone, doing this once will be an unbelievable time-savings. I will not go through here how to reorganize your file in code at this time, but I will describe it in prose. The point of this post is to consider the gains in running time, and demonstrate some of the code, but perhaps I can come back later and elaborate on this step more explicitly.

One way to do this would be to assign each letter in each word its ASCII integer value, and add them up for each word, giving each word a score that can then be used as a key to place it in the back in the file appropriately. This score would then be placed to the left of the word it represents, such that it is the actual key for the word being searched; the code below assumes such a format.

E.g. The word AND would be equivalent to the scores 1 + 14 + 4, or 29.

Now all you need to do is run a recursive binary search on the file to discover whether the word that has been placed on the board is real or not, and you may run the search with runtime of much closer to 𝚯(log n)!

Here is a sample of what that code might look like:

```objective-c
NSInteger numWordsInDictionary = 200000 //assumes you know the number of lines (words) in the file for simplicity of the example

- (void)findWordInBinaryTreeDictionary:(NSString *)aWord
{
	self.wordScore = 0;
	for (int i; i < [aWord length]; i++)
	{
		self.wordScore = self.wordScore + [aWord characterAtIndex:i];
	}

	NSInteger *foundSameIndexAsWordBeingSearched = [self recursiveBinarySearchOnDictionary:numWordsInDictionary/2];

	//finish this by adding code to check predecessors and successors for the same score. create an array of these and linearly (or with Cocoa API at faster speeds) search this subset only.

	//a further enhancement for a game like scrabble might be to use the number of letters in the user's word as a secondary strategic sort key to further limit the number of words to be searched linearly.

}

- (NSInteger)recursiveBinarySearchOnDictionary:(NSInteger)lineNumberToBeChecked
{
	NSInteger wordScoreOnLineNumber = [[NSString substringToIndex:[lineNumberToBeChecked rangeOfString:@" "]] integerValue];
	if (wordScoreOnLineNumber == self.wordScore
	{
		return lineNumberToBeChecked;
	}
	else if (wordScoreOnLineNumber < self.wordScore)
	{
		[self recursiveBinarySearchOnDictionary:lineNumberToBeChecked*1.5];
	}
	else if (wordScoreOnLineNumber > self.wordScore)
	{
		[self recursiveBinarySearchOnDictionary:lineNumberToBeChecked/2];
	}
}

```

Using code similar to the above (which is pseudo code for sure, given this is an early draft post) you would then want to run steps to check the wordScore values on predecessor and successor line numbers in the file to then linearly search a subset of words for the word you are seeking. And voila, you will efficiently locate (or not locate) the set of characters the user has tried placing on your scrabble board and in turn, created your own basic database using a combination of binary search.

Other potential uses for such code include datasets of every geography in the world (of which some databases catalogue over two million inclusive of all levels of geography). Can you think of a few?

One final thought: Using alternative data structures such as balanced binary search trees might make this implementation more robust in cases where the dataset is mutable. But given we do not need to add/remove data from the English Dictionary example with significant frequency, we can be satisfied with the above method.


---

