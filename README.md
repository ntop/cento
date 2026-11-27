![ntop][cento_logo]
# nProbe Cento
This repository contains open source components and extensions for nProbe Cento available at http://www.ntop.org/products/netflow-probes/nprobe-cento/

Here you can find:
* Add-on and extensions
* Code examples of applications that can interact with nProbe
* Documentation

Enjoy!

[cento_logo]: https://www.ntop.org/wp-content/uploads/2025/06/nprobe-cento-logo.png

[ntop_logo]: https://camo.githubusercontent.com/58e2a1ecfff62d8ecc9d74633bd1013f26e06cba/687474703a2f2f7777772e6e746f702e6f72672f77702d636f6e74656e742f75706c6f6164732f323031352f30352f6e746f702e706e67

### Documentation

If you want to learn more about nProbe Cento please visit the [User's Guide](https://www.ntop.org/guides/cento/).


### Building the User's Guide

The User's Guide is a [VitePress](https://vitepress.dev) project in `doc/` (Markdown pages, same
URLs as the old Sphinx guide: `/guides/cento/<page>.html`). Requires Node 18+ and `make`.

```
cd doc
make dist      # build the static site into doc/dist (runs npm install if needed)
make serve     # build, then serve it at http://localhost:4173/guides/cento/   (PORT=8080 make serve)
```

For live reload while editing a page use `npm run dev` (http://localhost:5173/guides/cento/).

### Details
For more information about nProbe Cento, please visit [http://ntop.org](https://www.ntop.org/products/netflow-probes/nprobe-cento/).
