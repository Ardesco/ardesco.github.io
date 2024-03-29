---
layout: post
author: "Mark Collin"
title: "Waiting for Angular"
image: /assets/images/feed/waiting.jpg
attribution: "Photo by Joshua Earle on Unsplash"
category: Testing
tags: [Java, AngularJS, Selenium, WebDriver]
---
Recently I've spent a fair bit of time working with Angular JS applications and as great as Angular is, it can be a pain when it comes to automating it.

The main problem you will probably see is due to the fact that Angular does everything asynchronously, so you're never quite sure when the page has finished loading, if only there was a way to know Angular had finished before you started doing stuff on a page...

Here's an ExpectedCondition that will wait for Angular to finish processing stuff on the page:

```java
public static ExpectedCondition angularHasFinishedProcessing() {
    return new ExpectedCondition() {
        @Override
        public Boolean apply(WebDriver driver) {
            JavascriptExecutor jsexec = ((JavascriptExecutor) driver);
            String result = (String) jsexec.executeScript("return (window.angular != null) && " +
                    "(angular.element(document).injector() != null) && " +
                    "(angular.element(document).injector().get('$http').pendingRequests.length === 0)");
            return Boolean.valueOf(result);
        }
    };
}
```
