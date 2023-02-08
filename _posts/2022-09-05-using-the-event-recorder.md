---
title: Using the Event Recorder 
date: 2022-09-05 12:00:00 -500
categories: [Tips & Tricks] 
tags: [lsretail, lscentral, eventrecorder, events]
---

The way I came to use the event recorder in Business Central is like this:

1. I would open the Event Recorder page and start recording
2. I would use the search bar to navigate to the page whose events I want to record
3. I would perform the task needed, then colose the page
4. Upon landing back to the event recorder page, I would stop recording.
5. I examine the events log.

## The problem

This was well and all, until I needed to record events from LS Central (previously known as LS Retail). What happens here is that LS Central will close every opened page upon returning to Business Central.

So, I had to look for other ways to use the event recording, since as you probably know, if I open the event recorder in another tab in the browser, it is considered another session, so the events are not recorded.

## The solution

In the documentation, Microsoft provides the solution, and if, like me, you haven't noticed, I comment on it below:

<https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/developer/devenv-events-discoverability#how-to-record-events>

>### How to record events
>
>The following steps describe how to record events when you are in the Event Recorder page.
>
>1. On the Event Recorder page, in the upper right corner, choose Open this page in a new window.
>2. Then on the Event Recorder page, choose Record Events, and then choose the Start button.
>3. In the original window, perform all the actions that you want to record while the event recorder session is on. For example, post a purchase order.
>4. After you have performed the actions of your scenario, navigate back to the window that has the Event Recorder page and click the Stop button to finnish recording.
>
>    All the events raised while performing the actions of your scenario are recorded and can be viewed on as shown below.
>
>5. Choose Get AL Snippet to get the event subscription code in AL. You can use the AL snippet code in Codeunits to subscribe to those events.

## My take

This makes sense since when you dettach the event recorder page, you are still in the same session.

This was useful to me, and to make sure I remember it in the future, I'm making this note for my future self.