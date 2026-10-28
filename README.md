# The Librecar Control Wiki

Wiki to illustrate Librecar Control Project.

The book covers LiteX and Linux bring-up on an Arty A7-100T, SocketCAN on JC, lab Ethernet on J9, and the Open Single Pair Ethernet path (LAN8670 PMOD on JB) toward a CAN–10BASE-T1S zonal gateway.

## Pull Requests

Pull requests are welcome. Please ensure they conform to the conventions described below.

## Viewing

See the [rendered book](https://librecar.dev/).

## Development

Install [`mdbook`](https://github.com/rust-lang/mdBook):
```
cargo install mdbook
```

Build the book:
```
mdbook build
```
The generated files are in the `book` directory.

View the built book:
```
mdbook serve
```
This will launch a local web server to serve the book. Navigate to
[localhost:3000](http://localhost:3000/) in a web browser. While the web server
is running, the rendered book will automatically update if the source files
change.

### Conventions

Each entry is of the form:
```
* [The Literal Title of The Book](https://a-link-to-the/book/) - an optional comment if the title isn't self-explanatory
```

The `.editorconfig` file captures basic source files formatting conventions.
Many editors support this file natively; others (such as VS code) require a plugin, see https://editorconfig.org/.

The default branch is `main`. See [Renaming the default branch from master](https://github.com/github/renaming) for the rationale.

Commit logs should describe the specific changes made by a commit. See [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/).

## Deployment

The book is deployed to the `gh-pages` branch by CI on a push to the `main`
branch. GitHub Pages serves it at [https://librecar.dev](https://librecar.dev/).
The workflow writes the `CNAME` file so the custom domain survives each
rebuild.
