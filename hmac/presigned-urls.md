---

copyright:
  years: 2017, 2019
lastupdated: "2021-1-30"

keywords: authorization, aws, hmac, signature, presign

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

# Creating a presigned URL
{: #presign-url}

Presigned URLs in {{site.data.keyword.cos_full}} create temporary links that can be used to share an object without requiring additional user credentials when accessed.
{: shortdesc}

Of course, one can also [provide a temporary target for sending a PUT request](https://medium.com/codait/keeping-your-secrets-between-cloud-object-storage-and-your-browser-part-1-68f4b83bbd38) also without needing to provide any more information for authentication. The easiest way to create presigned URLs is using the [AWS CLI](/docs/cloud-object-storage/cli?topic=cloud-object-storage-aws-cli). But first, you may need to run `aws configure` in order to set your Access Key ID and Secret Access Key from your own [HMAC-enabled service credential](/docs/cloud-object-storage?topic=cloud-object-storage-uhc-hmac-credentials-main). When you have completed configuring your CLI, use the following example as a template and replace the endpoint and name of your bucket with the appropriate information:

```bash
$ aws --endpoint-url=https://{endpoint} s3 presign s3://{bucket-name}/{new-file-key}
```

If the service credential used to generate the HMAC credentials (used as the Access Key ID and Secret Access Key configuration above) is deleted, the access for the pre-signed URL will fail.
{: note}

It is also possible to set an expiration time for the URL in seconds (default is 3600):

```bash
$ aws --endpoint-url=https://{endpoint} s3 presign s3://bucket-1/new-file --expires-in 600
```

It is also possible to construct them programmatically. Here are examples for basic `GET` operations written in Python. For more information about endpoints, see [Endpoints and storage locations](/docs/cloud-object-storage?topic=cloud-object-storage-endpoints#endpoints).

Unlike AWS S3, {{site.data.keyword.cos_full_notm}} does not enforce a maximum expiration time of 7 days (604800 seconds). While it is possible to create a presigned URL with a long expiration value, most use cases that require extended public access would be better served by [implementing a public access policy](/docs/cloud-object-storage/info?topic=cloud-object-storage-iam-public-access) on a bucket instead.
{: tip}

## Create a presigned URL to download an object
{: #presign-url-get}

### Python Example
{: #presign-url-get-python}

```python
import datetime
import hashlib
import hmac
import requests
from requests.utils import quote

access_key = '{COS_HMAC_ACCESS_KEY_ID}'
secret_key = '{COS_HMAC_SECRET_ACCESS_KEY}'

# request elements
http_method = 'GET'
region = ''
bucket = 'example-bucket'
cos_endpoint = '{endpoint}'
host = cos_endpoint
endpoint = 'https://' + host
object_key = 'example-object'
expiration = 86400  # time in seconds


# hashing methods
def hash(key, msg):
    return hmac.new(key, msg.encode('utf-8'), hashlib.sha256).digest()


# region is a wildcard value that takes the place of the AWS region value
# as COS doen't use regions like AWS, this parameter can accept any string
def createSignatureKey(key, datestamp, region, service):
    keyDate = hash(('AWS4' + key).encode('utf-8'), datestamp)
    keyRegion = hash(keyDate, region)
    keyService = hash(keyRegion, service)
    keySigning = hash(keyService, 'aws4_request')
    return keySigning


# assemble the standardized request
time = datetime.datetime.utcnow()
timestamp = time.strftime('%Y%m%dT%H%M%SZ')
datestamp = time.strftime('%Y%m%d')

standardized_querystring = ( 'X-Amz-Algorithm=AWS4-HMAC-SHA256' +
                             '&X-Amz-Credential=' + access_key + '/' + datestamp + '/' + region + '/s3/aws4_request' +
                             '&X-Amz-Date=' + timestamp +
                             '&X-Amz-Expires=' + str(expiration) +
                             '&X-Amz-SignedHeaders=host' )
standardized_querystring_url_encoded = quote(standardized_querystring, safe='&=')

standardized_resource = '/' + bucket + '/' + object_key
standardized_resource_url_encoded = quote(standardized_resource, safe='&')

payload_hash = 'UNSIGNED-PAYLOAD'
standardized_headers = 'host:' + host
signed_headers = 'host'

standardized_request = (http_method + '\n' +
                        standardized_resource + '\n' +
                        standardized_querystring_url_encoded + '\n' +
                        standardized_headers + '\n' +
                        '\n' +
                        signed_headers + '\n' +
                        payload_hash)

# assemble string-to-sign
hashing_algorithm = 'AWS4-HMAC-SHA256'
credential_scope = datestamp + '/' + region + '/' + 's3' + '/' + 'aws4_request'
sts = ( hashing_algorithm + '\n' +
        timestamp + '\n' +
        credential_scope + '\n' +
        hashlib.sha256(standardized_request).hexdigest() )

# generate the signature
signature_key = createSignatureKey(secret_key, datestamp, region, 's3')
signature = hmac.new(signature_key,
                     (sts).encode('utf-8'),
                     hashlib.sha256).hexdigest()

# create and send the request
# the 'requests' package autmatically adds the required 'host' header
request_url = ( endpoint + '/' +
                bucket + '/' +
                object_key + '?' +
                standardized_querystring_url_encoded +
                '&X-Amz-Signature=' +
                signature )

print 'request_url: %s' % request_url

print '\nSending `%s` request to IBM COS -----------------------' % http_method
print 'Request URL = ' + request_url
request = requests.get(request_url)

print '\nResponse from IBM COS ---------------------------------'
print 'Response code: %d\n' % request.status_code
print request.text


# this information can be helpful in troubleshooting, or to better
# understand what goes into signature creation
print 'These are the values used to construct this request.'
print 'Request details -----------------------------------------'
print 'http_method: %s' % http_method
print 'host: %s' % host
print 'region: %s' % region
print 'endpoint: %s' % endpoint
print 'bucket: %s' % bucket
print 'object_key: %s' % object_key
print 'timestamp: %s' % timestamp
print 'datestamp: %s' % datestamp

print 'Standardized request details ----------------------------'
print 'standardized_resource: %s' % standardized_resource_url_encoded
print 'standardized_querystring: %s' % standardized_querystring_url_encoded
print 'standardized_headers: %s' % standardized_headers
print 'signed_headers: %s' % signed_headers
print 'payload_hash: %s' % payload_hash
print 'standardized_request: %s' % standardized_request

print 'String-to-sign details ----------------------------------'
print 'credential_scope: %s' % credential_scope
print 'string-to-sign: %s' % sts
print 'signature_key: %s' % signature_key
print 'signature: %s' % signature

print 'Because the signature key has non-ASCII characters, it is'
print 'necessary to create a hexadecimal digest for the purposes'
print 'of checking against this example.'

def hex_hash(key, msg):
    return hmac.new(key, msg.encode('utf-8'), hashlib.sha256).hexdigest()
def createHexSignatureKey(key, datestamp, region, service):
    keyDate = hex_hash(('AWS4' + key).encode('utf-8'), datestamp)
    keyRegion = hex_hash(keyDate, region)
    keyService = hex_hash(keyRegion, service)
    keySigning = hex_hash(keyService, 'aws4_request')
    return keySigning

signature_key_hex = createHexSignatureKey(secret_key, datestamp, region, 's3')

print 'signature_key (hexidecimal): %s' % signature_key_hex

print 'Header details ------------------------------------------'
print 'pre-signed url: %s' % request_url
```

### Java Example
{: #presign-url-get-java}

```java
import java.io.BufferedReader;
import java.io.DataOutputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.time.format.DateTimeFormatter;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.ZonedDateTime;
import java.util.Formatter;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;

public class CoSHMAC {
    // constants
    private static final String accessKey = "{COS_HMAC_ACCESS_KEY_ID}";
    private static final String secretKey = "{COS_HMAC_SECRET_ACCESS_KEY}";
    private static final String httpMethod = "GET";
    private static final String host = "{endpoint}";
    private static final String region = "";
    private static final String endpoint = "https://" + host;
    private static final String bucket = "example-bucket";
    private static final String objectKey = "example-object";
    private static final String requestParameters = "";
    private static final int expiration = 86400;  // time in seconds
    
    public static void main(String[] args) {
        try {
            // assemble the standardized request
            ZonedDateTime time = ZonedDateTime.now(ZoneOffset.UTC);
            String datestamp = time.format(DateTimeFormatter.ofPattern("yyyyMMdd"));
            String timestamp = datestamp + "T" + time.format(DateTimeFormatter.ofPattern("HHmmss")) + "Z";

            String standardizedQuerystring = "X-Amz-Algorithm=AWS4-HMAC-SHA256" +
                "&X-Amz-Credential=" + URLEncoder.encode(accessKey + "/" + datestamp + "/" + region + "/s3/aws4_request", StandardCharsets.UTF_8.toString()) +
                "&X-Amz-Date=" + timestamp +
                "&X-Amz-Expires=" + String.valueOf(expiration) +
                "&X-Amz-SignedHeaders=host";

            String standardizedResource = "/" + bucket + "/" + objectKey;

            String payloadHash = "UNSIGNED-PAYLOAD";
            String standardizedHeaders = "host:" + host;
            String signedHeaders = "host";

            String standardizedRequest = httpMethod + "\n" +
                standardizedResource + "\n" +
                standardizedQuerystring + "\n" +
                standardizedHeaders + "\n" +
                "\n" +
                signedHeaders + "\n" +
                payloadHash;

            // assemble string-to-sign
            String hashingAlgorithm = "AWS4-HMAC-SHA256";
            String credentialScope = datestamp + "/" + region + "/" + "s3" + "/" + "aws4_request";
            String sts = hashingAlgorithm + "\n" +
                timestamp + "\n" +
                credentialScope + "\n" +
                hashHex(standardizedRequest);

            // generate the signature
            byte[] signatureKey = createSignatureKey(secretKey, datestamp, region, "s3");
            String signature = hmacHex(signatureKey, sts);

            // create and send the request
            String requestUrl = endpoint + "/" +
                bucket + "/" +
                objectKey + "?" +
                standardizedQuerystring +
                "&X-Amz-Signature=" +
                signature;

            URL urlObj = new URL(requestUrl);
            HttpURLConnection con = (HttpURLConnection) urlObj.openConnection();
            con.setRequestMethod(httpMethod);

            System.out.printf("\nSending %s request to IBM COS -----------------------", httpMethod);
            System.out.println("Request URL = " + requestUrl);

            int responseCode = con.getResponseCode();
            System.out.println("\nResponse from IBM COS ----------------------------------");
            System.out.printf("Response code: %d\n\n", responseCode);

            BufferedReader in = new BufferedReader(new InputStreamReader(con.getInputStream()));
            String inputLine;
            StringBuffer response = new StringBuffer();

            while ((inputLine = in.readLine()) != null) {
                response.append(inputLine);
            }
            in.close();

            //print result
            System.out.println(response.toString());

            con.disconnect();

            // this information can be helpful in troubleshooting, or to better
            // understand what goes into signature creation
            System.out.println("These are the values used to construct this request.");
            System.out.println("Request details -----------------------------------------");
            System.out.printf("http_method: %s\n", httpMethod);
            System.out.printf("host: %s\n", host);
            System.out.printf("region: %s\n", region);
            System.out.printf("endpoint: %s\n", endpoint);
            System.out.printf("bucket: %s\n", bucket);
            System.out.printf("object_key: %s\n", objectKey);
            System.out.printf("timestamp: %s\n", timestamp);
            System.out.printf("datestamp: %s\n", datestamp);
            
            System.out.println("Standardized request details ----------------------------");
            System.out.printf("standardized_resource: %s\n", standardizedResource);
            System.out.printf("standardized_querystring: %s\n", standardizedQuerystring);
            System.out.printf("standardized_headers: %s\n", standardizedHeaders);
            System.out.printf("signed_headers: %s\n", signedHeaders);
            System.out.printf("payload_hash: %s\n", payloadHash);
            System.out.printf("standardized_request: %s\n", standardizedRequest);
            
            System.out.println("String-to-sign details ----------------------------------");
            System.out.printf("credential_scope: %s\n", credentialScope);
            System.out.printf("string-to-sign: %s\n", sts);
            System.out.printf("signature_key: %s\n", signatureKey);
            System.out.printf("signature: %s\n", signature);
            
            System.out.println("Because the signature key has non-ASCII characters, it is");
            System.out.println("necessary to create a hexadecimal digest for the purposes");
            System.out.println("of checking against this example.");

            String signatureKeyHex = createHexSignatureKey(secretKey, datestamp, region, "s3");
            System.out.printf("signature_key (hexidecimal): %s\n", signatureKeyHex);

            System.out.println("Header details ------------------------------------------");
            System.out.printf("pre-signed url: %s\n", requestUrl);
        }
        catch (Exception ex) {
            System.out.printf("Error: %s\n", ex.getMessage());
        }
    }

    private static String toHexString(byte[] bytes) {
		Formatter formatter = new Formatter();
		
		for (byte b : bytes) {
			formatter.format("%02x", b);
		}

		return formatter.toString();
	}

    public static byte[] hash(byte[] key, String msg) {
        byte[] returnVal = null;
        try {
            SecretKeySpec signingKey = new SecretKeySpec(key, "HmacSHA256");
            Mac mac = Mac.getInstance("HmacSHA256");
            mac.init(signingKey);
            returnVal = mac.doFinal(msg.getBytes("UTF8"));
        }
        catch (Exception ex) {
            throw ex;
        }
        finally {
            return returnVal;
        }
    }

	public static String hmacHex(byte[] key, String msg) {
        String returnVal = null;
        try {
            returnVal = toHexString(hash(key, msg));
        }
        catch (Exception ex) {

        }
        finally {
            return returnVal;
        }
    }
    
    public static String hashHex(String msg) {
        String returnVal = null;
        try {
            MessageDigest digest = MessageDigest.getInstance("SHA-256");
            byte[] encodedhash = digest.digest(msg.getBytes(StandardCharsets.UTF_8));
            returnVal = toHexString(encodedhash);
        }
        catch (Exception ex) {
            throw ex;
        }
        finally {
            return returnVal;
        }
    }
    
    // region is a wildcard value that takes the place of the AWS region value
    // as COS doesn"t use the same conventions for regions, this parameter can accept any string
    public static byte[] createSignatureKey(String key, String datestamp, String region, String service) {
        byte[] returnVal = null;
        try {
            byte[] keyDate = hash(("AWS4" + key).getBytes("UTF8"), datestamp);
            byte[] keyString = hash(keyDate, region);
            byte[] keyService = hash(keyString, service);
            byte[] keySigning = hash(keyService, "aws4_request");
            returnVal = keySigning;
        }
        catch (Exception ex) {
            throw ex;
        }
        finally {
            return returnVal;
        }
    }

    public static String createHexSignatureKey(String key, String datestamp, String region, String service) {
        String returnVal = null;
        try {
            returnVal = toHexString(createSignatureKey(key, datestamp, region, service));
        }
        catch (Exception ex) {
            throw ex;
        }
        finally {
            return returnVal;
        }
    }
}
```

## Create a presigned URL to upload an object
{: #presign-url-put-python}

```python
import datetime
import hashlib
import hmac
import requests
from requests.utils import quote

access_key = '{COS_HMAC_ACCESS_KEY_ID}'
secret_key = '{COS_HMAC_SECRET_ACCESS_KEY}'

# request elements
http_method = 'PUT'
region = ''
bucket = 'example-bucket'
cos_endpoint = '{endpoint}'
host = cos_endpoint
endpoint = 'https://' + host
object_key = 'example-object'
expiration = 3600  # time in seconds


# hashing methods
def hash(key, msg):
    return hmac.new(key, msg.encode('utf-8'), hashlib.sha256).digest()


# region is a wildcard value that takes the place of the AWS region value
# as COS doen't use regions like AWS, this parameter can accept any string
def createSignatureKey(key, datestamp, region, service):
    keyDate = hash(('AWS4' + key).encode('utf-8'), datestamp)
    keyRegion = hash(keyDate, region)
    keyService = hash(keyRegion, service)
    keySigning = hash(keyService, 'aws4_request')
    return keySigning


# assemble the standardized request
time = datetime.datetime.utcnow()
timestamp = time.strftime('%Y%m%dT%H%M%SZ')
datestamp = time.strftime('%Y%m%d')

standardized_querystring = ('X-Amz-Algorithm=AWS4-HMAC-SHA256' +
                            '&X-Amz-Credential=' + access_key + '/' + datestamp + '/' + region + '/s3/aws4_request' +
                            '&X-Amz-Date=' + timestamp +
                            '&X-Amz-Expires=' + str(expiration) +
                            '&X-Amz-SignedHeaders=host')
standardized_querystring_url_encoded = quote(standardized_querystring, safe='&=')

standardized_resource = '/' + bucket + '/' + object_key
standardized_resource_url_encoded = quote(standardized_resource, safe='&')

payload_hash = 'UNSIGNED-PAYLOAD'
standardized_headers = 'host:' + host + '\n'
signed_headers = 'host'

standardized_request = (http_method + '\n' +
                        standardized_resource + '\n' +
                        standardized_querystring_url_encoded + '\n' +
                        standardized_headers + '\n' +
                        signed_headers + '\n' +
                        payload_hash)

# assemble string-to-sign
hashing_algorithm = 'AWS4-HMAC-SHA256'
credential_scope = datestamp + '/' + region + '/' + 's3' + '/' + 'aws4_request'
sts = (hashing_algorithm + '\n' +
       timestamp + '\n' +
       credential_scope + '\n' +
       hashlib.sha256(standardized_request).hexdigest())

# generate the signature
signature_key = createSignatureKey(secret_key, datestamp, region, 's3')
signature = hmac.new(signature_key,
                     (sts).encode('utf-8'),
                     hashlib.sha256).hexdigest()

# create and send the request
# the 'requests' package autmatically adds the required 'host' header
request_url = (endpoint + '/' +
               bucket + '/' +
               object_key + '?' +
               standardized_querystring_url_encoded +
               '&X-Amz-Signature=' +
               signature)

print 'request_url: %s' % request_url

print '\nSending `%s` request to IBM COS -----------------------' % http_method
print 'Request URL = ' + request_url
request = requests.put(request_url)

print '\nResponse from IBM COS ---------------------------------'
print 'Response code: %d\n' % request.status_code
print request.text


# this information can be helpful in troubleshooting, or to better
# understand what goes into signature creation
print 'These are the values used to construct this request.'
print 'Request details -----------------------------------------'
print 'http_method: %s' % http_method
print 'host: %s' % host
print 'region: %s' % region
print 'endpoint: %s' % endpoint
print 'bucket: %s' % bucket
print 'object_key: %s' % object_key
print 'timestamp: %s' % timestamp
print 'datestamp: %s' % datestamp

print 'Standardized request details ----------------------------'
print 'standardized_resource: %s' % standardized_resource_url_encoded
print 'standardized_querystring: %s' % standardized_querystring_url_encoded
print 'standardized_headers: %s' % standardized_headers
print 'signed_headers: %s' % signed_headers
print 'payload_hash: %s' % payload_hash
print 'standardized_request: %s' % standardized_request

print 'String-to-sign details ----------------------------------'
print 'credential_scope: %s' % credential_scope
print 'string-to-sign: %s' % sts
print 'signature_key: %s' % signature_key
print 'signature: %s' % signature

 print 'Because the signature key has non-ASCII characters, it is'
 print 'necessary to create a hexadecimal digest for the purposes'
 print 'of checking against this example.'

 def hex_hash(key, msg):
     return hmac.new(key, msg.encode('utf-8'), hashlib.sha256).hexdigest()
 def createHexSignatureKey(key, datestamp, region, service):
     keyDate = hex_hash(('AWS4' + key).encode('utf-8'), datestamp)
     keyRegion = hex_hash(keyDate, region)
     keyService = hex_hash(keyRegion, service)
     keySigning = hex_hash(keyService, 'aws4_request')
     return keySigning

 signature_key_hex = createHexSignatureKey(secret_key, datestamp, region, 's3')

 print 'signature_key (hexidecimal): %s' % signature_key_hex

 print 'Header details ------------------------------------------'
 print 'pre-signed url: %s' % request_url
```

### NodeJS Example
{: #presign-url-put-node}

```javascript
const crypto = require('crypto');
const moment = require('moment');
const https = require('https');

const accessKey = "{COS_HMAC_ACCESS_KEY_ID}";
const secretKey = "{COS_HMAC_SECRET_ACCESS_KEY}";
const httpMethod = 'GET';
const host = '{endpoint}';
const region = '';
const endpoint = 'https://' + host;
const bucket = 'example-bucket';
const objectKey = 'example-object'
const expiration = 86400  // time in seconds

// hashing and signing methods
function hash(key, msg) {
    var hmac = crypto.createHmac('sha256', key);
    hmac.update(msg, 'utf8');
    return hmac.digest();
}

function hmacHex(key, msg) {
    var hmac = crypto.createHmac('sha256', key);
    hmac.update(msg, 'utf8');
    return hmac.digest('hex');
}

function hashHex(msg) {
    var hash = crypto.createHash('sha256');
    hash.update(msg);
    return hash.digest('hex');
}

// region is a wildcard value that takes the place of the AWS region value
// as COS doesn't use the same conventions for regions, this parameter can accept any string
function createSignatureKey(key, datestamp, region, service) {
    keyDate = hash(('AWS4' + key), datestamp);
    keyString = hash(keyDate, region);
    keyService = hash(keyString, service);
    keySigning = hash(keyService, 'aws4_request');
    return keySigning;
}

function createHexSignatureKey(key, datestamp, region, service) {
    keyDate = hashHex(('AWS4' + key), datestamp);
    keyString = hashHex(keyDate, region);
    keyService = hashHex(keyString, service);
    keySigning = hashHex(keyService, 'aws4_request');
    return keySigning;
}

function printDebug() {
    // this information can be helpful in troubleshooting, or to better
    // understand what goes into signature creation
    console.log('These are the values used to construct this request.');
    console.log('Request details -----------------------------------------');
    console.log(`httpMethod: ${httpMethod}`);
    console.log(`host: ${host}`);
    console.log(`region: ${region}`);
    console.log(`endpoint: ${endpoint}`);
    console.log(`bucket: ${bucket}`);
    console.log(`objectKey: ${objectKey}`);
    console.log(`timestamp: ${timestamp}`);
    console.log(`datestamp: ${datestamp}`);

    console.log('Standardized request details ----------------------------');
    console.log(`standardizedResource: ${standardizedResource}`);
    console.log(`standardizedQuerystring: ${standardizedQuerystring}`);
    console.log(`standardizedHeaders: ${standardizedHeaders}`);
    console.log(`signedHeaders: ${signedHeaders}`);
    console.log(`payloadHash: ${payloadHash}`);
    console.log(`standardizedRequest: ${standardizedRequest}`);

    console.log('String-to-sign details ----------------------------------');
    console.log(`credentialScope: ${credentialScope}`);
    console.log(`string-to-sign: ${sts}`);
    console.log(`signatureKey: ${signatureKey}`);
    console.log(`signature: ${signature}`);

    console.log('Because the signature key has non-ASCII characters, it is');
    console.log('necessary to create a hexadecimal digest for the purposes');
    console.log('of checking against this example.');

    signatureKeyHex = createHexSignatureKey(secretKey, datestamp, region, 's3')

    console.log(`signatureKey (hexidecimal): ${signatureKeyHex}`);

    console.log('Header details ------------------------------------------');
    console.log(`pre-signed url: ${requestUrl}`);    
}

// assemble the standardized request
var time = moment().utc();
var timestamp = time.format('YYYYMMDDTHHmmss') + 'Z';
var datestamp = time.format('YYYYMMDD');

var standardizedQuerystring = 'X-Amz-Algorithm=AWS4-HMAC-SHA256' +
    '&X-Amz-Credential=' + encodeURIComponent(accessKey + '/' + datestamp + '/' + region + '/s3/aws4_request') +
    '&X-Amz-Date=' + timestamp +
    '&X-Amz-Expires=' + expiration.toString() +
    '&X-Amz-SignedHeaders=host';

var standardizedResource = '/' + bucket + '/' + objectKey;

var payloadHash = 'UNSIGNED-PAYLOAD';
var standardizedHeaders = 'host:' + host;
var signedHeaders = 'host';

var standardizedRequest = httpMethod + '\n' +
    standardizedResource + '\n' +
    standardizedQuerystring + '\n' +
    standardizedHeaders + '\n' +
    '\n' +
    signedHeaders + '\n' +
    payloadHash;

// assemble string-to-sign
var hashingAlgorithm = 'AWS4-HMAC-SHA256';
var credentialScope = datestamp + '/' + region + '/' + 's3' + '/' + 'aws4_request';
var sts = hashingAlgorithm + '\n' +
    timestamp + '\n' +
    credentialScope + '\n' +
    hashHex(standardizedRequest);

// generate the signature
signatureKey = createSignatureKey(secretKey, datestamp, region, 's3');
signature = hmacHex(signatureKey, sts);

// create and send the request
// the 'requests' package autmatically adds the required 'host' header
var requestUrl = endpoint + '/' +
    bucket + '/' +
    objectKey + '?' +
    standardizedQuerystring +
    '&X-Amz-Signature=' +
    signature;

console.log(`requestUrl: ${requestUrl}`);

console.log(`\nSending ${httpMethod} request to IBM COS -----------------------`);
console.log('Request URL = ' + requestUrl);

// create and send the request
console.log(`\nSending ${httpMethod} request to IBM COS -----------------------`);
console.log('Request URL = ' + requestUrl);

var request = https.get(requestUrl, function (response) {
    console.log('\nResponse from IBM COS ----------------------------------');
    console.log(`Response code: ${response.statusCode}\n`);

    response.on('data', function (chunk) {
        console.log('Response: ' + chunk);
        printDebug();
    });
});

request.end();
```
## Create a presigned URL - PHP
{: #presign-url-get-put-php}

```php
<?php

class Signature
{
    private $accessKey;
    private $secretKey;
    private $region;
    private $endpoint;
    private $host;
    public $httpMethod;
    public $bucket;
    public $objectKey;
    public $expiration;

    public function __construct()
    {
        $this->accessKey = "{COS_HMAC_ACCESS_KEY_ID}";
        $this->secretKey = "{COS_HMAC_SECRET_ACCESS_KEY}";

        $this->httpMethod = "GET"; //change the method to PUT for upload URL
        $this->host = "{endpoint}";
        $this->region = "";
        $this->endpoint = 'https://' . $this->host;
        $this->bucket = "example-bucket";
        $this->objectKey = "example-object";
        $this->expiration = 86400; //time in seconds
    }

    // hashing and signing methods
    public function hash($key, $msg)
    {
        return hash_hmac('sha256', utf8_encode($msg), $key, true);
    }

    public function hashWithoutUTFEncoding($key, $msg)
    {
        return hash_hmac('sha256', $msg, $key, true);
    }

    public function hmacHex($key, $msg)
    {
        return hash_hmac('sha256', $msg, $key, false);
    }

    public function hashHex($msg)
    {
        return hash('sha256', $msg, false);
    }

    // region is a wildcard value that takes the place of the AWS region value
    // as COS doesn't use the same conventions for regions, this parameter can accept any string
    public function createSignatureKey($key, $datestamp, $region, $service)
    {
        $keyDate = $this->hash(('AWS4' . $key), $datestamp);
        $keyString = $this->hashWithoutUTFEncoding($keyDate, $region);
        $keyService = $this->hashWithoutUTFEncoding($keyString, $service);
        $keySigning = $this->hashWithoutUTFEncoding($keyService, 'aws4_request');
        return $keySigning;
    }

    public function createHexSignatureKey($key, $datestamp, $region, $service)
    {
        $keyDate = $this->hashHex(('AWS4' . $key), $datestamp);
        $keyString = $this->hashHex($keyDate, $region);
        $keyService = $this->hashHex($keyString, $service);
        $keySigning = $this->hashHex($keyService, 'aws4_request');
        return $keySigning;
    }

    public function encodeURIComponent($str)
    {
        $revert = array('%21' => '!', '%2A' => '*', '%27' => "'", '%28' => '(', '%29' => ')');
        return strtr(rawurlencode($str), $revert);
    }


    public function getSignature()
    {
        // assemble the standardized request
        $dt = new DateTime();
        $dt->setTimeZone(new DateTimeZone('UTC'));
        $timestamp = $dt->format('Ymd') . "T" . $dt->format('hms') . 'Z';

        $datestamp = $dt->format('Ymd');

        $standardizedQuerystring = 'X-Amz-Algorithm=AWS4-HMAC-SHA256' .
            '&X-Amz-Credential=' . $this->encodeURIComponent($this->accessKey . '/' . $datestamp . '/' . $this->region . '/s3/aws4_request') .
            '&X-Amz-Date=' . $timestamp .
            '&X-Amz-Expires=' . strval($this->expiration) .
            '&X-Amz-SignedHeaders=host';
        $standardizedResource = '/' . $this->bucket . '/' . $this->objectKey;

        $payloadHash = 'UNSIGNED-PAYLOAD';
        $standardizedHeaders = 'host:' . $this->host;
        $signedHeaders = 'host';


        $standardizedRequest = $this->httpMethod . "\n" .
            $standardizedResource . "\n" .
            $standardizedQuerystring . "\n" .
            $standardizedHeaders . "\n" .
            "\n" .
            $signedHeaders . "\n" .
            $payloadHash;

        // assemble string-to-sign
        $hashingAlgorithm = 'AWS4-HMAC-SHA256';
        $credentialScope = $datestamp . '/' . $this->region . '/' . 's3' . '/' . 'aws4_request';
        $sts = $hashingAlgorithm . "\n" .
            $timestamp . "\n" .
            $credentialScope . "\n" .
            $this->hashHex($standardizedRequest);

        // generate the signature
        $signatureKey = $this->createSignatureKey($this->secretKey, $datestamp, $this->region, 's3');

        $signature = $this->hmacHex($signatureKey, $sts);

        // create and send the request
        // the 'requests' package autmatically adds the required 'host' header
        $requestUrl = $this->endpoint . '/' .
            $this->bucket . '/' .
            $this->objectKey . '?' .
            $standardizedQuerystring .
            '&X-Amz-Signature=' .
            $signature;

        error_log(" REQUEST URL  = " . $requestUrl . "\n\n");
        return $requestUrl;
    }
}


$obj = new Signature();
$obj->getSignature();



```
