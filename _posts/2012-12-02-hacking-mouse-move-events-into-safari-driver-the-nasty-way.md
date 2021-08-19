---
layout: post
author: "Mark Collin"
title: "Hacking Mouse Move Events Into Safari Driver The Nasty Way"
category: Testing
tags: [Java, Robot, Safari, WebDriver]
---
I thought long and hard before posting this entry because while it works, it's not the real solution (Which is adding the code to enable the Actions classes into Safari driver, something I'm looking at right now to see if I can contribute something useful back to the Selenium community).  I follow the [Selenium](https://groups.google.com/forum/?hl=en-US&fromgroups&pli=1#!forum/selenium-users){:target="_blank"} [WebDriver](https://groups.google.com/forum/?hl=en-US&fromgroups&pli=1#!forum/webdriver){:target="_blank"} mailing lists quite closley and regularly see people overcomplicating and hacking things, usually due to the fact that they did not get instant gratification when trying to do things the right way because they didn't quite get it working the first time.  I really hope people will not use this example to "try and get things working" in other browsers because they don't understand how to use the Actions class, or they can't be bothered to learn how to do things the right way.

Let me be very clear, this will work but it's a hack and there are consequences:

- You will no longer be able to run multiple safari instances on your local machine because there is only one mouse cursor and we will be using it (bye bye threading to speed your tests up in safari)
- You will have messy code because you are going to have to put in code branches specifically for Safari mouse events.
- You can't run the tests in the background, Safari will need to be in the foreground and have focus while tests are running

If you can live with these issues and you really need to get Safari working with mouse events in the short term, this may just work for you.

Now that the warnings are out of the way let's have a look at the problem...

Lot of modern websites are using hover events to make customised tooltips (well they aren't tooltips in the HTML sense, but everybody still calls them tooltips) that appear when hovering your mouse over something like a chart point, or maybe there is some drag and drop functionality you need to test.  The problem with Safari is that the Actions class hasn't yet been implemented and the Safari driver object does not have an underlying mouse object.  Game over?  Not quite there is a way to get around the current limitations of the Safari Driver that will enable you to start clicking and hovering away like a mad man, the Java AWT Robot class (I'll just call it the robot for the rest of this post).

The Robot is cross platform compliant so this solution could potentially work on any OS, but since Safari 6 is only available on OSX that is all we are interested in at the moment.  The solution I have still uses Selenium for the majority of the heavy lifting, it is simply using the robot in place of the actions class to manipulate the mouse.

For all of these examples I have created a class called RobotPowered with the following private variables and constructor:

```Java
private final Robot mouseObject;
private final WebDriver driver;
private final JavascriptExecutor executor;

public RobotPowered(WebDriver driver) throws AWTException {
  this.mouseObject = new Robot();
  this.driver = driver;
  this.executor = (JavascriptExecutor) driver;
}
```

Now we know what is available to us let's start creating the code to move the mouse in safari.  First of all we have a basic robot implementation that will allow you to move the mouse to a specific X/Y coordinate on the screen

```Java
public void robotPoweredMoveMouseToAbsoluteCoordinates(int xCoordinates, int yCoordinates) {
  mouseObject.mouseMove(xCoordinates, yCoordinates);
  mouseObject.waitForIdle();
}
```

It really doesn't get much easier than this, the code is self-explanatory and it will just work.  We do have a problem however; we don't know where the browser was loaded on the screen so if we tried to click on some coordinates we would effectively be clicking blind.  On to part two:

```Java
public void robotPoweredMoveMouseToCoordinatesOnPage(int xCoordinates, int yCoordinates) {
  //Get Browser dimensions
  int browserWidth = driver.manage().window().getSize().width;
  int browserHeight = driver.manage().window().getSize().height;

  //Get dimensions of the window displaying the web page
  int pageWidth = Integer.parseInt(executor.executeScript("return document.documentElement.clientWidth").toString());
  int pageHeight = Integer.parseInt(executor.executeScript("return document.documentElement.clientHeight").toString());

  //Calculate the space the browser is using for toolbars
  int browserFurnitureOffsetX = browserWidth - pageWidth;
  int browserFurnitureOffsetY = browserHeight - pageHeight;

  //Calculate the correct X/Y coordinates based upon the browser furniture offset and the position of the browser on the desktop
  int xPosition = driver.manage().window().getPosition().x + browserFurnitureOffsetX + xCoordinates;
  int yPosition = driver.manage().window().getPosition().y + browserFurnitureOffsetY + yCoordinates;

  //Move the mouse to the calculated X/Y coordinates
  mouseObject.mouseMove(xPosition, yPosition);
  mouseObject.waitForIdle();
}
```

This now calculates where the browser is on the screen, the size of the browser and how much space is used up by browser toolbars.  There is one caveat to the above code; you will need to disable the status bar.  I haven't found an easy way to work out how much space is used by the status bar, and how much is used by the rest of the browser so the simple solution is to just remove it.

This code is getting better, we now know that when we pass X/Y coordinates into the function it will click on the page, but that still leaves us guessing where on the page a specific WebElement is, well it's not a problem Selenium actually knows the coordinates of the elements on the page and it can tell you where they are:

```Java
public void robotPoweredMoveMouseToWebElementCoordinates(WebElement element) {
  //Get Browser dimensions
  int browserWidth = driver.manage().window().getSize().width;
  int browserHeight = driver.manage().window().getSize().height;

  //Get dimensions of the window displaying the web page
  int pageWidth = Integer.parseInt(executor.executeScript("return document.documentElement.clientWidth").toString());
  int pageHeight = Integer.parseInt(executor.executeScript("return document.documentElement.clientHeight").toString());

  //Calculate the space the browser is using for toolbars
  int browserFurnitureOffsetX = browserWidth - pageWidth;
  int browserFurnitureOffsetY = browserHeight - pageHeight;

  //Get the coordinates of the WebElement on the page and calculate the centre point
  int webElementX = ((Locatable) element).getCoordinates().getLocationOnScreen().x + Math.round(element.getSize().width / 2);
  int webElementY = ((Locatable) element).getCoordinates().getLocationOnScreen().y + Math.round(element.getSize().height / 2);

  //Calculate the correct X/Y coordinates based upon the browser furniture offset and the position of the browser on the desktop
  int xPosition = driver.manage().window().getPosition().x + browserFurnitureOffsetX + webElementX;
  int yPosition = driver.manage().window().getPosition().y + browserFurnitureOffsetY + webElementY;

  //Move the mouse to the calculated X/Y coordinates
  mouseObject.mouseMove(xPosition, yPosition);
  mouseObject.waitForIdle();
}
```

We are now taking in a WebElement and getting the coordinates of its top left point.  We then use the size and height of the element to work out its centre point which is where we move the mouse.

What about drag and drop?  Well that's easy we can use the robot to hold and release the mouse button as well.

```Java
public void robotPoweredMouseDown() {
  mouseObject.mousePress(InputEvent.BUTTON1_DOWN_MASK);
  mouseObject.waitForIdle();
}
```

```Java
public void robotPoweredMouseUp() {
  mouseObject.mouseRelease(InputEvent.BUTTON1_DOWN_MASK);
  mouseObject.waitForIdle();
}
```

You now have everything you need to perform mouse actions with the Safari driver, unfortunately there is another caveat.  I have found that when safari initially loads it doesn't always have focus and when it doesn't have focus it ignores the mouse events, the solution is to make the robot click on the window to set focus at the start of your test, here's a quick click function to let you do just that:

```Java
public void robotPoweredClick() {
  mouseObject.mousePress(InputEvent.BUTTON1_DOWN_MASK);
  mouseObject.mouseRelease(InputEvent.BUTTON1_DOWN_MASK);
  mouseObject.waitForIdle();
}
```

Finally here's a little trick to find out what browser the current driver object is driving, you can use this to add in some specific code branches for safari:

```Java
((RemoteWebDriver) driver).getCapabilities().getBrowserName();
```

Once again I must reiterate that the above code is a hack and has its limitations, it does however get you out of a tight spot if you need to run your hover/drag and drop automated tests against Safari.

All of the above code is available on github at [https://github.com/Ardesco/Powder-Monkey/blob/master/src/main/java/com/lazerycode/selenium/tools/RobotPowered.java](https://github.com/Ardesco/Powder-Monkey/blob/master/src/main/java/com/lazerycode/selenium/tools/RobotPowered.java){:target="_blank"}
