---
title: "Launching mediahelper.io."
date: 2021-09-21T17:07:00+02:00
draft: false
slug: "technical-writing/mediahelper-io"
description: "Web tool to deal with Video APIs in the browser."
keywords: ["web", "tool", "api", "video", "browser"]
toc: true
---

![mediahelper-logo](./mediahelper-logo.png)

## The streaming universe

For a few years now, the streaming universe is booming. Powerful actors have emerged from this new industry. They contribute to pushing the movement forwards in a relatively fast way.

However, distributing video content across the globe and to all the variety of devices is complex and requires real expertise.

After being in that field for more than 3 years now, I’ve decided to launch a website that aims to help developers and people that work in that industry, to facilitate their daily tasks and thus save them some time.

That website started from a constat that the browser APIs in the video industry are not well known from the developers who are not working in that field.

This is typically the case for me, when I first arrived in that area, I had to learn everything from scratch, MSE, EME API was something completely unknown to me.

However, the streaming industry on the internet is occupying more than 70% of time-consuming bandwidth, which means that almost every internet user has indirectly used it without knowing it (fortunately).

But as said before, distributing video on the internet is something that not a lot of companies can afford, the costs are enormous, server costs, video creation, etc...
It means that there is a restricted access to this field.

At that time, the internet was lacking of resources about this topic.

That's why I decided to build [mediahelper.io](https://mediahelper.vercel.app/), intending to facilitate the access of those APIs, for everyone working in that industry and beyond.

The website consists of several parts that aim to expose those APIs.

## Encryption EME API

EME is a powerful and yet not a very common API.

The EME API known as the Encrypted media extension, will help the user agent to communicate with the CDM (Content decryption module a sort of black box) to decipher the video content through a license containing the keys.

The EME API is also very handy to know what CDM module is contained in the browser.

The thing with the EME API is it requires many parameters to make it work efficiently.

This is not something that a developer can afford if he has to do it more than 1 time a day, it will be very time-consuming and so, not very productive, moreover, this is not very exciting to fill the parameters required by the API.

That’s why I tried to create the most accessible user experience possible through [mediahelper.io/encryption](https://mediahelper.vercel.app/encryption).

There is first, a block that doesn't need any configuration, that aims to be the most simple possible to give a first look at the what CDM is currently available into the browser.

Then, if you want a more precise configuration, you just have underneath a user interface, that allows the user to fully configure the parameters, thought to be the more accessible possible.

## Media source extension (MSE)

Depending on your device, on the one you want to play, you have certain codec technology that will allow you to decode the content (H265, H264, WebM, etc…)

However, every device doesn't have the same chances, for example, the h265 codec is relatively poorly supported on computer device but are fairly common on a device like a gaming console or a connected TV.

So, [mediahelper.io/decoding](https://mediahelper.vercel.app/decoding) has a user interface to know if your device can support the codec you want to use.

Moreover, the chrome team has developed a new API poorly known for the moment that allows you to know about useful thing like how your computer will perform when it comes to decoding video data.

There is also a UI that gives more practical access to that API.

## An [ISO BMFF Parser](https://www.w3.org/TR/mse-byte-stream-format-isobmff/)

To deliver video on the web, almost every videos players are based on adaptive streaming.

This means that instead of having to retrieve the whole chunk of video we divide the video into a smaller chunk of variable size usually 2sec for standard content.

This permits to download of a small part of video content but also enables adaptive streaming which is the fact to download a specific video chunk depending on the internet connection capabilities.

Those chunks of video are binary content.

Many useful metadata are contained in those chunks like the width or the height of the video chunk, but also the duration.

That tools parse the [isobmff byte stream format](https://www.w3.org/TR/mse-byte-stream-format-isobmff/) boxes inside the binary content and render them as a human-readable format. Moreover, the chrome team has developed a new API poorly known for the moment that allows you to know about a useful thing like how your computer will perform when it comes to decoding video data.

- [mediahelper.io/isobmff](https://mediahelper.vercel.app/isobmff)

## A Manifest Parser

To deliver video the player has to know where he can access the video files/segments on the internet.

For that tasks, the manifest is used to locate all those files but also is here to give many important pieces of information about the content we are about to play.

Few protocols exist, all based on the HTTP application layer.

- [HLS](https://datatracker.ietf.org/doc/html/rfc8216)
- [DASH](https://dashif.org/docs/DASH-IF-IOP-v4.3.pdf)
- [SMOOTH](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-sstr/8383f27f-7efe-4c60-832a-387274457251)

All those protocols have their precise specification written.

The website contains a manifest parser that allows checking how the manifest is interpreted by the player.

- [mediahelper.io/manifest](https://mediahelper.vercel.app/manifest)

## Few videos samples content

Most of the time, the video developer will need samples to test their features, but that is not always practical to create content, packaged and encoded itself, that’s why I propose a collection of samples files.

- [mediahelper.io/samples](https://mediahelper.vercel.app/samples)

## Conclusion

The [mediahelper.io](https://mediahelper.vercel.app/) website is especially concerned about the API embedded into a browser, those APIs have made their proof and are widely used even on embedded devices like ChromeCast devices, gaming consoles, or connected TV.

Of course, that website will never be finished and it's a good thing because it would be awesome that people add their features for their use case to the website to make it even better.

So feel free to contribute [here!](https://github.com/PaulRosset/mediahelper).
