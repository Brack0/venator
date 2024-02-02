+++
title = "Welcome Zola !"
date = 2024-02-02
+++

Big changes for me, small changes for you, I'm switching to [Zola](https://www.getzola.org/) for my personnal blog. And this is the first blog post. I'm using a new theme as well, which also convince me to change. Docusaurus is well suited for -technical- documentation but no so elegant for blogging (personal opinion).

<!-- more -->

## About Docusaurus

TBD : Things I liked and didn't like about docusaurus will be added later in this section. Spoiler : Docusaurus is great and was really helpful.

## About Zola

TBD : I will explain here why I chose Zola for my new blog.

## The blog setup

Not really a guide or a tutorial, but simply the way setup things and make it work.

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

```
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

#### Dev Mode

One big side effect of setting your production URL during zola init, base url is defined in config. All links will then be built on top of this base url. This is not what you want when having multiple environments and multiple URLs/domains. I switched back to `base_url = "/"` in config to get absolute link without any domain assumption.

````sh
> What is the URL of your site? (https://example.com): https://brack0.dev
# Will produce ```base_url = "https://brack0.dev"``` in config.toml
````

My launch command for dev as I'm working on a remote server (Linux dev server on LAN).

```sh
# Add interface 0.0.0.0 for LAN access
# Add base-url / because base-url in settings is not working during serve
zola serve --interface 0.0.0.0 --base-url /
```

#### Style

I don't really like font sizes for headings in this theme. There are to close of each other and I'm not able to distinguish them when reading this first blog post.

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

#### A11Y

There are multiple accessibility issues to fix in the theme such as multiple H1.

## Migrate blog articles

TBD but these elements will imply time and some changes :

- admonitions (tip, info, warn, etc)
- code blocks
- i18n
- font matter
- Config (SEO and RSS)
