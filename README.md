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
- [Integration](#integration)
  - [Self Mediation](#self-mediation)
    - [Self Mediation Swift Integration](#self-mediation-swift-integration)
    - [Self Mediation Obj-C Integration](#self-mediation-obj-c-integration)
  - [MoPub Mediation](#mopub-mediation)
    - [MoPub Mediation Swift Integration](#mopub-mediation-swift-integration)
    - [MoPub Mediation Obj-C Integration](#mopub-mediation-obj-c-integration)
    - [MoPub Custom SDK Network](#mopub-custom-sdk-network)
- [SKAdNetwork](#skadnetwork)

## Overview

### Latest Releases

- [LiftoffAds SDK][latest-display-sdk]
- Mediation Adapter SDKs
  - [Liftoff MoPub Adapter SDK for MoPub 5.12 and earlier][latest-mopub-pre5.13]
  - [Liftoff MoPub Adapter SDK for MoPub 5.13 and later][latest-mopub]

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

## Development Requirements

The LiftoffAds display SDK is written in Swift, compiled with the Swift 5.1
compiler, and distributed as a binary xcframework. Mediation adapter SDKs
provided by Liftoff are written in Objective-C and distributed as static
libraries.

To integrate the LiftoffAds display SDK, you will need at minimum:

- macOS 10.4.4 or later
- XCode 11.3 or later

## Integration

We currently support self-mediated setups and MoPub mediation. If you are using
a self-mediated setup, continue below. Otherwise, skip the section below and
continue with [MoPub Mediation](#mopub-mediation).

### Self Mediation

1. Download and unzip the [LiftoffAds SDK][latest-display-sdk].
2. Add `LiftoffAds.xcframework` to your app project.
3. In `General > Frameworks, Libraries, and Embedded Content`, select `Embed &
   Sign` for `LiftoffAds.xcframework`.

The following code samples may be used as reference for requesting and
displaying ads. Adjust the logic as necessary to meet the requirements of your
app.

#### Self Mediation Swift Integration

```swift
// AppDelegate.swift
import LiftoffAds

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(_ application: UIApplication,
                   didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Supported log levels: none, info, debug, error.
    Liftoff.logLevel = .error
    // Contact your Liftoff POC to retrieve your API key.
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

    loInterstitial = Liftoff.initInterstitialAdUnit(for: LIFTOFF_INTERSTITIAL_AD_UNIT)
    loInterstitial.delegate = self
    loInterstitial.requestAd()

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
    loBanner = Liftoff.initBannerAdUnit(for: LIFTOFF_BANNER_AD_UNIT, size: LOConstants.phoneBanner)
    loBanner.delegate = self
    loBanner.requestAd()
  }


  // MARK: LOInterstitialDelegate implementation
  // NOTE: These are optional implementations.

  // Called when the interstitial ad request is successfully filled.
  func loInterstitialDidLoad(_ interstitial: LOInterstitial) {
    // This will display the interstitial immediately after the ad request is
    // filled.
    interstitial.showAd(with: self)

    // To instead display the interstitial at an appropriate time as determined
    // by your app UX:
    // if loInterstitial.ready {
    //   loInterstitial.showAd(with: self)
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


  // MARK: LOBannerDelegate implementation
  // NOTE: These are optional implementations.

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
}
```

#### Self Mediation Obj-C Integration

```objective-c
// AppDelegate.m
@import LiftoffAds;

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  // Supported log levels: LOLoggingLevelNone, LOLoggingLevelInfo, LOLoggingLevelDebug, LOLoggingLevelError
  Liftoff.logLevel = LOLoggingLevelError;
  // Contact your Liftoff POC to retrieve your API key.
  [Liftoff initWithAPIKey:@"LIFTOFF_API_KEY"];

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
  // phoneBanner              = CGSizeMake(320, 50);
  // phoneBannerFlexWidth     = CGSizeMake(  0, 50);
  // tabletBanner             = CGSizeMake(728, 90);
  // tabletBannerFlexWidth    = CGSizeMake(  0, 90);
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

@end
```

### MoPub Mediation

LiftoffAds is a MoPub custom SDK network.

1. Download and unzip the [LiftoffAds SDK][latest-display-sdk].
2. Add `LiftoffAds.xcframework` to your app project.
3. In `General > Frameworks, Libraries, and Embedded Content`, select `Embed &
   Sign` for `LiftoffAds.xcframework`.
4. Download and unzip the Liftoff MoPub Adapter SDK.
   - [Liftoff MoPub Adapter SDK for MoPub 5.12 and earlier][latest-mopub-pre5.13]
   - [Liftoff MoPub Adapter SDK for MoPub 5.13 and later][latest-mopub]
5. In `General > Frameworks, Libraries, and Embedded Content`, select `Do Not
   Embed` for `LiftoffMoPubAdapter.xcframework`.
6. In `Build Settings`, add `-ObjC` to `Other Linker Flags` for your build
   target.

The following code samples may be used as reference for initializing the
LiftoffAds iOS SDK through MoPub. Adjust the logic as necessary to meet the
requirements of your app.

Instructions for requesting and displaying ads through MoPub can be found in the
[MoPub documentation](https://developers.mopub.com/publishers/ios/integrate/).

#### MoPub Mediation Swift Integration

```objective-c
// Bridging-Header.h
#import "LiftoffMoPubAdapter.h"
```

```swift
// AppDelegate.swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(_ application: UIApplication,
                   didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    let sdkConfig = MPMoPubConfiguration(adUnitIdForAppInitialization: "MOPUB_AD_UNIT_ID")
    sdkConfig.additionalNetworks = [LiftoffAdapterConfiguration.self]
    sdkConfig.setNetwork(
      ["apiKey": "LIFTOFF_API_KEY"],
      forMediationAdapter: "LiftoffAdapterConfiguration"
    )
    MoPub.sharedInstance().initializeSdk(with: sdkConfig)

    return true
  }

  // ...
}
```

#### MoPub Mediation Obj-C Integration

```objective-c
// AppDelegate.m
#import "LiftoffMoPubAdapter.h"

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  MPMoPubConfiguration *sdkConfig = [[MPMoPubConfiguration alloc] initWithAdUnitIdForAppInitialization:@"MOPUB_AD_UNIT_ID"];
  sdkConfig.additionalNetworks = @[LiftoffAdapterConfiguration.class];
  [sdkConfig setNetworkConfiguration:@{@"apiKey": @"LIFTOFF_API_KEY"}
                 forMediationAdapter:@"LiftoffAdapterConfiguration"];
  [[MoPub sharedInstance] initializeSdkWithConfiguration:sdkConfig completion:^{}];

  return YES;
}

// ...

@end
```

#### MoPub Custom SDK Network

You will need to create a new custom SDK network in the MoPub web portal for the
Liftoff ad network.

1. Create a new custom SDK network for Liftoff in [MoPub Networks](https://app.mopub.com/networks).
2. Create a new order for Liftoff in [MoPub Orders](https://app.mopub.com/orders).
3. After you create an order, create line items for your Liftoff ad units.
   Contact your Liftoff POC to set up ad units and retrieve your ad unit IDs.

The screenshots below show example configurations for Liftoff interstitial and
banner line items.

![](https://user-images.githubusercontent.com/573865/93147923-715b4680-f6a7-11ea-9584-11b2d9377cba.png)

![](https://user-images.githubusercontent.com/573865/93147999-994aaa00-f6a7-11ea-8e6f-5ba4c6513db0.png)

## SKAdNetwork

Liftoff is ready to support the upcoming changes to SKAdNetwork in iOS 14. To
enable SKAdNetwork for the LiftoffAds iOS SDK, which may increase your eCPM, add
the following to your app's plist:

```xml
<key>SKAdNetworkItems</key>
<array>
  <dict>
    <key>SKAdNetworkIdentifier</key>
    <string>7UG5ZH24HU.skadnetwork</string>
  </dict>
</array>
```

[latest-display-sdk]: https://github.com/liftoffio/LiftoffAds-iOS/releases/download/v1.1.0/LiftoffAds-v1.1.0.zip
[latest-mopub-pre5.13]: https://github.com/liftoffio/LiftoffAds-iOS/releases/download/mopub-v1.1.0/LiftoffMoPubAdapter-v1.1.0.zip
[latest-mopub]: https://github.com/liftoffio/LiftoffAds-iOS/releases/download/mopub-v2.1.0/LiftoffMoPubAdapter-v2.1.0.zip
