---
tags:
- rant
- considered harmful
- blog
- development
---

This post has been a long time coming. In 2021, through the pursuit of [[indieweb]] services, I switched to [[Drew DeVault]]'s [[Sourcehut]] platform instead of [[GitHub]]. I opted for a progressive style of migration, meaning new projects will be hosted at sourcehut. Some issues from the switch were evident right away, but many took time. Since my migration, I've endured [[growing pains]] around each corner.

The funny thing is that these problems are not at all related to sourcehut, but rather people's dependence on the service. Experiencing these issues has been (ironically) liberating, as it has made me increasingly more and more aware of the jail the developer community (myself included) has built for themselves. Freeing myself of these restrictions feels natural, just as [[rooting]] an [[android]] phone or migrating to [[linux]] has provided so many with the satisfaction of knowing they have gained some form of digital autonomy.

Detailed here are some such examples of issues I have experienced as a result of github's propitiatory architecture, or integrations people have built using it.

## How do deploy my website?

Up until this point, I had been using [[netlify]] to deploy my website. When switching to sourcehut, I went to create a new netlify page, because I could not figure out how to switch providers from an existing page. To my dismay, I discovered this would not work.

![Where is my option for custom URLS?](https://i.boehs.org/raw/50dh6f7y.png)

But *why*? It's standard across all git providers to allow cloning by appending `.git` to a repository. Even for private repositories, giving netlify a [[SSH]] key should be more than enough? What is there to lose by allowing arbitrary URI's?

Well, I suppose there is the touchy issue of automatic deployments. Presumably netlify sets up a webhook with github, gitlab, and bitbucket. These webhooks work because netlify developers have hand coded superpowers. Presumably these superpowers are not hard coded, that would be silly when you offer integrations for three services. [Sourcehut offers webhooks too](https://man.sr.ht/graphql.md#webhooks), I don't expect special handling for my niche service, but I would be down to hand code that handling for automatic deployments, provided 

Even if there was no webhooks, that's a pretty self imposed bottleneck, I see no reason that *manual* deployments would not work with this `.git` fallback approach.

[Colby Hubscher](https://colbyhub.com/) wrote an article on how to get this working using sourcehut's build service [here](https://colbyhub.com/deploy-to-netlify-with-sourcehut/), but It's too late, I can't think of a good reason this would not be offered in netlify.

### Vercel

In the [[jamstack]] community, the other netlify is [[vercel]], perhaps it fairs better.

![Spoiler: Yes](https://i.boehs.org/raw/fs6fxrjt.png)

Was that so hard netlify?

Wait...

![Huh?](https://i.boehs.org/raw/b1fzf154.png)

Huhhhhh?

### Cloudflare

Same story as netlify, but only github and gitlab

### The rest

- Surge works, that's kinda the point of surge though
- Fly.io works like surge

### Conclusion

There is nothing difficult to providing platform emancipation due to git fundamentals and the prevalence of webhooks in modern hosts, but for some reason every major service opposes it. Why?