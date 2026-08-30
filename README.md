# dial9 — RCN Lightning Talk

A [Slidev](https://github.com/slidevjs/slidev) deck, styled to match the
[TokioConf dial9 talk](https://github.com/rcoh/dial9-tokio-talk).

To start the slide show:

- `npm install`
- `npm run dev`
- visit <http://localhost:3030>

Edit [slides.md](./slides.md) to change the content. Speaker notes live in
`<!-- ... -->` blocks at the end of each slide.

`npm run export -- --format png --with-clicks` renders every click state to PNG
(requires `npm i --no-save playwright-chromium`).

## Styling conventions (inherited from the TokioConf deck)

- `theme: default`, `colorSchema: light`, `transition: fade` — restrained, no flashy transitions
- Layouts used: bare (default), `cover`, `statement`, `image`, `image-right`
- `class: text-center` on slides that are a single big idea
- UnoCSS/Tailwind utilities inline; accent colors are `blue-500` (dial9) and
  `amber-500` (cost/overhead), muted body text is `text-gray-400`
- `v-click` for reveals, `v-mark` for highlights; `$clicks` drives the two-phase
  bar chart on the serialization slide
