---
layout: post
author: "Mark Collin"
title: "Introducing The Driver Binary Downloader Maven Plugin For Selenium"
category: Testing
tags: [Automated Testing, Java, Selenium, WebDriver]
---
I'm a regular Maven user and one thing that has annoyed me for a while with Selenium is the standalone server binaries.  Not because I think the concept is a bad thing, but because it adds a dependency that has to be manually downloaded, in my mind this kind of defeats the object  of using Maven.  It wasn't so bad when it was just the googlechrome executable, but recently the IEDriver has gone the same way and from Jim Evans' presentation at at the Selenium conference  this year, it looks like everybody else will be following the trend at some point.

Well this left me with an itch to scratch and I have finally got around to doing something about it, introducing the driver-binary-downloader-maven-plugin.  It's a bit of a mouthful but I didn't want to use server in the name as people would think it was going to download the selenium server binaries, anyway it's just a name, how bad can it be?

I have released it today (and it's now available in the central Maven repository) at version 0.9.0 as it's fully functional, but in my mind still in beta stage.

## Basic Usage

The basic configuration is very simple, you tell it where to download things and it will go off and do it.
```xml
<plugins>
    <plugin>
        <groupId>com.lazerycode.selenium</groupId>
        <artifactId>driver-binary-downloader-maven-plugin</artifactId>
        <version>0.9.1</version>
        <configuration>
            <!-- root directory that downloaded driver binaries will be stored in -->
            <rootStandaloneServerDirectory>/my/location/for/driver/binaries</rootStandaloneServerDirectory>
            <!-- Where you want to store downloaded zip files -->
            <downloadedZipFileDirectory>/my/location/for/downloaded/zip/files</downloadedZipFileDirectory>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>selenium</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
</plugins>
```

The way the plugin works is by downloading the zipped binaries into the `<downloadedZipFileDirectory>` and then performing a hash check on them to ensure that the zip files are valid.  If the files are valid it extracts the contents of the file in a directory structure based upon your &amp;lt;rootStandaloneServerDirectory&amp;gt;.  The file structure will be ${root.server.directory}/${driver.name}/${operating.system}/${bit.rate}/${version}/$driver.binary}.  So for example if you have downloaded the 64bit googlechrome driver binary version 22 for linux and your `<rootStandaloneServerDirectory>` is "/binaries" the absolute path to it will be "/binaries/googlechrome/linux/64bit/22/googlechrome".

By default the plugin is run in the 'test-compile' phase so binaries will always be downloaded before your tests start to run.

## What Do I Do With It?

Your mavenised test suite can now automatically download any required standalone server binaries without you needing to manually download them and place then in a shared area.  If you had them checked into your source control repository (it happens) you can now remove them.

Since you now know where the binaries are going to exist once they have been downloaded you can use them in your selenium tests secure in the knowledge that if they don't exist they will don't once you have run a mvn verify

## Known Problems

This is only a beta and as a result may not be perfect, there is currently one problem that I currently know about

- If the latest version of a driver does not have binaries for all the specified operating systems (all are specified by default) it will go back to the previous version and download all of those binaries as well.

## Advanced Usage

While the above is useful, it's not exactly extensible and doesn't really help if you don't want to download things from the internet every time.  With this in mind there is a more advanced set of configuration options available.

```xml
<plugins>
    <plugin>
        <groupId>com.lazerycode.selenium</groupId>
        <artifactId>driver-binary-downloader-maven-plugin</artifactId>
        <version>0.9.1</version>
        <configuration>
            <!-- root directory that downloaded driver binaries will be stored in -->
            <rootStandaloneServerDirectory>/tmp/binaries</rootStandaloneServerDirectory>
            <!-- Where you want to store downloaded zip files -->
            <downloadedZipFileDirectory>/tmp/zips</downloadedZipFileDirectory>
            <!-- Location of a custom repository map -->
            <customRepositoryMap>/tmp/repo.xml</customRepositoryMap>
            <!-- Operating systems you want to download binaries for (Only valid options are: windows, linux, osx) -->
            <operatingSystems>
                <windows>true</windows>
                <linux>true</linux>
                <osx>true</osx>
            </operatingSystems>
            <!-- Download 32bit binaries -->
            <thirtyTwoBitBinaries>true</thirtyTwoBitBinaries>
            <!-- Download 64bit binaries -->
            <sixtyFourBitBinaries>true</sixtyFourBitBinaries>
            <!-- If set to false will download every version available (Other filters will be taken into account -->
            <onlyGetLatestVersions>false</onlyGetLatestVersions>
            <!-- Provide a list of drivers and binary versions to download (this is a map so only one version can be specified per driver) -->
            <getSpecificExecutableVersions>
                <googlechrome>18</googlechrome>
            </getSpecificExecutableVersions>
            <!-- Number of times to attempt to download each file -->
            <fileDownloadRetryAttempts>2</fileDownloadRetryAttempts>
            <!-- Number of ms to wait before timing out when trying to connect to remote server to download file -->
            <fileDownloadConnectTimeout>20000</fileDownloadConnectTimeout>
            <!-- Number of ms to wait before timing out when trying to read file from remote server -->
            <fileDownloadReadTimeout>10000</fileDownloadReadTimeout>
            <!-- Overwrite any existing binaries that have been downloaded and extracted -->
            <overwriteFilesThatExist>true</overwriteFilesThatExist>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>selenium</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
</plugins>
```

The above options are fairly self explanatory so I'm not going to go into all of them in detail here, the most useful option however is probably `<customRepositoryMap>`.  This option will allow you to specify your own RepositoryMap.xml (the file that specifies the drivers versions and download locations).  This can enable you to have a local mirror for the driver binary zip files that you connect to, to download your Selenium server binaries to your local machine.

The repository map is checked using an xsd when it is loaded in so if you are planning on rolling your own you will probably find it useful to look at the current [RepositoryMap.xml](https://github.com/Ardesco/selenium-standalone-server-plugin/blob/master/src/main/resources/RepositoryMap.xml){:target="_blank"} and the [RepositoryMap.xsd](https://github.com/Ardesco/selenium-standalone-server-plugin/blob/master/src/main/resources/RepositoryMap.xsd){:target="_blank"} that it is checked against.

If you want to have a look at the code for this plugin it is all publicly available here: [https://github.com/Ardesco/selenium-standalone-server-plugin](https://github.com/Ardesco/selenium-standalone-server-plugin){:target="_blank"}

## Feedback

The one thing I really want now is feedback.  Does it work the way you expect?  Can you think of any improvements?

If you find any bugs please raise them on the [Issues Page](https://github.com/Ardesco/selenium-standalone-server-plugin/issues){:target="_blank"}.  The only way it will improve is if I know there are problems.

**\*Edit**

Just pushed out 0.9.1 to fix a problem where extracted files were not always executable, the plugin now explicitly makes the files it extracts executable.  I have updated the example POM files above with the new version number.
