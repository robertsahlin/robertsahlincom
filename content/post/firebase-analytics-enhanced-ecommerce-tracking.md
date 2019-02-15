---
title: "Firebase Analytics e-commerce tracking"
date: 2019-02-15T7:50:33+01:00
draft: false
tags: ["Firebase Analytics", "Ecommerce"]
---

I recently implemented firebase analytics tracking in MatHem's native apps and in that process I discovered that the [documentation about tracking e-commerce](https://support.google.com/firebase/answer/6317499?hl=en) is very limited compared to how to implement it in Google Analytics. Hence, I thought I would share our implementation of Firebase Analytics.

First, Firebase Analytics has a completely different data model than Google Analytics. It is much more flexible and basically you log events with corresponding parameters or you log user properties. The current limits are:
* 500 distinct events per app (automatically collected events excluded)
* 25 event parameters per event
* 25 user properties per app

This is usually more than enough. However, due to the flexibility, it requires more careful consideration how to structure the implementation. This is what we ended up with (examples for android), I hope you find it useful:

#### Product impression
```java
//Loop through list of products
//start for-loop
Bundle productBundle = new Bundle();
productBundle.putString("product_id", "sku1234");
productBundle.putString("product_name", "Donut Scented T-Shirt");
productBundle.putString("product_category", "Apparel/Men/Shirts");
productBundle.putString("product_brand", "Google");
productBundle.putDouble("product_price", 29.99);
productBundle.putString("product_currency", "SEK");
productBundle.putString("product_action", "view");
productBundle.putString("product_action_list", "Search Results") //List name
productBundle.putLong( "product_position", 1 ); // Position in the list
productBundle.putDouble(Param.VALUE, 29.99);
productBundle.putString(Param.CURRENCY, "SEK");
mFirebaseAnalytics.logEvent("product_view", productBundle);
//end for-loop
```

#### Product click
```java
// Define product with relevant parameters
Bundle productBundle = new Bundle();
productBundle.putString("product_id", "sku1234");
productBundle.putString("product_name", "Donut Scented T-Shirt");
productBundle.putString("product_category", "Apparel/Men/Shirts");
productBundle.putString("product_brand", "Google");
productBundle.putDouble("product_price", 29.99);
productBundle.putString("product_currency", "SEK");
productBundle.putString("product_action", "click");
productBundle.putString("product_action_list", "Search Results") //List name
productBundle.putLong( "product_position", 1 ); // Position in the list
productBundle.putDouble(Param.VALUE, 29.99);
productBundle.putString(Param.CURRENCY, "SEK");
mFirebaseAnalytics.logEvent("product_click", productBundle);
```

#### Product detail view
```java
// Define product with relevant parameters
Bundle productBundle = new Bundle();
productBundle.putString("product_id", "sku1234");
productBundle.putString("product_name", "Donut Scented T-Shirt");
productBundle.putString("product_category", "Apparel/Men/Shirts");
productBundle.putString("product_brand", "Google");
productBundle.putDouble("product_price", 29.99);
productBundle.putString("product_currency", "SEK");
productBundle.putString("product_action", "detail");
productBundle.putDouble(Param.VALUE, 29.99);
productBundle.putString(Param.CURRENCY, "SEK");
mFirebaseAnalytics.logEvent("product_detail", productBundle);
```

#### Product add to cart
```java
// Define product with relevant parameters
Bundle productBundle = new Bundle();
productBundle.putString("cart_id", "1234-abcd");
productBundle.putString("product_id", "sku1234");
productBundle.putString("product_name", "Donut Scented T-Shirt");
productBundle.putString("product_category", "Apparel/Men/Shirts");
productBundle.putString("product_brand", "Google");
productBundle.putDouble("product_price", 29.99);
productBundle.putLong("product_quantity", 1);
productBundle.putString("product_currency", "SEK");
productBundle.putString("product_action", "add");
productBundle.putDouble(Param.VALUE, 29.99);
productBundle.putString(Param.CURRENCY, "SEK");
mFirebaseAnalytics.logEvent("product_add", productBundle);
```

