# Getting Started

This document describes the process of integrating the LiftoffAds SDK with an iOS app. It includes
instructions for integrating directly through a self-mediated setup as well as integrating through MoPub
mediation.

If you have any questions, please email them to: sdk@liftoff.io

## Overview

LiftoffAds supports the following formats and sizes:

__Formats__
- HTML
- HTML Video
- VAST Video

__Sizes__

- 320x480 (phone portrait)
- 480x320 (phone landscape)
- 768x1024 (tablet portrait)
- 1024x768 (tablet landscape)

## Requirements

LiftoffAds is written in Swift and compiled with the Swift 5.1 compiler. LiftoffAds supports iOS 10+.

__Development Environment__
- macOS 10.4.4 or later
- XCode 11.3 or later


## SKAdNetwork

To support SKAdNetwork with LiftoffAds, add the following to your app's plist file:

```xml
<key>SKAdNetworkItems</key>
<array>
  <dict>
    <key>SKAdNetworkIdentifier</key>
    <string>7UG5ZH24HU.skadnetwork</string>
  </dict>
</array>
```

## Setup for Self Mediation

Follow this setup if you are using custom or in-house mediation.

1. Download and unzip the latest version of LiftoffAds.zip
2. Add LiftoffAds.xcframework to your app project
3. In your app target's settings (General > Frameworks, Libraries, and Embedded Content), select Embed & Sign for LiftoffAds.xcframework

### Initializing the LiftoffAds SDK

#### Swift Apps

In your app's `ViewController.swift` or where appropriate:

1. Import the Liftoff framework:

   ``` swift
   import LiftoffAds
   ```

2. Initialize the LiftoffAds SDK with your API key:

   ``` swift
   Liftoff.initWithAPIKey("LIFTOFF_API_KEY")
   ```

3. Create an interstitial ad for an ad unit:

   ``` swift
   var interstitial = Liftoff.initInterstitialAdUnit(for: "LIFTOFF_AD_UNIT_ID")
   ```

4. Assign a delegate to be notified of ad lifecycle events:

   ``` swift
   interstitial.delegate = self
   ```

5. Request an ad:

   ``` swift
   interstitial.requestAd()
   ```

6. Display the ad:

   ``` swift
   if interstitial.ready {
     interstitial.showAd(with: self)
   }
   ```

#### Objective-C Apps

In your app's `ViewController.m` or where appropriate:

1. Import the Liftoff framework:

   ``` objective-c
   @import LiftoffAds;
   ```

2. Initialize the LiftoffAds SDK with your API key:

   ``` objective-c
   [Liftoff initWithAPIKey: @"LIFTOFF_API_KEY"];
   ```

3. Create an interstitial ad for an ad unit:

   ``` objective-c
   LOInterstitial *interstitial =
    [Liftoff initInterstitialAdUnitFor: @"LIFTOFF_AD_UNIT_ID"];
   ```

4. Assign a delegate to be notified of ad lifecycle events:

   ``` objective-c
   interstitial.delegate = self;
   ```

5. Request an ad:

   ``` objective-c
   [interstitial requestAd];
   ```

6. Display the ad:

   ``` objective-c
   if (interstitial.ready) {
     [interstitial showAdWith: self];
   }
   ```
