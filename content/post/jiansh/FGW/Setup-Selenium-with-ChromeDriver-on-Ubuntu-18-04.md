+++
title = "Setup-Selenium-with-ChromeDriver-on-Ubuntu-18-04"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "FGW"
]
toc = true
+++

[link to JianShu](https://www.jianshu.com/p/e6d4ef639ade)


[https://tecadmin.net/setup-selenium-chromedriver-on-ubuntu/](https://tecadmin.net/setup-selenium-chromedriver-on-ubuntu/)

https://stackoverflow.com/a/34343755/1087122

Selenium is only a library, and as such it does not particularly care if you are running it on a system that is equipped with a GUI. What you are probably asking is: If I use Selenium to open a browser, is that browser going to work on a system with no GUI. The answer to this is: it depends!

There are headless browsers: browsers that also do not have a GUI component. [HtmlUnit](http://htmlunit.sourceforge.net/) is packaged with Selenium. Another popular browser is [PhantomJS](http://phantomjs.org/), which has third-party Selenium bindings library called [GhostDriver](https://github.com/detro/ghostdriver). Personally I would **avoid both of these**! HtmlUnit uses a JavaScript engine that none of the current desktop browsers support, and as such the tests are not very reliable. GhostDriver has [not been maintained for 2 years](https://github.com/detro/ghostdriver#help-needed), and as such also makes for unreliable results. PahntomJS is definitely an option, as it uses WebKit - the engine that is in Safari and Chrome browsers, but you would have to write your own [API](http://phantomjs.org/api/).

Most systems will allow you to have a virtual GUI. You mentioned Ubuntu, which is a Debian derivative. There are several tutorials on the Net that tell you how to install Xvfb, most of which are incomplete or wrong. On a Debian you install a headless browser like this:

1.  Install Xvfb: `apt-get install xvfb`
2.  Install a browser. Assuming you are using a Debian server, you will not be able to install something like Firefox with apt-get, because the repositories are not present. Instead Google something like "Firefox off-line install", or whatever browser you want to use, and then use `wget` on your server to grab the package.
3.  Unpack the package somewhere like `/usr/local/lib`, and then create a soft link from `/usr/local/bin` to the binary that launches the browser.
4.  Now try launching your browser headless. For example, for Firefox you would try: `xvfb-run firefox`. This may produce some errors, which you have to fix. In my case, I was missing the library `libdbus-glib-1-2` which I could install just using apt-get.
5.  At this point you will need to launch Xvfb before running your Selenium tests. Most CI servers have a plugin for Xvfb, or you can do it from the command line: `Xvfb :99 &`. See the [docs](http://www.x.org/archive/X11R7.6/doc/man/man1/Xvfb.1.xhtml) for additional information.

[https://my.oschina.net/zjzhai/blog/295288](https://my.oschina.net/zjzhai/blog/295288)

```
sudo apt-get update
sudo apt-get install -y unzip xvfb libxi6 libgconf-2-4
```


### 独立在server端安装chrome
```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb
google-chrome --version
Google Chrome 80.0.3987.122
```


### chrome headless 模式
[https://tecadmin.net/google-chrome-headless-features/](https://tecadmin.net/google-chrome-headless-features/)


### 下载对应端chromedriver
[https://sites.google.com/a/chromium.org/chromedriver/downloads](https://sites.google.com/a/chromium.org/chromedriver/downloads)

```
mv chromedriver /usr/bin/chromedriver

chromedriver --version
ChromeDriver 80.0.3987.106 (f68069574609230cf9b635cd784cfb1bf81bb53a-refs/branch-heads/3987@{#882})
```

[https://tecadmin.net/google-chrome-headless-features/](https://tecadmin.net/google-chrome-headless-features/)

```
google-chrome --headless --disable-gpu --screenshot https://www.baidu.com/
[0226/113854.465613:ERROR:zygote_host_impl_linux.cc(89)] Running as root without --no-sandbox is not supported. See https://crbug.com/638180.
➜  ~ google-chrome --headless --disable-gpu --no-sandbox  --screenshot https://www.baidu.com/
[0226/113918.665763:INFO:headless_shell.cc(619)] Written to file screenshot.png.
```

![](https://upload-images.jianshu.io/upload_images/3296949-984643d69ffbfbb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


[https://tecadmin.net/setup-selenium-chromedriver-on-ubuntu/](https://tecadmin.net/setup-selenium-chromedriver-on-ubuntu/)

```
System.setProperty("webdriver.chrome.driver", "/usr/bin/chromedriver");
ChromeOptions chromeOptions = new ChromeOptions();
chromeOptions.addArguments("--headless");
chromeOptions.addArguments("--no-sandbox");

WebDriver driver = new ChromeDriver(chromeOptions);

driver.get("https://www.baidu.com");

Thread.sleep(1000);

if (driver.getPageSource().contains("hao")) {
		System.out.println("Pass");
} else {
		System.out.println("Fail");
}
driver.quit();
```
            

