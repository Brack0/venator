# TODO

## SEO

- check meta for all kind of pages (page, section, tags, etc)
- sitemap.xml: ignore tags (all langs)

## A11Y

- color contrast (warning: theme for code blocks)
- real audit (lighthouse is definitely not enough)

## Cosmetic

- light/dark theme switch ? (user preference for now)
- change colors ? (see A11Y)

## Content

- i18n (add french translations, hreflang alternates)
- Front-end chronicles episodes (blog posts, templates, RSS)
- Landing (crazy CSS ? OpenGL ? WebAssembly ?): be aware of browser/device support
- Algolia ?
- Footer content (Versus Nebulon)
- 404
- Authors in blog posts (this can lead to a rework of templates for taxonomies)

## Technical

### HTML

- Cleanup uneeded tags (simplify HTML structure)
- A lot of `| safe` is either missing or useless. Need some cleanup on these.

### CSS

- remove unused rules
- provide same visuals across browsers
- use BEM ? or tag based ?
- inject only relevant CSS (split)
- inline CSS (macro: inline_style)
- add a reset : <https://keithjgrant.com/posts/2024/01/my-css-resets/>

### Other

- Rename translation keys
- some doc as code for some features (only Tera comments `{# #}` as HTML comments can go to production)
- create a github issue on Zola: Code hightlight + HTML minify => break newlines in code
- <https://observatory.mozilla.org/analyze/venator.vercel.app> (and add a badge in Readme)
