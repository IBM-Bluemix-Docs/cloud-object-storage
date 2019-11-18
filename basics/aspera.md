---

copyright:
  years: 2017, 2018, 2019
lastupdated: "2019-04-12"

keywords: aspera, high speed, big data, packet loss

subcollection: cloud-object-storage

---
{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:important: .important}
{:note: .note}
{:download: .download}
{:java: .ph data-hd-programlang='java'} 
{:python: .ph data-hd-programlang='python'} 

# Use Aspera high-speed transfer
{: #aspera}

Aspera high-speed transfer overcomes the limitations of traditional FTP and HTTP transfers to improve data transfer performance under most conditions, especially in networks with high latency and packet loss. Instead of the standard HTTP `PUT`, Aspera high-speed transfer uploads the object by using the [FASP protocol](https://asperasoft.com/technology/transport/fasp/). Using Aspera high-speed transfer for uploads and downloads offers the following benefits:

- Faster transfer speeds
- Transfer large object uploads over 200 MB in the console and 1 GB by using an SDK or library
- Upload entire folders of any type of data, such as multi-media files, disk images, and any other structured or unstructured data
- Customize transfer speeds and default preferences
- Transfers can be viewed, paused, resumed, or cancelled independently

Aspera high-speed transfer is available in the {{site.data.keyword.cloud_notm}} [console](#aspera-console) and can also be used programmatically by using an [SDK](#aspera-sdk). 

Aspera high-speed transfer is available in certain regions only. See [Integrated Services](/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-service-availability) for more details.
{:tip}

It isn't possible to use Aspera high-speed transfer if the bucket has an Immutable Object Storage policy.
{:important}

## Using the console
{: #aspera-console}

If you add objects by using the console in a [supported region](/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-service-availability), you are prompted with an option to install the Aspera Connect client. This browser plug-in provides Aspera high-speed transfer to upload files or folders.

### Install Aspera Connect
{: #aspera-install}

1. Select **Install Aspera Connect** client.
2. Follow the installation instructions for your operating system and browser.
3. Resume file or folder upload.

The Aspera Connect plug-in can also be installed from the [Aspera website](https://downloads.asperasoft.com/connect2/) directly. For help troubleshooting issues with the Aspera Connect plug-in, [see the documentation](https://downloads.asperasoft.com/en/documentation/8).

After the plug-in is installed, you have the option to set Aspera high-speed transfer as the default for any uploads to the target bucket that use the same browser. Select **Remember my browser preferences**. Options are also available in the bucket configuration page under **Transfer options**. These options allow you to choose between Standard and High speed as the default transport for uploads and downloads.

Typically, using the IBM Cloud Object Storage web-based console isn't the most common way to use {{site.data.keyword.cos_short}}. The Standard transfer option limits objects size to 200 MB and the file name and key will be the same. Support for larger object sizes and improved performance (depending on network factors) is provided by Aspera high-speed transfer.

An Aspera server runs one SSH server on a configurable TCP port (22 by default, but often customers use port 33001). The firewall on the server side must allow this one TCP port to reach the Aspera server.
No servers are listening on UDP ports. When a transfer is initiated by an Aspera client, the client opens an SSH session to the SSH server on the designated TCP port and negotiates the UDP port over which the data will travel. By default, Aspera clients and servers are configured to use UDP port 33001.
After the session initiation step, both the client and the server will send and receive UDP traffic on the negotiated port. To allow the UDP session to start, the firewall on the Aspera server side must allow port UDP 33001 to reach the Aspera server.[Firewall Considerations](https://www.ibm.com/support/pages/firewall-considerations)
{:important}

### Transfer status
{: #aspera-console-transfer-status}

**Active:** Once you initiate a transfer, the transfer status displays as active. While the transfer is active, you can pause, resume, or cancel an active transfer. 

**Completed:** Upon completion of your transfer, information about this and all transfers in this session display on the completed tab. You can clear this information. You will only see information about transfers that are completed in the current session.

**Preferences:** You can set the default for uploads and downloads to High speed.

Downloads that use Aspera high-speed transfer incur egress charges. For more information, see the [pricing page](https://www.ibm.com/cloud/object-storage).
{:tip}

**Advanced Preferences:** You can set bandwidth for uploads and downloads.

----

## Using Libraries and SDKs
{: #aspera-sdk}

Aspera high-speed transfer supports Java and Python.

### When to use Aspera High-Speed Transfer
{: #aspera-guidance}

The FASP protocol that Aspera high-speed transfer uses is not suited for all data transfers:

1. Always make use of multiple sessions - at least two parallel sessions will best use Aspera high-speed transfers capabilities. See specific guidance for [Java](/docs/services/cloud-object-storage/libraries?topic=cloud-object-storage-java#java-examples-aspera) and [Python](/docs/services/cloud-object-storage/libraries?topic=cloud-object-storage-python#python-examples-aspera).
2. Aspera high-speed transfer is ideal for larger files, and any files or directories that contain a total amount of data less than one GB will instead transfer the object in multiple parts by using the standard Transfer Manager classes. Aspera high-speed transfers require a longer time-to-first-byte than normal HTTP transfers. The instantiation of many Aspera Transfer Manager objects to manage the transfers of individual smaller files can result in subpar performance. It is best to instantiate a single client to upload a directory of smaller files instead.
3. Aspera high-speed transfer was designed in part to improve performance in network environments with large amounts of packet loss, making the protocol performant over large distances and public wide area networks. Aspera high-speed transfer should not be used for transfers within a region or data center.

The Aspera high-speed transfer SDK is closed-source and thus an optional dependency for the COS SDK (which uses an Apache license). 
{:tip}

#### COS/Aspera High-Speed Transfer Packaging
{: #aspera-packaging}


<img alt="COS/Aspera High-Speed Transfer SDK." src="https://s3.us.cloud-object-storage.appdomain.cloud/docs-resources/aspera-packaging.png" height="200px" />
{: caption="Figure 1: COS/Aspera High-Speed Transfer SDK." caption-side="bottom"} 

### Supported Platforms
{: #aspera-sdk-platforms}

| OS                     | Version   | Architecture | Tested Java Version | Tested Python Version |
|------------------------|-----------|--------------|--------------|----------------|
| Ubuntu                 | 18.04 LTS | 64-Bit       | 6 and higher | 2.7, 3.6       |
| Mac OS X               | 10.13     | 64-Bit       | 6 and higher | 2.7, 3.6       |
| Microsoft&reg; Windows | 10        | 64-Bit       | 6 and higher | 2.7, 3.6       |

Each Aspera high-speed transfer session creates an individual `ascp` process that runs on the client machine to perform the transfer. Ensure that your computing environment can allow this process to run.
{:tip}

**More limitations**

* 32-bit binaries are not supported
* Windows support requires Windows 10
* Linux support is limited to Ubuntu (tested against the 18.04 LTS)
* Aspera Transfer Manager clients must be created that use IAM API keys and not HMAC credentials.

### Getting the SDK
{: #aspera-sdk-java} 
{: java}

There is two SDKS, Java and Python, which can be revealed in the links at the top of the page.
Return to the top by [clicking here]( /docs/services/cloud-object-storage?topic=cloud-object-storage-aspera).
{:note}

The best way to use {{site.data.keyword.cos_full_notm}} and Aspera high-speed transfer Java SDK is to use Maven to manage dependencies. If you aren't familiar with Maven, you get can get up and running with the [Maven in 5 Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html){:new_window} guide.
{: java}

Maven uses a file named `pom.xml` to specify the libraries (and their versions) needed for a Java project. Below is an example `pom.xml` file for using the {{site.data.keyword.cos_full_notm}} and Aspera high-speed transfer Java SDK
{: java}

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.cos</groupId>
    <artifactId>docs</artifactId>
    <packaging>jar</packaging>
    <version>2.0-SNAPSHOT</version>
    <name>docs</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>com.ibm.cos</groupId>
            <artifactId>ibm-cos-java-sdk</artifactId>
            <version>2.1.3</version>
        </dependency>
        <dependency>
            <groupId>com.ibm.cos-aspera</groupId>
            <artifactId>cos-aspera</artifactId>
            <version>0.1.163682</version>
        </dependency>
    </dependencies>
</project>
```
{: codeblock}
{: java}

Examples of initiating Aspera high-speed transfers with Java are available in the [Using Aspera High-Speed Transfer](/docs/services/cloud-object-storage/libraries?topic=cloud-object-storage-java#java-examples-aspera) section.
{: java}

### Getting the SDK
{: #aspera-sdk-python} 
{: python}

There is two SDKS, Java and Python, which can be revealed in the links at the top of the page.
Return to the top by [clicking here]( /docs/services/cloud-object-storage?topic=cloud-object-storage-aspera).
{:note}

The {{site.data.keyword.cos_full_notm}} and Aspera high-speed transfer Python SDKs are available from the Python Package Index (PyPI) software repository. 
{: python}

```
pip install cos-aspera
```
{: codeblock}
{: python}

Examples of initiating Aspera transfers with Python are available in [Using Aspera High-Speed Transfer](/docs/services/cloud-object-storage/libraries?topic=cloud-object-storage-python#python-examples-aspera) section.
