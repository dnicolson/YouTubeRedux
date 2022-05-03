
---

:warning: As of 03.03.2021 YouTube no longer supports the TVML app and this no longer works.

---

# YouTubeRedux

YouTubeRedux is an attempt to resurrect the classic tvOS YouTube app as an alternative to the current cross-platform app.

It's possible to sideload the `YouTube 1.2.1.ipa` app but the release of tvOS 14 has resulted in some playback issues, the audio can become out of sync or the video frames can freeze indefinitely.

## The Original YouTube App

The first step was to use [Hopper](https://www.hopperapp.com/) to disassemble the original IPA file, this showed the entry point and the linked frameworks. The first thing that was noticeable was how few symbols there were compared to a default tvOS app, it turns out that there is only a single function of compiled code.

Looking at the `Info.plist` file it appears as though it was built with a predecessor to TVML based on a JavaScript file referenced with the `TVLContentKitBootURL` key.

Looking for this string confirms it is part of the `ATVLegacyContentKit` private framework:

```bash
strings /Applications/Xcode.app/Contents/Developer/Platforms/AppleTVOS.platform/Library/Developer/CoreSimulator/Profiles/Runtimes/tvOS.simruntime/Contents/Resources/RuntimeRoot/System/Library/PrivateFrameworks/ATVLegacyContentKit.framework/ATVLegacyContentKit | grep TVLContentKitBootURL
```

Using Hopper and [Ghidra](https://ghidra-sre.org/), you can see that the entry point contained the `TVLApplication` and `TVLAppDelegate` strings. These are the `principalClassName` and the `delegateClassName` parameters for the `UIApplicationMain` function. The key is to call `UIApplicationMain` with both these, the app works by omitting the `principalClassName` but a network error alert is briefly shown when launching the app.

The `TVMLKit.framework` framework needs to be added to the project, `CoreFoundation.framework` was linked in the original app but doesn't seem to be needed anymore. The original project was built with Xcode 7.2 but Xcode 12 seems to able to produce a comparable binary.

## ATVLegacyContentKit Protocol

It's possible to inspect the HTTP traffic with a tool like [Proxyman](https://proxyman.io/) when running the project in the Simulator.

Attempting to use the YouTube JavaScript payload with a normal TVML app showed some incompatibilities, it appears as though the protocol is based on what the US Trailers app and [PlexConnect](https://github.com/iBaa/PlexConnect) uses.

Here are the response and request headers for the initial payload:

```
Host: www.youtube.com
Accept: */*
Cookie: VISITOR_INFO1_LIVE=Ngejf51Do6Q
Connection: keep-alive
X-Apple-TV-Version: 14.2
User-Agent: com.dnicolson.YouTubeRedux/1.0 iOS/14.2 model/AppleTV6,2 build/18K54
Accept-Language: en
X-Apple-Tz: 3600
Accept-Encoding: gzip, deflate, br
```

```
P3P: CP="This is not a P3P policy! See http://support.google.com/accounts/answer/151657?hl=en for more info."
X-Content-Type-Options: nosniff
Content-Length: 20437
X-Frame-Options: SAMEORIGIN
Content-Type: text/html; charset=utf-8
Content-Encoding: br
Cache-Control: no-cache
Expires: Tue, 27 Apr 1971 19:44:06 GMT
Date: Thu, 05 Nov 2020 21:08:34 GMT
Server: YouTube Frontend Proxy
X-XSS-Protection: 0
Alt-Svc: h3-Q050=":443"; ma=2592000,h3-29=":443"; ma=2592000,h3-T051=":443"; ma=2592000,h3-T050=":443"; ma=2592000,h3-Q046=":443"; ma=2592000,h3-Q043=":443"; ma=2592000,quic=":443"; ma=2592000; v="46,43"
```

## tvOS Compatibility

Building the app with the tvOS 10 Simulator appears to have no compatibility issues.

- tvOS 11 has slight alignment issues with view headings and video thumbnails are aligned to the right.
- tvOS 12 appears to have the same issues.
- tvOS 13 has the same issues, including more visible headings due to the smaller top navigation style.
- tvOS 14 has the existing issues in addition to the list highlight colour looking incorrect.

## Next Steps

There are some options to modernise the app, and maybe even add 4K support or remove ads.

1. Intercepting the HTTP responses to fix the layout issues on tvOS 11 and after.
2. Using the payload endpoint in a TVML app and replacing and injecting code to run in a TVML app.
3. Use the same authentication method and create an entirely new TVML app.
