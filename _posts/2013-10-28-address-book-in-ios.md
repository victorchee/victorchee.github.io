---
layout: post
title: "iOS中使用通讯录"
category:  Skill
---
App开发中用到通讯录的不多，现在接到个同步通讯录的需求，所以研究了一下通讯录开发，总结如下，设计到联系人的读取，修改和删除。

```Objective-C
@interface AddressBookViewController ()
{
    ABAddressBookRef addressBookRef;
}
@end
```

- 请求权限

```Objective-C
addressBookRef = ABAddressBookCreate();
ABAddressBookRequestAccessWithCompletion(addressBookRef, ^(bool granted, CFErrorRef error) {
    dispatch_async(dispatch_get_main_queue(), ^{
        if (granted) {
            CFIndex localContactsCount = ABAddressBookGetPersonCount(addressBookRef);
        } else {
            // failed
        }
    });
});
```

- 读取所有联系人

```Objective-C
CFArrayRef localContacts = ABAddressBookCopyArrayOfAllPeople(addressBookRef);
for (int i = 0; i < CFArrayGetCount(localContacts); ++i) {
    ABRecordRef person = CFArrayGetValueAtIndex(localContacts, i);

    NSString *firstName = (NSString *)ABRecordCopyValue(person, kABPersonFirstNameProperty);
    NSString *lastName = (NSString *)ABRecordCopyValue(person, kABPersonLastNameProperty);

    ABMutableMultiValueRef personEmail = ABRecordCopyValue(person, kABPersonEmailProperty);
    for (int j = 0; j < ABMultiValueGetCount(personEmail); ++j) {
        NSString *emailLabel = (NSString *)ABAddressBookCopyLocalizedLabel(ABMultiValueCopyLabelAtIndex(personEmail, j));
        NSString *email = (NSString *)ABMultiValueCopyValueAtIndex(personEmail, j);
    }

    ABMultiValueRef personPhone = ABRecordCopyValue(person, kABPersonPhoneProperty);
    for (int j = 0; j < ABMultiValueGetCount(personPhone); ++j)
    {
        NSString *phoneLabel = (NSString *)ABAddressBookCopyLocalizedLabel(ABMultiValueCopyLabelAtIndex(personPhone, j));
        NSString *phone = (NSString *)ABMultiValueCopyValueAtIndex(personPhone, j);
    }

    NSData *image = (NSData*)ABPersonCopyImageData(person);

    // 其他字段依此类推
}
CFRelease(localContacts);
```

- 写入联系人

```Objective-C
ABRecordRef person = ABPersonCreate();

NSString *firstName = @"First Name";
ABRecordSetValue(person, kABPersonFirstNameProperty, (__bridge CFTypeRef)firstName, NULL);

NSString *lastName = @"Last Name";
ABRecordSetValue(person, kABPersonLastNameProperty, (__bridge CFTypeRef)lastName, NULL);

NSString *phoneNumber = @"13212312312";
NSString *phoneLabel = @"Home";
ABMutableMultiValueRef phone = ABMultiValueCreateMutable(kABMultiStringPropertyType);
ABMultiValueAddValueAndLabel(phone, (__bridge CFTypeRef)phoneNumber, (__bridge CFTypeRef)phoneLabel, NULL);
ABRecordSetValue(person, kABPersonPhoneProperty, phone, NULL);

BOOL addSuccess = ABAddressBookAddRecord(addressBookRef, person, NULL);
if (addSuccess) {
    BOOL saveSuccess = ABAddressBookSave(addressBookRef, NULL);
}
```

如果报错`Can't return type callbacks for 3`，检查ABMutableMultiValueRef创建的时候是否传入的是kABMultiStringPropertyType。

- 删除联系人

```Objective-C
CFArrayRef localContacts = ABAddressBookCopyArrayOfAllPeople(addressBookRef);
for (int i = 0; i < CFArrayGetCount(localContacts); ++i) {
    ABRecordRef person = CFArrayGetValueAtIndex(localContacts, i);

    NSString *personFirstName = (NSString *)ABRecordCopyValue(person, kABPersonFirstNameProperty);

    if ([personFirstName isEqualToString:@"First Name"]) {
        ABAddressBookRemoveRecord(addressBookRef, person, NULL);
        BOOL saveSuccess = ABAddressBookSave(addressBookRef, NULL);
    }
}
CFRelease(localContacts);
```
