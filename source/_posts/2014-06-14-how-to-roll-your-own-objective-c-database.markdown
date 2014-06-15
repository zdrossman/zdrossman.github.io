---
layout: post
title: "How to: Roll your own Objective-C Database"
date: 2014-06-14 22:19:48 -0400
comments: true
categories: [Database,External Files]
published: false
---

First, a disclaimer: This is my first attempt at blogging in Octopress. Enough said. Surf my site at your own risk of getting frustrated. This disclaimer may have to appear on every blog post I create for the next few months too, but today's is particularly tenuous.

Second, another disclaimer: You will not see all of the code you would need to write in this post just yet. Though I plan to augment this post with additional code in the future. I will try to provide references for where you may find some of the more detailed code as needed in the meantime. It is quite a bit of code for a beginner like me to write without Xcode's autocomplete feature!

So, why would you roll your own Objective-C database? The Core Data API is built so you don't have to do this, but let's assume for a moment that you haven't gotten your head around Core Data yet. And, you need to be able to search, say, the English dictionary (and don't wish to use Apple's autocorrect dictionary due to the many non-English words included in it.) Maybe you are building a scrabble game, for instance. For the sake of this discussion, assume you already have a text file that contains every word in the English language on its own line, in alphabetical order. (If you did not, the preprocessing step would be to sort such a file and format it this way.)

The quickest code you could write might look like this (assuming you have already specified your filePath/fileEncoding):

```objective-C
NSMutableArray *arrayOfAllWordsInTheEnglishLanguage = [NSMutableArray alloc] init];
NSString *allWordsInTheEnglishLanguage = [NSString stringWithContentsOfFile:filePath encoding:fileEncoding error:NULL];
for (NSString *aWord in [allWordsInTheEnglishLanguage componentsSeparatedByString:@"\n"])
{
	[arrayOfAllWordsInTheEnglishLanguage addObject:aWord];
}
```

Objective-C is fast when it searches an array, but loading a dataset that spans into the hundreds of thousands of objects on an iPhone in this fashion might take a little while. In addition, you have to keep it all in memory if you don't want to reload it constantly. Instead, imagine you could just search the file to see if the word being suggested on the board was an appropriate word with great speed.

Final(?) disclaimer: I cannot say with what speed iOS can access and interpret an external file. This may impede the value of my theory here. Feel free to comment on this if you are more familiar!

With that disclaimer out of the way you must reorganize your file into a balanced binary search tree. And for this, you must, once read the file in the way I describe above. But given you will be storing a local copy of this dictionary on each user's phone, doing this once will be an unbelievable time-savings. I will not go through here how to reorganize your file in code at this time. The point of this post is to consider the gains in running time, and demonstrate some of the code, but perhaps I can come back later and elaborate.

If you don't know what a binary search tree is, see the following link for more detail.

//LINK GOES HERE

One way to do this would be to assign each letter in each word its ASCII integer value, and add them up for each word, giving each word a score that can then be used as a key to place it in the binary search tree appropriately. This score would then be placed to the left of the word it represents, such that it is the actual key for the word being searched; the code below assumes such a format.


Now all you need to do is run a recursive binary search on the file to discover whether the word that has been placed on the board is real or not, and voila!, you may run the search with runtime of much closer to theta log n! (Excuse my lack of proper notation! <-- again, first. octopress. post. ever.)

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

	//finish this by adding code to check predecessors and successors for the same score. create an array of these and linearly search them only.

	//a further enhancement for a game like scrabble might be to use the number of letters in the user's word as strategic key to further limit the number of words to be searched linearly.

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

Using code similar to the above (which is a pseudo code for sure, given I did not yet have time to test it) you would then want to run steps to check the wordScore values on predecessor and successor line numbers in the file to then linearly search a subset of words for the word you are seeking. And voila, you will efficiently locate (or not locate) the set of characters the user has tried placing on your scrabble board and in turn, created your own basic database using a combination of binary search 


---

