---
layout: post
author: "Mark Collin"
title: "Fixing JMeter security flaws with the JMeter Maven Plugin"
image: /assets/images/feed/jmeter_maven.png
attribution: "Jmeter Maven plugin"
category: OSS
tags: [Automated Testing, JMeter, Performance Testing]
---

You must have been living in a hole if you haven't heard of the recent Log4J vulnerabilities which have caused a flurry of releases of various software packages.  Unfortunately JMeter was also affected and the JMeter team has been hard at work releasing new versions to upgrade the affected Log4J library.  

We don't use the Log4J library directly in the Jmeter Maven Plugin, so in this respect we got lucky; however there have been a couple of requests for new releases to change the JMeter version used by the JMeter Maven Plugin from people worried about the Log4J vulnerabilities.  With this in mind I thought it may be a good idea to show how the JMeter Maven Plugin can be used to modify the JMeter version and work around library vulnerabilities.  

## Modify the system properties used by JMeter to disable the vulnerability

For this Log4J vulnerability, there is a workaround that will allow you to disable it a properties setting.  This is a straight forward fix with the JMeter Maven Plugin, we just need to modify the system properties in the plugin configuration block.

```xml
<plugin>
    <groupId>com.lazerycode.jmeter</groupId>
    <artifactId>jmeter-maven-plugin</artifactId>
    <version>3.5.0</version>
    <executions>
        <!-- Generate JMeter configuration -->
        <execution>
            <id>configuration</id>
            <goals>
                <goal>configure</goal>
            </goals>
        </execution>
        <!-- Run JMeter tests -->
        <execution>
            <id>jmeter-tests</id>
            <goals>
                <goal>jmeter</goal>
            </goals>
        </execution>
        <!-- Fail build on errors in test -->
        <execution>
            <id>jmeter-check-results</id>
            <goals>
                <goal>results</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!-- Add a System property version -->
        <propertiesSystem>
            <log4j2.formatMsgNoLookups>true</log4j2.formatMsgNoLookups>
        </propertiesSystem>
    </configuration>
</plugin>
```

If you want some more information showing how to add/change properties check out the JMeter Maven Plugin wiki at the [modifying properties](https://github.com/jmeter-maven-plugin/jmeter-maven-plugin/wiki/Modifying-Properties) section.

## Specify an updated JMeter version via config

Next we have the most obvious fix to the problem, update the version of JMeter used by the JMeter Maven Plugin.  

The JMeter Maven Plugin allows you to configure the version of JMeter that it uses, it's not hard coded.  It's as simple as adding a configuration element to specify the version of JMeter you wish to use.

```xml
<plugin>
    <groupId>com.lazerycode.jmeter</groupId>
    <artifactId>jmeter-maven-plugin</artifactId>
    <version>3.5.0</version>
    <executions>
        <!-- Generate JMeter configuration -->
        <execution>
            <id>configuration</id>
            <goals>
                <goal>configure</goal>
            </goals>
        </execution>
        <!-- Run JMeter tests -->
        <execution>
            <id>jmeter-tests</id>
            <goals>
                <goal>jmeter</goal>
            </goals>
        </execution>
        <!-- Fail build on errors in test -->
        <execution>
            <id>jmeter-check-results</id>
            <goals>
                <goal>results</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!-- Override JMeter version -->
        <jmeterVersion>5.4.3</jmeterVersion>
    </configuration>
</plugin>
```

This can get more complicated if the list of artifacts JMeter supplies changes, but it's fine you can use configuration to specify the artifact list as well, for more details check out the JMeter Maven Plugin wiki at the [specifying JMeter version](https://github.com/jmeter-maven-plugin/jmeter-maven-plugin/wiki/Specifying-JMeter-Version) section.

## Override the vulnerable version of log4j-core

What happens if there is no version of JMeter that uses the latest libraries?  Well you can force an override by specifying a library version.  Maven artifact resolution will ensure that only the latest version is downloaded.

Here is an example showing you how to override the version of log4j-core used by JMeter by updating the version copied over to the lib directory.

```xml
<plugin>
    <groupId>com.lazerycode.jmeter</groupId>
    <artifactId>jmeter-maven-plugin</artifactId>
    <version>3.5.0</version>
    <executions>
        <!-- Generate JMeter configuration -->
        <execution>
            <id>configuration</id>
            <goals>
                <goal>configure</goal>
            </goals>
        </execution>
        <!-- Run JMeter tests -->
        <execution>
            <id>jmeter-tests</id>
            <goals>
                <goal>jmeter</goal>
            </goals>
        </execution>
        <!-- Fail build on errors in test -->
        <execution>
            <id>jmeter-check-results</id>
            <goals>
                <goal>results</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!-- Specify a library version -->
        <testPlanLibraries>
            <artifact>org.apache.logging.log4j:log4j-core:2.17.0</artifact>
        </testPlanLibraries>
    </configuration>
</plugin>
```

To see the various options for overriding libraries check out the JMeter Maven Plugin wiki at the [adding/excluding libraries to/from the classpath](https://github.com/jmeter-maven-plugin/jmeter-maven-plugin/wiki/Adding-Excluding-libraries-to-from-the-classpath) section.
