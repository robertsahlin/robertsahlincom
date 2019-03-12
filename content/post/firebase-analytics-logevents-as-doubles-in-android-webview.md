---
title: "Firebase LogEvents as doubles in a webview"
date: 2019-02-15T7:33:33+01:00
draft: false
tags: ["Firebase Analytics", "Android", "Javascript"]
---

Are you planning to use Firebase Analytics in a WebView on Android or iOS? Beware if you want to log doubles (price for example) as event parameters.

Google is sunsetting the Analytics Services SDKs coming fall and promote the transition to tracking your apps with the Firebase SDKs. Firebase Analytics is awesome (BigQuery Export, User and Event based, etc.). However, there is an issue I'm certain many analysts will face when implementing Firebase Analytics in a WebView. If your app, like MatHem's, at some point in the user journey switch over to a webview (ex. checkout) you need to log events using [javascript that calls a native javascript interface](https://github.com/firebase/analytics-webview/blob/ce453095e26b506665a45a8d4968cd258462945f/web/public/index.js#L71). The data model in Firebase Analytics is based on events that has parameters and each parameter has a key and a value-object. The value object has three different value fields: String, Integer and Double.

```
{"event": "temperature_reading",
parameters : [
  {"key": "degrees",
  "value": {
    "string_val":"five",
    "int_val": 5,
    "double_val": 5.0
  }
]
```

However, javascript has a Number type and decimals don't show when a value is an integer. Hence, if you log the price of a product as a parameter, it will be represented as an integer if the price is 5.00 and double if the price is 5.01. This will really complicate analysis of that data since the data will end up in different columns in BigQuery due to how it is logged by Firebase Analytics for Android and iOS. 

### Android
The [reference code](https://github.com/firebase/analytics-webview/blob/ce453095e26b506665a45a8d4968cd258462945f/android/app/src/main/java/com/google/firebase/quickstart/analytics/webview/AnalyticsWebInterface.java#L76) for building the interface in android checks if the parameter value is string, int or double and stores it in the corresponding field in the firebase data model. 

```java
Object value = jsonObject.get(key);

   if (value instanceof String) {
       result.putString(key, (String) value);
   } else if (value instanceof Integer) {
       result.putInt(key, (Integer) value)      
   } else if (value instanceof Double) {
       result.putDouble(key, (Double) value);
   } else {
       Log.w(TAG, "Value for key " + key + " not one of [String, Integer, Double]");
   }
```
As you can see, an integer value is put as an Integer while a double is put as a double. 

One possible solution it is to try to cast the value (if it is a String or Integer) and log it to multiple columns. Example:

```java
if (value instanceof String) {
            try{
                result.putString(key, (String) value);
                result.putDouble(key, Double.parseDouble((String) value));
                result.putInt(key, Integer.parseInt((String) value));
            } catch(Exception e){}
        }
        if (value instanceof Integer) {
            try{
                result.putInt(key, (Integer) value);
                result.putDouble(key, Double.valueOf((Integer) value));
            }catch(Exception e){}
        }
        if (value instanceof Double){
            result.putDouble(key, (Double) value);
        }
```

However, this is not the best solution due to how the iOS logging works.

### iOS
As you can see in [the reference code example for iOS](https://github.com/firebase/analytics-webview/blob/ce453095e26b506665a45a8d4968cd258462945f/ios/swift/FirebaseAnalyticsWeb/ViewController.swift#L57), the parameters passed from the WebView are logged as [String : NSObject] and hence iOS logEvent function takes care of mapping the number to int or double. Hence, you don't control whether the number should be stored as int or double.

```swift
// [START handle_messages]
  func userContentController(_ userContentController: WKUserContentController,
                           didReceive message: WKScriptMessage) {
    guard let body = message.body as? [String: Any] else { return }
    guard let command = body["command"] as? String else { return }
    guard let name = body["name"] as? String else { return }

    if command == "setUserProperty" {
      guard let value = body["value"] as? String else { return }
      Analytics.setUserProperty(value, forName: name)
    } else if command == "logEvent" {
      guard let params = body["parameters"] as? [String: NSObject] else { return } //<-- String : Any !
      Analytics.logEvent(name, parameters: params)
    }
  }
  // [END handle_messages]
```

Of course we could parse through the params variable and map parameter keys with what type we want to store it as, but there is a simpler solution.

### Store all numbers as doubles
We decided to log all numbers as doubles to make sure that analysts don't have to understand when numbers may end up in different fields. Hence, we parse through all numbers and cast them to doubles if they are integers.
