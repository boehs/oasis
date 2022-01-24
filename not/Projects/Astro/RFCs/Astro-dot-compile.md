- Start Date: 2022-01-23
- Implementation PR: (leave this empty)
- Evans insane 3/4

# Summary
This RFC introduces a new feature of the `Astro` API: `Astro.compile`. It is designed to reduce javascript in built sites further by precompiling segments of external javascript, similar to frontmatter.
# Example
Evan is adding a really cool text to the bottom of his website. Inside a script tag, he writes
```js
let reallycooltext = "evan " + "is " + "really " + "cool!"
document.getElementById('footer').innerHTML = reallycooltext
```
Next, Evan refactors his code to do it the astro way
```html
---
let reallycooltext = "evan " + "is " + "really " + "cool!"
---
<script define:vars={{reallycooltext}}>
document.getElementById('footer').innerHTML = reallycooltext
</script>
```
This reduces his website size by getting rid of extra quotation marks and plus signs. The built HTML here looks like
```html
<script define:vars={{reallycooltext}}>
let reallycooltext="evan is really cool!"
document.getElementById('footer').innerHTML = reallycooltext
</script>
```
This is good progress, but Evan realizes that including this script tag on *every* page is excessive. He realizes it would be better to make another file, `script.js` and import it from the HTML. That way, the browser can cache it. He moves his code out of the astro component into it's own file.
```js
let reallycooltext = "evan " + "is " + "really " + "cool!"
document.getElementById('footer').innerHTML = reallycooltext
```
And imports it from the html. This is great improvement, but he is once again shipping the useless + symbols and " symbols.

This is where we are now, ideally Evan could have frontmatter inside his javascript file, but that's impossible. Right now, he could

1. Compile it manually, remove extra quotation marks and plus symbols
2. Accept the bigger js file size

Now, let's refactor the javascript file with our RFC.

```j
let reallycooltext = "evan " + "is " + "really " + "cool!"
document.getElementById('footer').innerHTML = reallycooltext
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