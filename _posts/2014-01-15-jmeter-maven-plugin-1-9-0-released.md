---
layout: post
author: "Mark Collin"
title: "JMeter Maven Plugin 1.9.0 Released"
redirect_from:
- "/index.php/2014/01/jmeter-maven-plugin-version-1-9-0-released/"
image: /assets/images/feed/jmeter_maven.png
attribution: "Jmeter Maven plugin"
category: OSS
tags: [Automated Testing, JMeter, Performance Testing]
---
It has been a while since the last release and there are quite a few fixes and improvements.

One notable exclusion is JMeter 2.11 support, this is due to some dependency problems with the 2.11 JMeter artifacts.  We plan to make another release to support 2.11 as soon as everything is in place.

So on to the release notes:

**Version 1.9.0 Release Notes**

- JMeter version 2.10 support added.
- Issue #56 - Now using a ProcessBuilder to isolate the JVM JMeter runs in.
- Merge pull request #70 from Erik G. H. Meade - Add requiresDirectInvocation true to JMeterMojo.
- Issue #71 - Fixed documentation errors.
- Issue #63 - Fixed remote configuration documentation errors.
- Merge pull request #73 from Zmicier Zaleznicenka - Added missed dependency causing file not found / error in NonGUIDriver error.
- Issue #72 - Remove the maven site from the plugin.
- Issue #73 - Add missing dependency for ApacheJMeter-native.
- Issue #84 - Correctly place explicit dependencies in the /lib directory.
- Issue #66 - Jmeter lib directory contains additional jars.
- Issue #75 - Allow empty propertiesUser properties.
- Issue #80 - Integration Tests Failing With Maven 2.
- Issue #77 - JMeter plugins artifacts now placed in lib/ext directory. You can specify which artifacts are JMeter plugins using the new jmeterPlugins configuration setting:

```xml
<configuration>
    <jmeterPlugins>
        <plugin>
            <groupId>my.group</groupId>
            <artifactId>my.artifact</artifactId>
        </plugin>
    </jmeterPlugins>
</configuration>
```

- Added the ability to configure the JMeter JVM:

```xml
<configuration>
    <jMeterProcessJVMSettings>
        <xms>1024</xms>
        <xmx>1024</xmx>
        <arguments>
            <argument>-Xprof</argument>
            <argument>-Xfuture</argument>
        </arguments>
    </jMeterProcessJVMSettings>
</configuration>
```

- Issue #82 - Allow users to specify the resultsDir:

```xml
<configuration>
    <resultsDirectory>/tmp/jmeter</resultsDirectory>
</configuration>
```

- Issue #64 - Remote execution seems to be stopping before agent stops running the tests.
- Merge pull request #78 from Mike Patel - Changes to allow system / global jmeter properties to be sent to remote clients.
- Issue #89 - Add support for advanced log config. If you add a "logkit.xml" into the &lt;testFilesDirectory&gt; it will now be copied into the /bin folder. If one does not exist the default one supplied with JMeter will be used instead. If you don't want to call your advanced log config file "logkit.xml", you can specify the filename using:

```xml
<configuration>
    <logConfigFilename>myFile.xml</logConfigFilename>
</configuration>
```

- Issue #88 - ApacheJMeter_mongodb dependency is not in POM
