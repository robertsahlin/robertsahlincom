---
title: "Google Analytics Campaign Timeout Settings"
date: 2019-03-05T16:00:33+01:00
draft: false
tags: ["Google Analytics", "Settings"]
---

Pretty much every Google Analytics configuration I've seen have left the default Campaign Timeout Settings untouched. Also, it came to my knowledge that the tutor/s (representing a renowned digital analytics consultancy and Google Analytics partner) at the biggest digital analytics post-secondary education in Sweden instruct the students to keep the default settings. Hence, I feel obliged to write a post to clarify why you shouldn't.

### Keeping default campaign settings is just plain wrong and will mess up your traffic data in all reports except attribution model comparison tool!

And I will show you how. Campaign timeout settings tells Google Analytics for how long it should attribute sessions to the last non-direct click. The default settings is 6 months! Hence, all direct traffic after a non-direct click (paid, organic search, campaign, etc.) will be categorized as the last non-direct traffic source for 6 months. If your site has returning users (hopefully it has) this will cause a problem whenever you add exclusion filters or try to understand behaviour where you include traffic dimensions in the report/analysis.

Changing the campaign timeout settings can have a huge impact on standard reports as you can see in the screenshot below where I compare default 6 months against 1 day.

![Campaign timeout settings impact on standard reports](/images/standard-report.png)

A similar comparison in the attribution model comparison report shows nearly no change using a Last Non-Direct Click model (different periods are compared, else it would result in the same metrics).

![Campaign timeout settings impact on attribution model comparison reports](/images/attribution-comparison-report.png)

Are you convinced that you should change the default settings to something much shorter than default 6 months? So, what timeout should you set? I usually set it to 1 day, but theoretically you should set it as short as possible without generating new sessions when it's just a long session, because a new session is created when the traffic source changes (if campaign timeout settings is shorter than the session). As long as campaign timeout settings is greater than the longest session you can see in your stats you should be ok (campaign timeout > MAX\[session duration\]). But if you set it to 1 day you should be safe since a new session is started at midnight every day.

### I usually set campaign timeout settings to 1 day since a new session is started at midnight every day anyway

The weird thing is that the campaign timeout settings is most likely a legacy from a time where cookies were used to set up an attribution model (last non-direct click) that was a bit more sophisticated than last click. But times have changed and the possibilities to analyze attribution today are near to limitless and not dependent on a last non-direct cookie. Hence, we have a setting that doesn't affect what it was invented for but only messing up all other reports. I guess you could speculate why it still exist and whatever reason you come up with it is still a fact that players (Google is one of them) that own a lot of traffic on the internet will benefit from exaggerated attribution caused by the default settings.
