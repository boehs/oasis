- Start Date: 2021-01-12

# Summary

This RFC discusses the manner in which astro will support backend frameworks like express, flask, and rocket, through support for server side templating. 

# Example

With the EJS renderer in use, with the following astro code:

test.astro
```html
---
let bool = true
---


{{:if bool}}

<div>{bool}</div>
<div>{{bool}}</div>

{{/if}}

{ bool ? <div>{{bool}}</div> : null}
```

testtwo.astro
```
---
let bool = true
---
{bool}
```

gets rendered to

test.ejs

```ejs
<%if (bool) { %>
<div>true</div>
<div><%= bool %></div>
<% } %>
<div>true</div>
<div><%= bool %></div>

<div><%= bool %></div>
```

testtwo.html

```
true
```

# Motivation

Frontend frameworks and backend frameworks have historically not gotten along. Frontend frameworks tend to generate code that is very hard for backend frameworks to work with. To negate this issue, it has been common for these frontend frameworks to bundle a backend framework. Examples include:

- Nuxt
- Next
- SvelteKit

The problem is that these backend frameworks don't offer flexibility. When working with raw HTML, the developer has a wide choice of routing methodologies and is free to pick and choose, he gains flexibility at the cost of convenience. Astro is unique because it generates static HTML. This means astro can be used in many routing contexts, while still being convenient. 

# Detailed design

## Definitions

* Delimiter: astro currently uses the `{` `}` delimiters for most things

# Drawbacks

Why should we *not* do this? Please consider:

- implementation cost, both in term of code size and complexity
- whether the proposed feature can be implemented in user space
- the impact on teaching people Astro
- integration of this feature with other existing and planned features
- cost of migrating existing Astro applications (is it a breaking change?)

There are tradeoffs to choosing any path. Attempt to identify them here.

# Alternatives

What other designs have been considered? What is the impact of not doing this?

# Adoption strategy

If we implement this proposal, how will existing Astro developers adopt it? Is
this a breaking change? Can we write a codemod? Can we provide a runtime adapter library for the original API it replaces? How will this affect other projects in the Astro ecosystem?

# Unresolved questions

Optional, but suggested for first drafts. What parts of the design are still
TBD?