- Start Date: (fill me in with today's date, 2022-01-22)
- Implementation PR: (leave this empty)
# Summary
This RFC defines a configuration option to make astro viable in applications where custom elements and delimiters are needed.

# Motivation
This RFC was driven by the lack of support for EJS style templating. This style of templates uses `<%`, invalid html that astro correctly flags. It would be counterproductive to provide an exception to users of EJS, so I began thinking about how this need *I* have can be applied to needs others have. Through this RFC, users of templating engines *regardless* of what one they use feel first class.

In part two, this is extended to allow users to create embedded components in any renderer, make templating support even better, and introduce new renderers for executing code.
# Levels
This RFC has been split into two levels, however both levels have multiple implementation steps that can be completed individually. Level one is more pressing than two.
- Level one introduces basic support and language server support
	- [ ] It works!
	- [ ] Language server support
- Level two adds support for renderers
	- [ ] Base render support
	- [ ] File name change API
## Level One
### Example

```html
<%= hi %> <!-- This does not crash astro now, it also is syntax highlighted like ejs templates would be -->
```

```html
{{This is it's own thing, not javascript}}
```

### Detailed design

In an implementation of this design, the config object is extended to introduce a new key

```ts
interface CustomElement {
    /** RegExp for opening tag, closing tag
     * 
     * For example, if you want to make `<% hi! %>` valid, you would set match to be `[/<%/,/%>/]`
     * 
     * Or, if you wanted to do something more complex, say `<😂 to="the" world>owo</😂>`, you could write a regexp like `[/<😂.*?>/,/<\/😂>/]`
     */
	delimiters: [RegExp,RegExp]
    /**
     * Syntax Highlighting Language
     * 
     * Examples: `typescript`, `ejs`, `handlebars`
     */
	language: string
}
interface Options {
	...

    /**
     * Define custom delimiters for syntax highlighting and telling the compiler to leave you alone.
     */
	CustomElements?: [CustomElement]
}
```

There are actions taken in two places
#### Compiler
For delimiters that match, the compiler essentially leaves it be, so the output will be `delimiter+`

### Drawbacks

Why should we *not* do this? Please consider:

- implementation cost, both in term of code size and complexity
- whether the proposed feature can be implemented in user space
- the impact on teaching people Astro
- integration of this feature with other existing and planned features
- cost of migrating existing Astro applications (is it a breaking change?)

There are tradeoffs to choosing any path. Attempt to identify them here.

### Alternatives

What other designs have been considered? What is the impact of not doing this?

# Adoption strategy

If we implement this proposal, how will existing Astro developers adopt it? Is
this a breaking change? Can we write a codemod? Can we provide a runtime adapter library for the original API it replaces? How will this affect other projects in the Astro ecosystem?

# Unresolved questions

Optional, but suggested for first drafts. What parts of the design are still
TBD?