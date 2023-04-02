# ElixirConfEU 2023 - Testing Elixir Training

To make this work you need to ...

* clone the repo and run `npm install`
* you can then take a first look at the slides by running `npx slidev`
  and select `open`
* to see/check the (SPA) website you need to ...
  * install `npm install http-server --global`
  * build the website `npx slidev build`
  * run the webserver `http-server dist -c-1`

Note: The CI/CD pipeline (GitHub actions) will deploy the (SPA)
[website][] to `gh-pages` with every push/merge to `trunk`.

[website]: http://tedn.life/slidev-elixirconfeu-2023/presenter
