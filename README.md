# Simply Wholesale Goods

A custom WordPress and WooCommerce plugin for
[Simply Homemade Goods](https://simplyhomemadegoods.com/).

Simply Wholesale Goods helps approved vendors order wholesale products through a bulk order form
or the store's existing product pages, using WooCommerce for checkout, payments, and order history.

## Status

This project is being prepared for implementation. The repository contains the design
specification and working instructions. Plugin code, development tooling, the sanitized site
fixture, and the hosted demo environment are planned.

Setup commands and installation instructions will be documented as those components are built.

## Planned features

- A **Wholesale Vendor** role with access to wholesale products and pricing.
- A **WooCommerce > Wholesale** admin page for viewing vendors and managing their access.
- Wholesale bags and cases as native product variations, with independently configured prices.
- A bulk order form with blank quantity inputs and whole-number ordering.
- A configurable offline payment discount, defaulting to **3%**, for cash, check, and direct bank
  transfer when enabled.
- Optional nearest-dollar rounding for those same payment methods, enabled by default.
- Separate retail and wholesale shopping experiences.
- Preserved order details when product prices or case sizes change.

## Development

Read [AGENTS.md](AGENTS.md) for working instructions and coding conventions.

The [design specification](docs/design-spec.md) defines the product requirements, implementation
plan, and acceptance criteria.

The planned tooling includes:

- **PHP and Composer** for plugin development, dependencies, and packaging.
- **PHPUnit** for unit tests and WordPress/WooCommerce integration tests.
- **JavaScript and Playwright** for browser tests.
- **wp-env and Docker** for repeatable development and test environments.
- **Mailpit** for capturing development and demonstration emails.
- **Docker Compose** for a persistent hosted demo environment.

Dependency versions will follow the compatibility targets in the design specification.
Commit the applicable Composer and npm lockfiles.

## Development and demonstration environments

### Automated tests

Use a clean WordPress/WooCommerce environment with generated products, users, and orders.

Tests must be repeatable and isolated from the persistent demo. Resetting an automated test
environment must never reset the demo database or storage.

### Sanitized site fixture

Prepare a reviewed test copy of the store that preserves the relevant product catalog, layouts,
configuration, and public assets. Commit this fixture so a fresh checkout can reproduce the
store-specific environment.

Remove real customer accounts, orders, form submissions, private documents, credentials, and
other sensitive or operational data. Generate synthetic accounts and representative orders
during setup.

Keep original backups and unreviewed exports outside tracked source files. Service credentials,
including Stripe test secret keys, must be supplied separately.

Document the fixture's source versions, preparation process, and any differences from the
production hosting environment.

### Email testing

Route development and demo emails to Mailpit so order notifications, account messages, payment
links, and invoice attachments can be inspected.

Leave Mailpit forwarding and relaying disabled. Verify that the configured WordPress mailer
actually sends to Mailpit.

Captured messages and Mailpit's runtime storage remain outside Git.

### Hosted demo

Provide a persistent Docker Compose environment for demonstrating the plugin before deployment.

The demo will run WordPress, WooCommerce, the sanitized site fixture, and a known plugin build.
It will use its own database and storage, with Mailpit for email and Stripe configured for testing.

Integrate the demo with the server's existing reverse proxy and access configuration. Keep
machine-specific addresses and credentials outside tracked configuration.

Preserve demonstration data across ordinary restarts and plugin updates. Provide an explicit
operation for restoring the starting fixture.

Document setup, access, plugin updates, test payment configuration, and resetting the demo.
Use the built plugin ZIP for release demonstrations to exercise the actual installation path.

## Local files and fixtures

The following directory conventions apply as the environment is implemented:

| Path | Purpose | Git behavior |
| --- | --- | --- |
| `backups/` | Original website backup archives | Ignored |
| `.local-site/` | Working copies and exports awaiting review | Ignored |
| `runtime/` | Repository-local database, Mailpit, and other runtime storage | Ignored |
| `tests/fixtures/store/` | Reviewed, sanitized site baseline and selected assets | Tracked |
| `dist/` | Generated installable plugin ZIPs | Ignored |

Commit environment templates such as `.env.example` with placeholder values. Keep populated
environment files and service credentials outside Git.

Database and Mailpit bind mounts should use `runtime/`. Named Docker volumes may also be used
and are stored outside the repository.

## Distribution

Releases will provide an installable WordPress plugin ZIP through GitHub Releases.

The ZIP must contain the plugin's required runtime files and dependencies. Exclude test fixtures,
demo infrastructure, backups, credentials, and development tooling.

Development tools such as Composer, Node, Docker, and Mailpit are not required on the store's
website to use the finished plugin.

Before release, verify the built ZIP and complete the required compatibility checks against
the store's hosting environment and installed extensions.

Deploy the plugin ZIP and documented configuration changes. Preserve the live store's database
and orders when moving from demonstration to production.

## License

[MIT](LICENSE)
