- Start Date: (fill me in with today's date, 2022-01-22)
- Implementation PR: (leave this empty)
# Summary
This RFC defines a configuration option to make astro viable in applications where custom elements and delimiters are needed.

# Definitions
In `<%= hi %>`,
* `startDelimiter`: `<%`
* `value`: `= hi `
* `endDelimiter`: `%>`
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

In an implementation of this design, the config object is extended to introduce a new property

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
	language: string,
	/**
	 * Include delimiters in output
	 * 
	 * true: `<% Value %>`,
	 * 
	 * false: ` Value `
	 * 
	 * default: true
	 */
	includeDelimiters?: boolean
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
For delimiters that match, the compiler essentially leaves it be, so the output will be `if (includeDelimiters) startDelimiter + value + if(includeDelimiters) endDelimiter`, it then stops processing and should not do any transformation of matched content.

This is effectively the same as `<Fragment data-astro-raw>startDelimiter value endDelimiter </Fragement>` (Note: `data-astro-raw`  is currently broken, see compiler/#263)
#### Language Server
The contents within matched delimiters are syntax highlighted as specified in `language`

---

 Test cases should be added for both compiler and language server to ensure invalid html `<%%>` elements and double braces `{{}}` are still rendered correctly, if in CustomElements.

### Drawbacks

- implementation cost, both in term of code size and complexity

### Alternatives

There are no good alternatives. Not implementing this makes astro unusable for a group of people.

## Level Two
### Example
```html
running on sh#node --version#sh
```
and, `index.astro` with
```html
<%= hi %>
```
becomes `index.ejs` instead of `index.html`
### Detailed description
In an implementation of this design, the `CustomElement` interface is extended to introduce a new property

```ts
interface CustomElement {
	...
    /** Renderer
     *
	 * Example `@boehs/astro-render-bash` or `@boehs/astro-render-ejs`
	 */
	renderer: string
}
```

For delimiter matches with the renderer property, instead of doing nothing, the value is passed to the renderer as if it was it's own component. 
# Adoption strategy

If we implement this proposal, how will existing Astro developers adopt it? Is
this a breaking change? Can we write a codemod? Can we provide a runtime adapter library for the original API it replaces? How will this affect other projects in the Astro ecosystem?

# Unresolved questions

Optional, but suggested for first drafts. What parts of the design are still
TBD?