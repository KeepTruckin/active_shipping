# AGENTS.md

## Cursor Cloud specific instructions

### What this project is
`active_shipping` is a single Ruby **gem/library** (not a deployable app) that abstracts
various shipping carrier web APIs (USPS, FedEx, Canada Post, etc.). There are **no local
services, databases, or daemons** to run. "Running" it means using it from Ruby (a script
or `rake console`). Carrier integrations live in `lib/active_shipping/carriers/`.

### Toolchain / environment notes
- Runs on the system **Ruby 3.2** (installed via apt) with **Bundler** and gems vendored
  into `vendor/bundle` (`bundle config path` is set to `vendor/bundle`).
- Gems are pinned via the **`Gemfile.lock`** (which the repo `.gitignore`s, so it lives in
  the environment snapshot, not in git). The pins below make this Ruby-2.x-era code run on
  Ruby 3.2. **Do not delete `Gemfile.lock` or run `bundle update`** without re-checking the
  suite — regenerating it from scratch resolves to modern gems that break loading:
  - `concurrent-ruby 1.3.4` — later versions dropped the transitive `require 'logger'`, which
    breaks `activesupport 6.0`'s `LoggerThreadSafeLevel` at load time.
  - `minitest 5.15.0` — later versions removed the top-level `MiniTest` (camelCase) alias that
    old mocha needs.
  - `mocha 1.9.0` — last 1.x that still ships the `mocha/mini_test` shim required by
    `test/test_helper.rb`.
- `tzdata-legacy` (apt) is installed so activesupport can resolve legacy zone names like
  `US/Eastern` / `US/Pacific` used by the FedEx tests (baked into the snapshot).

### Build / test / run (standard commands, from repo root)
- Install/refresh deps: `bundle install` (this is the startup update script).
- Unit tests (offline, mocked — the primary dev command): `bundle exec rake test:unit`
- Remote/E2E tests (hit live carrier APIs, need credentials, otherwise skip):
  `bundle exec rake test:remote` — avoid in CI/offline; missing-credential suites self-skip.
- Note: the default `bundle exec rake test` runs unit **and** remote; prefer `test:unit` offline.
- Build the gem: `bundle exec rake build` (outputs `pkg/active_shipping-*.gem`).
- Interactive console: `bundle exec rake console`.
- **Lint:** none is configured in this repo (no RuboCop config, no lint rake task).

### Known pre-existing test errors (not environment bugs)
`bundle exec rake test:unit` reports **2 errors** on Ruby 3.2, both from `URI.decode` /
`URI.encode` being removed in Ruby 3.0 (used in `test/unit/carriers/usps_test.rb` and
`lib/active_shipping/carriers/usps.rb`). These are codebase-vs-Ruby-3 incompatibilities, not
setup problems. The rest of the suite passes (about 442 tests, 7 skips).
