---
layout: post
title: "Working with the Contact List"
date: 2014-07-29 14:06:33 -0400
comments: true
categories: 
---

I've been in dire need of a refresher on how to work with the contact list, and so, I am creating a reference!


Step 1: Make sure you have authorization to use the contact data:


```objc
@property (nonatomic) BOOL isAuthorized;
@property (strong, nonatomic) NSArray *contactsArray;
```


```objc
-(ABAuthorizationStatus)isAuthorized {

	_isAuthorized = NO;

	if (ABAddressBookGetAuthorizationStatus() == kABAuthorizationStatusAuthorized) {
	        _isAuthorized = YES;
	    }

	return _isAuthorized;

}
```


Step 2: If not authorized, get authorized or don't do anything!

```objc
-(void)authorizeAppToUseAddressBookWithCompletion:(void (^)(ABAddressBookRef addressBook, NSError *error))completionBlock
{
	CFErrorRef error = NULL;
    ABAddressBookRef addressBook = ABAddressBookCreateWithOptions(NULL, &error);
    
    if (!isAuthorized)
    {
	    ABAddressBookRequestAccessWithCompletion(addressBook, ^(bool granted, CFErrorRef error){
            completionBlock(addressBook, error)
            CFRelease(addressBook);
	    });
    }
}
```


Step 3: Collect data from the addressbook; you will need to bridge_transfer the ArrayOfAllPeople since it is written in C and not objective-C!

```objc
- (void)collectContactsFromAddressBook
{

	[self authorizeAppToUseAddressBookWithCompletion:^(ABAddressBookRef addressBook, NSError *error) {

	    NSMutableArray *contactsFromAddressBook = [[NSMutableArray alloc] init];
	    
	    if (addressBook != nil)
	    {
	        NSArray *allContacts = (__bridge_transfer NSArray *)ABAddressBookCopyArrayOfAllPeople(addressBook);
	        
	        NSUInteger i = 0;
	        
	        for (i = 0; i < [allContacts count]; i++)
	        {
	            Contact *contact = [[Contact alloc] init];
	            
	            ABRecordRef contactPerson = (__bridge ABRecordRef)allContacts[i];
	            
	            NSDate *lastModiDate = (__bridge_transfer NSDate*)ABRecordCopyValue(contactPerson, kABPersonModificationDateProperty);
	            
	            //names
	            NSString *firstName = (__bridge_transfer NSString *)ABRecordCopyValue(contactPerson, kABPersonFirstNameProperty);
	            NSString *lastName =  (__bridge_transfer NSString *)ABRecordCopyValue(contactPerson, kABPersonLastNameProperty);
	            
	            NSString *fullName;
	            if (lastName == nil)
	            {
	                fullName = [NSString stringWithFormat:@"%@", firstName];
	            }
	            else
	            {
	                fullName = [NSString stringWithFormat:@"%@ %@", firstName, lastName];
	            }
	            
	            contact.contactFirstName = firstName;
	            contact.contactLastName = lastName;
	            contact.contactFullName = fullName;
	            contact.modiDate = lastModiDate;
	            
	            //email
	            ABMultiValueRef emails = ABRecordCopyValue(contactPerson, kABPersonEmailProperty);
	            
	            NSUInteger j = 0;
	            for (j = 0; j < ABMultiValueGetCount(emails); j++)
	            {
	                NSString *email = (__bridge_transfer NSString *)ABMultiValueCopyValueAtIndex(emails, j);
	                [contact.contactEmailArray addObject:email];
	                
	            }
	            
	            //phone number
	            ABMultiValueRef phonenums = ABRecordCopyValue(contactPerson, kABPersonPhoneProperty);

	            //Add to mutable contact array
	            if ([contact.contactPhoneNumberArray firstObject])
	            {
	            [contactsFromAddressBook addObject:contact];
	            }
	        }
	    }
	
		self.contactsArray = contactsFromAddressBook;

	}];
}
```


