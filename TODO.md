# TODO

## \<head> tag

- Favicon

## SEO

- robots.txt
- sitemap.xml

## A11Y

- color contrast
- real audit (lighthouse is definitely not enough)

## Cosmetic

- light/dark theme switch ? (user preference for now)
- change colors ? (see A11Y)

## Content

- i18n (add french translations, hreflang alternates)
- Front-end chronicles episodes (blog posts, templates, RSS)
- Landing (crazy CSS ? OpenGL ? WebAssembly ?): be aware of browser/device support
- Algolia ?
- Footer content
- 404
- Authors in blog posts

## Technical

### CSS

- inject only relevant CSS (split)
- use BEM ? or tag based ?
- inline CSS (macro: inline_style)
- Reset : <https://keithjgrant.com/posts/2024/01/my-css-resets/>

### Other

- A lot of `| safe` is either missing or useless. Need some cleanup on these.
- Rename translation keys
- Cleanup menu template in `base.html`
- some doc as code for some features (only Tera comments `{# #}` as HTML comments can go to production)
- create a github issue on Zola: Code hightlight + HTML minify => break newlines in code
- <https://observatory.mozilla.org/analyze/venator.vercel.app> (and add a badge in Readme)
