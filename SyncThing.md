---
tags:
- showcase
in: blog
---

Similar to my issues with [[Obsidian Publish]], I was not about to subscribe to 4$ a month to send markdown files from my phone to my computer. Obsidian's help docs suggest two mobile apps, but both synced to a server rather than device to device. Enter: SyncThing

## Syncing text files

Setup was difficult (I had to manually enter a 30 or something char long pairing code), and I found the app confusing at first. Some direction would be useful (save for the incomplete docs that I needed to Google for), but used my critical thinking skills and caught on quick. The majority of the systems make sense once you learn them, and there are quite a few goodies I did not feel the need to take advantage of but could see the usecase of (such as ignore lists and send or receive *only*).

Once I had my devices paired, I added my vault folder to SyncThing and checked the device(es) I wanted to sync with. I was relieved to discover I simply needed to consent on the other device and file syncing began perfectly.

When I say perfectly I almost mean perfectly (that's a compliment!). I configured SyncThing to "watch" the folder for changes, and it perfectly and instantly picked up on every change and synced it every time. The only annoyance I had was how it handled an error syncing a file. I made a markdown file with a `?` mark in its name, and was informed of a syncing error and directed to the console to "look for clues". To SyncThing's credit, no tracebacks or anything, just a clean log entry informing me windows does not support question marks in file names: but I would have much rathered that information be presented in app

## Syncing music

Next I synced my mp3 library between mobile and PC again. Setup went just as smoothly as in the second paragraph. Naturally, mp3 file size holds no candle to that of markdown files. I had previously manually copied the music over, But at a later date did the following

- Modified the metadata on my computer
- Downloaded a few additional songs

I was delighted to discover that not only did it perfectly bring these devices in sync, It did not "fully" sync the files that I updated the metadata of. Instead, it appeared only a portion of each file was synced, reducing my bandwidth usage! You can learn more [here](https://docs.syncthing.net/users/syncing.html) in the docs, but the relevant portion is quoted below:

> Blocks making up a file are all the same size (except the last one in the file which may be smaller). The block size is dependent on the file size and varies from 128 KiB up to 16 MiB. Each file is sliced into a number of these blocks, and the SHA256 hash of each block is computed. This results in a block list containing the offset, size and hash of all blocks in the file.
   To update a file, Syncthing compares the block list of the current version of the file to the block list of the desired version of the file. It then tries to find a source for each block that differs. This might be locally, if another file already has a block with the same hash, or it may be from another device in the cluster

Cool!

## Ebooks

I was able to make a small script with the guidance of [NiLuJe](https://www.mobileread.com/forums/member.php?u=69624) [on a post I found](https://www.mobileread.com/forums/showthread.php?t=330426) to sync my E-reader and phone after side loading [[KoReader]] onto my [[Kobo]]. Currently the script is blocking so I run it only when I want to update the device (a good practice anyway, wifi is a big battery hog!)

## A scare

As I wrote this post, I encountered a merge conflict. Somehow, I picked the wrong version and boom, 3570 characters gone. Luckily, Obsidian's <kbd>Ctrl</kbd>+<kbd>Z</kbd> worked. From then on I enabled trashcan versioning on all my devices. That should be default!

## Conclusion

Yeah, this is great, a quest to sync my writing resulted in finding the solution for all my synchronization needs. I suggest if you need to interface with the same offline data on multiple devices you give SyncThing a try and stick with it even if setup is frustrating - you will thank yourself later.