---
layout: post
title: "Were the Russian Tweets Really Russian?"
author: "Joshua Eckroth"
---

On Feb 14, 2018, NBC News released a [collection of 200k deleted tweets](https://www.nbcnews.com/tech/social-media/now-available-more-200-000-deleted-russian-troll-tweets-n844731) that it claims are produced by Russian "trolls" intending to influence or "hack" the 2016 US presidential election. Data Hunters wants to know, **were these tweets really produced by Russians?**

We'll conduct an analysis across several dimensions. For every experiment listed below, if the experimental question is answered in the positive, then we have evidence the tweets are Russian.

The tweets are likely Russian and intended to "hack the election" if the following are true:

- Newer accounts have an unexpectedly high number of followers, i.e., the followers were purchased. See section, "Age of Accounts vs. Followers"

Note, we do not have a comparison set of typical tweets. We cannot compare the supposed Russian tweets with normal tweets. Instead, we must imagine what normal behavior looks like and run statistical tests against these assumptions.

## Date of Account Creation

*Hypothesis*: If the tweets are Russian, we should see new accounts created in batches for the purposes of influencing the election. If they tweets are not Russian (i.e., random people across the world with an interest in politics), the accounts should be created uniformly throughout the period of time represented in the dataset.

*Method*: We can use R's `uniform.test` to check if the date of account creation follows a uniform distribution (i.e., accounts are not created in batches).

*Result*: We have a p-value <0.01 so we have reason to believe the accounts were created in batches, possibly for the purpose of influencing the election.

(plot)

**Evidence for Russian tweets? ✓**

This analysis was conducted by Trâm Nguyễn and Andrew Tompkins.

In further analysis, Andrew Tompkins found key dates and events when accounts were created.

(put stuff here)

## Age of Account vs. Followers

*Hypothesis*: Older accounts should have more followers, everything else being equal. If new accounts have lots of followers, perhaps those followers were purchased.

*Method*: Compute the correlation of age of account and number of followers. Run R's `cor.test` to test if we can reject the null hypothesis, which states the correlation is zero. If the correlation is negative, we see normal (expected) behavior: older accounts have more followers. If the correlation is positive, we see unexpected behavior: newer accounts have more followers. If there is no correlation, this is also evidence against normal behavior.

*Result*: The correlation between account creation date and number of followers is small: about 0.05. Naturally, the `cor.test` indicates the null hypothesis cannot be rejected, so we have insufficient evidence that there is any correlation.

Because the correlation is likely zero, we have abnormal behavior. Normally, older accounts should have more followers, everything else being equal. Thus, we have evidence that the tweets are Russian.

(plot)

**Evidence for Russian tweets? ✓**

This analysis was conducted by Sam Pratico and Dearvis Troutman.

## Tweet Day of Week

*Hypothesis*: If these tweets are produced by some kind of Russian "[troll farm](http://www.bbc.com/news/technology-43093390)," then perhaps the tweets happen at unusual days in the week. For example, if a normal Twitter user tweets virtually the same number of times all days of the week, but the Russian tweets more often occur on a Sunday than any other day, we have evidence that the Russian tweets are unusual, and therefore possibly intended to influence the election.

*Method*: In order to compare the Russian tweet days with "regular" tweet days, we need some kind of comparsion set. An easy comparison set to obtain is [President Trump's tweets](https://github.com/mkearney/trumptweets). We can also visually compare graphs of the Russian tweets by day and [HubSpot's graph from five million accounts](https://blog.hubspot.com/blog/tabid/6307/bid/5500/When-Do-Most-People-Tweet-At-the-End-of-the-Week.aspx).

**Evidence for Russian tweets? ✗**

This analysis was conducted by Brandon Belna.

## Summary

- **✓** - Date of Account Creation
- **✓** - Age of Account vs. Followers
- **✗** - Tweet Day of Week


