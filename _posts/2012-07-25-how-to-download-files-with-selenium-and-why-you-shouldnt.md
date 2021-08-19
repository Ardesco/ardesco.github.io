---
layout: post
author: "Mark Collin"
title: "How To Download Files With Selenium And Why You Shouldn’t"
redirect_from: "/index.php/2012/07/how-to-download-files-with-selenium-and-why-you-shouldnt/"
category: Testing 
tags: [Java, Maven, Selenium, WebDriver]
---
In this blog post I will try and make you think why you are performing automated file download tests, and I will provide some Java code that will enable you to perform file downloads in a cross platform way without resorting to hacks like AutoIT.

## First things first, don’t do file download tests!

Let’s start off with a scenario. You and the BA have talked to the product owner and they have said that they want to give the users some cool functionality that enables them to download some PDF's with useful information in them. Everybody agrees that this is easy to implement and that you can very quickly run an exploratory test to check that it works by clicking on the download link in your browser. It downloads the file, you open it in PDF reader, all looks good and everybody is happy.

Now comes the tricky bit, you are asked to automate this scenario. After all, we want the build to go red on the CI server if some changes the developers make break your shiny new PDF download functionality. So, you load up Selenium and start replicating the actions you would take if you were playing the scenario out manually:

* You load the page with the download link.
* You find the `<a>` element on the page.
* You click on it...

You have just fallen into a trap, the trap being that Selenium can't deal with OS level dialogues so as soon as you click on the download link your test stops, you do not pass go and you don’t collect £200.

You go and have a look at the Selenium mailing lists and see lots of posts about AutoIT or maybe a post about a Java robot class and start looking at implementing one of these to interact with your OS level dialogue box…

STOP RIGHT THERE!

Now is the time to take a step backward and work out exactly what you want to test.

## Do you really need to download that file?

I’m guessing your initial reaction is “Yes, I do. I need to make sure that the download functionality continues to work”. Sounds pretty reasonable so far; let’s go a further down this rabbit hole:
	
* How many files are you planning to download?
* How big are these files?
* Do you have disk space to hold all of these files?
* Do you have network capacity to continually download these files?
* *What are you planning to do with the downloaded file?*

The last questions is where people usually stop and realise that they aren’t actually planning to do anything with the downloaded file. They are just planning to download the file and as long as a file has been downloaded they are happy that the test has passed. Now ask yourself, do you really need to download a file to perform this test. All you are actually doing is checking that when you click on a link you are getting a valid response from the server. You aren’t checking that you can download the file, you are checking for broken links. This is a worthwhile test, but it doesn’t require you to actually download anything. So let’s put AutoIT back in its little box and give you some code that can check to see if the link is valid.

## Checking that links are valid

It’s actually pretty simple, all you need to do is find the link on the page, extract a URL from its href attribute and then check to see if sending an HTTP GET request to that URL results in a valid response. To do this I have a URLStatusChecker class:

```java
package com.lazerycode.selenium.urlstatuschecker;

import org.apache.http.client.methods.*;

public enum RequestMethod {
    OPTIONS(new HttpOptions()),
    GET(new HttpGet()),
    HEAD(new HttpHead()),
    POST(new HttpPost()),
    PUT(new HttpPut()),
    DELETE(new HttpDelete()),
    TRACE(new HttpTrace());

    private final HttpRequestBase requestMethod;

    RequestMethod(HttpRequestBase requestMethod) {
        this.requestMethod = requestMethod;
    }

    public HttpRequestBase getRequestMethod() {
        return this.requestMethod;
    }
}
```

