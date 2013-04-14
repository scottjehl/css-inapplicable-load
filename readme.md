# Testing for Asynchronous Load of Inapplicable CSS StyleSheets

As we aim to address more diverse conditions and environments with CSS via Responsive Design, our ability to target styles to certain contexts while minimizing their impact on others becomes increasingly critical. CSS3 Media Queries are a powerful tool for specifying where certain CSS rules should and should not apply, but media queries have no affect on whether StyleSheets are actually requested, only whether they are rendered or not after they've been requested. This was tested and verified in a previous experiment, [CSS Download Tests](http://scottjehl.github.io/CSS-Download-Tests/), which showed that all of the following StyleSheets will be downloaded by all modern browsers that were tested, regardless of whether their associated media types or queries apply:

````
<link href="a.css" rel="stylesheet">
<link href="a.css" rel="stylesheet" media="(min-width: 45em)">
<link href="a.css" rel="stylesheet" media="(min-resolution: 100dpi)">
<link href="a.css" rel="stylesheet" media="nonsense query weeee!">
````

Unfortunately, not only do browsers always request StyleSheets regardless of their applicability, but many browsers delay rendering (displaying the page content to the user) until those requests have returned. This presents a significant problem, as many browsers will wait as long as 30 seconds or more for a file to load, and a variety of network or server conditions could cause that to occur.

That said, some modern browsers (particularly those based on _Webkit_ and now _Blink_) will actually evaluate media queries (when present) to determine the priority at which a StyleSheet should be loaded before requesting that StyleSheet. In these browsers, StyleSheets paired with media queries that are found to be inapplicable at load time will be loaded asynchronously (and possibly deferred until higher-priority requests have been made), meaning the page content will be displayed to the user immediately and regardless of whether those de-prioritized StyleSheets have finished loading or not. Webkit's behavior in this case was first documented in this post by Ilya Grigorik(http://www.igvita.com/2012/06/14/debunking-responsive-css-performance-myths/).

Given the increasing number of contexts we're addressing with CSS, this prioritized loading behavior is highly desirable, but it is not implemented in all modern browsers. Unfortunately, if we build our websites to favor this browser behavior today, for example by separating our styles into @media-specific files and loading them separately, we will introduce potential points of failure for users with browsers that don't behave in this way. This dilemma led me to create [a previous experiment that mimicked the behavior with JavaScript](https://github.com/scottjehl/eCSSential), which worked fairly well but failed to perform quite as well as a native implementation.

Ideally, this behavior would be native and reliable across browsers, but we're not there today. This   experiment tests which modern browsers implement this prioritized loading behavior, so we have a clearer picture of where it applies today.

## Test Case Explained

The experiment includes an HTML file containing a single `link` element with a `media` attribute containing a probably-inapplicable media query value `(min-width: 40000px)`. The `link` references a stylesheet that is designed to take 45 seconds to respond when it is requested. The HTML file also contains a single heading to test whether the page content is displayed while this request is pending.

```` index.html:
<!doctype html>
<html lang="en">
<head>
	<meta charset="utf-8">
	<title>Inapplicable CSS loading tests</title>
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<link href="test.css.php" rel="stylesheet" media="(min-width: 40000px)">
</head>
<body>
	<h1>Loading</h1>
</body>
</html>
````

```` test.css.php:
<?
	header('Content-Type: text/css');
	sleep(45);
?>
````

To test behavior, I loaded the page in a variety of web browsers and observed the loading behavior in a fairly non-scientific manner to see how each browser behaved, as it's easy to see whether the behavior is present by simply observing page load.


## The Results

### The Good

- Chrome 26: Renders page immediately (while request is pending)
- Safari 6.0.2: Renders page immediately (while request is pending)
- iOS Safari 6.1: Renders page immediately (while request is pending)
- Android 4.2.1 Webkit Browser: Renders page immediately (while request is pending)
- Android 4.2.1 Chrome 18: Renders page immediately (while request is pending)
- Android 4.2.1 Chrome 18: Renders page immediately (while request is pending)
- Android 4.2.1 Opera Mini: Renders page immediately (after approx 3 seconds, while request is pending)


### The Bad

- Internet Explorer 10: Blank page for full timeout period (approx. 30 observed seconds)
- Internet Explorer 9: Blank page for full timeout period (approx. 30 observed seconds)
- Internet Explorer 8: Blank page for full timeout period (approx. 30 observed seconds)
- Firefox 20: Blank page for full timeout period (approx. 30+ observed seconds)
- Opera 12.15: Blank page for full timeout period (approx. 6 observed seconds)
- Android 4.2.1 Opera Mobile: Blank page for full timeout period (approx. 10 observed seconds)


## Follow-up

Unfortunately, this behavior is not present in some of the most popular browsers in the world. Let's try and fix that.

Below are links to various browser vendors' bug trackers with feature requests for implementing this behavior.

- Internet Explorer: link pending...
- Firefox: link pending...
- Opera Desktop: link pending...
- Opera Mini: link pending...





