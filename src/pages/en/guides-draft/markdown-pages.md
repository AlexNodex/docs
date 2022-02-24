---
layout: ~/layouts/MainLayout.astro
title: Markdown
description: Building Pages in Markdown
---

## Markdown Pages

Astro treats any `.md` file inside of the `/src/pages` directory as a page. Placing a file in this directory, or any sub-directory, will automatically build a page route using the pathname of the file.

This file's Markdown content is combined with a Layout, and can additionally use variables and components defined in the front matter, to produce a page on your site.

### Layouts

Markdown pages have a special front matter property for `layout` that defines the relative path to an `.astro` Layout component. This component will wrap your Markdown content, providing a page shell and any other page template elements. 

> 🛎️ Markdown pages are children for Layout components. The Markdown content is placed into the Layout's `<slot />` element, rendered as HTML.

All other front matter properties defined in your `.md` page can be passed to the layout component as properties of the `content` object prop.


```markdown
---
layout: ../layouts/BaseLayout.astro
title: My cool page
description: My first Astro Markdown page
---
# Hello World!

This is my markdown page.
```

```astro
// src/layouts/BaseLayout.astro
---
const { content } = Astro.props;
---
<html>
  <head>
    <title>{content.title}</title>
  </head>

  <body>
    <slot />
  </body>
</html>
```
### Variables 

Front matter variables can be used directly in your Markdown as properties of the `frontmatter` object.

```markdown
---
layout: ../layouts/BaseLayout.astro
author: Leon
age: 42
---
# About the Author

{frontmatter.author} is {frontmatter.age} and lives in Toronto, Canada.

```

### Components

You can import components into your Markdown file using `setup` and use them along with Markdown syntax. The `frontmatter` object is also available to any imported components.

```markdown
---
layout: ../layouts/BaseLayout.astro
setup: | 
  import Author from '../../components/Author.astro'
  import Biography from '../components/Biography.jsx'
author: Leon
---
# About the Author

<Author name={frontmatter.author}/>

<Biography client:visible >
{frontmatter.author} lives in Toronto, Canada and enjoys photography.
</Biography>

```


### Draft Pages

`draft: true` is an optional front matter value that will mark an individual `.md` page or post as "unpublished." By default, this page will be excluded from the site build.

Markdown pages with `draft: false` or without the `draft` property are unaffected and will be built.

```markdown
src/pages/post/blog-post.md
---
layout: ../../layouts/BaseLayout.astro
title: My Blog Post
draft: true
---

This is my in-progress blog post.

No page will be built for this post.

To build and publish this post:
- update the front matter to `draft: false` or
- remove the `draft` property entirely.
```


> ⚠️ Although `draft: true` will prevent a page from being built on your site, `Astro.fetchContent()` currently returns **all your Markdown files**. 
>
> To exclude the post data (e.g. title, link, description) from being included in your post archive, or list of most recent posts, be sure that your `Astro.fetchContent()` function also filters to exclude any draft posts.

⚙️ To enable building draft pages: 

Add `drafts: true` to `buildOptions` in `astro.config.mjs` 

```js
//astro.config.mjs

export default /** @type {import('astro').AstroUserConfig} */ ({
  < ... >
  buildOptions: {
    site: 'https://example.com/',
    drafts: true,
  },

});
``` 

 💡 You can also pass the `--drafts` flag when running `astro build` to build draft pages! 


## Markdown Component

Astro has a dedicated component used to let you render Markdown in `.astro` files. 

Import the Astro Markdown component in your Component Script and then write any Markdown you want between `<Markdown> </Markdown>` tags.

````astro
---
// For now, this import _must_ be named "Markdown" and _must not_ be wrapped with a custom component
// We're working on easing these restrictions!

import { Markdown } from 'astro/components';
import Layout from '../layouts/Layout.astro';

const expressions = 'Lorem ipsum';
---
<Layout>
  <Markdown>
    # Hello world!

    **Everything** supported in a `.md` file is also supported here!

    There is _zero_ runtime overhead.

    In addition, Astro supports:
    - Astro {expressions}
    - Automatic indentation normalization
    - Automatic escaping of expressions inside code blocks

    ```js
      // This content is not transformed!
      const object = { someOtherValue };
    ```

    - Rich component support like any `.astro` file!
    - Recursive Markdown support (Component children are also processed as Markdown)
  </Markdown>
</Layout>
````

### Remote Markdown

If you have Markdown in a remote source, you may pass it directly to the Markdown component through the `content` attribute.

```astro
---
import { Markdown } from 'astro/components';

const content = await fetch('https://raw.githubusercontent.com/withastro/docs/main/README.md').then(res => res.text());
---
<Layout>
  <Markdown content={content} />
</Layout>
```

