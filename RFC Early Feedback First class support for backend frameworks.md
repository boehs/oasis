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
One might think an alternative is the following:

```
{`<%= var %>`}
```

However, attempting to build this site results in `&`
# Adoption strategy

If we implement this proposal, how will existing Astro developers adopt it? Is
this a breaking change? Can we write a codemod? Can we provide a runtime adapter library for the original API it replaces? How will this affect other projects in the Astro ecosystem?

# Unresolved questions

Optional, but suggested for first drafts. What parts of the design are still
TBD?