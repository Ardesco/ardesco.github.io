---
layout: post
author: "Mark Collin"
title: "Stop Moving So I Can Click You Dammit!"
category: Testing
tags: [Java, WebDriver]
---
This is a little trick that some may find useful.

I re-factored some tests that were checking an accordion control on Friday to speed things up, unfortunately when I was done I started getting some intermittent failures.  It seemed that I was now sometimes unable to open up one of the accordion elements.  A bit of head scratching and some time in the debugger with nothing obvious jumping out at me I finally realised what it was.  I was sometimes clicking on an element to open up the next accordion whilst it was still moving (all because I made things run faster).  

The solution?  Wait until the element has finished moving.

Hers is an Expected condition that you can use with WebDriverWait:

```Java
public static ExpectedCondition<Boolean> elementHasStoppedMoving(final WebElement element) {
    new ExpectedCondition<Boolean>() {
        @Override
        Boolean apply(WebDriver driver) {
            Point initialLocation = ((Locatable) element).getCoordinates().inViewPort();
            Thread.sleep(50);
            Point finalLocation = ((Locatable) element).getCoordinates().inViewPort();
            initialLocation.equals(finalLocation);
        }
    }
}
```
