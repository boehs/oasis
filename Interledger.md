Interledger is disgusting, for the first time I can recall, save for being sick, I am on the verge of vomit. One discovery lead to another, and I quickly realized the amount of deception in interledger, and it's meanings for the [[indie web]].

> **[[PenPen]]'s note:** Evan rashly equivalated the *interledger livenet* to interledger. He did this for a noble reason, the livenet *is* often equivilated, but they are not the same. Interledger is a standard, the livenet is the implementation representing the standard. Evan also wanted me to mention that he thinks the standard is awesome! Evan also wanted to mention that he thinks the livenet is the right thing for the wrong reason. More power to the creators using it! 🦸‍♀️

At first, I thought interledger is crypto. It is not, it is a standard (largely powered by crypto (the connectors mostly use XRP on the backend)) to facilitate transfers of money, agnostic of the currency used by the sender and receiver. I thought this was cool, I added coil to my site when [[imgur]] emerald began using it. I never really thought about it again, until I became involved with the indie web. It started with an email to drew:

>> on interledger and web monetization. I agree with your stance on cryptocurrency, generally, but this feels different. I don't really like coil's implementation with a fixed donation amount or the direction they are going that is making it something to be traded, but I like the idea of paying people based on my usage of their product or work (What I can afford). What are your thoughts on this system?
> There is no place for cryptocurrency in a respectable society.

This answer left me with lots to be desired, although I should have expected it (I was aware drew is a [[crypto]] hater, even more so than me). I thought about cashing out, so I went to my interledger wallet. They wanted everything. What everything?

Let me show you.

> 1. Your current residential address
> 2. Any one of the following valid government photo IDs in Latin (e.g., English, Spanish etc) or Cyrillic or Greek characters only:
	> - Passport
	> - National ID card
	> -  Driver's license
> 3. A 'live selfie'
> 4.  Please note we do not support non-Latin IDs such as Chinese, Arabic etc

It gets worse, in these places you explicitly can't upload (let's not forget the large large part of the world excluded by latin)

- Anguilla
- Antarctica
- Armenia
- Azerbaijan
- Barbados
- Belarus
- Bouvet Island
- Cambodia
- Central African Republic
- Chad
- Cuba
- Eritrea
- Fiji
- French Southern Territories
- Guinea
- Heard and McDonald Islands
- Iran
- Lebanon
- Liberia
- Mali
- Mauritius
- Montenegro
- North Korea
- Palau
- Samoa
- Sudan
- Syria
- S. Minor Outlying Islands
- Vanuatu
And *temporarily*
- American Samoa
- Bangladesh
- Burundi
- China
- Congo (Democratic Republic)
- Congo (Republic)
- Crimea
- Equatorial Guinea
- Germany
- Guinea-Bisseau
- Haiti
- Indonesia
- Iraq
- Kenya
- Libya
- Myanmar
- North Korea*
- Somalia
- Turkmenistan
- Vietnam
- Yemen

> \* both temporarily and under us law!?

So now we have banned half the world from interledger, an open platform, because who cares if you speak latin or not. What about gatehub? I will save you the trouble, same deal here. Fine, it's ok, let's cash out of this mess.

![[Pasted image 20211127120130.png]]

What the fuck? It's not worth it for a total of 5$. Funny how a platform that

> puts humanity first.

Locks half of humanity out. How can you possibly

> build fair and equitable pathways to financial access, always working to connect and benefit each and every human, regardless of identity, geography, or income.

When you exclude our friends that don't speak latin?

> **[[PenPen]]'s Note:** Again, Evan confuses interledger and the interledger livenet. interledger has done this sometimes, https://mojaloop.io, but the public face of interledger has clearly not.

---

But, all hope is not lost. Interledger is open, right? I could set up my own pointer on a VPS and coil could stream to that. Easy peasy. The interledger network should work like this:

![[1638027890230.png]]

>**[[PenPen]]'s Note:** Evan made a mistake here, it should be coil instead of bob.

It can support many bounces, after all. How hard could it be? Well, turns out that each connector needs one peer. There are three peers in the livenet, [[coil]], [[uphold]], and [[gatehub]] (there might be more, but I don't know of any). As I understand it the model looks like

![[Pasted image 20211127123137.png]]

And no one want's to dedicate the resources to peering with your connector.

---

## Why can't an indie connector come?

For the same reason coil, uphold, and gatehub are closed source for profit companies. They are disgusting, making money off of cashing in and out while spewing nonsense about liberation.. Infultrating their fortress with a non profit would be horrible for them. Similarly,

### Can we run our own ledger?

I don't think so. I often wondered why coil was closed source, with all the love they were spewing. Now it makes sense, These products are deliberatly closed source & need to be to be sustainable. Who knows how much work it would take to develop an open coil? Then, by opting to use this new open coil clone you are loosing money off of the people who don't care & just use coil (just as you lose money by using librepay).

Plus, something I have not reaserched is if these connectors could start stealing money passing through them. Perhaps that's another reason for the closed ecosystem, to prevent fraud. I hope the answer is no, But I am inclined based on what I read to think the answer is yes (The connectors in most ledgers work by wiring XRP crypto). I don't think this will ever change, it's likely by design. The t

Crypto is bad
Interledger could be great (\*but is not and never will be)
