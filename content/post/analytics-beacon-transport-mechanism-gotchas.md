---
title: "Analytics beacon transport mechanism gotchas"
date: 2019-02-18T7:50:33+01:00
draft: false
tags: ["DataHem", "Tracker", "Google Analytics", "Javascript"]
---

An important component in DataHem's measurementprotocol application is the javascript tracker that emitts hits to the AppEngine collector endpoint. It is important that the tracker emitts all defined hits, if the tracker malfunction then there is no way to recover dropped hits downstream.

Like Google Analytics, the DataHem tracker supports three different transport mechanisms; 'image' (using an Image object), 'xhr' (using an XMLHttpRequest object), or 'beacon' using the new navigator.sendBeacon method. The former two methods share the problem where hits are often not sent if the page is being unloaded. The navigator.sendBeacon method, by contrast, is a new HTML feature created to solve this problem. Hence, I wanted to use the beacon method as default and downgrade gracefully. 

```javascript
function(){
  if (navigator.sendBeacon) {
	    return 'beacon'; 
  }
  else if (typeof new XMLHttpRequest().responseType === 'string') {
    return 'xhr';
  } 
  else {
    return 'image';
  }
}
```

However, when performing a QA-check of the beacon method I noticed that 5-10% less hits were reported compared to Google Analytics using the image method. 

The difference initialized a closer investigation comparing individual hits in Google Analytics and DataHem. The majority of dropped hits in DataHem stemmed from:

1. Samsung browser: ~40%
2. Safari in-app (i.e facebook, instagram and google search app): ~25% 
3. Chrome on iOS (iPad & iPhone): ~20%
4. Edge on windows: ~10%

The remaining hits were in most cases due to differences in IP- and bot-filters.

Apparently som browsers pass the navigator.sendBeacon test but never emitts the hit to the collector. Also, I found that there is a [bug in webkit](https://bugs.webkit.org/show_bug.cgi?id=193508) where sendBeacon to a previously-unvisited HTTPS domain would always fail (third party browser running on iPhone or iPad). There is a fix in release 75 of [safari technology preview](https://developer.apple.com/safari/technology-preview/release-notes/). To mitigate these sources of errors I modified the script to check if the user agent match Samsung browser, common in-app browsers, Chrome on iOS and Edge. Also, even if the browser pass that test, the first hit is always sent as a xhr to ensure that sendBeacon will never send hits to an unvisited https domain.

```javascript
function(){
  var userAgent = window.navigator.userAgent.toLowerCase(),
  downgrade = /samsungbrowser|crios|edge|gsa|instagram|fban|fbios/.test( userAgent );
  if (navigator.sendBeacon && sessionStorage.getItem('beaconPreflight') && !downgrade) {
	  return 'beacon'; 
	}
	else if (typeof new XMLHttpRequest().responseType === 'string' && window.sessionStorage) {
    sessionStorage.setItem('beaconPreflight', true);
    return 'xhr';
	} 
	else {
	  return 'image';
	}
}
```

After that, the difference between DataHems tracker and the Google Analytics tracker (using image) is less than 0,26 percent on our [website](https://www.mathem.se)! 

The script above can be used together with Google Analytics as well, that is what I don on this blog. You just copy the javascript function above and paste it in a google tag manager variable (Javascript function) and save it to something like "transport method". Then in your GA Settings variable you add a field with the name "transport" and give it the value of the tracker variable reference, i.e. {{transport method}}.

The xhr method differed a lot less than beacon, in general it reported a difference that were less than 1% compared to the image method.
