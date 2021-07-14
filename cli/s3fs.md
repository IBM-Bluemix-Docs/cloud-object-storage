---

copyright:
  years: 2017, 2019
lastupdated: "2019-11-11"

keywords: s3fs, open source, file system, gateway

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

# Mounting a bucket using `s3fs`
{: #s3fs}

Applications that expect to read and write to a NFS-style filesystem can use `s3fs`, which can mount a bucket as directory while preserving the native object format for files. 
{: shortdesc}

This allows you to interact with your cloud storage using familiar shell commands, like `ls` for listing or `cp` to copy files, as well as providing access to legacy applications that rely on reading and writing from local files. For a more detailed overview, [visit the project's official README](https://github.com/s3fs-fuse/s3fs-fuse).

Looking for instructions for how to use {{site.data.keyword.cos_full}} in an {{site.data.keyword.containerlong_notm}} cluster? Go to the [{{site.data.keyword.containerlong_notm}} documentation](/docs/containers?topic=containers-object_storage) instead. 
{: tip}

## Prerequisites
{: #s3fs-prereqs}

* IBM Cloud account and an instance of {{site.data.keyword.cos_full}}
* A Linux or OSX environment
* Credentials (either an [IAM API key](/docs/cloud-object-storage/iam?topic=cloud-object-storage-iam-overview) or [HMAC credentials](/docs/cloud-object-storage?topic=cloud-object-storage-uhc-hmac-credentials-main))

## Installation
{: #s3fs-install}

On OSX, use [Homebrew](https://brew.sh/):

```sh
brew install --cask osxfuse
brew install s3fs
```
{:codeblock}

On Debian or Ubuntu: 

```sh
sudo apt-get install automake autotools-dev fuse g++ git libcurl4-openssl-dev libfuse-dev libssl-dev libxml2-dev make pkg-config
```
{:codeblock}

The official `s3fs` documentation suggests using `libcurl4-gnutls-dev` instead of `libcurl4-openssl-dev`. Either work, but the OpenSSL version may result in better performance. 
{:tip}

You can also build `s3fs` from source. First clone the Github repository:

```sh
git clone https://github.com/s3fs-fuse/s3fs-fuse.git 
```
{:codeblock}

Then build `s3fs`:

```sh
cd s3fs-fuse
./autogen.sh
./configure
make

```
{:codeblock}

And install the binary:

```sh
sudo make install
```
{:codeblock}

## Configuration
{: #s3fs-config}

Store your credentials in a file containing either `<access_key>:<secret_key>` or `:<api_key>`. This file needs to have limited access so run:

```sh
chmod 0600 <credentials_file> 
```
{:codeblock}

Now you can mount a bucket using:

```sh
s3fs <bucket> <mountpoint> -o url=http{s}://<endpoint> –o passwd_file=<credentials_file>
```
{:codeblock}

If the credentials file only has an API key (no HMAC credentials), you'll need to add the `ibm_iam_auth` flag as well:

```sh
s3fs <bucket> <mountpoint> -o url=http{s}://<endpoint> –o passwd_file=<credentials_file> -o ibm_iam_auth
```
{:codeblock}

The `<bucket>` in the example refers to an existing bucket and the `<mountpoint>` is the local path where you want to mount the bucket. The `<endpoint>` must correspond to the [bucket's location](/docs/cloud-object-storage/basics?topic=cloud-object-storage-endpoints). The `credentials_file` is the file created with the API key or HMAC credentials.

Now, `ls <mountpoint>` will list the objects in that bucket as if they were local files (or in the case of object prefixes, as if they were nested directories).

## Performance optimization
{: #s3fs-performance}

While performance will never be equal to a true local filesystem, it is possible to use some advanced options to increase throughput. 

```sh
s3fs <bucket_name> <mountpoint> -o url=http{s}://<COS_endpoint> –o passwd_file=<credentials_file> \
-o cipher_suites=AESGCM \
-o kernel_cache \
-o max_background=1000 \
-o max_stat_cache_size=100000 \
-o multipart_size=52 \
-o parallel_count=30 \
-o multireq_max=30 \
-o dbglevel=warn
```
{:codeblock}

1. `cipher_suites=AESGCM` is only relevant when using an HTTPS endpoint. By default, secure connections to IBM COS use the `AES256-SHA` cipher suite. Using an `AESGCM` suite instead greatly reduces the CPU load on your client machine, caused by the TLS crypto functions, while offering the same level of cryptographic security.
2. `kernel_cache` enables the kernel buffer cache on your `s3fs` mountpoint. This means that objects will only be read once by `s3fs`, as repetitive reading of the same file can be served from the kernel’s buffer cache. The kernel buffer cache will only use free memory which is not in use by other processes. This option is not recommend if you expect the bucket objects to be overwritten from another process/machine while the bucket is mounted, and your use-case requires live accessing the most up-to-date content. 
3. `max_background=1000` improves `s3fs` concurrent file reading performance. By default, FUSE supports file read requests of up to 128 KB. When asking to read more than that, the kernel split the large request to smaller sub-requests and lets s3fs process them asynchronously. The `max_background` option sets the global maximum number of such concurrent asynchronous requests. By default, it is set to 12, but setting it to an arbitrary high value (1000) prevents read requests from being blocked, even when reading a large number of files simultaneously.
4. `max_stat_cache_size=100000` reduces the number of redundant HTTP `HEAD` requests sent by `s3fs` and reduces the time it takes to list a directory or retrieve file attributes. Typical file system usage makes frequent access to a file’s metadata via a `stat()` call which maps to `HEAD` request on the object storage system. By default, `s3fs` caches the attributes (metadata) of up to 1000 objects. Each cached entry takes up to 0.5 KB of memory. Ideally, you would want the cache to be able to hold the metadata for all of the objects in your bucket. However, you may want to consider the memory usage implications of this caching. Setting it to `100000` will take no more than 0.5 KB * 100000 = 50 MB.
5. `multipart_size=52` will set the maximum size of requests and responses sent and received from the COS server, in MB scale. `s3fs` sets this to 10 MB by default. Increasing this value also increases the throughput (MB/s) per HTTP connection. On the other hand, the latency for the first byte served from the file will also increase. Therefore, if your use-case only reads a small amount of data from each file, you probably do not want to increase this value. Furthermore, for large objects (say, over 50 MB) throughput increases if this value is small enough to allow the file to be fetched concurrently using multiple requests. I find that the optimal value for this option is around 50 MB. COS best practices suggest using requests that are multiples of 4 MB, and thus the recommendation is to set this option to 52 (MB).
6. `parallel_count=30` sets the maximum number of requests sent concurrently to COS, per single file read/write operation. By default, this is set to 5. For very large objects, you can get more throughput by increasing this value. As with the previous option, keep this value low if you only read a small amount of data of each file.
7. `multireq_max=30` When listing a directory, an object metadata request (`HEAD`) is sent per each object in the listing (unless the metadata is found in cache). This option limits the number of concurrent such requests sent to COS, for a single directory listing operation. By default it is set to 20. Note that this value must be greater or equal to the `parallel_count` option above.
8. `dbglevel=warn` sets the debug level to `warn` instead of the default (`crit`) for logging messages to /var/log/syslog.

## Limitations
{: #s3fs-limitations}

It is important to remember that s3fs may not be suitable for all applications, as object storage services have high-latency for time to first byte and lack random write access. Workloads that only read big files, like deep learning workloads, can achieve good throughput using `s3fs`. 
