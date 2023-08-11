# Sigstore the hardway

![Publish Status](https://github.com/stacklok/sigstore-the-hard-way/workflows/publish/badge.svg)

**Sigstore the hard way** is a document that explains the inner workings of Sigstore's infrastructure. It's hosted at <https://stacklok.github.io/sigstore-the-hard-way/> and is available for free.

This README will guide you through the process of setting up and running "Sigstore the hard way" locally and how to contribute to the project.

## Contributing

Thank you for considering contributing! If this is the first time contributing to an Open Source project, you can learn how to make your first pull request from this free video series:

[How to Contribute to an Open Source Project on GitHub](https://egghead.io/courses/how-to-contribute-to-an-open-source-project-on-github)

### Contribution Prerequisites

Before running Sigstore the hard way, ensure that you have the following tools installed:

1. Install [Rust](https://www.rust-lang.org/tools/install) that includes `cargo`
1. Run `cargo install mdbook` to install [mdBook](https://github.com/rust-lang-nursery/mdBook)

If you are using a mac, we highly recommend using [Homebrew](https://brew.sh/) to install these tools.

#### Run locally

Once installed, [clone this reporsitory](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) and in the root folder you can use the `mdbook` command to view in realtime the sigstore-the-hardway documentation.

```bash
mdbook serve --open
```

#### Building for production

You do not need to build before making a pull request, we have a CI action that will automatically build the site and push it to the live site.

Before you push, please run the tests:

```bash
mdbook test
```

### Translations

We'd love help translating the sigstore-the-hardway! See the [Translations] label to join in
efforts. Open a new issue to start working on a new language! We're waiting on [mdbook support] for multiple languages
before we merge any in, but feel free to start!

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang-nursery/mdBook/issues/5
