---
# theme id or package name
# Learn more: https://sli.dev/themes/use.html
theme: 'apple-basic'

# what transition to use
transition: 'fade-out'

# title of your slide, will auto infer from the first header if not specified
title: 'Testing Elixir'

# titleTemplate for the webpage, `%s` will be replaced by the page's title
titleTemplate: '%s - Slidev'

# information for your slides, can be a markdown string
info: false

# enable presenter mode, can be boolean, 'dev' or 'build'
presenter: true

# enabled pdf downloading in SPA build, can also be a custom url
download: true

# filename of the export file
exportFilename: 'slides'

# export options
# use export CLI options in camelCase format
# Learn more: https://sli.dev/guide/exporting.html
export:
  format: 'pdf'
  timeout: 30000
  dark: false
  withClicks: true
  withToc: false
  
# syntax highlighter, can be 'prism' or 'shiki'
highlighter: 'prism'

# show line numbers in code blocks
lineNumbers: false

# enable monaco editor, can be boolean, 'dev' or 'build'
monaco: 'dev'

# download remote assets in local using vite-plugin-remote-assets, can be boolean, 'dev' or 'build'
remoteAssets: false

# controls whether texts in slides are selectable
selectable: true

# enable slide recording, can be boolean, 'dev' or 'build'
record: 'dev'

# force color schema for the slides, can be 'auto', 'light', or 'dark'
colorSchema: 'auto'

# router mode for vue-router, can be "history" or "hash"
routerMode: 'history'

# aspect ratio for the slides
aspectRatio: '16/9'

# real width of the canvas, unit in px
canvasWidth: 980

# used for theme customization, will inject root styles as `--slidev-theme-x` for attribute `x`
themeConfig:
  primary: '#5d8392'

# favicon, can be a local file path or URL
favicon: 'https://cdn.jsdelivr.net/gh/slidevjs/slidev/assets/favicon.png'

# URL of PlantUML server used to render diagrams
plantUmlServer: 'https://www.plantuml.com/plantuml'

# fonts will be auto imported from Google fonts
# Learn more: https://sli.dev/custom/fonts
fonts:
  sans: 'Roboto'
  serif: 'Roboto Slab'
  mono: 'Fira Code'

# default frontmatter applies to all slides
defaults:
  layout: 'default'

# drawing options
# Learn more: https://sli.dev/guide/drawing.html
drawings:
  enabled: true
  persist: false
  presenterOnly: false
  syncAll: true
---
---
src: ./000-intro.md
---
---
src: ./010-lecture-1.md
---
---
src: ./011-lecture-1-exercise.md
---
---
src: ./break.md
---
---
src: ./020-lecture-2.md
---
---
src: ./021-lecture-2-exercise.md
---
---
src: ./break.md
---
---
src: ./030-lecture-3.md
---
---
src: ./031-lecture-3-exercise.md
---
---
src: ./break.md
---
---
src: ./040-lecture-4.md
---
---
src: ./041-lecture-4-exercise.md
---
---
src: ./099-wrapup.md
---
---
src: ./100-appendix.md
---
---
src: ./200-demo.md
---
