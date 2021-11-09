*Developed by [[@evan]] Boehs, [[unlicense]]d, powered by [[flancia]]*

Wikilinks everywhere is a chrome extension that brings [[wikilinks]], and through it the [[internet]], to life. It can be installed on a large number of browsers and services, and will intelligently determine where \[\[broken\]] wikilinks should go to based on your settings.

# Sources

You can order or disable any of these sources at will. When a wikilink (**regardless** of if it is properly linked already) is detected the extension will go through these sources in that order you provided. The default order is explained below.

## Options

### Global

- Error codes:
	- Comma separated HTTP status codes
	- Supports `n`xx
	- **Default:** `4xx, 5xx`

### Source specific

- Ignore errors
	- Checkbox
	- Should error cases be ignored?
	- **Default:** Unchecked

## Actual sources

> Note: The last source is the catchall, no matter what it will ignore errors and faithfully complete the link. This means do nothing has importance. Don't remove or reorder

### Existing

If a existing link is already present do nothing

> *Note:* the only difference between the existing source and others is it changes nothing in the browser. By default, if the existing link resolves to `global.error`, it will move on to the sources below and consider replacing it with them

> Note: this might have made no sense. Don't mess with it, probably, it exists for a reason

### Prefix

Things like `WP:link` for wikipedia, or `AG:link` for [[anagora]], the schema looks like `PREFIX:page`.

#### Options

- Prefixes
	- A table of prefixes and the corresponding url
		- use `{page}` to have the provided back link, so with the `WP` prefix the url would be `wikipedia.org/wiki/{page}/`
		- DON'T INCLUDE THE `:` IN THE PREFIX

### [[IWLEP]]

The user's [[IWLEP]] rules. See the IWLEP page for that config, but it is it looks a lot like your config, instead made by the author of the backlink. Don't worry, it can't be faked *unless the user gets hacked but lots worse things could happen.

### Unprefixed

Iterates through all the listed urls in your order until one does not error, or all error. 

`{page}` is the value between the brackets

#### Options

- List
	- **Default:**
		- `wikipedia.org/wiki/{page}/`
		- `anogora.org/{page}/`

### Nothing

What does it say on the tin?

# Concerns

## Bandwidth

We use an extraordinarily large amount of bandwidth, and there is not much that we can do. That being said, the whole web is obese so I highly doubt we end up using more than 0.2 of the bandwidth of a standard blog (~1Kb a page so ~15kb a wikilink so ~150kb a page). Below is what we tried

### ✔️ Request methods

There is no `STATUS` request so we had to make due with `HEAD`. This will only fetch the part of the website that has the page name and useful information. It does not get the page contents, saving bucket loads of bandwidth.

### 👷‍♀️ Local catch

Once implemented we will store the last 1000 pages we checked, so if a wikilink is frequencintly used it will only be loaded once

### ❌ Catch server

This means storing the status code of every website we checked. I don't really like the idea of storing personal information like that.

## Ratelimits

By

# Browsers

- Powercord 🔌 (Discord) Plugin
- Chrome 
- FireFox 🔥🦊