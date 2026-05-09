---
layout: post
title: "What's new in Safari 17"
date: 2023-06-07
categories: [Browser, Web Performance]
---

Apple has unveiled Safari 17, its latest browser release that delivers a significant upgrade to browsing technology across its desktop and mobile platforms.

## WWDC23 announcements

 The announcement was done at Apple's [Worldwide Developers Conference (WWDC23)](https://developer.apple.com/videos/wwdc2023/), where a host of exciting Apple product updates were showcased.

While the event attracted significant attention due to the next-generation hardware announcements by Apple, it is equally important to underscore the notable advancements announced for Safari.

In the latest WebKit open-source blog post, Apple revealed that Safari 17 will introduce an impressive [88 new web features](https://webkit.org/blog/14205/news-from-wwdc23-webkit-features-in-safari-17-beta/). Key among those features are integration of Web Apps on Mac (🎉), Spatial Web elements for immersive spatial computing, improved Media streaming capabilities and exciting updates for render elements.

Today we will delve into the key features that are specifically designed to elevate and optimize web performance.

Its important to note that currently the features are available only via [Safari Technology Preview](https://developer.apple.com/safari/technology-preview/), an experimental version of Apple's Safari browser for developers, while the official version upgrade is anticipated to be released for all users in the upcoming fall.

### JPEG XL

[JPEG XL](https://jpeg.org/jpegxl/) is a groundbreaking modern image format. Safari 17 will be the first browser to support JPEG XL officially.

JPEG XL offers the capability to recompress images without compromising data, leading to a significant reduction in file size of up to 60%. This advancement promises improved image quality and reduced transfer size, benefiting web performance and user experience.

What sets JPEG XL apart is its progressive loading capability, enabling users to start viewing images before the entire file is downloaded.

![](/images/safari17-jxl.png)

While it competes with AVIF and WebP, this new image format aims to become the unifying solution to raster images on the web, as similar to what SVG is to vectors.

### Use case

-   **Compression** Superior image quality and compression ratios compared to legacy JPEG. The compression ratios achieved by JPEG XL are superior to those of both AVIF and WebP, as the file size can be reduced up to 1/50th of the original size.
-   **Smooth transition** JPEG is the most popular web image format. Transcoding JPEG images to JPEG XL is seamless and lossless, ensuring compatibility with existing JPEG-based applications.
-   **Optimization** The codec can be optimized for specific criteria. For example retaining high fidelity (i.e. Compatiblity with Professional photography needs), or for encoding speed (i.e. Faster image generation) or for compression ratio (i.e. Smaller files).

Its a point to note that each format brings its own set of features and optimizations to the table, aiming to strike the perfect balance between image quality and file size.

While there is wide adoption for [AVIF](https://caniuse.com/?search=AVIF) and [WebP](https://caniuse.com/?search=webp), [JPEG XL](https://caniuse.com/?search=JPEG-XL) was trailing. Competing formats are bad for web standards, but it would be interesting to see the growth and development of this image format with the support of Apple.

## Preconnect via HTTP Early Hints

Safari 17 brings a welcome performance feature by introducing support for [preconnect](https://webkit.org/blog/14205/news-from-wwdc23-webkit-features-in-safari-17-beta/#networking) via HTTP Early Hints. While this functionality was previously limited to Chromium-based browsers, it is now being experimented with in Safari

While Safari has previously supported other Early hints such as DNS prefetch (desktop), prefetch (experimental), and preload (desktop and mobile), the introduction of preconnect in Safari 17 further enhances the browser's performance capabilities.

```
<link rel="preconnect" href="https://fonts.googleapis.com">
```

With preconnect, browsers have the ability to establish connections with external resources, like servers and APIs, ahead of time. This is particularly useful when a webpage includes references to external libraries or resources, such as Google Fonts.

![](/images/safari17-google.png)

Image 1: An example of preconnect tag to Google fonts servers

When a browser requests a resource, it does a DNS lookup, establishes a TCP connection and employs TLS encryption to ensure secure transmission. With preconnect, the browser can establish this early lookup and be prepared to accept the response while it idles until it receives a server response.

Testing by Cloudflare has demonstrated a noteworthy 30% enhancement in page load time for initial website visits using Early hints. This underscores the significance of widespread browser adoption for better web performance.

## Fetch priority

Fetch Priority is a markup-based signal that enables developers to prioritize resources on their websites.This was only available on Chromium and now comes to Safari as well.

As opposed to server/network Early hints, this is primarily a browser API based signal.

Each browser has its own prioritization system for different file types. By default for example, Chrome assigns high priority to CSS files as they are considered render blocking resources, while non-blocking JavaScript and images are set to low priority.

It can be primarily used to boost the priority of the Largest Contentful Paint (LCP) image by specifying fetchpriority="high" on the image element. This causes the LCP to happen sooner, resulting in a faster website load time.

LCP being a core web vital, this is one of the popular ways web developers utilize this feature.

```
<!-- Fetch the LCP image on high priority -->
<img src="lcp.jpg" fetchpriority="high">

<!-- Lower the priority of a below the fold image -->
<img src="-image.jpg" fetchpriority="low">
```

For Safari it is unclear how fetchpriority will be implemented at this point, given the browser resource prioritization. But there is a detailed guide on how [Google Chrome](https://web.dev/fetch-priority/) handles this feature.

## Why Safari matters

Safari's last major upgrade was version 16, which was released for macOS Monterey and macOS Big Sur on September 2022. It was also shipped on iOS 16 which covered iPhone and iPad devices.

When considering the usage of web browsers, Safari holds a notable position, although it falls well behind Google Chrome in terms of global browser market share. (Desktop 12.85% , Mobile 27.8% - [Source](https://gs.statcounter.com/browser-market-share/desktop/worldwide))

While prioritizing Chrome is essential for enhancing web performance due to its market dominance, it is equally crucial to recognize Safari's substantial presence on mobile devices.

Thus, if your target audience predominantly uses Safari, it is crucial to dedicate efforts to test, improve, and optimize your website specifically for Safari users.

**Links**

-   Safari 17 Beta Release Notes [https://developer.apple.com/documentation/safari-release-notes/safari-17-release-notes](https://developer.apple.com/documentation/safari-release-notes/safari-17-release-notes)
-   Comparison of next-Gen image formats [https://cloudinary.com/blog/time\_for\_next\_gen\_codecs\_to\_dethrone\_jpeg](https://cloudinary.com/blog/time_for_next_gen_codecs_to_dethrone_jpeg)
-   Safari supported media formats [https://developer.apple.com/videos/play/wwdc2023/10122](https://developer.apple.com/videos/play/wwdc2023/10122)