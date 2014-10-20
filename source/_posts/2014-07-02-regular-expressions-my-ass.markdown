---
layout: post
title: "Regular expressions my ass."
date: 2014-07-02 22:07:33 -0400
comments: true
categories: 
---

Regular expressions are anything but. Here are some ways to remember how to use them via example.

First, the NSRegularExpression class, and how it is used:

```objc
    NSString *phoneSentence = @"My phone number is 555-1212";
    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:@"." options:NSRegularExpressionCaseInsensitive error:nil];
    NSArray* matches = [regex matchesInString:phoneSentence                                      options:NSMatchingWithoutAnchoringBounds
                                        range:NSMakeRange(0, [phoneSentence length])];
    NSLog(@"All matches: %@", matches);
    NSTextCheckingResult *match = matches[0];
    NSRange range = match.range;
    NSString *matchString = [[phoneSentence substringFromIndex:range.location] substringToIndex:range.location + range.length];
    
    NSLog(@"First match: %@",matchString);
```

This code can be broken into the following parts:

1) + (NSRegularExpression *)regularExpressionWithPattern:(NSString *)pattern options:(NSRegularExpressionOptions)options error:(NSError **)error

Create a regular expression using NSRegularExpression. Consider options such as case insensitivity and continuation across lines (NSRegularExpressionDotMatchesLineSeparators). We did not record the error here, though you are certainly welcome to.

Our simple regex pattern here is ".", which effectively means, find every individual character.

2) - (NSArray *)matchesInString:(NSString *)string options:(NSMatchingOptions)options range:(NSRange)range

This method returns an array of NSTextCheckingResults, or ranges in which matches were found.

3) Everything after this method

The rest of this code is simply meant to produce the expected result for the first match, which will be a capital "M". What you do with the ranges once you have them is up to you. That's the easy part, because you are no longer dealing with regular expressions.


So what are some regular expression operators you should know?

{n} - match something exactly n times
{n, } - match at least n times (as many times as possible)
{n, m} - match between n and m times, inclusive
| - match either before or after the pipe

And what are some metacharacters you should know?

\d - Match any decimal digit in the Unicode set of characters
\D - Match any non decimal digit
\r - Match a carriage return
\s - Match a whitespace character
.  - Match any character
^  - Match only if at the beginning of the line
$  - Match only from the end of the line
\  - Character escape for the following characters: * ? + [ { } ( ) ^ $ | \ . /

So some nice regular expressions to have in your back pocket (please note I have not validated these!):

```objc

NSString *ssnRegex = @"[0-9]{3}-[0-9]{3}-[0-9]{4}";

NSString *emailRegex = @"[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,6}"; //1

NSString *htmltagRegex = @"<([a-z][a-z0-9]*)\b[^>]*>(.*?)""; //2

NSString *phoneRegex = @"^(?:(?:\+?1\s*(?:[.-]\s*)?)?(?:\(\s*([2-9]1[02-9]|[2-9][02-8]1|[2-9][02-8][02-9])\s*\)|([2-9]1[02-9]|[2-9][02-8]1|[2-9][02-8][02-9]))\s*(?:[.-]\s*)?)?([2-9]1[02-9]|[2-9][02-9]1|[2-9][02-9]{2})\s*(?:[.-]\s*)?([0-9]{4})(?:\s*(?:#|x\.?|ext\.?|extension)\s*(\d+))?$"; //3
```


Sources

1) http://stackoverflow.com/questions/800123/what-are-best-practices-for-validating-email-addresses-in-objective-c-for-ios-2

2) http://www.raywenderlich.com/30288/nsregularexpression-tutorial-and-cheat-sheet

3) http://stackoverflow.com/questions/123559/a-comprehensive-regex-for-phone-number-validation
