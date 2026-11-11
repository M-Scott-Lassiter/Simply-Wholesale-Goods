# Simply Wholesale Goods

A custom WordPress and WooCommerce plugin for
[Simply Homemade Goods](https://simplyhomemadegoods.com/).

Simply Wholesale Goods helps approved vendors order wholesale products through a bulk order form
or the store's existing product pages, using WooCommerce for checkout, payments, and order history.

## Status

This project is in early development. The design specification is available; plugin implementation,
setup instructions, and installable releases will follow.

## Planned features

- A **Wholesale Vendor** role with access to wholesale products and pricing.
- A **WooCommerce > Wholesale** admin page for viewing vendors and managing their access.
- Wholesale bags and cases as native product variations, with independently configured prices.
- A bulk-order form with blank quantity fields and whole-number ordering.
- A configurable offline payment discount, defaulting to **3%**, for cash, check, and direct bank
  transfer when enabled, plus optional whole-dollar rounding for eligible payments.
- Separate retail and wholesale shopping experiences.
- Preserved order details when product prices or case sizes change.

## Development

The planned tooling includes PHP and Composer, PHPUnit for backend and integration tests,
plain JavaScript and Playwright for browser tests, and wp-env with Docker for local development.

See the [design specification](docs/design-spec.md) for the requirements, implementation plan,
and acceptance criteria.

## Distribution

Releases will provide an installable WordPress plugin ZIP through GitHub Releases. Development
tools such as Composer, Node, and the test runners are not required on the store's website.

## License

[MIT](LICENSE)
