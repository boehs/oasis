# Domain, Raspberry Pi, 512gb SSD

```toc
    style: bullet | number (default: bullet)
    min_depth: number (default: 2)
    max_depth: number (default: 6)
```

## For me

### [[SyncThing]]

I already explained why I love syncthing, but one limitation is both devices must be online to sync, so this often happens:

![[Pasted image 20210925175359.png]]
![[Pasted image 20210925175325.png]]
<sup>sexy art I know</sup>

Having a server means the phone can sync to that server, and then when the computer goes online and the phone offline the computer can still sync.

## For friends and me

### Email forwarding 

Ditching immature email with zero compromises

```ad-example
* cool_dude420@gmail.com -> evan@boehs.io -> email@example.com
* cool_dude420@gmail.com <- evan@boehs.io <- email@example.com
```

Existing emails delivered to `cool_dude420@gmail.com`, but new contacts get my best face (`evan@boehs.io`). It forwards emails to `cool_dude420@gmail.com` without telling the sender, and vice versa

### Hosting projects (many low traffic node, rust, and python servers)

Host backends I write independent of my computer, so they are available when ever me or a friend needs them. Nothing high traffic in this section, but involves some MySQL and such. Mostly node, some rust.

## For the world, me, and my friends

### Email aliases

Problem: When signing up for services, they have your email now and can do as they wish

Solution: 
```ad-example
netflix -> netflix@alias.boehs.io -> cool_dude420@gmail.com
```

That way, if a service goes rogue

```ad-example
netflix -> netflix@alias.boehs.io 🚫 cool_dude420@gmail.com
```

### Photo hosting ([[Lychee]])

My photos are a large part of me, and I would like to have public galleries to share with not only my photography teacher, but anyone I give the link to. 

The software I use has password protected galleries support, so I can have fully public ones, and a special gallery for class, for my family, and for me.

A wonderful replacement for Google #photos, because it is no longer free

### Drive ([[NextCloud]])

I want to host a small file server that allows me to have public, unlisted, and password protected areas. In most cases, Google drive is appropriate, but occasionally drive.boehs.io/coolfile.png would be nice, to make it clear that information is by *me*. I really hate NextCloud, but it's the best option.

### Website Hosting ([[Why|Digital Garden]])

Serving static website that has my notes ([[Trinket]]).

## Security

Because I am hosting this shit from my own wifi, security is important. 

### [[CloudFlare]]

Can't get better than this, free and hides my IP from scary DoS and Doxx

### Some tips

- https://makezine.com/2017/09/07/secure-your-raspberry-pi-against-attackers/ *
- https://blog.robertelder.org/securing-a-raspberry-pi/
- https://raspberrytips.com/security-tips-raspberry-pi/
- https://pimylifeup.com/raspberry-pi-security/
- https://gist.github.com/boseji/c9e91ff3bd0b3cfb62a5e260fe505374 *