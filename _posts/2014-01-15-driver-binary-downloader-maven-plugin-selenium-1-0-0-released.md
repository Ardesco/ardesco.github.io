---
layout: post
author: "Mark Collin"
title: "The Driver Binary Downloader Maven Plugin for Selenium 1.0.0 Released"
date: 2014-01-15 23:02:00 +0100
categories: [Testing, WebDriver] 
tags: [Automated Testing, WebDriver, Maven]
---
The initial stable release of the driver-binary-downloader-maven-plugin has been released. This brings in the following changes:

- Improved the performance of the unzip code (things are much quicker now).
- Only download binaries for the current OS (note more pulling down windows binaries on your linux box).
- PhantomJS support so that you can get GhostDriver(PhantomJSDriver) up and running with minimal effort.

To use it is very simple, just add the following to your POM:

```xml
    <plugins>
        <plugin>
            <groupId>com.lazerycode.selenium</groupId>
            <artifactId>driver-binary-downloader-maven-plugin</artifactId>
            <version>1.0.0</version>
            <configuration>
                <!-- root directory that downloaded driver binaries will be stored in -->
                <rootStandaloneServerDirectory>/my/location/binaries</rootStandaloneServerDirectory>
                <!-- Where you want to store downloaded zip files -->
                <downloadedZipFileDirectory>/my/location/zips</downloadedZipFileDirectory>
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

For more information see the project on [Github](https://github.com/Ardesco/selenium-standalone-server-plugin">https://github.com/Ardesco/selenium-standalone-server-plugin){:target="_blank"}
If you want to see it in action have a look at my [Selenium Maven Template](https://github.com/Ardesco/Selenium-Maven-Template">https://github.com/Ardesco/Selenium-Maven-Template){:target="_blank"}