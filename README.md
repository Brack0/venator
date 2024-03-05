# Venator

![Gitlab status badge](https://img.shields.io/gitlab/pipeline-status/Brack0%2Fvenator?logo=gitlab&labelColor=black)
![Vercel deployed badge](https://img.shields.io/badge/Vercel-deployed-blue?logo=vercel&labelColor=black)
![Zola version badge](https://img.shields.io/badge/Zola-0.18.0-orange?logo=rust&labelColor=black)
![MIT License badge](https://img.shields.io/gitlab/license/Brack0%2Fvenator?labelColor=black)

Venator is a static website for a personal blog. It's based on [Zola](https://www.getzola.org/), a static site generator built with Rust. You can find more context in the blog article [Welcome Zola !](content/blog/2024-02-02-welcome-zola.md).

## Getting started

Install Zola : <https://www.getzola.org/documentation/getting-started/installation/>

### Launch dev mode

```sh
zola serve -u /
```

#### Trivia

We use `-u /`, which stands for `base_url = "/"`, in dev mode as we don't default value (`127.0.0.1:1111`). See `site_url` in `config.toml`.

### Build the app

```sh
zola build
```

## Integration

Deployed with [Vercel](https://vercel.com/) via Zola preset.

### Custom settings

- Node.js version: `20.x` (required for latest Zola versions)
- Environment Variables:
  - `ZOLA_VERSION=0.18.0`

## Roadmap

**Main goal: Replace previous blog (<https://gitlab.com/Brack0/nebulon>)**

Tasks are available in [TODO.md](./TODO.md).

## Contributing

Not open.

## License

MIT License. Check [LICENSE](./LICENSE) for more details.
