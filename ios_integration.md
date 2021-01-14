# Integrating Import services into Swyft Mobile for Salesforce

This document describes how to integrate Contact, Note and Document import services provided by Swyft Mobile for Salesforce into your iOS application. Swyft Mobile uses the [BlackBerry Dynamics](https://www.blackberry.com/us/en/products/blackberry-dynamics) framework and App Kinetics to import these data types into Salesforce

## Requirements

In order to test this functionality you will need to have Swyft Mobile for Salesforce installed on the same iOS device as your iOS application. Swyft Mobile requires a license from [Swyft Technology](https://swyftmobile.com). If you do not have a license, please reach out to Swyft Technology or BlackBerry for a free trial. You will also need to make sure you have an Entitlement on for Swyft Mobile on your UEM or Good Control envirement.

## Services provided by Swyft Mobile for Salesforce

* Import Contact (*com.swyftmobile.smsf.create-contact*)
* Import Document (*com.swyftmobile.smsf.create-document*)
* Import Note (*com.swyftmobile.smsf.create-note*)
* Save Edited File Service (*com.good.gdservice.save-edited-file*)

These services are all advertised in the [BlackBerry Marketplace](https://marketplace.blackberry.com/services).

## Import Contact

Swyft Mobile can import attachments through App Kinetics that are in a 'vcard' format. To invoke the service you will need to have your contact in a 'vcard' [format](https://tools.ietf.org/html/rfc6350). App Kinetics requires that files are referenced from the dynamics file system. The example provided below shows how to create a file reference to your vcard.

```Objective-C
NSData   *fileData            = [vcardNSString dataUsingEncoding:NSUTF8StringEncoding];
NSArray  *paths               = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask ,YES );
NSString *documentsDirectory  = [paths objectAtIndex:0];
NSString *fileName            = [[[NSProcessInfo processInfo] globallyUniqueString] stringByAppendingPathExtension:@"vcf"];
NSString *documentPathForFile = [documentsDirectory stringByAppendingPathComponent:fileName];
[[GDFileManager defaultManager] createFileAtPath:documentPathForFile contents:fileData attributes:nil];

```

The String for the `documentPathForFile` will be used when the vcard document is sent to Swyft Mobile. 

Your application will need to query BlackBerry Dynamics for a matching service. Use the following example to get an array of services using the following constants;

```Objective-C
NSString* const kCreateContactService = @"com.swyftmobile.smsf.create-contact";
NSString* const kFileTransferServiceVersion = @"1.0.0.0";
NSMutableArray *arrayGDServiceProvider = [[[GDiOS sharedInstance] getServiceProvidersFor:kCreateContactService
                                                                                  andVersion:kFileTransferServiceVersion
                                                                                     andServiceType:GDServiceTypeApplication] mutableCopy];
```

Verify that you have at least one matching service that is not your application in your array, otherwise Dynamics will not be able to send the document;

```Objective-C
for (int i = 0; i < arrayGDServiceProvider.count; ++i)
{
    GDServiceProvider *details = [arrayGDServiceProvider objectAtIndex:i];
    
    if ([details.identifier isEqualToString:[[NSBundle mainBundle] bundleIdentifier]])
    {
        [arrayGDServiceProvider removeObjectAtIndex:i];
        break;
    }
}
```

To send the document to Swyft Mobile once you have your document and service, call the following Dynamics method;

```Objective-C
NSError *error                       = nil;
NSArray           *attachments       = [NSArray arrayWithObject:documentPathForFile];
GDServiceProvider *serviceProvider   = [arrayGDServiceProvider objectAtIndex:0];
NSDictionary *params                 = nil;
NSString* const kImportFileMethod    = @"importFile";

[GDServiceClient sendTo:serviceProvider.identifier
                      withService:kCreateContactService
                      withVersion:kFileTransferServiceVersion
                       withMethod:kImportFileMethod
                       withParams:params
                  withAttachments:attchments
              bringServiceToFront:GDEPreferPeerInForeground
                        requestID:nil
                            error:error];
```

There is no need to implement the GDServiceClientDelegate because Swyft Mobile does not invoke a service response.

## Import Document

Swyft Mobile can import document attachments through App Kinetics. App Kinetics requires that files are referenced from the dynamics file system. 

This service requires two parameters;

* Filename
* Mimetype

The example provided below shows how to create a file reference to a text file in your application.

```Objective-C
// In this example we take text from an String, and save it to the file system using the GDFileManager.
NSData   *fileData            = [textNSString dataUsingEncoding:NSUTF8StringEncoding];
NSArray  *paths               = NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask ,YES );
NSString *documentsDirectory  = [paths objectAtIndex:0];
NSString *fileName            = [[[NSProcessInfo processInfo] globallyUniqueString] stringByAppendingPathExtension:@"txt"];
NSString *documentPathForFile = [documentsDirectory stringByAppendingPathComponent:fileName];
[[GDFileManager defaultManager] createFileAtPath:documentPathForFile contents:fileData attributes:nil];

```


The String for the `documentPathForFile` will be used when the text document is sent to Swyft Mobile. 

Your application will need to query BlackBerry Dynamics for a matching service. Use the following example to get an array of services using the following constants;

```Objective-C
NSString* const kCreateDocumentService = @"com.swyftmobile.smsf.create-document";
NSString* const kFileTransferServiceVersion = @"1.0.0.0";
NSMutableArray *arrayGDServiceProvider = [[[GDiOS sharedInstance] getServiceProvidersFor:kCreateDocumentService
                                                                              andVersion:kFileTransferServiceVersion
                                                                          andServiceType:GDServiceTypeApplication] mutableCopy];
```


Verify that you have at least one matching service that is not your application in your array, otherwise Dynamics will not be able to send the document;

```Objective-C
for (int i = 0; i < arrayGDServiceProvider.count; ++i)
{
    GDServiceProvider *details = [arrayGDServiceProvider objectAtIndex:i];
    
    if ([details.identifier isEqualToString:[[NSBundle mainBundle] bundleIdentifier]])
    {
        [arrayGDServiceProvider removeObjectAtIndex:i];
        break;
    }
}
```

To send the document to Swyft Mobile once you have your document and service, call the following Dynamics method;

```Objective-C
NSError *error                       = nil;
NSArray           *attachments       = [NSArray arrayWithObject:documentPathForFile];
GDServiceProvider *serviceProvider   = [arrayGDServiceProvider objectAtIndex:0];
NSDictionary *params                 = @{ 
    @"Filename": @"sampletext.txt", 
    @"Mimetype": @"text/plain" 
};
NSString* const kImportFileMethod    = @"importFile";

[GDServiceClient sendTo:serviceProvider.identifier
                      withService:kCreateDocumentService
                      withVersion:kFileTransferServiceVersion
                       withMethod:kImportFileMethod
                       withParams:params
                  withAttachments:attchments
              bringServiceToFront:GDEPreferPeerInForeground
                        requestID:nil
                            error:error];
```

There is no need to implement the GDServiceClientDelegate because Swyft Mobile does not invoke a service response.

## Import Note

Swyft Mobile can import notes through App Kinetics. App Kinetics requires that files are referenced from the dynamics file system. 

This service require the following two parameters;

* Title
* Body

Your application will need to query BlackBerry Dynamics for a matching service. Use the following example to get an array of services using the following constants;

```Objective-C
NSString* const kCreateNoteService = @"com.swyftmobile.smsf.create-note";
NSString* const kFileTransferServiceVersion = @"1.0.0.0";
NSMutableArray *arrayGDServiceProvider = [[[GDiOS sharedInstance] getServiceProvidersFor:kCreateNoteService
                                                                              andVersion:kFileTransferServiceVersion
                                                                          andServiceType:GDServiceTypeApplication] mutableCopy];
```

Verify that you have at least one matching service that is not your application in your array, otherwise Dynamics will not be able to send the document;

```Objective-C
for (int i = 0; i < arrayGDServiceProvider.count; ++i)
{
    GDServiceProvider *details = [arrayGDServiceProvider objectAtIndex:i];
    
    if ([details.identifier isEqualToString:[[NSBundle mainBundle] bundleIdentifier]])
    {
        [arrayGDServiceProvider removeObjectAtIndex:i];
        break;
    }
}
```


To send the note to Swyft Mobile once you have your note and service, call the following Dynamics method;

```Objective-C
NSError *error                       = nil;
GDServiceProvider *serviceProvider   = [arrayGDServiceProvider objectAtIndex:0];
NSDictionary *params                 = @{ 
    @"Title": @"This is my title", 
    @"Body": @"This is my sample body for this note." 
};
NSString* const kImportFileMethod    = @"importFile";

[GDServiceClient sendTo:serviceProvider.identifier
                      withService:kCreateNoteService
                      withVersion:kFileTransferServiceVersion
                       withMethod:kImportFileMethod
                       withParams:params
                  withAttachments:nil
              bringServiceToFront:GDEPreferPeerInForeground
                        requestID:nil
                            error:error];
```

There is no need to implement the GDServiceClientDelegate because Swyft Mobile does not invoke a service response.
