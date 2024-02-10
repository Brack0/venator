+++
title = "Welcome Zola !"

[taxonomies]
tags = ["zola"]
+++

Big changes for me, small changes for you, I'm switching to [Zola](https://www.getzola.org/) for my personal blog. This is the first blog post. I'm using a new theme as well, which also convince me to change. [Docusaurus](https://docusaurus.io/) is well suited for (technical) documentation but not so elegant for blogging (opinion).

<!-- more -->

## About Docusaurus

I started my blog in early 2022. My requirements were simple: an open source (and free) solution based on something I know. I discovered Docusaurus via documentation of tools I use, such as Jest. It comes with everything I need out of the box and many features I will later configure. These features include i18n, SEO, plugins, extended Markdown (MDX, Front matter, remark, etc).

As many developers, I love programming. However, this can lead to appreciate the tools more than the applications built using them. This runs against a pragmatic way of solving things. I wanted to tackle this for my blog. When working on side projects, the journey AND the destination matter as one will learn a lot and get something at the end (kind of, [insert unfinished side project meme here]). But when it comes to blogging, the journey is you writing posts, not you building the best blogging app. That was the main goal when I started my blog.

I chose Docusaurus because I was able to focus on content (blog posts, podcast episodes, etc) instead of the project itself. As an example, I was able to write and deploy in production some blog post with only a GitLab access (thanks to Web IDE). Writing text or markdown and produce HTML/CSS/JS is what I expect from a static-site generator.

## About Zola

Before dive into Zola, let me explain why I was exploring other options. I'm just going to be honest, I'm tired of JS Framework being the [golden hammer](https://en.wikipedia.org/wiki/Law_of_the_instrument) of the web. I love working on Frontend, especially with Angular. Should I use Angular for every single project that write HTML tags? No obviously. Should I use React for a blog? I could but this is not necessary. I don't need JS here, or maybe just a few snippets for minor interaction. A real static-site generator is sufficient for blogging.

Second reason is focused on the visual part. Only one theme is production ready. Not even one has been added since the beginning of Docusaurus. The default theme can be tuned, but everyone got pretty much the same design overall. I'm bored of mine lately. So, I tried to find a tool that have multiple themes.

Why Zola then? Not going to lie, I'm on the Rust hype train 🦀 (Rust mentioned BTW). A rust engine was a secret requirement for this blog. Vercel has a preset for Zola. I don't have to deal with `node`, `node_modules` and dependencies. Basic solution, straight to the point, multiple themes, all in one. This is exactly what I need.

## The blog setup

Not really a guide nor a tutorial, but simply the way I setup Zola and make it work.

### Install Zola on a Linux

This is how I installed on ParrotOS (debian/ubuntu fork)

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

One big side effect of setting your production URL during zola init, base url is defined in config. All links will then be built on top of this base url. This is not what you want when having multiple environments and multiple URLs/domains. I switched back to `base_url = "/"` in config to get absolute link without any domain assumption.

```txt
> What is the URL of your site? (https://example.com): https://brack0.dev
  ➡️ Will produce ```base_url = "https://brack0.dev"``` in config.toml
```

My launch command for dev as I'm working on a remote server (Linux dev server on LAN).

```sh
# Add interface 0.0.0.0 for LAN access
# Add base-url / because base-url in settings is not working during serve
zola serve --interface 0.0.0.0 --base-url /
```

## What's next

### Style

I don't really like font sizes for headings in this theme. There are too close of each other and I'm not able to distinguish them when reading this first blog post.

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

### A11Y

There are multiple accessibility issues to fix in the theme such as multiple H1.

### Migrate blog articles

TBD but these elements will imply time and some changes:

- admonitions (tip, info, warn, etc)
- code blocks
- i18n
- front matter
- Config (SEO and RSS)

_Where did the 3D animation go?_

It will come back eventually (but maybe with another visual), still available [here](https://nebulon.vercel.app/).
