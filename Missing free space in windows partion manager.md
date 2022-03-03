---
tags:
- operating systems
- windows
- storage
- faq
- stackoverflow
repost: https://superuser.com/questions/1403035/why-is-the-size-of-available-shrink-space-is-only-13286-mb-on-250-gb-samsung-ssd
---
# Question

Help! I have a drive with 100gb free and I can only reduce it by about 10gb in the partion manager!


![See Here](https://i.stack.imgur.com/tEX4Z.png)

# Answer

> [[PenPen]]'s Note: Evan was initially skeptical that the pagefile could be the cause of this issue, it was only 16 gigabytes on his system - but after disabling it the issue went away.

The main reason for not being able to shrink the disk are that there are unmovable files on the disk at the time of trying to shrink the volume (as your screenshot says).

The most common "unmoveable" files are files which are locked during normal computer operation such as virtual memory/pagefile/system restore files as well as a few other files which may be open, but not running "in memory"

Having come across this myself previously on both server and desktop operating systems - I can say the most likely culprit is the pagefile.

To fix this:

```
Right-click Computer
Select Properties
Select Advanced system settings
Select the Advanced tab and then the Performance radio button
Select the Change box under Virtual memory
Un-check Automatically manage paging file size for all drives
Select No paging file, and click the Set button
Select OK to allow and restart.
```

Once your machine has rebooted and you know you have no page file (check at the root of C: with hidden and system files showing) - try a defrag and then try shrinking the volume again.

Don't forget to reset your pagefile back to its original size afterwards! Failure to do so will potentially cause significant performance issues with any machine.

Hope this helps

-> https://superuser.com/questions/1403035/why-is-the-size-of-available-shrink-space-is-only-13286-mb-on-250-gb-samsung-ssd

---

[[stackoverflow]]