```java
package com.lazerycode.selenium.urlstatuschecker;

import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpRequestBase;
import org.apache.http.client.params.ClientPNames;
import org.apache.http.client.protocol.ClientContext;
import org.apache.http.impl.client.BasicCookieStore;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.impl.cookie.BasicClientCookie;
import org.apache.http.params.HttpParams;
import org.apache.http.protocol.BasicHttpContext;
import org.apache.log4j.Logger;
import org.openqa.selenium.Cookie;
import org.openqa.selenium.WebDriver;

import java.io.IOException;
import java.net.MalformedURLException;
import java.net.URI;
import java.net.URISyntaxException;
import java.net.URL;
import java.util.Set;

public class URLStatusChecker {

    private static final Logger LOG = Logger.getLogger(URLStatusChecker.class);
    private URI linkToCheck;
    private WebDriver driver;
    private boolean mimicWebDriverCookieState = true;
    private boolean followRedirects = false;
    private RequestMethod httpRequestMethod = RequestMethod.GET;

    public URLStatusChecker(WebDriver driverObject) throws MalformedURLException, URISyntaxException {
        this.driver = driverObject;
    }

    /**
     * Specify a URL that you want to perform an HTTP Status Check upon
     *
     * @param linkToCheck
     * @throws MalformedURLException
     * @throws URISyntaxException
     */
    public void setURIToCheck(String linkToCheck) throws MalformedURLException, URISyntaxException {
        this.linkToCheck = new URI(linkToCheck);
    }

    /**
     * Specify a URL that you want to perform an HTTP Status Check upon
     *
     * @param linkToCheck
     * @throws MalformedURLException
     */
    public void setURIToCheck(URI linkToCheck) throws MalformedURLException {
        this.linkToCheck = linkToCheck;
    }

    /**
     * Specify a URL that you want to perform an HTTP Status Check upon
     *
     * @param linkToCheck
     */
    public void setURIToCheck(URL linkToCheck) throws URISyntaxException {
        this.linkToCheck = linkToCheck.toURI();
    }

    /**
     * Set the HTTP Request Method (Defaults to 'GET')
     *
     * @param requestMethod
     */
    public void setHTTPRequestMethod(RequestMethod requestMethod) {
        this.httpRequestMethod = requestMethod;
    }

    /**
     * Should redirects be followed before returning status code?
     * If set to true a 302 will not be returned, instead you will get the status code after the redirect has been followed
     * DEFAULT: false
     *
     * @param value
     */
    public void followRedirects(Boolean value) {
        this.followRedirects = value;
    }

    /**
     * Perform an HTTP Status check and return the response code
     *
     * @return
     * @throws IOException
     */
    public int getHTTPStatusCode() throws IOException {

        HttpClient client = new DefaultHttpClient();
        BasicHttpContext localContext = new BasicHttpContext();

        LOG.info("Mimic WebDriver cookie state: " + this.mimicWebDriverCookieState);
        if (this.mimicWebDriverCookieState) {
            localContext.setAttribute(ClientContext.COOKIE_STORE, mimicCookieState(this.driver.manage().getCookies()));
        }
        HttpRequestBase requestMethod = this.httpRequestMethod.getRequestMethod();
        requestMethod.setURI(this.linkToCheck);
        HttpParams httpRequestParameters = requestMethod.getParams();
        httpRequestParameters.setParameter(ClientPNames.HANDLE_REDIRECTS, this.followRedirects);
        requestMethod.setParams(httpRequestParameters);

        LOG.info("Sending " + requestMethod.getMethod() + " request for: " + requestMethod.getURI());
        HttpResponse response = client.execute(requestMethod, localContext);
        LOG.info("HTTP " + requestMethod.getMethod() + " request status: " + response.getStatusLine().getStatusCode());

        return response.getStatusLine().getStatusCode();
    }

    /**
     * Mimic the cookie state of WebDriver (Defaults to true)
     * This will enable you to access files that are only available when logged in.
     * If set to false the connection will be made as an anonymouse user
     *
     * @param value
     */
    public void mimicWebDriverCookieState(boolean value) {
        this.mimicWebDriverCookieState = value;
    }

    /**
     * Load in all the cookies WebDriver currently knows about so that we can mimic the browser cookie state
     *
     * @param seleniumCookieSet
     * @return
     */
    private BasicCookieStore mimicCookieState(Set seleniumCookieSet) {
        BasicCookieStore mimicWebDriverCookieStore = new BasicCookieStore();
        for (Cookie seleniumCookie : seleniumCookieSet) {
            BasicClientCookie duplicateCookie = new BasicClientCookie(seleniumCookie.getName(), seleniumCookie.getValue());
            duplicateCookie.setDomain(seleniumCookie.getDomain());
            duplicateCookie.setSecure(seleniumCookie.isSecure());
            duplicateCookie.setExpiryDate(seleniumCookie.getExpiry());
            duplicateCookie.setPath(seleniumCookie.getPath());
            mimicWebDriverCookieStore.addCookie(duplicateCookie);
        }

        return mimicWebDriverCookieStore;
    }
}
```

This will take a URL supplied to it and then return an HTTP status code. If it’s there I would expect a 200 (OK) or maybe even a 302 (Redirect). If it’s not there, I would expect a 404 (Not found) or if things really went badly a 500 (Server Error). It’s up to you to define which HTTP status code is a pass or a fail, the above code will simply tell you what the HTTP status code is. The above code is a little more complex than just performing a HTTP GET, it also mirrors your WebDriver session so that you can access the same resources as the user you are currently logged in as.
To use it you would simply do the following:

