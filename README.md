# sigstore the hardway

sigstore the hard way is hosted at https://sthw.decodebytes.sh and is available for free.

## contributing

Should you wish to contribute, you're in the right place.

Building the sigstore the hard way requires [mdBook].

[mdBook]: https://github.com/rust-lang-nursery/mdBook
[rust-mdbook]: https://github.com/rust-lang/rust/blob/master/src/tools/rustbook/Cargo.toml

```bash
$ cargo install mdbook
```

Once installed you can use the `mdbook` command to view in realtime the sigstore-the-hardway documentation.

```bash
mdbook serve --open
```

You do not need to build before making a pull request, we have a CI action that will automatically
build the site and push it to the live site.

Before you push, please test:

To run the tests:

```bash
$ mdbook test
```

### Translations

We'd love help translating the sigstore-the-hardway! See the [Translations] label to join in
efforts. Open a new issue to start working on a new language! We're waiting on [mdbook support] for multiple languages
before we merge any in, but feel free to start!

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang-nursery/mdBook/issues/5