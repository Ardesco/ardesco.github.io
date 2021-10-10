---
layout: post
author: "Mark Collin"
title: "JMeter Maven Plugin 3.5.0 Released"
image: /assets/images/feed/jmeter_maven.png
attribution: "Jmeter Maven plugin"
category: OSS
tags: [Automated Testing, JMeter, Performance Testing]
---
This is a minor release that performed a bit of code cleanup and added a small new feature.

As always the source code is available on [Github](https://github.com/jmeter-maven-plugin/jmeter-maven-plugin).

## Release Notes

- Updated all dependencies to get the build working on JDK 16 and JDK 17.  Hopefully this should resolve the issues reported in [Issue #412](https://github.com/jmeter-maven-plugin/jmeter-maven-plugin/issues/412).
- Refactored the results file scanning code.  Parsing of CSV files should be a bit quicker now.
- Added the ability to fail the build if the results file is empty (i.e. no successes or failures)

```xml
<configuration>
    <failBuildIfResultFileIsEmpty>true</failBuildIfResultFileIsEmpty>
</configuration>
```
