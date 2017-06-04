---
layout: post
author: "Mark Collin"
title: "JMeter Maven Plugin 1.8.0 Released"
date: 2013-03-13 22:11:00 +0100
categories: [JMeter, Testing] 
tags: [Automated Testing, JMeter, Performance Testing]
---
I'm a bit late adding this here (I've been distracted updating the wiki for the plugin and doing a bit of running around closing off issues) but thought it would be a good idea to start posting stuff about the JMeter Maven plugin here as well.  Version 1.8.0 of the JMeter Maven Plugin is now available in maven central.

The source code is available on [Github](https://github.com/Ronnie76er/jmeter-maven-plugin){:target="_blank"} and there is now also an up to date [Wiki](https://github.com/Ronnie76er/jmeter-maven-plugin/wiki){:target="_blank"} as well.

## Release Notes


- Added support for JMeter version 2.9.
- Fixed issue **#61** - Added skipTests ability.  You can now add a configuration option to skip tests, use it like this:

```xml
<properties>
    <skipTests>false</skipTests>
</properties>

<plugin>
    <groupId>com.lazerycode.jmeter</groupId>
    <artifactId>jmeter-maven-plugin</artifactId>
    <version>1.8.0</version>
    <executions>
        <execution>
            <id>jmeter-tests</id>
            <phase>verify</phase>
            <goals>
                <goal>jmeter</goal>
            </goals>
            <configuration>
                <skipTests>${skipTests}</skipTests>
            </configuration>
        </execution>
    </executions>
</plugin>
```


If you now run:

```bash
mvn verify –DskipTests=true
```

The performance tests will be skipped.

- **#58**,**#59** - Add dependencies with custom function to /lib/ext folder (pull request made by dpishchukhin that has been merged in).
- Removed jmx file sorting code as it was not sorting files into a deterministic order.  Tests are run in the order the plugin discovers them on disk now.
- Removed checks for **`<error>true</error>`** and **`<failure>true</failure>`** in .jtl files, these elements do not occur in JMeter 2.9.
- Added ability to choose whether to Append or Prepend date to filename using the new `<appendResultsTimestamp>` configuration option (Valid values are: **TRUE**,**FALSE**):

```xml
<appendResultsTimestamp>false</appendResultsTimestamp>
```


- Set default timestamp to an ISO_8601 timestamp.  The formatter now used in the configuration option `<resultsFileNameDateFormat>`is a JodaTime DateTimeFormatter (See [http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html){:target="_blank"}):

```xml
<resultsFileNameDateFormat >MMMM, yyyy</resultsFileNameDateFormat >
```


- Added the ability to override the root log level using the new “overrideRootLogLevel” configuration option (Valid log levels are **FATAL_ERROR**, **ERROR**, **WARN**, **INFO** and **DEBUG**):

```xml
<overrideRootLogLevel>DEBUG</overrideRootLogLevel>
```


- Failure scanner refactored to use a Boyer-Moore algorithm to increase performance on large results files, you should hopefully see some improvements in speed when the plugin is checking your results files for the presence of failures.
- Added the ability to set the result file format  using a new “resultsFileFormat” configuration option (Valid options are **XML** and **CSV**, it will default to **XML**):

```xml
<resultsFileFormat>CSV</resultsFileFormat>
```


- Modified remote configuration settings, configuration options are now:

```xml
<remoteConfiguration>
	<startAndStopServersForEachTest>false</startAndStopServersForEachTest>
	<startServersBeforeTests>true</startServersBeforeTests>
	<stopServersAfterTests>true</stopServersAfterTests>
	<serverList>server1,server2</serverList>
</remoteConfiguration>
```

If you use “**startAndStopServersForEachTest**” it will override “**startServersBeforeTests**” and “**stopServersAfterTests**” if they have been configured as well.
