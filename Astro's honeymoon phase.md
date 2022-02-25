I have a connection with [[Astro]] and will continue to do so. It's close to my heart and I will continue to hack on it.

I will be switching away from Astro for my personal site.

## What went wrong

My goal has and always will be performance. When I discovered Astro, I was so excited to be able to write code in svelte and then ship no JavaScript. While astro delivers, I began to realize it just did not feel as good as making a pure svelte site. To be clear, this is not the core team's fault, it's just a result of abstracting something not designed to be abstracted. So I begin using astro files for my astro site. That would be cool, but the editor support is not great.

Ok so nothing really feels right. To make it harder, the project is not ready yet. Until the last version, I was experiencing random build failures, and there is so much I just can't do. Again, not Astro's fault. Beta software guys!

And then as my site develops I realize I need more.

1. I want to integrate with the [[indie web]]
2. I want to run (hide right now! Buzzword alert) serverless functions
3. I want to generate [[gemini]] in the same build

Non of these really work.

- Indieweb is such a vast term, I won't go into specifics
- For serverless functions/SSR, in astro you just don't. <% breaks the compiler (this is intentional, invalid HTML!), And no matter what editor support would be lackluster because it's just hacks to get it working
- Gemini is possible now, but not really. Essentially I need to hand craft each page separate

On top of that, I'm not a crazy anarchist who combines 15 frameworks together like a madman. That feature has proved useless to me. So my astro honeymoon phase is over, it's not right for me for this usecase.

## Ok so whatchugonnado

So what next? Well what do we want

- 0.5pt: templating/logic (really 100pt but everyone has this)
- 1pt: support for postcss, html minification, javascript minification
- 1pt: I can use wikilinks in markdown in peace
- 1.5pt: SSR/Serverless
- 1.2pt: [[gemini]]
- 0.5pt: ecosystem/indieweb

== 6.7

### A baseline

As a baseline, astro is `0.5 + 0.9 + 0.25 (hack hack) + 0.2 (soon) + 0.1 (I guess I can) + 0.5 (not indieweb but everyone is so nice) = 2.45`

### Hugo

First hugo, my friends use hugo, my friends like hugo.

I don't. First, go sucks, also javascript sucks but go sucks more. I ignored it with astro but the url is *go*hugo, so I'm reminded each time I use the docs. Also it does not score very well `0.4 + 1 + 0.5 (less hack hack) + 0 + 1 + 0.2 - 0.1 (I hate go fee) = 3.0`. Not enough to warrent a switch

Notes:

- https://portable-hugo-links.netlify.app


### Jekyll

No

### Nuxt

No

### Next

No

### And finally....

11ty!

It checks all the boxes of things I want to do. Serverless is on the developer's mind, so is the Indieweb, and Gemini support is possible.

Let's see how it scores out, bullet points this time because i'm convinced!

- 0.5:
	- Templating in 11ty works! It's like astro but for templating languages haha
- 0.9pt:
	- PostCSS: https://gist.github.com/liamfiddler/f7d0ef9184770750578260978534e7e2
	- JS: https://gist.github.com/fourjuaneight/5c0981aa7b97d55fe159db55a822cf07
	- HTML: https://www.11ty.dev/docs/config/#transforms-example-minify-html-output
	- Notes: I really like this approach of do whateverism, but it's not fair to those who go out of their way to support postcss (hi astro!)
- 0.5pt:
	- Same as hugo
		- https://www.11ty.dev/docs/languages/markdown/#add-your-own-plugins
		- https://github.com/alexjv89/markdown-it-obsidian
- 0.9pt:
	- Not as I need it really but A for effort
- 0.1:
	- Probably some way to do this but no one has yet
- 0.45:
	- Astro ecosystem is stronger but 11ty supports indieweb!!!!!!!!!!!

Total: 3.35

So 11ty is the winner I guess!

### K bye

I still think Astro probably has good usecases and I will continue to develop it, but I'm no longer using it for my site.

See you in a month when I switch again.