### Nested Markdown

`<Markdown>` components can be nested. 

```astro
---
import { Markdown } from 'astro/components';

const content = await fetch('https://raw.githubusercontent.com/withastro/docs/main/README.md').then(res => res.text());
---

<Layout>
  <Markdown>
    ## Markdown example

    Here we have some __Markdown__ code. We can also dynamically render remote content.

    <Markdown content={content} />
  </Markdown>
</Layout>
```

⚠️ Use of the `Markdown` component to render remote Markdown can open you up to a [cross-site scripting (XSS)](https://en.wikipedia.org/wiki/Cross-site_scripting) attack. If you are rendering untrusted content, be sure to _sanitize your content **before** rendering it_.


## Markdown Parsers

Astro comes with Markdown support powered by [remark](https://remark.js.org/). 

The `@astrojs/markdown-remark` package is included by default with the following plugins pre-enabled:

- [GitHub-flavored Markdown](https://github.com/remarkjs/remark-gfm)
- [remark-smartypants](https://github.com/silvenon/remark-smartypants)
- [rehype-slug](https://github.com/rehypejs/rehype-slug)


> ⚙️ You can include a custom Markdown parser inside `astro.config.mjs` by providing a function that follows the `MarkdownParser` type declared inside [this file](https://github.com/withastro/astro/blob/main/packages/astro/src/@types/astro.ts).

```js
// astro.config.mjs
export default {
  markdownOptions: {
    render: [
      'parser-name', // or import('parser-name') or (contents) => {...}
      {
        // options
      },
    ],
  },
};
```

### Remark and Rehype Plugins

Astro supports third-party plugins for Markdown. You can provide your plugins in `astro.config.mjs`.

> **Note:** Enabling custom `remarkPlugins` or `rehypePlugins` removes Astro’s built-in support for the plugins previously mentioned. You must explicitly add these plugins to your `astro.config.mjs` file, if desired.

### Add a Markdown plugin in Astro

1. Install the npm package dependency in your project.

2. Update `remarkPlugins` or `rehypePlugins` inside the `@astrojs/markdown-remark` options:

```js
// astro.config.mjs
export default {
  markdownOptions: {
    render: [
      '@astrojs/markdown-remark',
      {
        remarkPlugins: [
          // Add a Remark plugin that you want to enable for your project.
          // If you need to provide options for the plugin, you can use an array and put the options as the second item.
          // ['remark-autolink-headings', { behavior: 'prepend'}],
        ],
        rehypePlugins: [
          // Add a Rehype plugin that you want to enable for your project.
          // If you need to provide options for the plugin, you can use an array and put the options as the second item.
          // 'rehype-slug',
          // ['rehype-autolink-headings', { behavior: 'prepend'}],
        ],
      },
    ],
  },
};
```

You can provide names of the plugins as well as import them:

```js
// astro.config.mjs
import autolinkHeadings from 'remark-autolink-headings';

export default {
  markdownOptions: {
    render: [
      '@astrojs/markdown-remark',
      {
        remarkPlugins: [[autolinkHeadings, { behavior: 'prepend' }]],
      },
    ],
  },
};
```

### Syntax Highlighting

Astro comes with built-in support for [Prism](https://prismjs.com/) and [Shiki](https://shiki.matsu.io/). 

By default, Prism is enabled but no Prism stylesheet is included. (Choose from the available [Prism Themes](https://github.com/PrismJS/prism-themes) and see the [list of languages supported by Prism](https://prismjs.com/#supported-languages) for options and usage.)

You can modify this behavior by configuring the `@astrojs/markdown-remark` options:

```js
// astro.config.mjs
export default {
  markdownOptions: {
    render: [
      '@astrojs/markdown-remark',
      {
        // Pick a syntax highlighter. 
        // Can be 'prism' (default), 'shiki' or false to disable any highlighting.
        syntaxHighlight: 'prism',
        // If you are using shiki, here you can define a global theme and
        // add custom languages.
        shikiConfig: {
          theme: 'github-dark',
          langs: [],
          wrap: false,
        },
      },
    ],
  },
};
```

You can read more about custom Shiki [themes](https://github.com/shikijs/shiki/blob/main/docs/themes.md#loading-theme) and [languages](https://github.com/shikijs/shiki/blob/main/docs/languages.md#supporting-your-own-languages-with-shiki).

(See also the [`<Prism />` Astro component](/en/reference/builtin-components/#prism-) and the [`<Code />` Astro component](/en/reference/builtin-components/#code-) powered by Shiki.)