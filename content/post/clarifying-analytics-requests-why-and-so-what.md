---
title: "Clarifying Analytics Requests = 5 x Why + So What"
date: 2018-01-23T16:36:15+01:00
draft: false
tags: ["Method", "Analytics"]
id: "2"
---

Most analysts want to avoid a situation where they spend a lot of time working on ad-hoc, vague and misguided analytics/data requests instead of focusing on hypothesis testing. But hey, you will not be able to avoid those requests entirely, so what can you do in order to clarify the requests you can't hold at bay? I use two simple methods to clarify analytics requests:

# 5 x why?

[5 Whys](https://en.wikipedia.org/wiki/5_Whys?lipi=urn%3Ali%3Apage%3Ad_flagship3_pulse_read%3B9ooMqnfoT3KI6bkNmvRHFw%3D%3D) is an iterative interrogative technique used to explore the cause-and-effect relationships underlying a particular problem and I've found that it also works really well to clarify analytics requests. You basically ask the requester why he/she wants the data/analysis and when you get the answer you just repeat the question five times (or as many as needed to nail what the requester really need - not wants).

*Example: The content manager of the company site comes up to you and asks for the average conversion rate and the conversion rate for the customers which browse videos on your website.*

## 1. Why?

- I want to know to know if users which browse video make purchases to a greater extent than the average user.

## 2. Why?

- I'm preparing a business case and need som data to support that.

## 3. Why?

- Well, I want to produce more high quality product videos for our website and it is a quite big investment but I think it will have positive ROI.

## 4. Why?

- I think that our website users appreciate high quality product videos and that will result in more sales.

## 5. Why?

- It is a relatively common user feedback from our web surveys that our users want more product videos on our website and we think it is an important component in the customers purchase decision.

## Clarified
Ok, now the requester has formulated a hypothesis: *"more high quality product videos will lift sales (i.e. conversion rate)"*. Since users which browse videos are more likely to convert than the average user to start with (selection effect), giving the conversion rates as originally requested would have misguided the business case. You'll have to A/B-test this in order to find out the impact of product videos on conversion rate. One way of doing that is simply to hide existing product videos from a share of users (treatment) and show it for the rest (control) and compare the conversion rates if/when the results are significant. An ad-hoc, vague and misguided analytics/data request turned into hypothesis testing.

# So what?
Avinash Kaushik has a [post about the "so what" test](http://www.kaushik.net/avinash/kill-useless-web-metrics-apply-so-what-test/?lipi=urn%3Ali%3Apage%3Ad_flagship3_pulse_read%3B9ooMqnfoT3KI6bkNmvRHFw%3D%3D). This is to test if the analytics request is actionable. If the requester can't take any action, based on your analysis, why even bother making the analysis? If the five whys didn't reveal possible actions, then a simple way forward is to give the requester your two best guesses (a positive and a negative) on what the outcome from the analysis will be. Then ask the requester what he/she would do different given your two guesses. If the action is the same or none at all, then ask yourself if there is any point doing the analysis.

I encourage you to share your own methods to avoid or clarify ad-hoc and vague analytics requests.
