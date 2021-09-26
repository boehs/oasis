One might be tempted to think at first glance that Trinket is another static site generator, and in many ways it is

1. It renders markdown to html
2. It provides templating for easy style and layout creation
3. It supports YAML front matter
4. It has rich plugin support to extend it's already steller markdown support

On top of that, trinket is a special hybrid of amazing things we took from our favorite static site generators, like

2. Incredibly preformant, safe, and multithreaded code written in Rust 🦀
3. Modern features to accommodate new way's of thinking, like digital gardens
4. First class terminal support

But what makes trinket *special* are it's trinkets, charms, toys, worrinklers and shamens

# The content ecosystem

TD;LR: Trinket makes it easy to get content from many abstract sources like Instagram, Twitter, local files, ect than turn them into a standardized layout ready to be like a conventional blog feed.

## Trinkets

If only one feature defines trinket, it's got to be our trinkets. Unlike conventional SSG's, our core tooling has no method of accessing markdown. Instead, we rely on trinkets to hand us structured data. Or built in local file parser is a trinket! It scans the file tree, and for each markdown file parses it into two sections, content and metadata. The result for each post looks something like

```json
{
	"metadata": {
		"creationDate": "May 31",
		"title": "ye"
	},
	"content": {
		"body": "<h1>hi</h1>"
	}
}
```

You then access it from your post template using the templates you know and love.

Imagine the possibilities for a second. Create a feed where Twitter posts are mixed around block posts, show off your Instagram gallery and your LinkedIn activity all from your blog with no effort

### Shamans

Shamens are good for when trinkets would be too intensive. They are quite similar, but are run in the DOM vs at compile time. Good for dynamic content and high file sizes. This also allows them to leverage all normal JavaScript features that might be relevant, like timeouts.

## Charms

Charms are extensions for the parser. They can hook onto events the parser emits and action them accordingly. Our API has useful methods like `content.enclosedIn("**")` and `content.line.startswith("#")`. Similarly to Trinkets, our markdown to html is entirely written in in Charms

## Toys

Toys mimic components in conventional static site generators.

## Wrinklers

Wrinkles "suck" cooki.. I mean posts, during compile time right after charms are applied. Metadata in it's raw format is still available despite the final HTML page being built. This makes wrinklers useful to organize posts based on metadata, because we are sure the data won't change at this stage. For instance, we ship with three wrinklers.

1. Is used to build an indexDB of posts for search functionally
2. Is used to generate feeds of posts meeting criteria, like tags

Wrinklers are run sync, and they are allowed to modify metadata, although that is discouraged

# magic

Magic is a dedicated package manager for trinket. You can easily search new trinkets, charms, toys, and wrinklers to add to your site. Each one can have dependencies that are automatically resolved. 

What's more, the API for trinkets and shamens are the same, so in most cases it's up to you what you want to use it for.

# Powerful markdown parsing

We are 100% commonmark complient, with extra features

- \[\[wikilinks\]\]
- Front matter
  - Aliases
- inline \#tagging
- $La^{tex}$
- ==\=\=Highlighting==\=\=
- Footnotes
- Citations^[1]
- superscript and subscript