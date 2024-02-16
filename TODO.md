# TODO

## \<head> tag

- Title logic (reduce brain overload for sections, pages and others)
- Lang
- \<meta> (author and description)
- OpenGraph (`<html prefix="og: https://ogp.me/ns#">`)
- Twitter
- Favicon

## SEO

- robots.txt
- sitemap.xml
- RSS

## A11Y

- color contrast
- real audit (lighthouse is definitely not enough)

## Cosmetic

- light/dark theme switch ? (user preference for now)
- change colors ? (see A11Y)
- lang switch

## Content

- i18n
- Front-end chronicles episodes
- Landing (crazy CSS ? OpenGL ? WebAssembly ?): be aware of browser/device support
- Algolia ?
- Footer content
- 404

## Technical

### CSS

- inject only relevant CSS (split)
- use BEM ? or tag based ?
- inline CSS (macro: inline_style)

### Other

- some doc as code for some features (only Tera comments `{# #}` as HTML comments can go to production)
- create a github issue on Zola: Code hightlight + HTML minify => break newlines in code
- <https://observatory.mozilla.org/analyze/venator.vercel.app> (and add a badge in Readme)