```java
@Test
public void statusCode404FromString() throws Exception {
    urlChecker.setURIToCheck(webServerURL + ":" + webServerPort + "/doesNotExist.html");
    urlChecker.setHTTPRequestMethod(RequestMethod.GET);
    assertThat(urlChecker.getHTTPStatusCode(), is(equalTo(404)));
}
```

That’s nice but I really do want to download the file
I know that there are some people who really do want to download the actual file and perform checks on it, so how should we do it?

***Everybody raves about AutoIT, that’s a good solution right?***

Well, no actually it’s not. AutoIT will only work on Windows so you can kiss goodbye to your cross platform testing. AutoIT will also be looking for a specific window name so you are going to need to have an AutoIT script for every different download dialogue that you trigger and if you are not calling it programmatically, but leaving it running in the background, it is going to automatically click on every download box that appears, not just the ones you want to interact with during your testing. Oh, did I also mention that you are going to have problems renaming the file you download?

***OK that doesn’t sound so good, how about a Java robot class? Lots of people talk about them as well***

That’s better; it can be cross platform compliant and you can rename files that you download, but it still has issues. With a Java robot class you will be either blindly clicking at a specific location, or trying to send keystrokes to the pop up dialogue in the hope that it is in the state you expect it to be in. This again means that you are probably going to have to have different robot classes for different operating systems and maybe for different browsers. It’s a lot of work and not guaranteed to be successful.

***So what do I do then? Forget about it? Use Sikuli?***

There is another option, you can use the information provided by Selenium to programmatically download the file and completely bypass the OS level dialogue:

