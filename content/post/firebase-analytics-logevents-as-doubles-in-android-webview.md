---
title: "Firebase LogEvents as doubles in a webview"
date: 2019-02-15T7:33:33+01:00
draft: false
tags: ["Firebase Analytics", "Android", "Javascript"]
---

Are you planning to use Firebase Analytics in a WebView on Android? Beware if you want to log doubles (price for example) as event parameters.

Google is sunsetting the Analytics Services SDKs coming fall and promote the transition to tracking your apps with the Firebase SDKs. Firebase Analytics is awesome (BigQuery Export, User and Event based, etc.). However, there is an issue I'm certain many analysts will face when implementing Firebase Analytics for Android. If your app, like MatHem's, at some point in the user journey switch over to a webview (ex. checkout) you need to log events using [javascript that calls a native javascript interface](https://github.com/firebase/analytics-webview/blob/ce453095e26b506665a45a8d4968cd258462945f/web/public/index.js#L71). The [reference code](https://github.com/firebase/analytics-webview/blob/ce453095e26b506665a45a8d4968cd258462945f/android/app/src/main/java/com/google/firebase/quickstart/analytics/webview/AnalyticsWebInterface.java#L76) for building the interface in android checks if the parameter value is string, int or double and stores it in the corresponding field in the firebase data model. 

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

However, javascript has a Number type and decimals don't show when a value is an integer. Hence, if you log the price of a product as a parameter, it will end up as intValue if the price is 5.00 and floatValue if the price is 5.01. This will really complicate analysis of that data. One possible way of solving it is to try to cast the value (if it is a String or Integer) and log it to multiple columns. Example:

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
