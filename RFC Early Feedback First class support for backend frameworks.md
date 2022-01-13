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


{{IF bool}}

<div>{bool}</div>
<div>{{bool}}</div>

{{/IF}}

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

While upcoming SSR support through rendering with astro is exciting, in certain contexts it does not make sense. Such examples are

- High performance situations where the effort needed to render complex components (header, footer, etc) would better be spent on what actually changes between different pages 
- Build scripts that transform html

This RFC adds flexability.
# Detailed design
## Definitions
* Delimiter: astro currently uses the `{` `}` delimiters for most things. For the context of this RFC delimiter = `{{}}`, 
* Syntax: Please refer to "Unresolved Questions"

## Design

When specifying code within the delimiter, the author may reference variables, preform if statements, use while loops, and more, while using a syntax defined by astro.

The difference is code within the delimiter is *not* filled at build time. For instance, consider the following astro code
```
{var}
``` 
vs the new syntax
```
{{var}}
```

In the first example, during build time var will be replaced with it's value. In the second, it will remain as is (`var`), but with appropriate delimiters for their template language.

Also, some more operations are preformed:

1. If file has a template, do not include in sitemap
2. If file has a template, change file extension.
# Drawbacks

* This feature may make people less likely to use SSR, although I doubt it.
* This feature is actually not very complex, it adds very little
* See alternatives!
# Alternatives
### One
One might think an alternative is to simply  use multiline strings:

```
{`<%= var %>`}
```

However, attempting to build this site results in `&gt;`, this is counterproductive. There is no way that I am aware of to just have the raw `<`
### Two
String arguments! Currently the builder uses `&lt;`/`&gt;` if it's not a valid html element or just the element `</div>`, without the lts and gts. It would be better if for a given string you could enable or disable this (Always replace with lt/gt or never replace with lt/gt)
# Unresolved questions
There are varying questions
## Is this RFC just to add a separate mode without &gt;s and %lt;s
### Yes?
This could work and it would be simple. The con is that template file extension support would be lost
### No?
A separate `Astro.raw()` mode could be added to avoid file renaming
