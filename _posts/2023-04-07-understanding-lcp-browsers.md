---
layout: post
title: "Understanding how browsers identify the LCP element"
date: 2023-04-07
categories: [Web Performance, Core Web Vitals]
---

Largest Contentful Paint (LCP) is a key performance metric that measures the loading speed of a webpage and is part of the Google Web Vitals initiative, which was introduced in May 2020.

Specifically, LCP measures the time it takes for the largest visible element on a page to become fully loaded and rendered in the viewport.

This element could be an image, video, or another content element that occupies a significant amount of screen real estate.

## How does Chrome determine the Largest Contentful Paint (LCP) Element?

Here are the general steps that Chrome takes to determine the LCP element:

1.  The browser begins to load the webpage and parses the HTML code to construct the Document Object Model (DOM) and the Render Object Model (ROM).
2.  As the page loads, the browser searches for all visible block-level or replaced elements (such as images or videos) **within the viewport**.
3.  For each eligible element, the browser calculates the size of the render box, which takes into account the element's width, height, padding, and border. This calculation doesn't include the margin or size of any child elements.
4.  The browser selects the element with the largest render box size as the candidate for Largest Contentful Paint (LCP). This element must also be loaded via an HTTP request and be visible in the viewport at the time of measurement.
5.  After identifying the LCP candidate, the browser measures the time it takes for the element to become fully visible in the viewport. This occurs when the element's content is rendered and its layout is stable.
6.  The time it takes for the largest contentful paint (LCP) element to become fully visible is recorded as the LCP value for the webpage.

It's important to note that the LCP value may [change](https://web.dev/lcp/#when-is-largest-contentful-paint-reported) as the page continues to load, and the final LCP value is typically the one recorded when the page finishes loading or when the user interacts with the page.

![](/images/lcp-1.png.webp)
Figure 1: LCP element changed after the page is fully loaded

## Factors that could affect LCP value

As per Google, the LCP metric can be primarily affected by these four factors,

1.  Slow server response times
2.  Render-blocking JavaScript and CSS
3.  Resource load times
4.  Client-side rendering

Slow server response times can cause delays in loading content, while render-blocking JavaScript and CSS can prevent the browser from rendering content until they're fully loaded.

Resource load times refer to the time taken for the browser to download and process images, videos, and other media, while client-side rendering involves processing and rendering content on the user's device.

### Mobile and Desktop may have different LCP elements

Mobile devices and desktops have different screen sizes and resolutions, which can affect how a webpage is rendered and which elements are considered the largest on the page.

For example, an image that takes up a significant portion of a desktop screen may not take up as much space on a mobile screen, where the LCP element might instead be a text block or a button.

Additionally, mobile devices often have slower network speeds and processing power compared to desktops, which can impact how quickly the LCP element is loaded and rendered on the screen.

![](/images/lcp-2.png.webp)
Figure 2: LCP element for [dailymail.co.uk](http://dailymail.co.uk/) is different on mobile and desktop

### Common LCP elements on a webpage

Google specifies what specific elements may be considered as LCP. Refer [https://web.dev/lcp/#what-elements-are-considered](https://web.dev/lcp/#what-elements-are-considered).

Below are some common elements in web pages that are eligible:

1.  Hero section image or video
2.  Background image
    Background images which are loaded via the [url()](https://developer.mozilla.org/docs/Web/CSS/url()) function
3.  Video thumbnail
4.  Image carousel
5.  Text
    Text can also be an LCP element, especially if it is the primary content on a webpage.

![](/images/lcp-3.png.webp)
Figure 3: LCP element can defer according to a specific website

### What elements may not be considered

It's understood that not all elements are considered candidates for LCP. For instance, elements that are not visible within the viewport, such as elements located at the bottom of the page or behind other elements, are not considered.

Additionally, elements that are dynamically generated, such as ads or pop-ups, may not be considered if they are not loaded via an HTTP request.

While background images can be an LCP element, solid background colours or patterns are typically not considered since they don't require an HTTP request to load.

Lazy-loaded content may not be applicable as LCP as well. Lazy-loading is a technique used to defer the loading of non-critical resources until they are needed, such as images or videos that are below the fold.

Since the lazy load resources are not loaded immediately, they may not be considered part of the LCP calculation.

SVG elements on a page although the largest visible element may not be considered for LCP as well.

![](/images/lcp-4.png.webp)
Figure 4: A larger SVG element may be ignored

### How to find the LCP element on my webpage

To replicate and test the LCP (Largest Contentful Paint) values of your webpage, you can run audits using an online tool like Google's PageSpeed Insights, GTMetrix and Webpagetest.

In a local environment, you can use Chrome's DevTools:

1.  Open the webpage of interest in your browser, such as **[cnn.com](https://www.cnn.com/)**.
2.  Open the Developer Tools in your browser. In Chrome, you can do this by clicking on the top right menu > More Tools > Developer Tools, or by pressing Ctrl+Shift+I.
3.  Navigate to the Performance tab in the Developer Tools and click the Reload button to load the webpage.
4.  After the page has loaded, locate the Largest Contentful Paint (LCP) option under the timings row and hover over it to view the LCP time.
5.  To identify the LCP element, look for the element highlighted in blue in the screenshot shown in the LCP element section of the Developer Tools.

![](/images/lcp-5.png.webp)
Figure 5: Identify the LCP element on Google Chrome Dev Tools

Alternatively, you can run a free webpage test on [https://www.webpagetest.org/](https://www.webpagetest.org/) where it has advanced tools to track LCP changes.

1.  Select Start a Site Performance test
2.  Enter the website url, such as [http://bbc.com/](http://bbc.com/)
3.  Select either mobile or desktop, and click on Start Test
4.  Once the test report is generated, select the Web Vitals view
5.  Go to the Largest Contentful Paint section and select the Filmstrip view
6.  Click on Adjust Filmstrip Settings > Filmstrip options > Highlight Largest Contentful Paints
7.  You can observe the LCP element as it loads on the filmstrip view

![](/images/lcp-6.png.webp)
Figure 6: Identify the LCP element by running a [webpagetest.org](http://webpagetest.org/) audit

UPDATE 10/Apr/2023 : To deep dive into LCP and recent trends from CrUX report refer [https://almanac.httparchive.org/en/2022/performance#largest-contentful-paint-lcp](https://almanac.httparchive.org/en/2022/performance#largest-contentful-paint-lcp)

Happy discovering!