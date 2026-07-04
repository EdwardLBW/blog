---
title: WebDriver
date: 2022-08-29 10:45:59
tags: ["前端"]
---

Recently, I am developing an industrial software, which has high requirements for software quality. So, the developers must write unit test code that coverage rate must be 100%. At the same time, the automated testing is a must, but, we met some problems about automated testing. just do some records and summaies.

We found out that the shadowdom is not very friendly to the automated testing tool webdriver, so I have this summary

## What`s the ShadowDom

The ShadowDom is a standard that comes with the browser and is used for WebComponent. Maybe, You do not usually pay too much attention on it, but the input, vedio, etc. that we often used are all standard WebComponent, and they all have ShadowDom.We need to change the settings in your browser to see it.

Let's take the input element of the Bing homepage as an example. When shadowDom is not set to be visible, the input tag still looks like a normal tag when inspecting the element.

![First Pic](/images/6.awebp)

First Pic

step 1: Enable the Viewable ShadowDom option in the browser console:

![Second Pic](/images/7.awebp) step 2:Look at the same input input box just now, we can see a shadow root:

![third Pic](/images/8.awebp)

third Pic

In the html standard, there are many elements that are natively implemented webComponent with shadow-root, for example: input, video, etc.\
If you use the WebComponent standard to develop a web page, then all custom components will be wrapped in ShadowDom

## How do we do automatic test

For automated testing, our task is to complete a series of automated processes in a pipeline and issue a report. The team uses Selenium WebDriver, the development tool is VisualStudio2022, the development environment is .net6.0, and the development language is naturally C#.

## What`s the Selenium WebDriver

Here's an official explanation: WebDriver is a remote control interface that enables introspection and control of user agents. It provides a platform- and language-neutral wire protocol as a way for out-of-process programs to remotely instruct the behavior of web browsers.

Provided is a set of interfaces to discover and manipulate DOM elements in web documents and to control the behavior of a user agent. It is primarily intended to allow web authors to write tests that automate a user agent from a separate controlling process, but may also be used in such a way as to allow in-browser scripts to control a — possibly separate — browser.

As mentioned above, WebDriver can be understood as a tool library that can operate browsers. For example, behaviors similar to using jquery to manipulate dom can be realized under WebDriver, which is convenient for establishing black-box automated tests.

A short code example：

```
require("chromedriver");
const webDriver = require("selenium-webdriver");
const { By } = require("selenium-webdriver");
const {readJSCode} = require('./utils/files');
const init = async (baseUrl,JSCodeUrl) => {

    const driver = new webDriver.Builder().forBrowser('chrome').build();
    driver.get(baseUrl);
    driver.manage().window().maximize();
    await driver.sleep(5000);
    const JSCode = readJSCode(JSCodeUrl,1);
    const shadow = await driver.executeScript(JSCode);
    console.log(shadow);
    // console.log(shadownRoot);
    // driver.quit();
}
```

## What problem we met

During the coding process, the test engineer encountered a problem. The WebDriver apis is not very effective for the locating of elements with shadowdomdom, the support is not enough, and it is difficult to obtain elements. This is a problem we have to solve.

## How do we to solve it。

Automated testing needs to be done using C#, so there are the coding tools and technical frameworks I mentioned above.After our discussion and experimentation, we can directly execute javascript statements using WebDriver's function ExecuteScript. Using native JavaScript statements to complete element locating and other operations, and then assign the return value to the driver, then, we solved the problem we met.

![Forth Pic](/images/9.awebp)

Forth Pic

But here is another problem,too much javascript code string made the automation code hard to read, and this is a disaster for code reviews.

So, based on the above question,for me, a nodeJS enthusiast, used Nodejs and webdriver package to achieve a friendly web automation test solution that is **friendly to shandowroot**,**loads and executes javascript statements on demand**. But, I want to say that it is still a demo, if you want to use it in a production environment, I think you need to add some customized functions.And,the most important part is file read-write, it means that no matter what tech stack you use，you can make it. But, from my said, the nodejs support for javascript is the best.

![Fifth Pic](/images/10.awebp)

Fifth Pic

as you can see a pic,The file structure is very simple,The project entry is app.js, and the config.js is the global configuration file that you can centralize some global configuration information here.

All source code exists in the src folder.Among them, JSCode is the javascript code file to be executed in webdriver, and under utils is the file reading function collection and the tool function collection. The main code of webdriver is in index.js.

I have uploaded the demo project to **[github:FriendlyNodeDriver](https://github.com/lililbwl/FriendlyNodeDriver.git)**, you can get it yourself if you need it. And,excute ternimal command **'npm install' && 'npm start'** , it`s will running.

## Finally

All the above contents do not contain sensitive information and encrypted information, hope to help those in need.  
If you have some different ideas and opinions, welcome to exchange

