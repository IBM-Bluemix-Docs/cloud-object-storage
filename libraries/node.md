---

copyright:
  years: 2017, 2019
lastupdated: "2019-11-19"

keywords: object storage, node, javascript, sdk

subcollection: cloud-object-storage


---
{:new_window: target="_blank"}
{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:note: .note}
{:download: .download} 
{:http: .ph data-hd-programlang='http'} 
{:javascript: .ph data-hd-programlang='javascript'} 
{:java: .ph data-hd-programlang='java'} 
{:python: .ph data-hd-programlang='python'}
{:faq: data-hd-content-type='faq'}
{:support: data-reuse='support'}

# Using Node.js
{: #node}

The {{site.data.keyword.cos_full}} SDK for Node.js provides modern capabilities that make the most of {{site.data.keyword.cos_full_notm}}.
{: shortdesc}

## Installing the SDK
{: #node-install}

[Node.js](https://cloud.ibm.com/docs/node?topic=nodejs-getting-started){: external} is an excellent way to build [web applications](https://developer.mozilla.org/en-US/docs/Learn/Server-side/First_steps){: external}, and customize your instance of {{site.data.keyword.cos_short}} for your end users. The preferred way to install the {{site.data.keyword.cos_short}} SDK for Node.js is to use the [`npm`](https://www.npmjs.com){: external} package manager for Node.js. Type the following command into a command line:

```sh
npm install ibm-cos-sdk
```
{: codeblock}

To download the SDK directly, the source code is hosted on [GitHub](https://github.com/IBM/ibm-cos-sdk-js){: external}.

More detail on individual methods and classes can be found in [the API documentation for the SDK](https://ibm.github.io/ibm-cos-sdk-js/){: external}.

## Getting Started
{: #node-gs}

### Minimum requirements
{: #node-gs-prereqs}

To run the SDK, you need **Node 4.x+**.

### Creating a client and sourcing credentials
{: #node-gs-credentials}

To connect to COS, a client is created and configured by providing credential information (API Key, Service Instance ID, and IBM Authentication Endpoint). These values can also be automatically sourced from a credentials file or from environment variables.

After generating a [Service Credential](/docs/services/cloud-object-storage/iam?topic=cloud-object-storage-service-credentials), the resulting JSON document can be saved to `~/.bluemix/cos_credentials`. The SDK will automatically source credentials from this file unless other credentials are explicitly set during client creation. If the `cos_credentials` file contains HMAC keys the client authenticates with a signature, otherwise the client uses the provided API key to authenticate with a bearer token.

The `default` section heading specifies a default profile and associated values for credentials. You can create more profiles in the same shared configuration file, each with its own credential information. The following example shows a configuration file with the default profile:
```
[default]
ibm_api_key_id = <DEFAULT_IBM_API_KEY>
ibm_service_instance_id = <DEFAULT_IBM_SERVICE_INSTANCE_ID>
ibm_auth_endpoint = <DEFAULT_IBM_AUTH_ENDPOINT>
```
{: codeblock}

If migrating from AWS S3, you can also source credentials data from  `~/.aws/credentials` in the format:

```
aws_access_key_id = <DEFAULT_ACCESS_KEY_ID>
aws_secret_access_key = <DEFAULT_SECRET_ACCESS_KEY>
```
{: codeblock}

If both `~/.bluemix/cos_credentials` and `~/.aws/credentials` exist, `cos_credentials` takes preference.

## Code Examples
{: #node-examples}

Getting started with [Node.js](https://nodejs.org/en/about/){: external}&mdash;once it's installed&mdash;usually involves configuration and invocation, like in [this example from Nodejs.org](https://nodejs.org/en/docs/guides/getting-started-guide/){: external}. We'll follow a similar model

### Initializing configuration
{: #node-examples-init}

```javascript
const myCOS = require('ibm-cos-sdk');

var config = {
    endpoint: '<endpoint>',
    apiKeyId: '<api-key>',
    ibmAuthEndpoint: 'https://iam.cloud.ibm.com/identity/token',
    serviceInstanceId: '<resource-instance-id>',
};

var cosClient = new myCOS.S3(config);
```
*Key Values*
* `<endpoint>` - public endpoint for your cloud object storage (available from the [IBM Cloud Dashboard](https://cloud.ibm.com/resources){: external}). For more information about endpoints, see [Endpoints and storage locations](/docs/services/cloud-object-storage?topic=cloud-object-storage-endpoints#endpoints).
* `<api-key>` - API key generated when creating the service credentials (write access is required for creation and deletion examples)
* `<resource-instance-id>` - resource ID for your cloud object storage (available through [IBM Cloud CLI](/resources?topic=resources-crn) or [IBM Cloud Dashboard](https://cloud.ibm.com/resources){: external})

### Creating a bucket
{: #node-examples-new-bucket}

A list of valid provisioning codes for `LocationConstraint` can be referenced in [the Storage Classes guide](/docs/services/cloud-object-storage?topic=cloud-object-storage-classes#classes).

```javascript
function createBucket(bucketName) {
    console.log(`Creating new bucket: ${bucketName}`);
    return cos.createBucket({
        Bucket: bucketName,
        CreateBucketConfiguration: {
          LocationConstraint: 'us-standard'
        },        
    }).promise()
    .then((() => {
        console.log(`Bucket: ${bucketName} created!`);
    }))
    .catch((e) => {
        console.error(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [createBucket](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#createBucket-property){: external}

### Creating a text object
{: #node-examples-new-file}

```javascript
function createTextFile(bucketName, itemName, fileText) {
    console.log(`Creating new item: ${itemName}`);
    return cos.putObject({
        Bucket: bucketName, 
        Key: itemName, 
        Body: fileText
    }).promise()
    .then(() => {
        console.log(`Item: ${itemName} created!`);
    })
    .catch((e) => {
        console.error(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [putObject](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#putObject-property){: external}

### List buckets
{: #node-examples-list-buckets}

```javascript
function getBuckets() {
    console.log('Retrieving list of buckets');
    return cos.listBuckets()
    .promise()
    .then((data) => {
        if (data.Buckets != null) {
            for (var i = 0; i < data.Buckets.length; i++) {
                console.log(`Bucket Name: ${data.Buckets[i].Name}`);
            }
        }
    })
    .catch((e) => {
        console.error(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [listBuckets](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#listBuckets-property){: external}

### List items in a bucket
{: #node-examples-list-objects}

```javascript
function getBucketContents(bucketName) {
    console.log(`Retrieving bucket contents from: ${bucketName}`);
    return cos.listObjects(
        {Bucket: bucketName},
    ).promise()
    .then((data) => {
        if (data != null && data.Contents != null) {
            for (var i = 0; i < data.Contents.length; i++) {
                var itemKey = data.Contents[i].Key;
                var itemSize = data.Contents[i].Size;
                console.log(`Item: ${itemKey} (${itemSize} bytes).`)
            }
        }    
    })
    .catch((e) => {
        console.error(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [listObjects](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#listObjects-property){: external}

### Get file contents of particular item
{: #node-examples-get-contents}

```javascript
function getItem(bucketName, itemName) {
    console.log(`Retrieving item from bucket: ${bucketName}, key: ${itemName}`);
    return cos.getObject({
        Bucket: bucketName, 
        Key: itemName
    }).promise()
    .then((data) => {
        if (data != null) {
            console.log('File Contents: ' + Buffer.from(data.Body).toString());
        }    
    })
    .catch((e) => {
        console.error(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [getObject](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#getObject-property){: external}

### Delete an item from a bucket
{: #node-examples-delete-object}

```javascript
function deleteItem(bucketName, itemName) {
    console.log(`Deleting item: ${itemName}`);
    return cos.deleteObject({
        Bucket: bucketName,
        Key: itemName
    }).promise()
    .then(() =>{
        console.log(`Item: ${itemName} deleted!`);
    })
    .catch((e) => {
        console.error(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [deleteObject](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#deleteObject-property){: external}

### Delete multiple items from a bucket
{: #node-examples-multidelete}

The delete request can contain a maximum of 1000 keys that you want to delete. While deleting objects in batches is very useful in reducing the per-request overhead, be mindful when deleting many keys that the request may take some time to complete. Also, take into account the sizes of the objects to ensure suitable performance.
{:tip}

```javascript
function deleteItems(bucketName) {
    var deleteRequest = {
        "Objects": [
            { "Key": "deletetest/testfile1.txt" },
            { "Key": "deletetest/testfile2.txt" },
            { "Key": "deletetest/testfile3.txt" },
            { "Key": "deletetest/testfile4.txt" },
            { "Key": "deletetest/testfile5.txt" }
        ]        
    }
    return cos.deleteObjects({
        Bucket: bucketName,
        Delete: deleteRequest
    }).promise()
    .then((data) => {
        console.log(`Deleted items for ${bucketName}`);
        console.log(data.Deleted);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });    
}
```
{: codeblock}
{: javascript}

*SDK References*
* [deleteObjects](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#deleteObjects-property){: external}

### Delete a bucket
{: #node-examples-delete-bucket}

```javascript
function deleteBucket(bucketName) {
    console.log(`Deleting bucket: ${bucketName}`);
    return cos.deleteBucket({
        Bucket: bucketName
    }).promise()
    .then(() => {
        console.log(`Bucket: ${bucketName} deleted!`);
    })
    .catch((e) => {
        console.error(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [deleteBucket](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#deleteBucket-property){: external}


### Execute a multi-part upload
{: #node-examples-multipart}

```javascript
function multiPartUpload(bucketName, itemName, filePath) {
    var uploadID = null;

    if (!fs.existsSync(filePath)) {
        log.error(new Error(`The file \'${filePath}\' does not exist or is not accessible.`));
        return;
    }

    console.log(`Starting multi-part upload for ${itemName} to bucket: ${bucketName}`);
    return cos.createMultipartUpload({
        Bucket: bucketName,
        Key: itemName
    }).promise()
    .then((data) => {
        uploadID = data.UploadId;

        //begin the file upload        
        fs.readFile(filePath, (e, fileData) => {
            //min 5MB part
            var partSize = 1024 * 1024 * 5;
            var partCount = Math.ceil(fileData.length / partSize);
    
            async.timesSeries(partCount, (partNum, next) => {
                var start = partNum * partSize;
                var end = Math.min(start + partSize, fileData.length);
    
                partNum++;

                console.log(`Uploading to ${itemName} (part ${partNum} of ${partCount})`);  

                cos.uploadPart({
                    Body: fileData.slice(start, end),
                    Bucket: bucketName,
                    Key: itemName,
                    PartNumber: partNum,
                    UploadId: uploadID
                }).promise()
                .then((data) => {
                    next(e, {ETag: data.ETag, PartNumber: partNum});
                })
                .catch((e) => {
                    cancelMultiPartUpload(bucketName, itemName, uploadID);
                    console.error(`ERROR: ${e.code} - ${e.message}\n`);
                });
            }, (e, dataPacks) => {
                cos.completeMultipartUpload({
                    Bucket: bucketName,
                    Key: itemName,
                    MultipartUpload: {
                        Parts: dataPacks
                    },
                    UploadId: uploadID
                }).promise()
                .then(console.log(`Upload of all ${partCount} parts of ${itemName} successful.`))
                .catch((e) => {
                    cancelMultiPartUpload(bucketName, itemName, uploadID);
                    console.error(`ERROR: ${e.code} - ${e.message}\n`);
                });
            });
        });
    })
    .catch((e) => {
        console.error(`ERROR: ${e.code} - ${e.message}\n`);
    });
}

function cancelMultiPartUpload(bucketName, itemName, uploadID) {
    return cos.abortMultipartUpload({
        Bucket: bucketName,
        Key: itemName,
        UploadId: uploadID
    }).promise()
    .then(() => {
        console.log(`Multi-part upload aborted for ${itemName}`);
    })
    .catch((e)=>{
        console.error(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [abortMultipartUpload](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#abortMultipartUpload-property){: external}
* [completeMultipartUpload](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#completeMultipartUpload-property){: external}
* [createMultipartUpload](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#createMultipartUpload-property){: external}
* [uploadPart](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#uploadPart-property){: external}

## Using Key Protect
{: #node-examples-kp}

Key Protect can be added to a storage bucket to manage encryption keys. All data is encrypted in IBM COS, but Key Protect provides a service for generating, rotating, and controlling access to encryption keys using a centralized service.

### Before You Begin
{: #node-examples-kp-prereqs}
The following items are necessary in order to create a bucket with Key-Protect enabled:

* A Key Protect service [provisioned](/docs/services/key-protect?topic=key-protect-provision#provision)
* A Root key available (either [generated](/docs/services/key-protect?topic=key-protect-create-root-keys) or [imported](/docs/services/key-protect?topic=key-protect-import-root-keys))

### Retrieving the Root Key CRN
{: #node-examples-kp-root}

1. Retrieve the [instance ID](/docs/services/key-protect?topic=key-protect-retrieve-instance-ID#retrieve-instance-ID) for your Key Protect service
2. Use the [Key Protect API](/docs/services/key-protect?topic=key-protect-set-up-api#set-up-api) to retrieve all your [available keys](https://cloud.ibm.com/apidocs/key-protect)
    * You can either use `curl` commands or an API REST Client such as [Postman](/docs/services/cloud-object-storage?topic=cloud-object-storage-postman) to access the [Key Protect API](/docs/services/key-protect?topic=key-protect-set-up-api#set-up-api).
3. Retrieve the CRN of the root key you will use to enabled Key Protect on the your bucket. The CRN will look similar to below:

`crn:v1:bluemix:public:kms:us-south:a/3d624cd74a0dea86ed8efe3101341742:90b6a1db-0fe1-4fe9-b91e-962c327df531:key:0bg3e33e-a866-50f2-b715-5cba2bc93234`

### Creating a bucket with Key Protect enabled
{: #node-examples-kp-new-bucket}

```javascript
function createBucketKP(bucketName) {
    console.log(`Creating new encrypted bucket: ${bucketName}`);
    return cos.createBucket({
        Bucket: bucketName,
        CreateBucketConfiguration: {
          LocationConstraint: '<bucket-location>'
        },
        IBMSSEKPEncryptionAlgorithm: '<algorithm>',
        IBMSSEKPCustomerRootKeyCrn: '<root-key-crn>'
    }).promise()
    .then((() => {
        console.log(`Bucket: ${bucketName} created!`);
        logDone();
    }))
    .catch(logError);
}
```
{: codeblock}
{: javascript}

*Key Values*
* `<bucket-location>` - Region or location for your bucket (Key Protect is only available in certain regions. Ensure your location matches the Key Protect service) A list of valid provisioning codes for `LocationConstraint` can be referenced in [the Storage Classes guide](/docs/services/cloud-object-storage?topic=cloud-object-storage-classes#classes)..
* `<algorithm>` - The encryption algorithm used for new objects added to the bucket (Default is AES256).
* `<root-key-crn>` - CRN of the Root Key that is obtained from the Key Protect service.

*SDK References*
* [createBucket](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html#createBucket-property){: external}

## Using Archive Feature
{: #node-examples-archive}

Archive Tier allows users to archive stale data and reduce their storage costs. Archival policies (also known as *Lifecycle Configurations*) are created for buckets and applies to any objects added to the bucket after the policy is created.

### View a bucket's lifecycle configuration
{: #node-examples-get-lifecycle}

```javascript
function getLifecycleConfiguration(bucketName) {
    return cos.getBucketLifecycleConfiguration({
        Bucket: bucketName
    }).promise()
    .then((data) => {
        if (data != null) {
            console.log(`Retrieving bucket lifecycle config from: ${bucketName}`);
            console.log(JSON.stringify(data, null, 4));
        }
        else {
            console.log(`No lifecycle configuration for ${bucketName}`);
        }
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [getBucketLifecycleConfiguration](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html){: external}

### Create a lifecycle configuration 
{: #node-examples-put-lifecycle}

Detailed information about structuring the lifecycle configuration rules are available in the [API Reference](/docs/services/cloud-object-storage/api-reference?topic=cloud-object-storage-compatibility-api-bucket-operations#compatibility-api-new-bucket)

```javascript
function createLifecycleConfiguration(bucketName) {
    //
    var config = {
        Rules: [{
            Status: 'Enabled', 
            ID: '<policy-id>',
            Filter: {
                Prefix: ''
            },
            Transitions: [{
                Days: <number-of-days>, 
                StorageClass: 'GLACIER'
            }]
        }]
    };
    
    return cos.putBucketLifecycleConfiguration({
        Bucket: bucketName,
        LifecycleConfiguration: config
    }).promise()
    .then(() => {
        console.log(`Created bucket lifecycle config for: ${bucketName}`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*Key Values*
* `<policy-id>` - Name of the lifecycle policy (must be unqiue)
* `<number-of-days>` - Number of days to keep the restored file

*SDK References*
* [putBucketLifecycleConfiguration](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html){: external}

### Delete a bucket's lifecycle configuration
{: #node-examples-delete-lifecycle}

```javascript
function deleteLifecycleConfiguration(bucketName) {
    return cos.deleteBucketLifecycle({
        Bucket: bucketName
    }).promise()
    .then(() => {
        console.log(`Deleted bucket lifecycle config from: ${bucketName}`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [deleteBucketLifecycle](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html){: external}

### Temporarily restore an object
{: #node-examples-restore-object}

Detailed information about the restore request parameters are available in the [API Reference](/docs/services/cloud-object-storage/api-reference?topic=cloud-object-storage-object-operations#object-operations-archive-restore)

```javascript
function restoreItem(bucketName, itemName) {
    var params = {
        Bucket: bucketName, 
        Key: itemName, 
        RestoreRequest: {
            Days: <number-of-days>, 
            GlacierJobParameters: {
                Tier: 'Bulk' 
            },
        } 
    };
    
    return cos.restoreObject(params).promise()
    .then(() => {
        console.log(`Restoring item: ${itemName} from bucket: ${bucketName}`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*Key Values*
* `<number-of-days>` - Number of days to keep the restored file

*SDK References*
* [restoreObject](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html){: external}

### View HEAD information for an object
{: #node-examples-lifecycle-head-object}
```javascript
function getHEADItem(bucketName, itemName) {
    return cos.headObject({
        Bucket: bucketName,
        Key: itemName
    }).promise()
    .then((data) => {
        console.log(`Retrieving HEAD for item: ${itemName} from bucket: ${bucketName}`);
        console.log(JSON.stringify(data, null, 4));
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

*SDK References*
* [headObject](https://ibm.github.io/ibm-cos-sdk-js/AWS/S3.html){: external}

## Updating Metadata
{: #node-examples-metadata}

There are two ways to update the metadata on an existing object:
* A `PUT` request with the new metadata and the original object contents
* Executing a `COPY` request with the new metadata specifying the original object as the copy source

### Using PUT to update metadata
{: #node-examples-metadata-put}

**Note:** The `PUT` request overwrites the existing contents of the object so it must first be downloaded and re-uploaded with the new metdata


```javascript
function updateMetadataPut(bucketName, itemName, metaValue) {
    console.log(`Updating metadata for item: ${itemName}`);

    //retrieve the existing item to reload the contents
    return cos.getObject({
        Bucket: bucketName, 
        Key: itemName
    }).promise()
    .then((data) => {
        //set the new metadata
        var newMetadata = {
            newkey: metaValue
        };

        return cos.putObject({
            Bucket: bucketName,
            Key: itemName,
            Body: data.Body,
            Metadata: newMetadata
        }).promise()
        .then(() => {
            console.log(`Updated metadata for item: ${itemName} from bucket: ${bucketName}`);
        })
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

### Using COPY to update metadata
{: #node-examples-metadata-copy}

```javascript
function updateMetadataCopy(bucketName, itemName, metaValue) {
    console.log(`Updating metadata for item: ${itemName}`);

    //set the copy source to itself
    var copySource = bucketName + '/' + itemName;

    //set the new metadata
    var newMetadata = {
        newkey: metaValue
    };

    return cos.copyObject({
        Bucket: bucketName, 
        Key: itemName,
        CopySource: copySource,
        Metadata: newMetadata,
        MetadataDirective: 'REPLACE'
    }).promise()
    .then((data) => {
        console.log(`Updated metadata for item: ${itemName} from bucket: ${bucketName}`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

## Using Immutable Object Storage
{: #node-examples-immutable}

### Add a protection configuration to an existing bucket
{: #node-examples-immutable-add}

Objects written to a protected bucket cannot be deleted until the protection period has expired and all legal holds on the object are removed. The bucket's default retention value is given to an object unless an object specific value is provided when the object is created. Objects in protected buckets that are no longer under retention (retention period has expired and the object does not have any legal holds), when overwritten, will again come under retention. The new retention period can be provided as part of the object overwrite request or the default retention time of the bucket will be given to the object. 

The minimum and maximum supported values for the retention period settings `MinimumRetention`, `DefaultRetention`, and `MaximumRetention` are 0 days and 365243 days (1000 years) respectively. 


```js
function addProtectionConfigurationToBucket(bucketName) {
    console.log(`Adding protection to bucket ${bucketName}`);
    return cos.putBucketProtectionConfiguration({
        Bucket: bucketName,
        ProtectionConfiguration: {
            'Status': 'Retention',
            'MinimumRetention': {'Days': 10},
            'DefaultRetention': {'Days': 100},
            'MaximumRetention': {'Days': 1000}
        }
    }).promise()
    .then(() => {
        console.log(`Protection added to bucket ${bucketName}!`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

### Check protection on a bucket
{: #node-examples-immutable-check}

```js
function getProtectionConfigurationOnBucket(bucketName) {
    console.log(`Retrieve the protection on bucket ${bucketName}`);
    return cos.getBucketProtectionConfiguration({
        Bucket: bucketName
    }).promise()
    .then((data) => {
        console.log(`Configuration on bucket ${bucketName}:`);
        console.log(data);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

### Upload a protected object
{: #node-examples-immutable-upload}

Objects in protected buckets that are no longer under retention (retention period has expired and the object does not have any legal holds), when overwritten, will again come under retention. The new retention period can be provided as part of the object overwrite request or the default retention time of the bucket will be given to the object.

|Value	| Type	| Description |
| --- | --- | --- | 
|`Retention-Period` | Non-negative integer (seconds) | Retention period to store on the object in seconds. The object can be neither overwritten nor deleted until the amount of time specified in the retention period has elapsed. If this field and `Retention-Expiration-Date` are specified a `400`  error is returned. If neither is specified the bucket's `DefaultRetention` period will be used. Zero (`0`) is a legal value assuming the bucket's minimum retention period is also `0`. |
| `Retention-expiration-date` | Date (ISO 8601 Format) | Date on which it will be legal to delete or modify the object. You can only specify this or the Retention-Period header. If both are specified a `400`  error will be returned. If neither is specified the bucket's DefaultRetention period will be used. |
| `Retention-legal-hold-id` | string | A single legal hold to apply to the object. A legal hold is a Y character long string. The object cannot be overwritten or deleted until all legal holds associated with the object are removed. |

```js
function putObjectAddLegalHold(bucketName, objectName, legalHoldId) {
    console.log(`Add legal hold ${legalHoldId} to ${objectName} in bucket ${bucketName} with a putObject operation.`);
    return cos.putObject({
        Bucket: bucketName,
        Key: objectName,
        Body: 'body',
        RetentionLegalHoldId: legalHoldId
    }).promise()
    .then((data) => {
        console.log(`Legal hold ${legalHoldId} added to object ${objectName} in bucket ${bucketName}`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}

function copyProtectedObject(sourceBucketName, sourceObjectName, destinationBucketName, newObjectName, ) {
    console.log(`Copy protected object ${sourceObjectName} from bucket ${sourceBucketName} to ${destinationBucketName}/${newObjectName}.`);
    return cos.copyObject({
        Bucket: destinationBucketName,
        Key: newObjectName,
        CopySource: sourceBucketName + '/' + sourceObjectName,
        RetentionDirective: 'Copy'
    }).promise()
    .then((data) => {
        console.log(`Protected object copied from ${sourceBucketName}/${sourceObjectName} to ${destinationBucketName}/${newObjectName}`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

### Add or remove a legal hold to or from a protected object
{: #node-examples-immutable-legal-hold}

The object can support 100 legal holds:

*  A legal hold identifier is a string of maximum length 64 characters and a minimum length of 1 character. Valid characters are letters, numbers, `!`, `_`, `.`, `*`, `(`, `)`, `-` and `.
* If the addition of the given legal hold exceeds 100 total legal holds on the object, the new legal hold will not be added, a `400` error will be returned.
* If an identifier is too long it will not be added to the object and a `400` error is returned.
* If an identifier contains invalid characters, it will not be added to the object and a `400` error is returned.
* If an identifier is already in use on an object, the existing legal hold is not modified and the response indicates the identifier was already in use with a `409` error.
* If an object does not have retention period metadata, a `400` error is returned and adding or removing a legal hold is not allowed.

The user making adding or removing a legal hold must have `Manager` permissions for this bucket.

```js
function addLegalHoldToObject(bucketName, objectName, legalHoldId) {
    console.log(`Adding legal hold ${legalHoldId} to object ${objectName} in bucket ${bucketName}`);
    return cos.client.addLegalHold({
        Bucket: bucketName,
        Key: objectId,
        RetentionLegalHoldId: legalHoldId
    }).promise()
    .then(() => {
        console.log(`Legal hold ${legalHoldId} added to object ${objectName} in bucket ${bucketName}!`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}

function deleteLegalHoldFromObject(bucketName, objectName, legalHoldId) {
    console.log(`Deleting legal hold ${legalHoldId} from object ${objectName} in bucket ${bucketName}`);
    return cos.client.deleteLegalHold({
        Bucket: bucketName,
        Key: objectId,
        RetentionLegalHoldId: legalHoldId
    }).promise()
    .then(() => {
        console.log(`Legal hold ${legalHoldId} deleted from object ${objectName} in bucket ${bucketName}!`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

### Extend the retention period of a protected object
{: #node-examples-immutable-extend}

The retention period of an object can only be extended. It cannot be decreased from the currently configured value.

The retention expansion value is set in one of three ways:

* additional time from the current value (`Additional-Retention-Period` or similar method)
* new extension period in seconds (`Extend-Retention-From-Current-Time` or similar method)
* new retention expiry date of the object (`New-Retention-Expiration-Date` or similar method)

The current retention period stored in the object metadata is either increased by the given additional time or replaced with the new value, depending on the parameter that is set in the `extendRetention` request. In all cases, the extend retention parameter is checked against the current retention period and the extended parameter is only accepted if the updated retention period is greater than the current retention period.

Objects in protected buckets that are no longer under retention (retention period has expired and the object does not have any legal holds), when overwritten, will again come under retention. The new retention period can be provided as part of the object overwrite request or the default retention time of the bucket will be given to the object.

```js
function extendRetentionPeriodOnObject(bucketName, objectName, additionalSeconds) {
    console.log(`Extend the retention period on ${objectName} in bucket ${bucketName} by ${additionalSeconds} seconds.`);
    return cos.extendObjectRetention({
        Bucket: bucketName,
        Key: objectName,
        AdditionalRetentionPeriod: additionalSeconds
    }).promise()
    .then((data) => {
        console.log(`New retention period on ${objectName} is ${data.RetentionPeriod}`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}

### List legal holds on a protected object
{: #node-examples-immutable-list-holds}

This operation returns:

* Object creation date
* Object retention period in seconds
* Calculated retention expiration date based on the period and creation date
* List of legal holds
* Legal hold identifier
* Timestamp when legal hold was applied

If there are no legal holds on the object, an empty `LegalHoldSet` is returned.
If there is no retention period specified on the object, a `404` error is returned.


```js
function listLegalHoldsOnObject(bucketName, objectName) {
    console.log(`List all legal holds on object ${objectName} in bucket ${bucketName}`);
    return cos.listLegalHolds({
        Bucket: bucketName,
        Key: objectId
    }).promise()
    .then((data) => {
        console.log(`Legal holds on bucket ${bucketName}: ${data}`);
    })
    .catch((e) => {
        console.log(`ERROR: ${e.code} - ${e.message}\n`);
    });
}
```
{: codeblock}
{: javascript}
