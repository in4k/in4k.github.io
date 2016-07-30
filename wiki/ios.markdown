---
title: "iOS"
layout: "wiki-page"
---

** WORK IN PROGRESS, DO NOT DISTURB **

# Comment

The App Store: Not for demos.
-----------------------------

The App Store is not your friend. It's great for some things, but not demos. The first demo ever submitted to the App Store was rejected with this rejection reason: "Demos are not allowed. Please consider making a lite version instead". Forget it.

Xcode: Not so hot either.
-------------------------
You can code in Xcode (or whatever else) and distribute source. OK, but few people are going to see it realtime. And there are no executable compressors for iOS I know of. It's possible, but you're on your own.

Swift Playgrounds: Oh yes.
--------------------------

[Swift Playgrounds](https://developer.apple.com/swift/playgrounds/) (I'll call it SP from now on) is a new app for iPad and iOS 10 (which is still in (public) beta as I write). It's designed mainly to help kids learn to code. But guess what? We can code on it too, and it provides full hardware access including OpenGL ES and Metal!

You get just 1 choice of language: Swift. If your demo is 90% shader though, you'll get GLSL or [MSL](https://developer.apple.com/library/ios/documentation/Metal/Reference/MetalShadingLanguageGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40014364-CH1-SW1) (Metal Shading Language - C++11 based shader language).

You can code entirely on iPad (Apple provide a custom, code-oriented keyboard plus heavy use of autocomplete that makes it surprisingly good). You can also 

# Tools

# "Cross platform" (iOS / macOS) dev

As good as the SP soft keyboard on iPad is, you don't want to use that if you have a mac handy. Copy the playground to the mac (airdrop usually works best), and open it in Xcode.

If you use any platform-specific APIs you'll need to work around them. Common things:

* Platform specific code is done like this:

#if os(iOS)
// iOS code
#elseif os(OSX)
// mac code
#endif

* Types like NSRect (mac) and CGRect (iOS): Make a typealias for each to say Rect.

**Important note for Metal**: Metal is NOT supported for iOS projects in Xcode. To fix this, open the Utilities pane (right hand side) in Xcode, and change the Platform from iOS to macOS. Remember to change it back to iOS before sending it back to the iPad, because Mac projects won't open at all on iPad.

# References

# Examples
* [Playground Punk / CRTC](http://www.pouet.net/prod.php?which=67806)
