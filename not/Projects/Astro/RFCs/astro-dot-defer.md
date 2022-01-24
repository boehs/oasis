- Start Date: 2022-01-23
- Implementation PR: (leave this empty)
- Insane 4/4

# Summary
This RFC introduces a new feature to the `Astro` API: `Astro.defer`. It allows you to execute code *after* your website has been built.
# Example
Let's say we have a setup like this
`layouts/main`
```js
import Stf from '$scripts/title'
const {title} = Astro.props;
if (title) Stf.add(title)
```
`pages/titles`
```jsx
    {[...Stf].map((item) => (
        <li>{item}</li>
    ))}
``` 
`$scripts/title`
```js
let titles = new Set([])
export default titles;
```
In this setup, the user wants a page with a list of all the titles on their website. The issue is there is no way to guarantee that `pages/titles` will be built last. Let's introduce `Astro.defer`
`pages/titles`
```jsx
{Astro.defer(function(){[...Stf].map((item) => (
return(<li>{item}</li>)
))})};
```
# Motivation

In this section, step back and ask yourself: 'why are we doing this?', 'why does it matter?', 'what use cases does it support?', and 'what is the expected outcome?'.

Please focus on explaining the motivation so that if this RFC is not accepted,
the motivation could be used to develop alternative solutions. In other words,
enumerate the constraints you are trying to solve without coupling them too
closely to the solution you have in mind.

# Detailed design

This is the bulk of the RFC. Explain the design in enough detail for somebody
familiar with Astro to understand, and for somebody familiar with the
implementation to implement. This should get into specifics and corner-cases,
and include examples of how the feature is used. Any new terminology should be
defined here.

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