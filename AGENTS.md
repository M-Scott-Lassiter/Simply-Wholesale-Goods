# AGENTS.md

## Project purpose and scope

Simply Wholesale Goods is a custom WordPress and WooCommerce plugin for Simply Homemade Goods.

- Keep the implementation focused on this store's wholesale ordering workflow.
- Prefer native WooCommerce products, variations, carts, checkout, payments, and orders.
- Keep the code straightforward for a small team to understand and maintain.
- Avoid speculative features, unnecessary dependencies, and general-purpose frameworks.
- Implement plugin behavior within this repository without modifying WordPress, WooCommerce,
  third-party plugins, or the active theme.

## Requirements and project documentation

Read `README.md`, `docs/design-spec.md`, and the relevant existing code before implementation.

- `README.md` provides the project overview and development entry point.
- `docs/design-spec.md` defines product behavior, architecture, and acceptance criteria.
- `AGENTS.md` defines working practices, coding conventions, and verification expectations.

Follow the current task and documented requirements. An explicit change requested by the
maintainer takes precedence over older project requirements.

If requirements materially conflict, identify the conflict before implementing the affected
behavior. Continue independent work where possible. Do not silently invent business rules.

Keep detailed payment rules, calculations, compatibility targets, and acceptance criteria in
the design specification. Update affected documentation when an authorized change alters them.

## Coding and documentation conventions

### JavaScript

- Use ES6+ JavaScript with JSDoc. Do not introduce TypeScript.
- Prefer vanilla JavaScript over frameworks.
- Use four-space indentation, single quotes, no semicolons, and no trailing commas.
- Target a maximum line width of approximately 100 characters.
- Write browser tests and JavaScript development scripts using these same conventions.
- Add JSDoc for exported functions and public APIs where it clarifies their contract.

### PHP and other files

- Maintain compatibility with the PHP, WordPress, and WooCommerce targets in the design spec.
- Use four-space indentation and valid PHP syntax, including required semicolons.
- Follow established WordPress and WooCommerce API conventions.
- Use PHPDoc where it clarifies public methods, parameters, return values, or non-obvious behavior.
- Use two-space indentation for YAML.
- Save text files as UTF-8.

### Comments and documentation

- Explain intent, assumptions, edge cases, and non-obvious behavior.
- Avoid comments that merely repeat the code.
- Write comments for a contributor with intermediate programming experience.
- Keep public documentation clear, practical, and accessible to a new contributor.
- Keep documented commands consistent with the actual development tooling.
- Use automated formatting and lint checks once configured.

## Implementation constraints

- Enforce wholesale authorization on the server for every relevant request and purchase path.
- Keep administrative management permissions separate from wholesale shopping eligibility.
- Validate input and escape output using appropriate WordPress and WooCommerce APIs.
- Use capability checks and nonce verification for applicable mutations; a nonce alone does
  not authorize an action.
- Apply catalog visibility rules consistently to rendered pages, AJAX/API responses, and caches.
- Use WooCommerce APIs for products, carts, and orders. Avoid direct order-table assumptions.
- Model wholesale bags and cases as native variations with independently configured prices.
- Treat quantity as the number of the selected variation. Two cases means two cases.
- Preserve purchased prices and packaging descriptions in historical orders.
- Never reconstruct historical order details from the current product configuration.
- Use deterministic monetary calculations. Displayed totals must match saved and payable totals.
- Keep discount eligibility and rounding eligibility explicit and consistent with the design spec.
- Keep recalculation idempotent so repeated requests cannot duplicate discounts or adjustments.

## Development tooling

Use Composer for PHP dependencies, PHP test commands, and plugin packaging tasks.
Use PHPUnit for unit tests and WordPress/WooCommerce integration tests.
Use a small private Node package for Playwright and `wp-env` development tooling.
Write Playwright tests in JavaScript.

During initial scaffolding, establish the setup, test, lint, and build commands and document them
in `README.md`. Do not assume those commands exist before their configuration is present.

- Read `composer.json`, `package.json`, and relevant configuration before selecting commands.
- Use repository scripts when available.
- Keep dependency versions compatible with the documented runtime and test environments.
- Commit applicable dependency lockfiles and keep them consistent with their manifests.
- Development tooling must not be required on the production website.

## Testing and verification

Run checks appropriate to the change and follow the repository's configured CI and release gates.

- Add meaningful tests for new behavior and regression coverage for bug fixes.
- Prioritize monetary calculations, authorization, cart/order integration, and order history.
- Use real WordPress/WooCommerce integration tests for behavior that depends on those systems.
- Cover important vendor and administrator workflows with browser tests.
- Verify access rules across login state, role changes, and relevant cache behavior.
- Test payment-method changes and rounding boundaries against the design specification.
- Use isolated test environments and payment test modes.
- For releases, verify that the built ZIP installs, activates, and contains required runtime files.
- Documentation-only changes require appropriate review, not an unrelated full application suite.

Do not weaken tests or change expected behavior solely to make a failure disappear.
Report exactly what was run, what passed, and what could not be verified.

## Scope and change discipline

- Inspect repository state and relevant files before editing.
- Make focused changes that complete the requested task.
- Preserve unrelated modifications and existing user work.
- Report unrelated problems instead of automatically fixing them.
- When asked for a review or explanation, provide findings without applying changes.
- Avoid opportunistic refactoring, broad formatting changes, and unrelated dependency upgrades.
- Make reasonable implementation decisions autonomously within the agreed requirements.
- Raise unresolved business decisions when they materially affect correctness.
- Keep Git operations, release publication, and deployment within the authorized task scope.

## Packaging and completion

- Preserve the stable `simply-wholesale-goods` plugin directory and established identifiers.
- Build release ZIPs from the intended production files.
- Include required runtime dependencies while excluding tests and development material.
- Keep plugin versions, release tags, and distributable filenames consistent.
- Update relevant documentation when behavior, setup, commands, or packaging changes.

Finish each task with a concise explanation of what changed, why it changed, and how it was
verified. Identify remaining limitations or blockers. Do not claim that tests, commits, pushes,
releases, or deployments succeeded unless their results were verified.
