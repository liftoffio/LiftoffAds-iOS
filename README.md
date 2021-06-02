# LiftoffAds iOS SDK

This repository describes the LiftoffAds iOS SDK technical integration. For help
configuring and testing ad units or setting up reporting, please contact your
Liftoff POC.

For any other questions, please email sdk@liftoff.io.

## Table of Contents

- [Overview](#overview)
  - [Latest Releases](#latest-releases)
  - [Supported Devices](#supported-devices)
  - [Supported Ad Sizes](#supported-ad-sizes)
  - [Supported Ad Types](#supported-ad-types)
- [Development Requirements](#development-requirements)
  - [SKAdNetwork](#skadnetwork)
- [Integrating the SDK](#integrating-the-sdk)
  - [Downloading the SDK](#downloading-the-sdk)
    - [CocoaPods](#cocoapods)
    - [Direct Download](#direct-download)
  - [Code Changes](#code-changes)
    - [MoPub Mediation](#mopub-mediation)
      - [Swift](#swift)
      - [Objective-C](#objective-c)
    - [Self Mediation](#self-mediation)
      - [Swift](#swift-1)
      - [Objective-C](#objective-c-1)
    - [GDPR/CCPA and User Consent](#gdprccpa-and-user-consent)
    - [Test Ad Units](#test-ad-units)
  - [Creating a MoPub Custom SDK Network](#creating-a-mopub-custom-sdk-network)
- [COPPA](#coppa)
- [Troubleshooting](#troubleshooting)

## Overview

### Latest Releases

- [LiftoffAds SDK][latest-display-sdk]
- Mediation Adapter SDKs
  - [Liftoff MoPub Adapter SDK][latest-mopub]

### Supported Devices

- iPhone
- iPad

### Supported Ad Sizes

- 320x480 (iPhone portrait interstitial)
- 480x320 (iPhone landscape interstitial)
- 768x1024 (iPad portrait interstitial)
- 1024x768 (iPad landscape interstitial)
- 320x50 (iPhone banner)
- 728x90 (iPad banner)
- 300x250 (medium rectangle)

### Supported Ad Types

- HTML and HTML video
- VAST video
- Rewarded

## Development Requirements

The LiftoffAds display SDK is written in Swift, compiled with the Swift 5.3
compiler, and distributed as a binary xcframework. Mediation adapter SDKs
provided by Liftoff are written in Objective-C and distributed as static
libraries.

To integrate the latest version of the LiftoffAds display SDK, you will need at
minimum:

- macOS >= 11 (Big Sur)
- XCode >= 12.5

### SKAdNetwork

LiftoffAds uses SKAdNetwork in iOS 14+. To enable SKAdNetwork for the LiftoffAds
network, add the following to your app's plist:

```xml
<key>SKAdNetworkItems</key>
<array>
  <dict>
    <key>SKAdNetworkIdentifier</key>
    <string>7UG5ZH24HU.skadnetwork</string>
  </dict>
</array>
```

NOTE: SKAdNetwork is likely to be required for the LiftoffAds network in the
near future. When this requirement is added, your fill rate may drop
substantially if you do not include the plist entry above.

## Integrating the SDK

### Downloading the SDK

You can download the SDK through CocoaPods or directly.

#### CocoaPods

**Only CocoaPods versions >= 1.10 are supported since the SDK is packaged as an
xcframework.**

The easiest way to add the LiftoffAds SDK to your project is via CocoaPods.

Include `LiftoffAds` as a dependency in your PodFile:

```ruby
source "https://github.com/CocoaPods/Specs.git"

target "MyApp" do
  pod "LiftoffAds"
  pod "LiftoffMoPubAdapter" # Only if you are using MoPub mediation.
end
```

#### Direct Download

If you aren't using CocoaPods as described above, you can download the SDK and
manually add it to your project:

1. Download and unzip the [LiftoffAds SDK][latest-display-sdk].
2. Add `LiftoffAds.xcframework` to your app's project.
3. In `General > Frameworks, Libraries, and Embedded Content`, select `Embed &
   Sign` for `LiftoffAds.xcframework`.

Only if you are using MoPub mediation, download the Liftoff MoPub Adapter SDK
and add it to your project:

4. Download and unzip the [Liftoff MoPub Adapter SDK][latest-mopub].
5. In `General > Frameworks, Libraries, and Embedded Content`, select `Do Not
   Embed` for `LiftoffMoPubAdapter.xcframework`.
6. In `Build Settings`, add `-ObjC` to `Other Linker Flags` for your build
   target.

### Code Changes

We currently support MoPub mediation and self-mediated setups. If you are using
a self-mediated setup, skip the section below and continue with [Self
Mediation](#self-mediation).

#### MoPub Mediation

**Only MoPub SDK versions >= 5.13 are supported. The latest Liftoff MoPub
adapter requires MoPub SDK >= 5.16.**

LiftoffAds can be added as a MoPub custom SDK network.

Instructions for requesting and displaying ads through MoPub can be found in the
[MoPub documentation](https://developers.mopub.com/publishers/ios/integrate/).

The following code samples may be used as reference for initializing the
LiftoffAds iOS SDK through MoPub. Adjust the logic as necessary to meet the
requirements of your app.

##### Swift

```objective-c
// Bridging-Header.h
#if __has_include("LiftoffMoPubAdapter.h")
  #import <LiftoffMoPubAdapter.h>
#else
  #import <LiftoffMoPubAdapter/LiftoffMoPubAdapter.h>
#endif
```

```swift
// AppDelegate.swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(_ application: UIApplication,
                   didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    let sdkConfig = MPMoPubConfiguration(adUnitIdForAppInitialization: "MOPUB_AD_UNIT_ID")

    // BEGIN: Required for Liftoff custom SDK network.
    sdkConfig.additionalNetworks = [LiftoffAdapterConfiguration.self]
    sdkConfig.setNetwork(
      // Contact your Liftoff POC to retrieve your API key. Define this constant
      // elsewhere or replace with a hardcoded string.
      ["apiKey": LIFTOFF_API_KEY],
      forMediationAdapter: "LiftoffAdapterConfiguration"
    )
    // END

    MoPub.sharedInstance().initializeSdk(with: sdkConfig)

    return true
  }

  // ...
}
```

##### Objective-C

```objective-c
// AppDelegate.m
#if __has_include("LiftoffMoPubAdapter.h")
  #import <LiftoffMoPubAdapter.h>
#else
  #import <LiftoffMoPubAdapter/LiftoffMoPubAdapter.h>
#endif

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  MPMoPubConfiguration *sdkConfig = [[MPMoPubConfiguration alloc] initWithAdUnitIdForAppInitialization:@"MOPUB_AD_UNIT_ID"];

  // BEGIN: Required for Liftoff custom SDK network.
  sdkConfig.additionalNetworks = @[LiftoffAdapterConfiguration.class];
  // Contact your Liftoff POC to retrieve your API key. Define this constant
  // elsewhere or replace with a hardcoded string.
  [sdkConfig setNetworkConfiguration:@{@"apiKey":LIFTOFF_API_KEY}
                 forMediationAdapter:@"LiftoffAdapterConfiguration"];
  // END

  [[MoPub sharedInstance] initializeSdkWithConfiguration:sdkConfig completion:^{}];

  return YES;
}

// ...

@end
```

#### Self Mediation

The following code samples may be used as reference for requesting and
displaying ads. Adjust the logic as necessary to meet the requirements of your
app.

##### Swift

```swift
// AppDelegate.swift
import LiftoffAds

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(_ application: UIApplication,
                   didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Supported log levels: none, info, debug, error.
    Liftoff.logLevel = .error
    // Contact your Liftoff POC to retrieve your API key. Define this constant
    // elsewhere or replace with a hardcoded string.
    Liftoff.initWithAPIKey(LIFTOFF_API_KEY)

    return true
  }

  // ...
}
```

```swift
// ViewController.swift
import LiftoffAds

class ViewController: UIViewController, LOInterstitialDelegate, LOBannerDelegate {
  var loInterstitial: LOInterstitial!
  var loBanner: LOBanner!

  override func viewDidLoad() {
    super.viewDidLoad()

    // Contact your Liftoff POC to retrieve your ad unit IDs.
    // NOTE: Liftoff interstitial and banner objects cannot be reused to request
    // multiple ads. You must initialize a new object for each ad request.

    self.loInterstitial = Liftoff.initInterstitialAdUnit(for: LIFTOFF_INTERSTITIAL_AD_UNIT)
    self.loInterstitial.delegate = self
    self.loInterstitial.requestAd()

    // The size argument may be any CGSize, but we recommend choosing from a
    // preset list of sizes from LOConstants (a 0 indicates a flexible
    // dimension):
    // phoneBanner              = CGSize(width: 320, height: 50)
    // phoneBannerFlexWidth     = CGSize(width:   0, height: 50)
    // tabletBanner             = CGSize(width: 728, height: 90)
    // tabletBannerFlexWidth    = CGSize(width:   0, height: 90)
    // mediumRectangle          = CGSize(width: 300, height: 250)
    // mediumRectangleFlexWidth = CGSize(width:   0, height: 250)
    // flexAll                  = CGSize.zero
    self.loBanner = Liftoff.initBannerAdUnit(for: LIFTOFF_BANNER_AD_UNIT, size: LOConstants.phoneBanner)
    self.loBanner.delegate = self
    self.loBanner.requestAd()
  }


  // MARK: LOInterstitialDelegate implementation

  // Called when the interstitial ad request is successfully filled.
  func loInterstitialDidLoad(_ interstitial: LOInterstitial) {
    // This will display the interstitial immediately after the ad request is
    // filled.
    interstitial.showAd(with: self)

    // To instead display the interstitial at an appropriate time as determined
    // by your app UX:
    // if self.loInterstitial.ready {
    //   self.loInterstitial.showAd(with: self)
    // }
  }

  // Called when the interstitial ad request cannot be filled.
  func loInterstitialDidFailToLoad(_ interstitial: LOInterstitial) {}

  // Called before the interstitial view controller is presented.
  func loInterstitialWillShow(_ interstitial: LOInterstitial) {}

  // Called after the interstitial view controller is presented.
  func loInterstitialDidShow(_ interstitial: LOInterstitial) {}

  // Called before the interstitial view controller is hidden.
  func loInterstitialWillHide(_ interstitial: LOInterstitial) {}

  // Called after the interstitial view controller is hidden.
  func loInterstitialDidHide(_ interstitial: LOInterstitial) {}

  // Called when the interstitial becomes visible to the user.
  func loInterstitialImpressionDidTrigger(_ interstitial: LOInterstitial) {}

  // Called when the user will be directed to an external destination.
  func loInterstitialClickDidTrigger(_ interstitial: LOInterstitial) {}

  // Called when the user has earned a reward by watching a rewarded ad.
  func loInterstitialWillRewardUser(_ interstitial: LOInterstitial) {}


  // MARK: LOBannerDelegate implementation

  // Called when the banner ad request is successfully filled. The view argument
  // is the banner's UIView.
  func loBannerDidLoad(_ banner: LOBanner, view: UIView) {
    self.view.addSubview(view)
  }

  // Called when the banner ad request cannot be filled.
  func loBannerDidFailToLoad(_ banner: LOBanner) {}

  // Called when the banner becomes visible to the user.
  func loBannerImpressionDidTrigger(_ banner: LOBanner) {}

  // Called when the user will be directed to an external destination.
  func loBannerClickDidTrigger(_ banner: LOBanner) {}

  // Implementing this by returning a root view controller allows banners to
  // present a StoreKit modal for app installs, which may improve conversion
  // rates and your eCPM.
  func loBannerViewControllerForPresentingModalView(_ banner: LOBanner) -> UIViewController? {
    return self
  }

  // Called when a modal view controller will be displayed after a user click.
  func loBannerModalWillShow(_ banner: LOBanner) {}

  // Called when a modal view controller is displayed after a user click.
  func loBannerModalDidShow(_ banner: LOBanner) {}

  // Called when a modal view controller will be dismissed.
  func loBannerModalWillHide(_ banner: LOBanner) {}

  // Called when a modal view controller is dismissed.
  func loBannerModalDidHide(_ banner: LOBanner) {}

  // Called when the user will be directed to an external destination.
  func loBannerWillLeaveApplication(_ banner: LOBanner)
}
```

##### Objective-C

```objective-c
// AppDelegate.m
@import LiftoffAds;

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  // Supported log levels: LOLoggingLevelNone, LOLoggingLevelInfo, LOLoggingLevelDebug, LOLoggingLevelError
  Liftoff.logLevel = LOLoggingLevelError;
  // Contact your Liftoff POC to retrieve your API key. Define this constant
  // elsewhere or replace with a hardcoded string.
  [Liftoff initWithAPIKey:LIFTOFF_API_KEY];

  return YES;
}

// ...

@end
```

```objective-c
// ViewController.m
@import LiftoffAds;

@interface ViewController () <LOInterstitialDelegate, LOBannerDelegate>

@property (nonatomic) LOInterstitial *loInterstitial;
@property (nonatomic) LOBanner *loBanner;

@end

@implementation ViewController

- (void)viewDidLoad {
  [super viewDidLoad];

  // Contact your Liftoff POC to retrieve your ad unit IDs.
  // NOTE: Liftoff interstitial and banner objects cannot be reused to request
  // multiple ads. You must initialize a new object for each ad request.

  self.loInterstitial = [Liftoff initInterstitialAdUnitFor:@"LIFTOFF_INTERSTITIAL_AD_UNIT_ID"];
  self.loInterstitial.delegate = self;
  [self.loInterstitial requestAd];

  // The size argument may be any CGSize, but we recommend choosing from a
  // preset list of sizes from LOConstants (a 0 indicates a flexible
  // dimension):
  // phoneBanner              = CGSizeMake(320,  50);
  // phoneBannerFlexWidth     = CGSizeMake(  0,  50);
  // tabletBanner             = CGSizeMake(728,  90);
  // tabletBannerFlexWidth    = CGSizeMake(  0,  90);
  // mediumRectangle          = CGSizeMake(300, 250):
  // mediumRectangleFlexWidth = CGSizeMake(  0, 250);
  // flexAll                  = CGSizeZero;
  self.loBanner = [Liftoff initBannerAdUnitFor:@"LIFTOFF_BANNER_AD_UNIT_ID" size:LOConstants.phoneBanner];
  self.loBanner.delegate = self;
  [self.loBanner requestAd];
}


#pragma mark - LOInterstitialDelegate implementation

// Called when the interstitial ad request is successfully filled.
- (void)loInterstitialDidLoad:(LOInterstitial *)interstitial {
  // This will display the interstitial immediately after the ad request is
  // filled.
  [interstitial showAdWith:self];

  // To instead display the interstitial at an appropriate time as determined
  // by your app UX:
  // if (self.loInterstitial.ready) {
  //   [self.loInterstitial showAdWith:self];
  // }
}

// Called when the interstitial ad request cannot be filled.
- (void)loInterstitialDidFailToLoad:(LOInterstitial *)interstitial {}

// Called before the interstitial view controller is presented.
- (void)loInterstitialWillShow:(LOInterstitial *)interstitial {}

// Called after the interstitial view controller is presented.
- (void)loInterstitialDidShow:(LOInterstitial *)interstitial {}

// Called before the interstitial view controller is hidden.
- (void)loInterstitialWillHide:(LOInterstitial *)interstitial {}

// Called after the interstitial view controller is hidden.
- (void)loInterstitialDidHide:(LOInterstitial *)interstitial {}

// Called when the interstitial becomes visible to the user.
- (void)loInterstitialImpressionDidTrigger:(LOInterstitial *)interstitial {}

// Called when the user will be directed to an external destination.
- (void)loInterstitialClickDidTrigger:(LOInterstitial *)interstitial {}

// Called when the user has earned a reward by watching a rewarded ad.
- (void)loInterstitialWillRewardUser:(LOInterstitial *)interstitial {}


#pragma mark - LOBannerDelegate implementation

// Called when the banner ad request is successfully filled. The view argument
// is the banner's UIView.
- (void)loBannerDidLoad:(LOBanner *)banner view:(UIView *)view {
  [self.view addSubview:view];
}

// Called when the banner ad request cannot be filled.
- (void)loBannerDidFailToLoad:(LOBanner *)banner {}

// Called when the banner becomes visible to the user.
- (void)loBannerImpressionDidTrigger:(LOBanner *)banner {}

// Called when the user will be directed to an external destination.
- (void)loBannerClickDidTrigger:(LOBanner *)banner {}

// Implementing this by returning a root view controller allows banners to
// present a StoreKit modal for app installs, which may improve conversion rates
// and your eCPM.
- (UIViewController *)loBannerViewControllerForPresentingModalView:(LOBanner *)banner {
  return self;
}

// Called when a modal view controller will be displayed after a user click.
- (void)loBannerModalWillShow:(LOBanner *)banner {}

// Called when a modal view controller is displayed after a user click.
- (void)loBannerModalDidShow:(LOBanner *)banner {}

// Called when a modal view controller will be dismissed.
- (void)loBannerModalWillHide:(LOBanner *)banner {}

// Called when a modal view controller is dismissed.
- (void)loBannerModalDidHide:(LOBanner *)banner {}

// Called when the user will be directed to an external destination.
- (void)loBannerWillLeaveApplication:(LOBanner *)banner {}

@end
```

#### GDPR/CCPA and User Consent

LiftoffAds complies with the EU's General Data Protection Regulation (GDPR) and
the California Consumer Privacy Act (CCPA). However, LiftoffAds does not
currently manage its own consent mechanism, so you will be required to pass user
consent information to our SDK. The following code samples can be used as
reference:

##### Swift

```swift
LOPrivacySettings.setHasUserConsent(true)
```

##### Objective-C

```objective-c
[LOPrivacySettings setHasUserConsent:true];
```

#### Test Ad Units

Use the following ad unit IDs to display a LiftoffAds test creative and verify
successful integration.

| Ad Unit ID                        | Size           | Type                       |
| --------------------------------- | -------------- | -------------------------- |
| `liftoff-banner-mrect-test`       | Banner / MRECT | VAST video, HTML video     |
| `liftoff-interstitial-video-test` | Interstitial   | VAST video                 |
| `liftoff-interstitial-html-test`  | Interstitial   | HTML video                 |
| `liftoff-rewarded-video-test`     | Rewarded Interstitial | VAST video          |

### Creating a MoPub Custom SDK Network

*You may skip this section if you are not using MoPub mediation.*

You will need to create a new custom SDK network in the MoPub web portal for the
Liftoff ad network.

1. Create a new custom SDK network for Liftoff in [MoPub Networks](https://app.mopub.com/networks).
2. Create a new order for Liftoff in [MoPub Orders](https://app.mopub.com/orders).
3. Create line items for your Liftoff ad units. Contact your Liftoff POC to set
   up ad units and retrieve your ad unit IDs.

The screenshots below show example configurations for Liftoff banner/medium
rectangle, standard interstitial, and rewarded interstitial line items.

#### Banner / Medium Rectangle (MRECT)

Both banner and mrect ads use the `LiftoffBannerCustomEvent` class.

![](https://user-images.githubusercontent.com/573865/93147999-994aaa00-f6a7-11ea-8e6f-5ba4c6513db0.png)

#### Interstitial

![](https://user-images.githubusercontent.com/573865/93147923-715b4680-f6a7-11ea-9584-11b2d9377cba.png)

#### Rewarded Interstitial

![](https://user-images.githubusercontent.com/573865/96619293-d6afe200-12ba-11eb-8a14-133be3d8f775.png)

## COPPA

LiftoffAds does not serve end users who fall under the restrictions of the
Children’s Online Privacy Protection Act (COPPA), specifically age 12 years and
younger. If you collect information that indicates a user falls under this
category, you must not use the LiftoffAds SDK for the user's sessions.

## Troubleshooting

Set the log level to `debug` before troubleshooting.

Common integration issues:

- `"No API key provided"`: Missing API key. Verify that you've included the
  proper initialization code (see above).
- `"Error authentication: Check API key"`: Incorrect API key. Check for any
  typos in your API key.
- `"Unable to fetch ad unit: <PROVIDED_AD_UNIT_ID>"`: Could not fill ad request.
  Check ad unit ID for typos.

[latest-display-sdk]: https://github.com/liftoffio/LiftoffAds-iOS/releases/download/v1.6.0/LiftoffAds-v1.6.0.zip
[latest-mopub]: https://github.com/liftoffio/LiftoffAds-iOS/releases/download/mopub-v2.4.1/LiftoffMoPubAdapter-v2.4.1.zip