```java
package com.lazerycode.selenium.filedownloader;

import org.apache.commons.io.FileUtils;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.params.ClientPNames;
import org.apache.http.client.protocol.ClientContext;
import org.apache.http.impl.client.BasicCookieStore;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.impl.cookie.BasicClientCookie;
import org.apache.http.params.HttpParams;
import org.apache.http.protocol.BasicHttpContext;
import org.apache.log4j.Logger;
import org.openqa.selenium.Cookie;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;

import java.io.File;
import java.io.IOException;
import java.net.URISyntaxException;
import java.net.URL;
import java.util.Set;

public class FileDownloader {

    private static final Logger LOG = Logger.getLogger(FileDownloader.class);
    private WebDriver driver;
    private String localDownloadPath = System.getProperty("java.io.tmpdir");
    private boolean followRedirects = true;
    private boolean mimicWebDriverCookieState = true;
    private int httpStatusOfLastDownloadAttempt = 0;

    public FileDownloader(WebDriver driverObject) {
        this.driver = driverObject;
    }

    /**
     * Specify if the FileDownloader class should follow redirects when trying to download a file
     *
     * @param value
     */
    public void followRedirectsWhenDownloading(boolean value) {
        this.followRedirects = value;
    }

    /**
     * Get the current location that files will be downloaded to.
     *
     * @return The filepath that the file will be downloaded to.
     */
    public String localDownloadPath() {
        return this.localDownloadPath;
    }

    /**
     * Set the path that files will be downloaded to.
     *
     * @param filePath The filepath that the file will be downloaded to.
     */
    public void localDownloadPath(String filePath) {
        this.localDownloadPath = filePath;
    }

    /**
     * Download the file specified in the href attribute of a WebElement
     *
     * @param element
     * @return
     * @throws Exception
     */
    public String downloadFile(WebElement element) throws Exception {
        return downloader(element, "href");
    }

    /**
     * Download the image specified in the src attribute of a WebElement
     *
     * @param element
     * @return
     * @throws Exception
     */
    public String downloadImage(WebElement element) throws Exception {
        return downloader(element, "src");
    }

    /**
     * Gets the HTTP status code of the last download file attempt
     *
     * @return
     */
    public int getHTTPStatusOfLastDownloadAttempt() {
        return this.httpStatusOfLastDownloadAttempt;
    }

    /**
     * Mimic the cookie state of WebDriver (Defaults to true)
     * This will enable you to access files that are only available when logged in.
     * If set to false the connection will be made as an anonymouse user
     *
     * @param value
     */
    public void mimicWebDriverCookieState(boolean value) {
        this.mimicWebDriverCookieState = value;
    }

    /**
     * Load in all the cookies WebDriver currently knows about so that we can mimic the browser cookie state
     *
     * @param seleniumCookieSet
     * @return
     */
    private BasicCookieStore mimicCookieState(Set seleniumCookieSet) {
        BasicCookieStore mimicWebDriverCookieStore = new BasicCookieStore();
        for (Cookie seleniumCookie : seleniumCookieSet) {
            BasicClientCookie duplicateCookie = new BasicClientCookie(seleniumCookie.getName(), seleniumCookie.getValue());
            duplicateCookie.setDomain(seleniumCookie.getDomain());
            duplicateCookie.setSecure(seleniumCookie.isSecure());
            duplicateCookie.setExpiryDate(seleniumCookie.getExpiry());
            duplicateCookie.setPath(seleniumCookie.getPath());
            mimicWebDriverCookieStore.addCookie(duplicateCookie);
        }

        return mimicWebDriverCookieStore;
    }

    /**
     * Perform the file/image download.
     *
     * @param element
     * @param attribute
     * @return
     * @throws IOException
     * @throws NullPointerException
     */
    private String downloader(WebElement element, String attribute) throws IOException, NullPointerException, URISyntaxException {
        String fileToDownloadLocation = element.getAttribute(attribute);
        if (fileToDownloadLocation.trim().equals("")) throw new NullPointerException("The element you have specified does not link to anything!");

        URL fileToDownload = new URL(fileToDownloadLocation);
        File downloadedFile = new File(this.localDownloadPath + fileToDownload.getFile().replaceFirst("/|\\\\", ""));
        if (downloadedFile.canWrite() == false) downloadedFile.setWritable(true);

        HttpClient client = new DefaultHttpClient();
        BasicHttpContext localContext = new BasicHttpContext();

        LOG.info("Mimic WebDriver cookie state: " + this.mimicWebDriverCookieState);
        if (this.mimicWebDriverCookieState) {
            localContext.setAttribute(ClientContext.COOKIE_STORE, mimicCookieState(this.driver.manage().getCookies()));
        }

        HttpGet httpget = new HttpGet(fileToDownload.toURI());
        HttpParams httpRequestParameters = httpget.getParams();
        httpRequestParameters.setParameter(ClientPNames.HANDLE_REDIRECTS, this.followRedirects);
        httpget.setParams(httpRequestParameters);

        LOG.info("Sending GET request for: " + httpget.getURI());
        HttpResponse response = client.execute(httpget, localContext);
        this.httpStatusOfLastDownloadAttempt = response.getStatusLine().getStatusCode();
        LOG.info("HTTP GET request status: " + this.httpStatusOfLastDownloadAttempt);
        LOG.info("Downloading file: " + downloadedFile.getName());
        FileUtils.copyInputStreamToFile(response.getEntity().getContent(), downloadedFile);
        response.getEntity().getContent().close();

        String downloadedFileAbsolutePath = downloadedFile.getAbsolutePath();
        LOG.info("File downloaded to '" + downloadedFileAbsolutePath + "'");

        return downloadedFileAbsolutePath;
    }

}
```

The above code will mimic your current WebDriver session and programmatically download your file to the system temp directory where you can perform further checks upon it (It tells you where it downloaded it to). It’s relatively simple to use, a couple of basic examples are shown below:

```java
@Test
public void downloadAFile() throws Exception {
    FileDownloader downloadTestFile = new FileDownloader(driver);
    driver.get("http://www.localhost.com/downloadTest.html");
    WebElement downloadLink = driver.findElement(By.id("fileToDownload"));
    String downloadedFileAbsoluteLocation = downloadTestFile.downloadFile(downloadLink);

    assertThat(new File(downloadedFileAbsoluteLocation).exists(), is(equalTo(true)));
    assertThat(downloadTestFile.getHTTPStatusOfLastDownloadAttempt(), is(equalTo(200)));
}

@Test
public void downloadAnImage() throws Exception {
    FileDownloader downloadTestFile = new FileDownloader(driver);
    driver.get("http://www.localhost.com//downloadTest.html");
    WebElement image = driver.findElement(By.id("ebselenImage"));
    String downloadedImageAbsoluteLocation = downloadTestFile.downloadImage(image);

    assertThat(new File(downloadedImageAbsoluteLocation).exists(), is(equalTo(true)));
    assertThat(downloadTestFile.getHTTPStatusOfLastDownloadAttempt(), is(equalTo(200)));
}
```

***But that’s not the same as clicking on a link and downloading the file…***

