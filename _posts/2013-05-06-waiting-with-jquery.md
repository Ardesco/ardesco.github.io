---
layout: post
author: "Mark Collin"
title: "Waiting with jQuery"
image: /assets/images/feed/wait.jpg
attribution: "Photo by Phil Hearing on Unsplash"
category: Testing
tags: [jQuery, Selenium, WebDriver]
---
Waiting can be hard, so here are a couple of useful tricks to use with jQuery:

First of all, have you ever tried to interact with something on the screen only for some background AJAX call to change what is on the screen at the last possible moment as if it was purposfully trying to break your test?  Well lets get rid of that problem by waiting until all AJAX calls have finished processing:

```java
public static ExpectedCondition<Boolean> jQueryAJAXCallsHaveCompleted() {
    return new ExpectedCondition<Boolean>() {

        @Override
        public Boolean apply(WebDriver driver) {
            return (Boolean) ((JavascriptExecutor) driver).executeScript("return (window.jQuery != null) && (jQuery.active === 0);");
        }
    };
}
```

Bear in mind that this will wait until there are no outstanding AJAX calls, once this condition has been met something sneaky could then fire off another AJAX call just to be awkward so it's not totally foolproof, it should help increase reliability however.

Secondly, have you ever tried to click on an element that is supposed to do something special (e.g. save a form, sort a table, make something magical happen on screen, make a drop down box appear on mouseover, etc) and once you have clicked on it found that nothing happened?  The usual thing to do is blame Selenium because it didn't click on your element, but have you ever thought that Selenium is so fast that it managed to click on the element before the JavaScript that is rendering the page managed to register a listener on the element that you are about to interact with?  You may want to try this:

```java
public static ExpectedCondition<Boolean> listenerIsRegisteredOnElement(final String listenerType, final WebElement element) {
    return new ExpectedCondition<Boolean>() {
        @Override
        public Boolean apply(WebDriver driver) {
            Map<String, Object> registeredListeners = (Map<String, Object>) ((JavascriptExecutor) driver).executeScript("return jQuery._data(jQuery(arguments[0]).get(0), 'events')", element);
            for (Map.Entry<String, Object> listener : registeredListeners.entrySet()) {
                if (listener.getKey().equals(listenerType)) {
                    return true;
                }
            }
            return false;
        }
    };
}
```

You would use it like this:

```java
WebElement myDropDownMenu = driver.findElement(By.id("menu"));
wait.until(listenerIsRegisteredOnElement("mouseover", myDropDownMenu ))
```

This would make selenium wait until a mouseover listener has been applied to a dropdown menu element (Obviously this example assumes that the menu dropdown is being performed using jQuery).

The above will only work if your site is using jQuery and jQuery is triggering the relevant actions so it is limited, however it is hopefully useful as well, enjoy.
