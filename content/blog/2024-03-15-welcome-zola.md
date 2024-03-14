+++
title = "Welcome Zola !"

[taxonomies]
tags = ["Zola", "Rust", "Content"]
+++

Small change for you, big change for me, I'm switching to [Zola](https://www.getzola.org/) for my personal blog. This marks my first blog post using Zola. I'm using a new theme as well, which also convinced me to change. [Docusaurus](https://docusaurus.io/) is well suited for (technical) documentation but not so elegant for blogging (opinion).

<!-- more -->

## About Docusaurus

I started my blog in early 2022. My requirements was simple: an open source (and free) solution based on something I know. I discovered Docusaurus via documentation of tools I use, like Jest. It comes with everything I need out of the box and many features I will configure later. These features include i18n, SEO, plugins, extended Markdown (MDX, Front matter, remark, etc).

As many developers, I love to program. However, this can lead to appreciate the tools more than the applications built using them. This runs against a pragmatic way of solving things. I wanted to tackle this for my blog. When working on side projects, the journey AND the destination matter as one will learn a lot and get something at the end (kind of, [insert unfinished side project meme here]). But when it comes to blogging, the journey is you writing posts, not you building the best blogging app. That was the main goal when I started my blog.

I chose Docusaurus because I was able to focus on content (blog posts, podcast episodes, etc) instead of the project itself. As an example, I was able to write and deploy in production some blog post with only a GitLab access (thanks to Web IDE). Writing text or markdown and produce HTML/CSS/JS is what I expect from a static-site generator.

## About Zola

Before diving into Zola, let me explain why I was exploring other options. I'm just going to be honest, I'm tired of JS Framework being the [golden hammer](https://en.wikipedia.org/wiki/Law_of_the_instrument) of the web. I love working on Frontend, especially with Angular. Should I use Angular for every single project that write HTML tags? No obviously. Should I use React for a blog? I could but this is not necessary. I don't need JS here, or maybe just a few snippets for minor interaction. A real static-site generator is sufficient for blogging.

Second reason is focused on the visual part. Only one Docusaurus theme is production ready. Not even one has been added since the beginning of Docusaurus. The default theme can be tuned, but everyone got pretty much the same design overall. I'm bored of mine lately. So, I tried to find a tool that have multiple themes and where you can easily build your own.

_Don't get me wrong, we can customize Docusaurus with a technic called [Swizzling](https://docusaurus.io/docs/swizzling) but it's risky. One can replace layout components with his own implementation. Since these components can have breaking changes (props, renaming, splitting, etc), you have to check and maybe fix manually every custom layout components. Ain't nobody got time for that!_

Why Zola then? I won't lie, I'm on the Rust hype train 🦀 (Rust mentioned BTW). A Rust engine was a secret requirement for this blog. Vercel offers a preset for Zola. I don't have to deal with `node`, `node_modules` and dependencies. Basic solution, straight to the point, multiple themes, all in one. This is exactly what I need.

## The blog setup

Not really a guide nor tutorial, but simply the way I setup Zola and get it to work.

### Install Zola on Linux

This is how I installed on ParrotOS (a Debian/Ubuntu fork)

```sh
# Get last release from https://github.com/getzola/zola/releases
wget https://github.com/getzola/zola/releases/download/v0.18.0/zola-v0.18.0-x86_64-unknown-linux-gnu.tar.gz

# Quick and dirty "install"
tar xvzf zola-v0.18.0-x86_64-unknown-linux-gnu.tar.gz
sudo cp zola /usr/local/bin
```

### Get started

```sh
zola init myblog
```

```txt
> What is the URL of your site? (https://example.com): https://brack0.dev
> Do you want to enable Sass compilation? [Y/n]: Y
> Do you want to enable syntax highlighting? [y/N]: Y
> Do you want to build a search index of the content? [y/N]: N
```

### Import an awesome theme

```sh
git submodule add https://github.com/pawroman/zola-theme-terminimal.git themes/terminimal
```

```toml
# https://github.com/pawroman/zola-theme-terminimal
theme = "terminimal"
```

### Tweaks

One big side effect of setting your production URL during zola init, base url is defined in config. All links will then be built on top of this base url. This is not what you want when having multiple environments and multiple URLs/domains. I switched back to `base_url = "/"` in config to get absolute link without any domain assumption. However we need absolute urls for some meta tags, such as canonical and alternates. I created an extra value in config : `site_url = "https://brack0.dev"` to build absolute urls.

```txt
> What is the URL of your site? (https://example.com): https://brack0.dev
  ➡️ Will produce ```base_url = "https://brack0.dev"``` in config.toml
```

My launch command for dev as I'm working on a remote server (Linux dev server on LAN).

```sh
# Add base-url="/" because base-url is "http://127.0.0.1:1111" when I'm using the dev server
zola serve --base-url /
```

## What's next ?

### Style / Design

I moved Terminimal theme inside the codebase which allowing me to update things I don't like. You can see below some CSS for headings. Font sizes are too close of each other and we are not able to distinguish them very well. Anyway, I've made some design updates to match my expectations (visual, accessibility, etc).

```css
h1 {
  font-size: 1.4rem;
}

h2 {
  font-size: 1.3rem;
}

h3 {
  font-size: 1.2rem;
}

h4, h5, h6 {
  font-size: 1.15rem;
}
```

### A11Y (Accessibility)

There were multiple issues to fix in the theme such as multiple H1. Accessibility is easier for blogging since we have a lot of structured text content and few complex elements such as dropdown, modal, carousel, etc. Once I've updated colors and contrast, I used tools ([WAVE](https://wave.webaim.org/extension/) and [AXE](https://www.deque.com/get-started-axe-devtools-browser-extension/)) to detect and fix remaining issues.

### Migrate content from previous blog

Besides few things, it was pretty easy. Basically it goes like this : copy [MDX](https://mdxjs.com/) files, update front matter (Markdown header with some metadata), replace React components with Zola [shortcodes](https://www.getzola.org/documentation/content/shortcodes/), add some polish and that's it.

I have also created shortcodes for [admonitions](https://docusaurus.io/docs/markdown-features/admonitions) and code blocks. Admonition does not exist in markdown and code block is missing a title (which is really useful in my opinion).

One cool behavior of Zola is that all translations of a markdown file is next to the original (english for me). In Docusaurus, original blog posts were behind `/blog` and translated ones were hidden in `/i18n/[locale]/docusaurus-plugin-content-blog`. It's also better to configure two separate contents (blog posts and podcast episodes) with SEO and RSS

## Conclusion

And voila ! If you can read this, you're on my new blog generated by Zola. Hope you will enjoy it. No JavaScript included, built with Rust (another Rust mention, let's go) and blazingly fast.

Say goodbye to [Nebulon](https://nebulon.vercel.app/). So Long, and Thanks for All the Fish 👋.
