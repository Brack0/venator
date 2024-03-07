# TODO

## Mandatory

### Content

- Front-end chronicles episodes (blog posts, templates, RSS)
- Authors in blog posts (this can lead to a rework of templates for taxonomies)
- Update and translate "welcome zola"

## Later

### Cosmetic

- light/dark theme switch ? (user preference for now)
- change colors ? (see A11Y)
- highlight inline code (text inside these backticks ``)
- highlight advices (background color ?)

### Content bis

- Landing (crazy CSS ? OpenGL ? WebAssembly ?): be aware of browser/device support
- Algolia ?
- Footer content (Versus Nebulon) in "about me" page ?
- Better 404 ? 
- Post order mismatch :
  - at the end of blog section, older posts = arrow right / newer posts = arrow left
  - at the end of a post, older post = arrow left / newer post = arrow right

### Technical

#### HTML

- Cleanup uneeded tags (simplify HTML structure)
- A lot of `| safe` is either missing or useless. Need some cleanup on these.

#### CSS

- remove unused rules
- provide same visuals across browsers
- use BEM ? or tag based ?
- inject only relevant CSS (split)
- inline CSS (macro: inline_style)
- add a reset : <https://keithjgrant.com/posts/2024/01/my-css-resets/>

#### Other

- Rename translation keys
- some doc as code for some features (only Tera comments `{# #}` as HTML comments can go to production)
- create a github issue on Zola: Code hightlight + HTML minify => break newlines in code
- <https://observatory.mozilla.org/analyze/venator.vercel.app> (and add a badge in Readme)
