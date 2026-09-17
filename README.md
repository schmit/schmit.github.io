# Random Projections

Sven Schmit's personal website and blog at <https://svenschmit.com>, built with
Jekyll and the Lagrange theme.

## Local development

Use Ruby 3.4 (the tested patch version is in `.ruby-version`) and Bundler.
The Ruby bundled with macOS is too old for this project. With Homebrew:

```sh
brew install ruby@3.4
export PATH="$(brew --prefix ruby@3.4)/bin:$PATH"
bundle install
bundle exec jekyll serve
```

Open <http://localhost:4000>. Restart the server after changing `_config.yml`.
Ruby version managers can use `.ruby-version` instead of the Homebrew setup.

## Validation and dependency updates

```sh
bundle exec jekyll build
bundle outdated
```

To update installed gems within the declared version constraints, run
`bundle update`, then build again. `Gemfile.lock` remains local and ignored;
dependency constraints live in `Gemfile` and `lagrange.gemspec`.

Edit source files, not generated `_site/` output. Use Jujutsu (`jj`) for local
version control.

Shared style variables are in `_sass/_variables.scss`; the Sass entry point is
`assets/css/main.scss`. Keep Sass `@import` syntax: production currently uses
GitHub Pages' branch-based build, whose legacy Sass compiler does not support
`@use`. Local Dart Sass deprecation warnings for these imports are expected.
The production builder uses GitHub Pages' bundled gems rather than this
project's local Ruby/Jekyll versions. A future move to Sass modules must also
migrate deployment to a custom build using the project's dependencies.
