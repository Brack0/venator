# TODO

## Cosmetic

- menu links for mobile (visual bug)
- highlight inline code (text inside these backticks ``)
- highlight advices (background color ?)

## Content

- Translate "welcome zola"
- Landing (crazy CSS ? OpenGL ? WebAssembly ?): be aware of browser/device support
- Algolia ?
- Footer content (Versus Nebulon) in "about me" page ?
- Split section templates (blog and podcast)
- Better 404 ?
- Post order mismatch :
  - at the end of blog section, older posts = arrow right / newer posts = arrow left
  - at the end of a post, older post = arrow left / newer post = arrow right

## A11Y

- Fix remaining issues

## Technical

### HTML

- Cleanup uneeded tags (simplify HTML structure)
- A lot of `| safe` is either missing or useless. Need some cleanup on these.
- Favicon : <https://web.dev/learn/html/document-structure#favicon>
- All CSP stuff (<https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html>)

### CSS

- consistency in sizing (0.5 rem, 2 rem, 8 rem instead of 10px, 40px, 150px)
- remove unused rules
- provide same visuals across browsers
- use BEM ? or tag based ?
- inject only relevant CSS (split)
- inline CSS (macro: inline_style)
- add a reset : <https://keithjgrant.com/posts/2024/01/my-css-resets/>

### Other

- Rename translation keys
- some doc as code for some features (only Tera comments `{# #}` as HTML comments can go to production)
- <https://observatory.mozilla.org/analyze/venator.vercel.app> (and add a badge in Readme)