#### Checkout
Since our apps switch over to a webview in the checkout step, the implementation is done with javascript and google tag manager and the tag triggers on a checkout event and read the same datalayer as when tracking checkout for google analytics (web client). But the event and parameters are the same if you implement it in your native code as in the events described above. Also, take a look at my [post about tracking doubles in webviews](https://robertsahlin.com/firebase-logevents-as-doubles-in-a-webview/).
```javascript
<script>
  
function logEvent(name, params) {
  if (!name) {
    return;
  }

  if (window.AnalyticsWebInterface) {
    // Call Android interface
    window.AnalyticsWebInterface.logEvent(name, JSON.stringify(params));
  } else if (window.webkit
      && window.webkit.messageHandlers
      && window.webkit.messageHandlers.firebase) {
    // Call iOS interface
    var message = {
      command: 'logEvent',
      name: name,
      parameters: params
    };
    window.webkit.messageHandlers.firebase.postMessage(message);
  }else {
    // No Android or iOS interface found
    console.log("No native AnalyticsWebInterface APIs found.");
  }
}

function setUserProperty(name, value) {
  if (!name || !value) {
    return;
  }

  if (window.AnalyticsWebInterface) {
    // Call Android interface
    window.AnalyticsWebInterface.setUserProperty(name, value);
  } else if (window.webkit
      && window.webkit.messageHandlers
      && window.webkit.messageHandlers.firebase) {
    // Call iOS interface
    var message = {
      command: 'setUserProperty',
      name: name,
      value: value
   };
    window.webkit.messageHandlers.firebase.postMessage(message);
  }else {
    // No Android or iOS interface found
    console.log("No native AnalyticsWebInterface APIs found.");
  }
}
  
//helper function to traverse nested javascript objects that may contain undefined properties  
function get(p,o){
    return p.reduce(function(xs, x){
       return (xs && xs[x]) ? xs[x] : null;}, o);
}

  
  if (window.AnalyticsWebInterface || (window.webkit && window.webkit.messageHandlers
      && window.webkit.messageHandlers.firebase)) {
try {
    var params = {};
    params.checkout_step = get(['checkout', 'actionField', 'step'], {{dl ecommerce}});
    params.checkout_option = get(['checkout', 'actionField', 'option'], {{dl ecommerce}});
    logEvent('checkout_progress', params);
  } catch(e) {console.log(e);}

}else {
    // No Android or iOS interface found
    console.log("No native AnalyticsWebInterface APIs found.");
  }
</script>
```

#### Purchase
The same setup as for checkout, implemented with tag manager but triggered on a purchase event this time. One event for the transaction and one event for each product in the transaction.
```javascript
<script>

function logEvent(name, params) {
  if (!name) {
    return;
  }

  if (window.AnalyticsWebInterface) {
    // Call Android interface
    window.AnalyticsWebInterface.logEvent(name, JSON.stringify(params));
  } else if (window.webkit
      && window.webkit.messageHandlers
      && window.webkit.messageHandlers.firebase) {
    // Call iOS interface
    var message = {
      command: 'logEvent',
      name: name,
      parameters: params
    };
    window.webkit.messageHandlers.firebase.postMessage(message);
  }else {
    // No Android or iOS interface found
    console.log("No native AnalyticsWebInterface APIs found.");
  }
}

function setUserProperty(name, value) {
  if (!name || !value) {
    return;
  }

  if (window.AnalyticsWebInterface) {
    // Call Android interface
    window.AnalyticsWebInterface.setUserProperty(name, value);
  } else if (window.webkit
      && window.webkit.messageHandlers
      && window.webkit.messageHandlers.firebase) {
    // Call iOS interface
    var message = {
      command: 'setUserProperty',
      name: name,
      value: value
   };
    window.webkit.messageHandlers.firebase.postMessage(message);
  }else {
    // No Android or iOS interface found
    console.log("No native AnalyticsWebInterface APIs found.");
  }
}
  
//helper function to traverse nested javascript objects that may contain undefined properties    
function get(p,o){
    return p.reduce(function(xs, x){
       return (xs && xs[x]) ? xs[x] : null;}, o);
}

  if (window.AnalyticsWebInterface || (window.webkit && window.webkit.messageHandlers
      && window.webkit.messageHandlers.firebase)) {

  try {
    var params = {};
    params.transaction_id = get(['purchase', 'actionField', 'id'], {{dl ecommerce}});
    params.transaction_revenue = get(['purchase', 'actionField', 'revenue'], {{dl ecommerce}});
    params.transaction_tax = get(['purchase', 'actionField', 'tax'], {{dl ecommerce}});
    params.transaction_shipping = get(['purchase', 'actionField', 'shipping'], {{dl ecommerce}});
    params.transaction_coupon_code = get(['purchase', 'actionField', 'coupon'], {{dl ecommerce}});
    params.transaction_currency = 'SEK';
    params.value = get(['purchase', 'actionField', 'revenue'], {{dl ecommerce}});
    params.currency = 'SEK';
    logEvent('ecommerce_purchase', params);
  } catch(e) {console.log(e);}

try {
    {{dl ecommerce}}.purchase.products.forEach(function(prod) {
        var params = {};
        params.transaction_id = get(['purchase', 'actionField', 'id'], {{dl ecommerce}});
        params.product_id = get(['id'], prod);
        params.product_name = get(['name'], prod);;
        params.product_category = get(['category'], prod);;
        params.product_brand = get(['brand'], prod);;
        params.product_price = get(['price'], prod);;
        params.product_quantity = get(['quantity'], prod);;
        params.product_action = get(['action'], prod);;
        params.product_currency = 'SEK';
        params.value = get(['price'], prod);;
        params.currency = 'SEK';
        logEvent('product_purchase', params);
    });
  } catch(e) {console.log(e);}

}else {
    // No Android or iOS interface found
    console.log("No native AnalyticsWebInterface APIs found.");
  }
</script>
```

#### Promotion impressions
```java
// Define promotion with relevant parameters
Bundle promotionBundle = new Bundle();
promotionBundle.putString("promotion_id", "promo_1234");
promotionBundle.putString("promotion_name", "summer sale");
promotionBundle.putString("promotion_creative", "summer_banner");
promotionBundle.putString("promotion_position", "banner_slot1");
promotionBundle.putString("promotion_action", "view");
mFirebaseAnalytics.logEvent("promotion_view", promotionBundle);
```

#### Promotion click
```java
// Define promotion with relevant parameters
Bundle promotionBundle = new Bundle();
promotionBundle.putString("promotion_id", "promo_1234");
promotionBundle.putString("promotion_name", "summer sale");
promotionBundle.putString("promotion_creative", "summer_banner");
promotionBundle.putString("promotion_position", "banner_slot1");
promotionBundle.putString("promotion_action", "click");
mFirebaseAnalytics.logEvent("promotion_click", promotionBundle);
```