Well, actually it is. When you click on the link your browser sends a HTTP GET request over to the webserver and then downloads the file to a temporary location, then it hands the file over to the operating system which then pops up a dialogue asking you where you really want to save it. All you are doing is taking the browser and the operating system out of the equation. Let’s face it, if the browsers download mechanism doesn’t work, there isn’t anything much you can do about it anyway (apart from raise a bug with the browser vendor).

## I’ve tried it and it works, but how do I know I have the right file?

The most simple and obvious way to check that the file is correct is to compare it to a known good copy of the file. If the file we have downloaded matches the original file it must be the correct file. No doubt you are now thinking “But hang on a moment, that means I need to keep a copy of every file that I download and some of them are massive…”. Not quite, there is another option you can just store a hash of the known good file. Taking an unsalted MD5/SHA1 hash of a file will always produce the same hash for the same file. So all you need to do is take a hash of the file you have downloaded and compare it to a known good hash of the file. If the hash doesn’t match you can fail the test and then examine the file manually later to find out what went wrong.
The final bit of code I have to offer is a class that will perform a hash check for you:

```java
package com.lazerycode.selenium.filedownloader;

public enum HashType {
    MD5,
    SHA1;
}
```

```java
package com.lazerycode.selenium.filedownloader;

import org.apache.commons.codec.digest.DigestUtils;
import org.apache.log4j.Logger;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;

public class CheckFileHash {

    private static final Logger LOG = Logger.getLogger(CheckFileHash.class);
    private HashType typeOfHash = null;
    private String expectedFileHash = null;
    private File fileToCheck = null;

    /**
     * The File to perform a Hash check upon
     *
     * @param fileToCheck
     * @throws FileNotFoundException
     */
    public void fileToCheck(File fileToCheck) throws FileNotFoundException {
        if (!fileToCheck.exists()) throw new FileNotFoundException(fileToCheck + " does not exist!");

        this.fileToCheck = fileToCheck;
    }

    /**
     * Hash details used to perform the Hash check
     *
     * @param hash
     * @param hashType
     */
    public void hashDetails(String hash, HashType hashType) {
        this.expectedFileHash = hash;
        this.typeOfHash = hashType;
    }

    /**
     * Performs a expectedFileHash check on a File.
     *
     * @return
     * @throws IOException
     */
    public boolean hasAValidHash() throws IOException {
        if (this.fileToCheck == null) throw new FileNotFoundException("File to check has not been set!");
        if (this.expectedFileHash == null || this.typeOfHash == null) throw new NullPointerException("Hash details have not been set!");

        String actualFileHash = "";
        boolean isHashValid = false;

        switch (this.typeOfHash) {
            case MD5:
                actualFileHash = DigestUtils.md5Hex(new FileInputStream(this.fileToCheck));
                if (this.expectedFileHash.equals(actualFileHash)) isHashValid = true;
                break;
            case SHA1:
                actualFileHash = DigestUtils.shaHex(new FileInputStream(this.fileToCheck));
                if (this.expectedFileHash.equals(actualFileHash)) isHashValid = true;
                break;
        }

        LOG.info("Filename = '" + this.fileToCheck.getName() + "'");
        LOG.info("Expected Hash = '" + this.expectedFileHash + "'");
        LOG.info("Actual Hash = '" + actualFileHash + "'");

        return isHashValid;
    }

}
```

You can then use it in a test like the one below:

```java
private final URL testFile = this.getClass().getResource("/download.zip");

@Test
public void checkValidMD5Hash() throws Exception {
    CheckFileHash fileToCheck = new CheckFileHash();
    fileToCheck.fileToCheck(new File(testFile.toURI()));
    fileToCheck.hashDetails("def3a66650822363f9e0ae6b9fbdbd6f", MD5);
    assertThat(fileToCheck.hasAValidHash(), is(equalTo(true)));
}
```

Hopefully I have managed to make you think twice about downloading files in your automated tests and provided a good cross platform/cross browser solution that will remove the need to add in yet another application to your test framework. The code above is a snapshot in time and will continue to be tweaked and updated as I’m made aware of problems, or think of better ways to do things.

One thing you may have noticed is that the code performing a status check and a file download is very similar.  I'm aware of this but wanted to keep it separate to reinforce the idea that you do not need to do file downloads, I will be merging both parts together in the near future.

If you want to have a look at the latest revision it’s available on Github as part of [https://github.com/Ardesco/Powder-Monkey](https://github.com/Ardesco/Powder-Monkey){:target="_blank"}.

All feedback appreciated :)
