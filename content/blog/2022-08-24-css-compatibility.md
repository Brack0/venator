+++
title = "Managing CSS Compatibility"

[taxonomies]
tags = ["CSS", "Dev"]
+++

With the omnipresence of Babel and TypeScript in the Frontend universe, the JavaScript compatibility of our applications is (almost) no longer an issue. For HTML, only a few attributes have been added since HTML5 for bonus features (lazyload, prefetch, etc.). They are generally ignored by incompatible browsers. What about CSS? There are no proper polyfills, very heterogeneous support, and even a slight misstep with `display: grid` can be disastrous for the visuals. So, how do we rationalize our CSS? Let's explore that together.

<!-- more -->

## Browserslist

[Browserslist](https://github.com/browserslist/browserslist) is a library used by many frontend developers. There's a good chance that in your current projects, you have a `.browserslistrc` file or a configuration in your `package.json`. Whether you are aware of it or not, Browserslist is present in **React** applications (via create-react-app), **Angular**, **Vue**, and many others (I've verified only the three most well-known 😬).

Here's a quick example of how Browserslist works by providing a configuration and then checking which browsers are associated with that configuration:

{% codeblock(name="package.json") %}

```json
{
  "browserslist": [
    "last 1 version",
    "> 5% and not dead",
    "BlackBerry 10",
    "not ie 11"
  ]
}
```

{% end %}

{% codeblock(name="$ npx browserslist") %}

```sh
and_chr 103
and_ff 101
and_qq 10.4
and_uc 12.12
android 103
bb 10
chrome 103
chrome 102
edge 103
firefox 103
ios_saf 15.5
kaios 2.5
op_mini all
op_mob 64
opera 89
safari 15.6
samsung 17.0
```

{% end %}

In our case, Browserslist (with the help of [Can I Use](https://caniuse.com/)) will help us configure the two tools I present below. Indeed, we will use it as a reference for the browsers to support. The advantage is that Babel and [ESLint](https://github.com/amilajack/eslint-plugin-compat) also use this repository for JavaScript support.

{% advice(type="tip") %}

You can also harness the power of Browserslist for your Node.js backends. More information can be found on the [Browserslist repository](https://github.com/browserslist/browserslist).

{% end %}

## PostCSS Autoprefixer Plugin

If you have Angular, React, or VueCLI projects, you are already using [Autoprefixer](https://github.com/postcss/autoprefixer). Regardless, its operation is very simple: inspect your CSS and add the necessary prefixes for certain browsers (`-webkit-`, `-moz-`, etc.). As you may have guessed, Autoprefixer relies on Browserslist (+ Can I Use) to determine when and what type of prefix to add.

In short, this [PostCSS](https://github.com/postcss/postcss) plugin allows you to focus on the main CSS rules while offloading some of the compatibility management. I refer you to the documentation for its setup if it's not already present.

## Stylelint as a Safeguard

[Stylelint](https://stylelint.io/) is a linter for CSS/SASS/Less/etc. similar to ESLint for JavaScript. Stylelint highlights anomalies in your stylesheets based on configurable rules, with the goal of preventing errors and establishing conventions for developers.

In addition to best practices, you can also identify rules that are not supported by your target browsers. Just like Autoprefixer, we rely on Browserslist (+ Can I Use) to bridge the gap between your target browsers and the CSS rules used. This is exactly what the Stylelint plugin [no-unsupported-browser-features](https://github.com/ismay/stylelint-no-unsupported-browser-features) does, and I encourage you to try it out. Don't forget about your IDEs to reduce the feedback loop.

{% advice(type="tip") %}

When you combine Stylelint and Autoprefixer (which I highly recommend), consider enabling rules of the type `xxx-no-vendor-prefix` because Autoprefixer handles support for you. No need to clutter your style files.

{% end %}

## Tweaking

Once all these elements are in place, you can adjust the list of supported browsers according to your needs. Browserslist's default settings are suitable for the majority of projects with no specific requirements (`> 0.5%, last 2 versions, Firefox ESR, not dead`). If that's not sufficient, you can define a set of rules that are more or less restrictive and tailored to your audience.

Finally, I draw your attention to a more targeted approach for your product. Through small utilities ([Browserslist-GA](https://github.com/browserslist/browserslist-ga) and [browserslist-new-relic](https://github.com/syntactic-salt/browserslist-new-relic)), you can gather **your** statistics and use them in Browserslist (with rules like `> Y% in my stats`). This can make sense when public statistics are not relevant, such as in very specific markets (e.g., China) or private platforms.

## Conclusion

By using these tools, you can secure your CSS production. Stylelint allows us to spot subtleties in our practices. For example, the `gap` property combined with a `flex` display has only been available since mid-2020! In comparison, when used with `grid`, this property has been available since mid-2018, and even since early 2017 _[Editor's Note: that is, at the same time as CSS Grid]_ under its old name (`grid-gap`). There are many other examples like this, and unless you spend all your time on MDN, you'll miss more than one in your projects. In addition, Autoprefixer takes care of aliases, prefixes, and other refinements required by our favorite browsers. This helps us avoid tricky bugs and headaches.
