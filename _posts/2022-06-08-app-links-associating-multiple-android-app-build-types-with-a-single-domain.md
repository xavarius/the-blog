---
layout: default
title: "App Links: Associating Multiple Android App Build Types with a Single Domain"
date: 2022-06-08
---

### **App Links: Associating Multiple Android App Build Types with a Single Domain**

Although I won’t go into the configuration nor benefits of Android App Links,
you can find them [here](https://developer.android.com/training/app-links), [here](https://developer.android.com/training/app-links/verify-site-associations), and [here](https://developers.google.com/digital-asset-links/v1/create-statement), I would like to elaborate on the missing documentation bit.

Specifically, how to associate and verify ownership between your domain
(a website) and multiple build types **of the same** Android application
using a Digital Asset Links JSON file.
Simply put, how to get rid of disambiguous dialog in such an environment.

![We do not want that, ever!]({{ '/assets/1_sNCApKUp4caWwqZWqXZrMA.jpg' | relative_url }})
*We do not want that, ever! Source: [https://www.flickr.com/photos/adewale_oshineye/21830113332](https://www.flickr.com/photos/adewale_oshineye/21830113332) Licence CC.*

While it’s not groundbreaking research, this guide can spare you
a couple of hours or even days of waiting impatiently to see if ownership
over an app has been verified.

In most cases, you’re not able to deploy a Digital Asset Links JSON file
on your own, thus you’re dependent on a third party to do so.
It means waiting. It’s better to Do it once, properly.

App links (deep links) are usually not a standalone functionality of an app
yet rather a collaboration between many teams that drives users
into an app (for example, marketing campaigns).

More precisely app links usually require a bit of collaboration from a mobile team and marketing/DevOps/web/backend/you-name-it team from your company.

Let’s give an answer by starting with a question.

**Given** the fact that an app has multiple build types, each having its own, distinct certificate fingerprint SHA-1
**When** a subset of build types points into the same domain
**Then** could ownership be verified for all build types using a single domain?

![This is the “complexity” we’re facing.]({{ '/assets/1_MSFfUt4ondPMJ5aVmTPGuQ.jpg' | relative_url }})
*This is the “complexity” we’re facing. Source: DIY.*

Let’s give an example since it might be confusing.

An app has DEV (*debug*) and QA/UAT/SIT (*debug* and *release*) build type configurations that are both pointing to the same domain (website),
say, *uat.example.com*. That means you have two “apps” with
two **distinct** signing key configs: two distinct SHA-1 certificate fingerprints are being used to validate ownership of App Links via a single domain that hosts the assetlinks.json file.

If you have a single domain, can you verify ownership of two apps using two distinct certificates?

Yes, you can, it’s [documented](https://developer.android.com/training/app-links/verify-site-associations#multiple-apps).

Our case is a bit different. What if an app also has links configured for *uat.mobile.example.com* subdomain and both *debug* and *release* build types of the same app are pointed into it? Could I verify ownership upon a second subdomain using both *debug* and *release* certificates again?

The answer is: Yes, it can be easily achieved.

This is an assetlinks.json content you have to [upload](https://developer.android.com/training/app-links/verify-site-associations) to your subdomains.

{% gist xavarius/2a532bf2d187b5911d38d00fb3de7385 %}

In the example above, the Digital Asset Links JSON file is configured to support a particular build type

> “package_name”: “com.example.app.buildType1”

which should be verified using debug SHA-1 certificate

> “sha256_cert_fingerprints”: [“debugCertificateFingerprintHere”]

while the second build type has a “release” nature and uses a release SHA-1 certificate for ownership to be verified

> “package_name”: “com.example.app.buildType2”,

> “sha256_cert_fingerprints”: [“releaseCertificateFingerprintHere”]

Lemme explain a bit. The Digital Asset Links JSON file is a plain JSON file.

It contains a list (table) of elements as a top-level element.

All you have to do is to provide an entry for each of your build types with an appropriate SHA-1 fingerprint. And [deploy](https://developer.android.com/training/app-links/verify-site-associations) it in [each domain](https://developers.google.com/digital-asset-links/v1/create-statement#website_statement_file) you’re interested in. Voila!

This configuration is extremely useful if you love the flexibility and the opportunity of using any number of environments with any of your build types. It enhances the testability of an app and increases data diversity while easing debugging— all because you can choose which environment will be used while implementing and testing your application.

![Yeah, sharing is caring, remember that!]({{ '/assets/1_pqei5mOJjnLprLAotZuAZg.jpg' | relative_url }})
*Yeah, **sharing is caring**, remember that!*

Thank you, [Wojtek](https://www.linkedin.com/in/wdawiskiba/) for the peer review. It’s delightful to have such support. :)
