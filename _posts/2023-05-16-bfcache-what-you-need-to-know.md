---
layout: post
title: "Back Forward cache: What you need to know"
date: 2023-05-16
categories: [Browser, Web Performance]
---

Imagine you're browsing an online store, looking at different products. You click on an item to view its details but then decide to go back and continue exploring. Thanks to bfcache, when you return to the previous page, it instantly appears with all the information and images loaded, without having to wait for it to load again from the server.

## What is bfcache?


Back/forward cache (bfcache) is a feature of web browsers that allows pages to be restored from memory after the user has navigated away from them and then back again. This can improve the performance of web pages by reducing the number of times that pages need to be loaded from the server.

This seamless experience saved you time, bandwidth and made your shopping journey smoother.

Let's delve into the details of this feature, which is supported by [modern browsers](https://caniuse.com/?search=bfcache), including mobile devices.

## How does bfcache work?

Bfcache operates by saving a comprehensive snapshot of a page, including the JavaScript heap, in memory as the user navigates away. With the complete page stored in memory, the browser can swiftly and effortlessly restore it when the user chooses to return.

This snapshot encompasses the HTML, CSS, JavaScript, and all other resources essential for rendering the page. Subsequently, if the user navigates away from and subsequently back to the page, the browser can restore it from memory instead of fetching it again from the server.

![](/images/bfcache-what-stored.jpg)

### PageTransitionEvent

The PageTransitionEvent plays a role in signaling page transitions, including when a page is being unloaded or when a new page is being loaded. When a user navigates away from a page, such as by clicking a link or using the browser's back or forward button, the PageTransitionEvent is triggered to notify the page that it is being transitioned out of view.

It encompasses various event types, including "pageshow", "pagehide", "beforeunload" and "unload" each corresponding to a specific stage in the page transition process.

This offers developers a way to observe and respond to page transitions within the browser, enabling them to enhance user experiences, implement custom behaviors, and optimize resource management.

**Note:** The unload event has been available in web browsers for a long time and is triggered when a page is being unloaded, either by navigating away or closing the browser window. On the other hand, the pagehide event was introduced later and provides an additional opportunity for developers to perform actions or cleanup tasks before a page is hidden or unloaded.

## How can I make my pages eligible for bfcache?

There are a few things that you can do to make your pages eligible for bfcache:

-   **Avoid using the unload event**
    The unload event is fired when the user is about to navigate away from a page. If you use the unload event to do anything that might affect the page's state, such as saving data or closing a modal dialog, then the page will not be eligible for bfcache.
-   **Avoid attaching event listeners to the beforeunload event**
    The beforeunload event is fired when the user is about to navigate away from a page and gives the page a chance to ask the user if they're sure they want to leave. If you attach an event listener to the beforeunload event and the user clicks "Cancel," then the page will not be eligible for bfcache.
-   **Do not make any network requests after the user has navigated away from your page**
    If you make any network requests after the user has navigated away from your page, then the browser will not be able to restore the page from memory. This is because the browser will not know what resources are needed to render the page.

**Note:** If there are any unload event listeners on your page, you should convert them to pagehide event listeners

## How can I check if my pages are eligible for bfcache?

You can check if your pages are eligible for bfcache using the Chrome DevTools. To do this, open the DevTools and go to the Applications tab. In the Cache section, look for the Back/forward cache entry. If your pages are eligible for bfcache, they will be listed in this section.

![](/images/bfcache-check-1.jpg)
![](/images/bfcache-check-2.jpg)

You can find more information about testing page optimization for instant loads and identifying issues that may affect eligibility for back-forward cache in the [Chrome DevTools documentation](https://developer.chrome.com/docs/devtools/application/back-forward-cache/).

## Limitations of bfcache

-   **Size**
    The size of the bfcache is limited by the amount of memory available on the device. When the bfcache is full, the browser will start to evict pages that have not been visited recently. This means that pages that are frequently visited or that contain a lot of resources may not be cached if the bfcache is full.
-   **Persistence**
    The bfcache is not persistent. When the browser is closed, the bfcache is cleared. This means that any pages that are stored in the bfcache will be lost when the browser is closed.
-   **Security**
    The bfcache is not secure. Any sensitive information that is stored in the bfcache can be accessed by anyone who has access to the device. This means that sensitive information, such as passwords or credit card numbers, should not be stored in the bfcache.

## What are the benefits?

Bfcache can improve the performance of web pages by reducing the number of times that pages need to be loaded from the server. This can lead to faster page load times, which can improve the user experience.

-   **Improved performance**
    bfcache can significantly improve the performance of web pages, especially for pages that are frequently visited or that contain a lot of resources. By storing a copy of the page in memory, the browser can avoid having to make a new request to the server, which can save time.
-   **Reduced bandwidth usage**
    bfcache can also help to reduce bandwidth usage by the browser. When a page is stored in bfcache, the browser does not have to download the page from the server again when the user navigates back to it. This can save bandwidth for both the user and the host.
-   **Better user experience**
    bfcache can provide a better user experience by making it faster and easier for users to navigate back to pages that they have recently visited. This can help to improve the overall usability of a website and make it more enjoyable for users to visit.

## Key observations

In her recent talk on [How to Prioritize Web Performance Optimizations](https://www.youtube.com/watch?v=bZV7XxsCNb8&t=647s) by Melissa Ada, she presented some interesting observations with regard to bfcache.
![](/images/bfcache-ada-1.jpg)
(Slide from [Melissa Ada](https://www.welovespeed.com/assets/docs/2023/melissa-ada-prioritizing-web-werformance-pptimizations.pdf))

The sharp increase in CLS scores for mobile devices in 2022 is believed to be closely related to the introduction of bfcache. The Web Almanac team suspects this as one of the primary factors contributing to the trend.

You can refer to the [Web Almanac report](https://almanac.httparchive.org/en/2022/performance#bfcache-eligibility) for 2022 to explore the detailed analysis and reasoning behind this correlation. It aligns with the [launch](https://web.dev/bfcache/#test-to-ensure-your-pages-are-cacheable) of bfcache on Chrome in late 2021.

**Resources:**

-   Bfcache eligibility criteria (100+) [https://docs.google.com/spreadsheets/d/1li0po\_ETJAIybpaSX5rW\_lUN62upQhY0tH4pR5UPt60/edit#gid=0](https://docs.google.com/spreadsheets/d/1li0po_ETJAIybpaSX5rW_lUN62upQhY0tH4pR5UPt60/edit#gid=0)