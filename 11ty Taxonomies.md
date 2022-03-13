---
title: Zero maintenance taxonomies in 11ty
tags:
- thinking aloud
- 11ty
- jamstack
- webdev
in: blog
date: 2022-03-12
---

Recently, I became interested in making my blog more navigatable, so I began looking into [taxonomies](https://indieweb.org/taxonomy). To my disappointment, 11ty has no concept of taxonomies! The collection object only looks for tags.

There have been a few blog posts on this topic that I have disregarded because they did not have the flexibility I craved. I want to just make a list of taxonomies I use and never need to think about it again.

::: details Other posts on taxonomies in 11ty

- https://www.webstoemp.com/blog/basic-custom-taxonomies-with-eleventy/

:::

Well folks, if there is one thing I'm good at, it's abusing software. Let's do this.

[[toc]]

## Getting started

To have flexible taxonomies in 11ty, we need two collections. Because 11ty does not allow multiple levels of pagination, the first collection must be one dimensional by nature, so we can paginate over it.

Our architecture will look as follows:

```json
{
    // What we use for paginating
    "taxAndValues": [
        ["in", "blog"],
        ["tag", "lemons"]
        ["tag", "alpacas"]
        ...
    ],
    // What we use for populating tax pages
    "nestedTax": {
        "tag": {
            "leomons": {
                "data": ...
                ...
            }
            ...
        }
        ...
    }
}
```

Naturally, the first thing we need to do is set up paginating.

### Before you get too caught up

I am using *similar* code, but not the same, on my website. This code has been adapted to be more general than what I use on given site. I have not fully tested the *modified* code, however I see no reason it should not work. If you encounter issues, please email *u.boehs.oasis@todo.sr.ht*

## Paginating

First, let's set up our `taxValue.njk` file. For now, we are just going to set up the frontmatter

{% raw %}

```yaml
---
layout: base.njk
pagination:
  data: taxAndValues
  size: 1
  alias: value
permalink: /{{ value[0] }}/{{ value[1] }}/
eleventyComputed:
  title: On "{{ value[1] }}"
---
```

{% endraw %}

This will paginate over our one dimensional `taxAndValues` collection that we are yet to make. Let's make it now.

First, lets make a list of the taxonomies we are using.

```js
const taxonomies = [
    "tags",
    "in"
}]
```

Assuming your taxonomies list is indeed just called `taxonomies`, you can paste this code in without trouble

```js
eleventyConfig.addCollection('taxAndValues',function(collectionApi) {
    // lets make a variable to hold our taxonomies and values
    let taxAndValues = [] 
    // We need to get each post in our posts folder. In my case this is /c
    const nodes = collectionApi.getFilteredByGlob('pages/c/*.md')
    // next lets iterate over all the nodes
    nodes.forEach(node => {
        // and then iterate over the taxonomies
        taxonomies.forEach(taxonomy => {
            // I don't want to paginate date, for instance
            // this is why my collectionControl is using objects instead of arrays
            if (node?.data?.[taxonomy]) {
                // this is typeof on drugs
                switch(({}).toString.call(node.data[taxonomy]).match(/\s([a-zA-Z]+)/)[1].toLowerCase()) {
                    // if it is an array (for tags especially)
                    case 'array':
                        node.data[taxonomy].forEach(item =>
                            taxAndValues.push([taxonomy,item])
                        )
                        break
                    // otherwise
                    default: taxAndValues.push([taxonomy,node.data[taxonomy]])
                }
            }
        })
    });

    // custom set, sets don't work with objects
    const unique = [...new Set(taxAndValues.map(JSON.stringify))].map(JSON.parse)
    
    return unique
})
```

## A quick break

Lets git commit! We are making great progress!

```bash
$ git commit --all -m "start working on taxonomies"
```

I would also like to take a moment to remember my English bulldog. Today, two years ago, she left us at the age of 14. She was a wonderful dog, and I feel so lucky to have lived those 14 years with her. May she snore forever in heaven.

::: details Bulldog picture
![Bulldog picture](https://i.boehs.org/raw/dew2l3tt.jpg)
:::

## Creating useful pages

We are now generating these pages a proper way, but something is still missing. These pages don't have content!

Let's reopen `taxValue.njk`, and add to the page

{% raw %}
```liquid
{% set taglist = collections.nestedTax[value[0]][value[1]] %}
<ul>
{% for post in taglist %}
    <li>
        <a href="{{ post.url | url }}">{{ post.data.title }}</a>
    </li>
{% endfor %}
</ul>
```
{% endraw %}

We will now largely copy and paste the same code that we did in the first collection

```js
eleventyConfig.addCollection("nestedTax", function (collectionApi) {
    let nestedTax = {};
    const nodes = collectionApi.getFilteredByGlob("pages/c/*.md");

    nodes.forEach(node => {
        taxonomies.forEach(taxonomy => {
            if (node?.data?.[taxonomy]) {
                // if the taxonomy in the object does not yet exist
                if (!nestedTax[taxonomy]) nestedTax[taxonomy] = {};
                // like "computing" or "blog"
                const taxValue = node.data[taxonomy]
                switch ({}.toString.call(taxValue).match(/\s([a-zA-Z]+)/)[1].toLowerCase()) {
                    case "array": {
                        taxValue.forEach(item => {
                            // if the value in the object does not yet exist
                            if (!nestedTax[taxonomy][item]) nestedTax[taxonomy][item] = [];
                            // then add the entire page to it
                            nestedTax[taxonomy][item].push(node);
                        });
                        break;
                    }
                    default: {
                        // if the value in the object does not yet exist
                        if (!nestedTax[taxonomy][taxValue])
                        nestedTax[taxonomy][taxValue] = [];
                        // then add the entire page to it
                        nestedTax[taxonomy][taxValue].push(node);
                    }
                }
            }
        })
    });

    return nestedTax;
});
```

## Conclusion

I'm sorry for the complex code! If you have any suggestions to improve readability, please let me know!

### Legal (You don't want to get sued)

All code on this page (text within `<code>` html tags) is hereby released under the [unlicense](/unlicense.txt). Consider sending a small tip via librepay!
