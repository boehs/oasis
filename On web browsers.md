I have come to the conclusion that web browsers are flawed like many, though to what extent I differ. **I believe the issue with [[productivity]] [[online]] is that with literally one button and zero thought you can go in any direction**

# Proposed Solution

> [[PenPen]]'s Note: This is a user story, not a spec

> [[PenPen]]'s Note: [[october]] is a working name

When installing october, the first app is made. This app is `google.com`.

When leaving google.com, a new window is opened. It will ask us to give this collection of windows a name, let's call it `months`. We now have two windows with two very specific states.

> [[Sidetrack]]: Keep in mind that the first window is always google, this is the app named october. You could delete the october app, really, it's just a pointer. Replace it with duckduckgo, whatever.

Go back to the google window, this time you are prompted. You can merge it into the window you just opened, or open it in a new window. I think you get the window idea, so let's open it in the window we just opened. The months window is quickly becoming an app! Let's continue adding pages to months, one for each month.

![[Pasted image 20211215003642.png]]

Now, let's discard the google window. It's ok, it can come back, reopen a new instance of october, and note all the windows are self contained, but this begs the question: we have something quite good going on with our months app, what do we do now? Glad you asked.

We have three options, saving the session, saving the collection, or minting the collection.

* Saving the session allows us to reopen the window exactly as it was
* Saving the collection allows us to open the urls exactly as they are at the given moment, in this case 12 urls all of months
* Minting the collection saves it as an app, in this case the months app

---

Let's step away from our months app. Magically, a google drive app appears. The google drive app contains google drive, google sheets, google slides, and google docs. Very poggers.

Magically, a window with a link to google docs opens! How cool. Let's click that link..... And, that was underwhelming. October asks you what you want to do with it again! Let's open the settings by pressing `Ctrl+:`, and going to the handlers section. Let's create a basic handler. we have two sections.

```
For: <>
Do: <>
```

by default, urls are opened as `october://go/<url>`, let's override this.

The `for` section is easy, set it to be `docs.google.com/document/*`

The `do` bit is harder. Let's start with `october://app/<app>/<page>`

> A page is an item in a collection.

We need to replace \<app> and \<page> with something. October uses id's for this, handy thing's that look like `ঠ⍉ᯙ♀भᏃ` (no that's not corruption). Let's open the hamburger menu where we found settings, click the id symbol right next to settings, it's kinda hidden! replace \<app> with what was copied to your clipboard.

Now, let's right click the page we are on in the sidebar, copy that id. Replace that with the page. in this case, this was all that was needed for october to figure it out. It's smart! And cool! If you need something more complicated, right click the page, go to advanced, click on protocol script. You can write scripts that run whenever that page get's launched!


---

Sometimes tabs are relevant, unfortunately. Right click on a page, click on `Enable tabs`. By default you can do whatever with tabs, but we want it to be more locked down. You can write scripts for this using right click page > advanced > tab script. Use it to restrict what a given page can be used for!

---

# Vision

Eventually, you will cultivate a network of apps. One for word processing, one for news, one for all the git sources, etc. Hopefully, if you are distractable like me, this can help you stay on task. A good october configuration does this.