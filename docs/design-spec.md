# Simply Wholesale Goods

## Design and implementation specification

**Specification version:** 1.0  
**Prepared:** November 10, 2026  
**Store:** Simply Homemade Goods, https://simplyhomemadegoods.com/  
**Repository owner and maintainer:** m-scott-lassiter  
**Repository and plugin slug:** simply-wholesale-goods  

Build a small, maintainable WordPress plugin for this one WooCommerce store. Approved
vendors must be able to purchase wholesale variations through the existing product
pages or a bulk order form, then use the existing cart, checkout, payment gateways,
and order management.

This is an implementation handoff, not a record of completed implementation or tests.
The repository has not been created as part of preparing this specification.

## 1. Authority and implementation defaults

The business requirements below consolidate the owner's decisions in the planning
conversation. Requirements marked **MUST** are acceptance conditions. The few choices
not explicitly settled by the owner are listed here so they cannot be mistaken for
previously approved business rules.

### 1.1 Selected implementation defaults

1. **Discount base:** apply the percentage discount to the wholesale merchandise
   subtotal. Exclude shipping and unrelated fees. Include those charges when rounding
   the final payable cash/check total. Shipping treatment was not explicitly specified
   by the owner; merchandise-only discounting is the selected v1 default.
2. **Rounding midpoint:** round to the nearest whole USD dollar, with an exact
   half-dollar rounded upward.
3. **Bulk submission:** add to the existing eligible cart, combine quantities of
   identical variations, then continue to native checkout.
4. **Batch errors:** invalid quantity fields count as zero. A positive-quantity row
   referencing an unauthorized, invalid, or unavailable variation rejects the batch
   before cart changes.
5. **History:** catalog restrictions govern current shopping and reordering. A
   customer's own historical orders remain accessible after their role changes.
6. **Existing orders:** preserve the payment policy captured when the order was
   created. Later settings changes apply to new carts/orders, not old transactions.
7. **Simple products:** ordinary simple products remain retail-only in v1. Wholesale
   offerings use native variations, including a one-variation variable product if
   necessary.
8. **Repository visibility:** use a private repository initially if visibility has not
   otherwise been specified. Repository publication and production installation are
   separate from building the plugin.

Direct bank transfer eligibility is an explicit user decision. It receives the same
discount and whole-dollar rounding when enabled.

### 1.2 Scope boundaries

The plugin is maintained for this store. It MUST NOT introduce volume pricing,
vendor-specific price lists, an application approval system, a separate wholesale
checkout, a new payment gateway, a tax engine, resale-certificate management, inventory
tracking, case-to-bag stock conversion, or a custom invoice system.

There is no requirement for a commercial plugin framework, automatic WordPress.org
distribution, a custom update server, or automatic deployment to the live store.

## 2. Site baseline

The following versions were supplied in the site's November 10, 2026 status report.
They are the initial compatibility target, not claims that every component is the latest
release or that compatibility has already been tested.

| Component | Reported configuration |
| --- | --- |
| WordPress | 7.1.4, single site |
| WooCommerce | 11.2.2 |
| PHP | 8.2.35 |
| Database server | MySQL 8.4.7-7 |
| Theme | Astra 4.14.2, classic theme |
| WooCommerce theme template overrides | None reported |
| Cart page | /cart/, contains [woocommerce_cart] |
| Checkout page | /checkout/, contains [woocommerce_checkout] |
| Account page | /my-account/, contains [woocommerce_my_account] |
| Currency | USD, two decimal places |
| Order storage | Traditional CPT datastore; HPOS disabled |
| Persistent object cache | Object Cache Pro 1.25.7 |
| Catalog size | 36 products and 90 existing variations |
| Keldy's account role | Administrator |

### 2.1 Relevant installed extensions

| Extension | Version | Required integration coverage |
| --- | --- | --- |
| Checkout Plugins - Stripe for WooCommerce | 1.11.2 | Checkout amount, payment changes, order-pay, enabled express paths |
| CartFlows | 3.3.2 | Confirm the actual checkout route |
| Auto Coupons for WooCommerce, RLDD | 3.1.0 | Prevent promotional discounts on wholesale orders |
| Advanced Product Search, aThemeArt | 1.2.0 | Audience-filtered live search and prices |
| Smart Variation Swatches, aThemeArt | 2.0.1 | Eligible variation controls and underlying data |
| Variation Swatches, CartFlows | 1.0.14 | Eligible variation controls and underlying data |
| Modal Fly Cart and AJAX Add to Cart, aThemeArt | 2.0.1 | Authorized cart mutations and quantity validation |
| PDF Invoices and Packing Slips, WP Overnight | 5.16.6 | Frozen packaging descriptions and consistent totals |
| Cart Abandonment Recovery | 2.2.2 | Restored carts still obey current authorization and coupon rules |

Elementor and related page-building extensions are also active. Test the actual pages
in use; do not infer a conflict solely from the installed-plugin list.

The supplied CartFlows screenshot shows "Create a global store checkout." The working
inference is that global Store Checkout is unconfigured. Target classic WooCommerce
checkout and confirm that inference on staging. Individual funnels may still exist.
CartFlows replaces the store checkout after its Store Checkout feature is enabled.
[S2]

The Stripe integration is the Checkout Plugins product, not an assumed gateway named
"stripe." Inspect the installed gateway IDs and enabled checkout paths. Its author
changed distribution channels; reproduce the actual installed package for compatibility
testing rather than assuming a WordPress.org download will be available. [S18]

### 2.2 Tax and fulfillment assumptions

Wholesale sales are tax-free under the store's configured business arrangement.
Test fixtures use zero-tax products/orders. The plugin does not determine exemption
eligibility or manage permits.

Reuse existing WooCommerce fulfillment, shipping, pickup, and payment availability
settings. No new shipping calculation or inventory model is required. Products are
made to order and numeric stock tracking is out of scope.

## 3. Plugin identity and lifecycle

| Identifier | Value |
| --- | --- |
| Display name | Simply Wholesale Goods |
| Directory and text domain | simply-wholesale-goods |
| Main plugin file | simply-wholesale-goods.php |
| PHP namespace | SimplyWholesaleGoods |
| Function, hook, and metadata prefix | swg |
| Vendor role slug | swg_wholesale_vendor |
| Vendor role display name | Wholesale Vendor |
| Wholesale ordering capability | swg_order_wholesale |
| Wholesale management capability | swg_manage_wholesale |
| Bulk form shortcode | [swg_wholesale_order] |
| Admin menu | WooCommerce > Wholesale |

Use the plugin for all wholesale behavior. A child theme is not required.

Activation MUST check for WooCommerce and fail gracefully when it is unavailable.
Initialize roles and default settings idempotently. Updates MUST preserve users,
products, settings, and historical order metadata.

Provide a normal WordPress page containing the bulk form shortcode. Store its page ID
in the plugin settings. A one-click "Create wholesale order page" setup action is
sufficient; it must not overwrite unrelated pages or create duplicates when repeated.
Use "Wholesale ordering" as the default page title.

Ordinary deactivation or uninstall MUST NOT delete customer accounts, orders, purchased
option snapshots, or store product data. Preserve plugin data by default.

Wholesale privacy depends on the plugin being active. The maintenance instructions MUST
tell the maintainer to disable wholesale variations before deactivating/removing the
plugin on the live site. Do not claim that WooCommerce understands the custom wholesale
marker while the plugin is inactive.

## 4. Roles and vendor management

### 4.1 Role model

Wholesale Vendor has customer-level privileges plus swg_order_wholesale. It MUST NOT
receive administrative, product-editing, user-management, or private-page-reading
privileges merely to access wholesale ordering.

Grant swg_manage_wholesale to Administrator. Keep administration and shopping capabilities
separate: Keldy does not become a wholesale-only shopper because she manages the plugin.

The actual WordPress role assignment is the source of truth. Do not create a second
vendor database or approval flag.

### 4.2 WooCommerce > Wholesale

Register a submenu under WooCommerce using normal WordPress admin styling. It contains:

- **Vendors**, the default tab.
- **Settings**.

The Vendors tab MUST provide a vendor count, search by name/email, the current vendor
list, an Add vendor action, and a link to native user creation.

| List column | Content |
| --- | --- |
| Name | Customer display name with an edit-account link |
| Business | Existing billing company, when available |
| Email | Existing account email |
| Actions | Edit account; Remove wholesale access |

Use ordinary pagination for the account list. Do not add sales analytics, messaging,
application review, or additional business-profile fields.

### 4.3 Granting access

1. Keldy chooses Add vendor.
2. She searches existing customer accounts by name or email.
3. She selects an account and grants wholesale access.
4. The account becomes Wholesale Vendor and appears in the list.

The convenience workflow accepts ordinary Customer accounts and existing Wholesale
Vendors. Exclude staff accounts and accounts with additional privileged roles or
capabilities. Never replace Keldy's Administrator role through this picker.

Approving an existing vendor is an idempotent "already a vendor" result.

For someone without an account, link to the native WordPress Add User screen. Keldy
selects Wholesale Vendor there and can use WordPress's normal account notification and
password setup. Do not build another account-registration system or assume an
undocumented role-preselection URL parameter.

### 4.4 Removing access

"Remove wholesale access" changes the eligible vendor account back to Customer. It
preserves the account ID, addresses, orders, and invoices. The list updates immediately,
and the person can be approved again later.

Changes made through the native Users screen MUST appear automatically in the custom
vendor list. Both interfaces operate on the same role data.

### 4.5 Authorization

Every administration mutation MUST validate the management capability, applicable
native user/product editing permissions, and a valid nonce. Menu visibility and browser
controls are not authorization.

A privileged catalog-management request may see the full catalog. Do not use a blanket
is_admin() bypass: WordPress defines that function as an administration-context check,
not a check that the current user is an administrator. [S3]

## 5. Product and case model

### 5.1 Native variations

The representative soup already uses Size variations:

- Family - feeds 4-6.
- Half - feeds 2-3.

Wholesale bags and cases MUST be native WooCommerce variations under the same product.
Every variation has its independently entered native WooCommerce price. The same
wholesale variation price applies to all approved vendors, regardless of quantity.

Add a **Wholesale only** checkbox to the existing variation editor. Save its boolean
value as protected variation metadata, proposed key _swg_wholesale_only. Unmarked
variations are retail-only. Disabled or otherwise unpurchasable variations remain
unavailable.

Use an explicit marker, not labels, category names, or a comparison between retail and
wholesale prices. Do not create a separate wholesale-price table.

The plugin MUST work with the product's configured attributes. A mandatory migration to
a particular attribute schema is not required. If a new packaging attribute is added
during setup, preserve existing retail variation identities and assign complete attribute
combinations; avoid wildcard combinations that overlap wholesale choices.

WooCommerce already provides per-variation pricing and attributes. [S4]

### 5.2 Bags and cases

Keldy can offer wholesale bags, cases, or both for a product. Disabling the wholesale
bag variation makes that size available wholesale only by case, while the corresponding
retail variation can remain available to retail shoppers.

Example prices are illustrative:

| Offering | Native variation price | Native quantity meaning |
| --- | ---: | --- |
| Retail bag | $10.00 | One retail bag |
| Wholesale bag | $5.00 | One wholesale bag |
| Wholesale Case of 10 | $50.00, later changed manually to $45.00 | One case |

Two $45 cases MUST create quantity 2 and a $90 line subtotal. Do not expand that into
quantity 20 or calculate the case price from the bag price.

There is no automatic server calculation or admin JavaScript helper for case prices.
There is no extra numeric case-size field needed for inventory or pricing.

All wholesale purchase quantities MUST be positive whole quantities on product,
bulk-form, cart, fly-cart, API, and checkout paths.

### 5.3 Changing packaging

Use descriptive options such as Case of 10 and Case of 20.

When packaging changes, create the new option/variation and disable the old variation
for future purchases. Do not silently change the meaning of an existing purchased
variation. Reordering an unavailable Case of 10 MUST NOT silently substitute Case of 20.

Purchased wording must remain stable even if a product, variation, or taxonomy label is
later edited or deleted. Section 10 defines the order snapshot.

## 6. Storefront audience policy

### 6.1 Shopping rules

| Shopper/context | Visible and purchasable offerings |
| --- | --- |
| Logged out | Retail-only |
| Ordinary Customer | Retail-only |
| Wholesale Vendor | Wholesale-only |
| Authorized catalog management | Full editable catalog |

A Wholesale Vendor MUST NOT see retail choices as shopping options. If a product has no
eligible offerings for the current audience, hide it from storefront discovery and
return an appropriate not-found/unavailable result for a direct product URL.

For v1, unmarked simple, grouped, and external products are retail-only. Wholesale
offerings use the variation model. There is no mixed retail/wholesale cart mode.

The "wholesale-only anywhere" shopping rule does not erase a customer's historical
purchases. Owners retain authorized access to their own existing order records.

### 6.2 Apply one policy to all relevant paths

Use one authoritative eligibility service throughout the plugin. Cover:

- Shop and category lists, search, related products, and enabled recommendations.
- Product options, both installed swatch integrations, and Additional Information.
- Variation JSON embedded in HTML and variation AJAX responses.
- Public-facing product API responses, including exposed variation endpoints.
- Displayed minimum prices, "from" prices, price ranges, and structured offer data.
- Normal add-to-cart, direct variation-ID requests, and AJAX add-to-cart.
- Quantity updates, main cart, mini/fly cart, restored carts, and checkout.
- Reorder requests and Cart Abandonment Recovery paths in use on this site.

CSS or client-side hiding alone does not satisfy this requirement. Rendering one filtered
control while returning unauthorized variations elsewhere in the response also fails.

Preserve fully authorized product management/API operations. Determine their permissions
and purpose explicitly; an authenticated customer or an administrative AJAX URL does not
establish management authority.

### 6.3 Cache behavior

Audience-specific variation sets and prices MUST NOT be written into shared product
state or reused across retail and wholesale contexts. Use appropriate audience factors
for caches that contain filtered results. All vendors currently share one price list, so
per-vendor price caches are unnecessary.

WooCommerce documents role/user factors for dynamic variation-price cache keys. [S5]
The site's Object Cache Pro makes warm-cache tests mandatory. Also inspect full-page
cache behavior on staging; object caching alone does not describe that configuration.

Role changes and catalog edits MUST take effect without Keldy manually clearing a cache.

### 6.4 Session changes

On login, logout, role changes, and cart restoration, revalidate item eligibility before
displaying cart contents or accepting checkout. Remove currently ineligible shopping
items with a clear notice, while preserving eligible cart contents.

Apply this to session merging and stale mini-cart output as well as the main cart.
Historical order ownership and payment authorization are separate from new shopping
eligibility; revocation does not itself cancel an existing order.

## 7. Wholesale page and bulk order form

### 7.1 Access states

| State | Page content |
| --- | --- |
| Logged out | Approved-vendor explanation, sign-in option, and instructions to request access |
| Logged in without vendor access | Not-approved explanation and existing store contact route |
| Approved vendor | Wholesale bulk order form |

After login, return the vendor to the wholesale page.

Use server-side gating of the shortcode and its submission endpoint. The page can be a
published WordPress page with protected content; do not rely on WordPress's Private page
status as the vendor authorization model. Add noindex behavior to the wholesale page
and show its shopping navigation link to approved vendors.

Default access copy:

> Wholesale ordering is available to approved vendors. Sign in with your vendor account,
> or contact Simply Homemade Goods to request wholesale access.

Use the site's existing contact route when configured. Do not create a vendor application
form or approval-email automation.

### 7.2 Form layout

List eligible, enabled wholesale variations grouped under their product names. Include
the purchased option label, unit price, blank quantity input, and line subtotal. Show a
running merchandise subtotal and an **Add to cart and checkout** button.

The displayed subtotal is a merchandise estimate. The selected payment method and any
existing delivery charge determine the final amount at checkout.

Keep the form responsive, keyboard accessible, and readable in Astra. Each quantity input
needs an accessible label that identifies its product and option. Submission feedback must
be visible and announced appropriately. Use vanilla JavaScript for convenience calculations.

### 7.3 Quantity normalization

Quantity inputs MUST start blank. Zero is never required.

Normalize quantities in both PHP and JavaScript. The server is authoritative. Parse the
entire value; never use partial parsing or truncate fractions.

| Submitted quantity | Result |
| --- | --- |
| Missing, null, empty, or whitespace | Ignore as zero |
| Zero | Ignore |
| Positive whole number, e.g. 2 or "2" | Use that quantity |
| Whitespace around a valid number | Trim, then validate |
| Leading zeros, e.g. "002" | Use 2 |
| Ordinary decimal representation with zero fractional part, e.g. "2.0" | Use 2 |
| Negative or nonintegral, e.g. -1 or 2.7 | Ignore as zero |
| Malformed, e.g. "2abc" or "two" | Ignore as zero |
| NaN, Infinity, booleans, arrays, objects | Ignore as zero |
| Exponent, hex, or thousands-separated input, e.g. "2e3", "0x10", "1,000" | Ignore as zero |

The explicit input grammar above is a v1 implementation default. Accept ordinary
base-10 numeric input only, with an optional fractional part consisting entirely of zeros.
Ensure quantities and monetary multiplication remain within safe representable bounds.
Do not introduce a business quantity cap or a volume-price tier; quantities in the
millions are not intrinsically invalid.

Share normalization fixtures between PHP and browser tests. Browser-native validity
checking MUST NOT block the whole form before invalid rows can be normalized to zero.
An optional invalid field must not prevent another valid row from being ordered.

If all rows are ignored, show: **Enter a quantity for at least one item.**

### 7.4 Product validation and atomic submission

After quantity normalization, validate every positive row's variation identity, actual
parent product, wholesale marker, native purchasability, current price, and the current
user's permission.

An invalid quantity is skipped. An unauthorized or stale positive product selection is a
validation error. Do not silently treat product-authorization failures as zero quantities.

Validate the complete selected batch before changing the cart. If a selected item became
unavailable, show an actionable message and preserve entries for correction. Avoid partial
additions; if a later cart hook rejects an item, restore the pre-submission cart state for
that batch.

Combine identical variation lines and add them to existing eligible quantities. The normal
product page and bulk form MUST result in equivalent native cart lines and prices.

### 7.5 Submission integrity

Require vendor access and a valid nonce on every request. Ignore client-submitted prices,
discounts, totals, and claimed user roles.

Use a server-side submission token with an atomic claim/check so a double click, retried
request, or concurrent replay of the same batch cannot double-add it. A nonce or disabled
button alone is not duplicate-submission protection.

A fresh intentional submission uses a new token. Replaying a completed request returns
its existing success result. Failed validation must allow correction and resubmission.

## 8. Payments, discounts, and rounding

### 8.1 Gateway rules

Use core gateway IDs for the confirmed offline methods. They are cod, cheque, and bacs.
The corresponding WooCommerce classes define these IDs. [S6]

| Payment method | Current configuration | Percentage discount | Whole-dollar rounding |
| --- | --- | --- | --- |
| Cash on delivery, cod | Enabled | Yes | Yes, when rounding is on |
| Check, cheque | Enabled | Yes | Yes, when rounding is on |
| Direct bank transfer, bacs | Disabled | Yes whenever enabled | Yes |
| Stripe and other methods | Existing configuration | No | No |

The plugin MUST respect gateway availability. Supporting bank-transfer discounts does
not enable bank transfer. If Keldy later enables it through WooCommerce, the discount
works without a code change.

Discount eligibility and rounding eligibility MUST be separate policies. Do not treat
every non-Stripe method, or a display label containing "cash," as eligible.

These rules apply to wholesale purchases. Ordinary retail orders receive none of the
plugin's payment adjustments.

### 8.2 Settings

| Setting | Default | Behavior |
| --- | --- | --- |
| Offline payment discount percentage | 3.00% | Applies to cod, cheque, and bacs |
| Round cash/check total to a whole dollar | On | Applies only to cod and cheque |
| Wholesale page | Configured page ID | Used for navigation and login return |
| Vendor access instructions/contact link | Existing store contact route | Shown to visitors without approval |

Validate the percentage as a finite decimal from 0 through 100, with at most two decimal
places. A zero rate disables the percentage deduction. Rounding remains an independent
setting. This validation range is a technical input contract, not a volume-pricing feature.

Proposed settings storage is one versioned swg_settings option. Use decimal strings or
integer basis points for the percentage, not uncontrolled floating-point arithmetic.

### 8.3 Arithmetic contract

Use integer cents or exact decimal arithmetic for USD.

- S = wholesale merchandise subtotal at the selected variations' current native prices.
- F = existing shipping and unrelated fees.
- D = eligible percentage of S, rounded half-up to cents; otherwise zero.
- U = S - D + F.
- T = nearest whole dollar to U, ties upward, only for cash/check with rounding enabled;
  otherwise T = U.
- R = T - U, the signed rounding adjustment.

Tax is zero under the project's configured sales assumption. Do not add independent tax
calculations.

Calculate the aggregate discount consistently and allocate any cent remainder across
lines deterministically. Do not round each bag's percentage separately and accidentally
change the order-level discount. Native discount allocation is suitable when it satisfies
this arithmetic contract.

The payable WooCommerce order total MUST equal T. Formatting a decimal total as a whole
number without changing the actual amount is not acceptable.

### 8.4 Acceptance examples

| Scenario | Merchandise | Shipping | Discount | Before dollar rounding | Rounding | Payable |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Cash/check, rounding on | $120.00 | $0.00 | -$3.60 | $116.40 | -$0.40 | $116.00 |
| Cash/check, rounding off | $120.00 | $0.00 | -$3.60 | $116.40 | $0.00 | $116.40 |
| Bank transfer when enabled | $120.00 | $0.00 | -$3.60 | $116.40 | $0.00 | $116.40 |
| Stripe | $120.00 | $0.00 | $0.00 | $120.00 | $0.00 | $120.00 |
| Cash/check with delivery charge | $100.00 | $7.50 | -$3.00 | $104.50 | +$0.50 | $105.00 |

Additional rounding fixtures:

| Before rounding | Rounding adjustment | Final total |
| ---: | ---: | ---: |
| $149.59 | +$0.41 | $150.00 |
| $116.49 | -$0.49 | $116.00 |
| $116.50 | +$0.50 | $117.00 |
| $116.00 | $0.00 | $116.00 |

Follow nearest-dollar rounding literally. On a very small order it can erase a small
discount or increase the result above the card amount: $5.80 minus a $0.17 discount is
$5.63, which rounds to $6.00. A sufficiently small positive total can round to zero.
Do not silently add a minimum order, cap, or "round down only" exception.

### 8.5 Totals integration

Use a shared monetary service for checkout and existing-order recalculation.

The percentage deduction may use native WooCommerce discount machinery, including a
server-controlled internal coupon, or appropriately allocated native line discounts.
If an internal coupon is used, it MUST NOT be obtainable by manually submitting its code
and MUST NOT be confused with a promotional coupon.

Show clear logical rows such as:

- Offline payment discount (3%).
- Cash/check rounding.
- Total.

The displayed percentage label follows the configured or saved order rate. Store the
actual discount and signed adjustment. The displayed components must reconcile to the
saved grand total.

Rounding needs a dedicated adapter for authoritative cart totals, order creation, native
order recalculation, and document output. A possible implementation uses the cart total
filter and the order's post-recalculation hook with persisted signed adjustment data.
The exact hooks must be verified against the installed versions before release. [S7]

The integration MUST be idempotent through repeated AJAX checkout updates, session
restoration, order save/reload, admin recalculation, and payment retries. Never calculate
a new discount from an already-discounted amount.

Do not pass negative values to WC_Cart::add_fee; its documented contract prohibits them.
A one-time order set_total call is also insufficient because normal recalculation rebuilds
totals. Prove the entire lifecycle with tests. [S7]

Before creating, updating, or confirming a Stripe payment attempt, the persisted order
amount and the amount sent to Stripe MUST reflect the currently selected method.

### 8.6 Promotional coupons

Wholesale orders MUST reject promotional coupons, including manual entry, direct
requests, automatic application, URL-based application, and restored-session coupons.

Hide wholesale coupon entry controls and enforce the restriction on the server. Retail
coupon behavior remains native. Auto Coupons is installed and documents automatic and
URL application, making these explicit integration paths. [S8]

A controlled internal payment adjustment is not a promotional coupon. If implemented
using coupon APIs, distinguish it securely so the wholesale restriction does not reject
the plugin's own incentive.

## 9. Switching an existing offline order to card

A vendor may submit a cash/check/bank-transfer order and later request card payment.
The offline discount MUST be removed, and any cash/check rounding MUST also be removed,
before collecting the card payment.

### 9.1 Stored amounts govern the conversion

Use the original order's stored base prices, quantities, packaging, shipping/fees, and
saved policy. Do not fetch current catalog prices or current case labels to reconstruct
the amount owed.

A $120 order placed for cash at $116 returns to $120 for Stripe even if the current
catalog price has changed. A bank-transfer version of that same order is $116.40.

Do not change paid, captured, partially refunded, or refunded transactions through an
automatic method-switch recalculation. Refunds remain native WooCommerce operations
against the amount actually paid; preserve the original adjustment records.

### 9.2 Concrete admin workflow

Provide a focused **Prepare card payment** action on an eligible wholesale order:

1. Keldy confirms that the offline payment has not been collected.
2. Validate authorization, nonce, original order ownership/data, and that the order has
   no captured online payment, refund, completion, or other evidence of settlement.
3. Restore saved base amounts and remove the offline discount and rounding.
4. Persist the corrected amount and put the eligible order into an appropriate payable
   state.
5. Expose the native WooCommerce payment URL for the existing Stripe integration.
6. Add an order note recording the actor, original method/amount, new amount, and the
   confirmation that payment was uncollected.

Use the existing customer-invoice/payment-link email mechanism when Keldy chooses to send
it. Do not add a separate messaging system.

Core COD commonly puts physical orders into Processing before collection, and WooCommerce's
is_paid check is status-based. Do not use status alone as evidence that cash was received.
The normal order-pay eligibility also needs attention for On hold/Processing offline
orders. Verify the installed behavior, including date-paid handling, and keep this
conversion explicit and auditable. [S9]

The conversion must not bypass native order ownership/payment-key authorization.
Wholesale-role revocation stops new wholesale shopping but does not itself erase an
existing debt or authorize another person to pay/view the order.

If an order lacks the required snapshots or its lines were edited inconsistently with
them, stop the conversion with an actionable admin message. Do not guess a historical
price from today's catalog. Native unpaid-order edits must be reconciled with the saved
base amounts before conversion.

### 9.3 Method changes during an authorized payment flow

For any method selection offered by the existing payable-order flow, derive totals from
the same saved base:

| Change | Result |
| --- | --- |
| Cash/check to Stripe | Remove discount and rounding |
| Cash/check to bank transfer | Keep discount; remove rounding |
| Bank transfer to cash/check | Keep discount; apply saved rounding policy |
| Any eligible offline method to another ineligible method | Remove both adjustments |

A retry must not compound adjustments or allow a stale payment attempt to charge an
earlier amount. Paid transactions must remain immutable under this logic.

Do not reprice an order while a payment attempt is in flight or authorized but awaiting
capture. Resolve or cancel that attempt through the existing gateway, confirm its resulting
state, and only then permit another payment-method conversion. Use an order-level lock
or equivalent atomic guard so a concurrent conversion cannot race payment confirmation.

## 10. Historical order data

Use WooCommerce order and order-item CRUD APIs. The current site uses traditional order
storage, but direct order reads/writes through posts/postmeta would unnecessarily tie the
plugin to that storage. Validate HPOS compatibility before declaring it. [S10]

Native order lines already store product/variation identity, quantity, subtotals, and
totals. Add immutable human-readable option snapshots because resolving a saved taxonomy
slug against today's term name is not sufficient to preserve wording. [S11]

### 10.1 Proposed metadata contract

Names below are implementation identifiers; maintain a schema version for future changes.

| Location | Proposed metadata | Meaning |
| --- | --- | --- |
| Variation | _swg_wholesale_only | Explicit wholesale eligibility marker |
| Order | _swg_wholesale_order | Classification captured at order creation |
| Order | _swg_schema_version | Snapshot/adjustment schema version |
| Order | _swg_payment_policy | Saved percentage, rounding toggle, and eligibility policy version |
| Order | _swg_discount_total | Actual plugin discount |
| Order | _swg_rounding_adjustment | Signed rounding amount |
| Order | _swg_unrounded_total | Payable amount before whole-dollar rounding |
| Order item | _swg_option_snapshot | Attribute labels and human-readable purchased values |
| Order item | _swg_base_unit_price | Native purchased unit price before this plugin's adjustments |
| Order item | _swg_base_line_subtotal | Base merchandise amount for the saved quantity |
| Order/item as needed | _swg_base_snapshot | Shipping/fee and reconciliation data required for safe method changes |

Avoid redundant copies of data already persisted natively unless a snapshot is necessary
to restore the pre-adjustment state. Internal metadata must not clutter customer output.

Display the frozen purchased option wording once in order details, emails, admin views,
and PDFs. Avoid duplicated attribute rows or silently substituting current labels.

A later bag price, case price, case size, role assignment, or plugin-setting change MUST
NOT rewrite completed purchase history. An older invoice for quantity 2 of Case of 10
must remain two cases of ten.

## 11. Implementation structure and standards

A small set of focused PHP modules is sufficient:

| Responsibility | Purpose |
| --- | --- |
| Plugin lifecycle | Dependencies, roles, defaults, setup, version migrations |
| Audience policy | One source of truth for current shopping eligibility |
| Administration | Vendor management, settings, variation checkbox |
| Catalog integration | Queries, selectors, public data, price displays, caches |
| Bulk ordering | Quantity normalization, validation, duplicate protection, cart additions |
| Payment policy | Exact monetary calculations and gateway eligibility |
| Cart/order integration | Persist and reconcile adjustments before payment |
| Order presentation | Frozen options and consistent totals across existing outputs |

These are boundaries, not a requirement for a framework or a separate class for every
hook. Prefer direct WordPress/WooCommerce APIs and narrowly scoped adapters.

Use PHP compatible with the reported PHP 8.2 environment. Use namespaces/prefixes to
avoid collisions. Use suitable PHPDoc and comments explaining intent and non-obvious
lifecycle behavior.

Use plain ES6+ JavaScript with JSDoc where helpful:

- Four-space indentation.
- No semicolons.
- Single quotes.
- No trailing commas.
- Approximately 100-character line width.
- Vanilla JavaScript unless a concrete integration requires an existing dependency.

Use two-space YAML indentation and UTF-8-safe files. PHP retains required PHP syntax,
including semicolons; JavaScript formatting rules do not make invalid PHP acceptable.

Validate and sanitize input, escape output for its context, and use capability checks and
nonces where appropriate. Do not log secrets, payment credentials, full request bodies,
or unnecessary customer details.

Use meaningful comments for monetary decisions, snapshot invariants, authorization
boundaries, and replay protection. Avoid comments that merely restate simple code.

## 12. Developer tooling

### 12.1 Composer and PHP

Composer manages PHP development dependencies and named scripts. Commit composer.json
and composer.lock. Composer supports executable commands and PHP script callbacks,
including the @php executable selector. [S12]

Pin PHPUnit to the actual WordPress/WooCommerce test harness. WordPress publishes a
runner compatibility matrix; the latest PHPUnit release is not automatically the right
choice for the target integration environment. [S13]

Composer installation alone is not the entire integration-test setup. Explicitly provide
the WordPress test library, WooCommerce bootstrap, isolated database, and fixture setup.

### 12.2 Node and the local site

Use a small package.json with private set to true for development tools:

- Playwright Test, with tests written in JavaScript.
- @wordpress/env for a repeatable local site and test environment.

Commit the npm lock file. Use the Docker runtime for the WordPress/database integration
environment. Pin the intended WordPress/WooCommerce/PHP versions in the environment
configuration and document the installed Stripe/plugin fixtures. Playwright's JavaScript
runner and wp-env use Node. [S14]

Keldy's production site requires none of Composer, Node, PHPUnit, Docker, or the test
dependencies. She installs the completed plugin ZIP.

### 12.3 Command interface to implement

These are proposed repository scripts, not commands that already exist.

| Command | Contract |
| --- | --- |
| composer install | Install locked PHP development dependencies |
| npm ci | Install locked Node development dependencies |
| npm run env:start | Start the configured local WordPress environment |
| npm run env:stop | Stop it |
| composer test:unit | Run isolated PHP logic tests |
| composer test:integration | Run real WordPress/WooCommerce integration tests |
| composer test | Run both PHP suites |
| npm run test:e2e | Run Playwright browser/API acceptance tests |
| composer build | Assemble the installable ZIP |
| composer check | Run the documented PHP release checks |

Document prerequisites and the exact host/container invocation for each command. Make
the wrapper scripts consistent so the maintainer does not have to remember container
paths or manually bootstrap a database for every run. Include setup troubleshooting in
the developer README.

Jest is optional only if a meaningful isolated JavaScript unit suite justifies it. Do not
add another runner merely to duplicate Playwright or PHP coverage.

## 13. Automated acceptance criteria

Use deterministic fixtures: Administrator, ordinary Customer, two Wholesale Vendors,
a retail-only product, a product with wholesale bags and cases, a cases-only wholesale
product, an unavailable variation, and representative existing orders.

Use realistic selected plugin fixtures for compatibility checks and test payment modes.
The normal test environment must not send live customer emails or collect real payments.

### 13.1 Roles, administration, and access

| ID | Required result |
| --- | --- |
| A01 | Administrator can list, search, approve, revoke, and reapprove vendors |
| A02 | Native Users changes and the Wholesale vendor list stay synchronized |
| A03 | Grant/revoke preserves account identity, addresses, and order history |
| A04 | Repeated grant/revoke requests are idempotent |
| A05 | Customers/guests cannot invoke administration mutations; staff accounts cannot be demoted by the vendor picker |
| A06 | Guests and Customers see/buy retail only; Wholesale Vendors see/buy wholesale only |
| A07 | Products with no eligible choices are absent from discovery and unavailable by direct URL |
| A08 | Unauthorized variation IDs cannot be purchased through direct, AJAX, API, fly-cart, or reorder requests |
| A09 | Embedded data, search results, price ranges, attributes, and structured offers obey the audience policy |
| A10 | Retail-first and vendor-first warm-cache sequences return correct results |
| A11 | Role revocation, login/logout, and restored carts revalidate shopping eligibility without a manual cache clear |
| A12 | Historical order access remains correct after role changes; other customers cannot read those orders |

### 13.2 Product and bulk-order behavior

| ID | Required result |
| --- | --- |
| B01 | Each wholesale variation uses its independently entered native price |
| B02 | Two $45 Case of 10 variations produce quantity 2 and $90 subtotal |
| B03 | Disabling wholesale bags produces cases-only wholesale availability without hiding retail bags from retail shoppers |
| B04 | Blank/invalid quantity fields are skipped and valid rows still submit |
| B05 | Fractional, negative, malformed, nonfinite, and structured quantity inputs never become unintended positive purchases |
| B06 | An all-skipped form shows the specified friendly message |
| B07 | Browser validation does not prevent the server's normalization behavior |
| B08 | A stale/unauthorized positive product row rejects the batch without partial additions |
| B09 | Existing eligible cart quantities are preserved and incremented correctly |
| B10 | Duplicate/retried/concurrent requests with the same submission token do not double-add |
| B11 | Ordinary product and bulk form purchases produce equivalent cart lines |
| B12 | Whole quantities are enforced on every wholesale quantity-update path |
| B13 | Case label and price edits cannot rewrite an older order or invoice |

### 13.3 Payments and monetary history

| ID | Required result |
| --- | --- |
| M01 | cod and cheque receive the configured percentage discount |
| M02 | bacs receives the discount when enabled, receives no dollar rounding, and is not enabled by plugin activation |
| M03 | Stripe/other ineligible methods receive neither adjustment |
| M04 | Rounding defaults on; switching it off preserves the percentage discount |
| M05 | All section 8 arithmetic fixtures pass, including half-dollar and small-total cases |
| M06 | Multi-line discount allocation reconciles exactly to the declared aggregate deduction |
| M07 | Repeated cart and order recalculations never compound adjustments |
| M08 | Manual, automatic, URL, direct-request, and restored promotional coupons cannot affect wholesale totals |
| M09 | Any controlled internal adjustment still works while promotional coupons are denied |
| M10 | Offline-to-card conversion restores historical base prices before payment, even after catalog prices change |
| M11 | COD Processing status alone does not block a legitimate explicitly uncollected conversion or prove collection |
| M12 | Settled/refunded orders cannot be repriced by method switching |
| M13 | Retries cannot collect stale Stripe amounts; in-flight/authorized attempts and concurrent conversions are resolved before repricing |
| M14 | Checkout, saved order, native emails, My Account, admin totals, and PDF invoice totals agree |
| M15 | Later plugin-setting changes do not reprice old orders |
| M16 | Rounding and discount amounts remain auditable and preserve actual-paid/refund history |

### 13.4 Packaging and installation

| ID | Required result |
| --- | --- |
| P01 | Built ZIP contains one stable simply-wholesale-goods root directory and the main plugin file |
| P02 | Development dependencies, tests, caches, local config, and Git metadata are absent from the distributable |
| P03 | Any required runtime dependency/autoloader is present |
| P04 | Version tag, plugin header, and ZIP filename agree |
| P05 | The built ZIP installs and activates on a clean target WordPress/WooCommerce fixture |
| P06 | Upgrade preserves settings, roles, variation flags, and historical orders |
| P07 | Missing WooCommerce produces a useful administrative result rather than an uncontrolled fatal error |

CI MUST run the PHPUnit unit and integration suites and the deterministic Playwright
browser/API acceptance suite. A release MUST fail if any required suite fails. Use
Playwright for real UI and request boundaries, and give critical calculation and
authorization rules direct tests rather than relying on screenshots alone.

Maintain a focused compatibility run against Keldy's actual stack. Do not claim a broad
WordPress/plugin-version support matrix from tests of one site.
The fuller compatibility run with her configured third-party extensions may be a separate
staging release gate; it does not replace the browser suite required in CI.

## 14. Packaging and GitHub releases

Use the repository m-scott-lassiter/simply-wholesale-goods for source, issues, documentation,
and release tracking. The display name remains **Simply Wholesale Goods**.

Include at least:

| Repository path | Purpose |
| --- | --- |
| simply-wholesale-goods.php | Plugin bootstrap/header |
| src/ | PHP implementation |
| assets/ | Production JavaScript/CSS |
| templates/ | Plugin-owned output templates, if needed |
| tests/ | Unit, integration, and browser tests |
| tools/build.php | Packaging script |
| docs/design-spec.md | This specification |
| README.md | Developer setup and Keldy's operating instructions or links to them |
| composer.json and composer.lock | PHP tooling |
| package.json and package-lock.json | Development Node tooling |
| .wp-env.json | Local environment |
| .github/workflows/ | CI and release workflows |

The packaging script uses a deliberate production-file inclusion list. PHP's ZipArchive
extension is suitable; provision it on the build machine/CI runner. Include runtime
dependencies intentionally and omit development-only files. [S15]

Example output:

- dist/simply-wholesale-goods-1.0.0.zip
- Archive root: simply-wholesale-goods/
- Entry point inside archive: simply-wholesale-goods/simply-wholesale-goods.php

Keep the internal directory and entry point stable across versions. Read the release
version from the maintained plugin version source and fail if it disagrees with the tag.

### 14.1 CI and release sequence

1. Push/PR checks run the documented required tests.
2. A version tag, such as v1.0.0, identifies the release source.
3. The release workflow checks out that exact tag and installs locked dependencies.
4. It runs required checks, builds the ZIP, and performs the installation smoke test.
5. It attaches the validated ZIP and release notes to the GitHub Release.
6. The maintainer downloads that asset and supplies it to Keldy.

GitHub supports tagged releases and attached assets; Actions can use the GitHub CLI to
upload the ZIP. Its automatic "Source code" archive contains the repository at the tag
and is distinct from the deliberately packaged plugin asset. [S16]

Keldy uses WordPress's plugin upload/install workflow. Updates use the same stable plugin
identity. No production credentials or automatic live-site deployment belong in the
initial release workflow.

## 15. Implementation order and release gates

### Phase 1: Repository and test foundation

Create the agreed repository, establish Composer/Node commands, configure the isolated
WordPress/WooCommerce environment, and add realistic fixtures. Build an installable empty
plugin package and verify activation/dependency handling.

### Phase 2: Prove the two integration risks

Before extensive UI work, prove:

- Native variation audience separation, including the site's swatches, search, direct
  purchase requests, and populated caches.
- Discount/rounding persistence and an offline-to-Stripe payment conversion through the
  actual installed Stripe plugin.

Document the selected WooCommerce hooks/adapters and their tested lifecycle behavior.
A display-only proof is not sufficient for either risk.

### Phase 3: Finish the functional workflow

Implement vendor administration, settings, the variation marker, the bulk form, snapshots,
promotional-coupon rejection, and document presentation. Complete the acceptance cases.

### Phase 4: Staging and release

Confirm the real cart-to-checkout route, actual payment IDs, pickup/shipping behavior,
tax-free configuration, existing invoice templates, and any express payment path in use.
Run the focused compatibility suite with the reported extensions.

Build and install the release ZIP on staging, then verify a retail purchase and each
enabled wholesale payment path. Production installation is a separate deployment step.

## 16. Keldy's operating guide requirements

Include concise instructions for:

1. Opening WooCommerce > Wholesale to view/search vendors.
2. Adding an existing customer or creating a new account through Users.
3. Removing/reinstating wholesale access.
4. Creating wholesale variations, checking Wholesale only, and entering their prices.
5. Offering bags and cases together or disabling wholesale bags.
6. Moving from Case of 10 to Case of 20 without changing historical purchases.
7. Adjusting the percentage and the independent cash/check rounding toggle.
8. Enabling bank transfer in WooCommerce later, with the discount applying automatically.
9. Recording offline collection through her existing order workflow and preparing an
   explicitly uncollected order for card payment.
10. Installing an update ZIP and the live-site deactivation precaution.

Keep these instructions in ordinary store-management language. Implementation identifiers,
test tooling, and cache keys belong in developer documentation.

## 17. Source notes and technical references

Business requirements and site facts come from the owner's planning conversation, the
November 10, 2026 WooCommerce status report, and the supplied CartFlows/product screenshots.
The external references below support platform behavior; the plugin's requirements are
design decisions, not claims that those platforms already provide the complete feature.

- **S1 - Classic WooCommerce pages:** [Page shortcodes](https://woocommerce.com/document/woocommerce-shortcodes/page-shortcodes/).
- **S2 - CartFlows global checkout:** [Store Checkout](https://cartflows.com/docs/store-checkout/).
- **S3 - Administration context and permissions:** [is_admin](https://developer.wordpress.org/reference/functions/is_admin/), [roles and capabilities](https://developer.wordpress.org/plugins/users/roles-and-capabilities/), [add_submenu_page](https://developer.wordpress.org/reference/functions/add_submenu_page/).
- **S4 - Native variations:** [Variable products](https://woocommerce.com/document/variable-product/).
- **S5 - Variation-price caching:** [WooCommerce variation-price cache guidance](https://developer.woocommerce.com/2026/01/08/variation-prices-caching-improvements-in-woocommerce-10-5/).
- **S6 - Offline gateway identities:** [COD source](https://woocommerce.github.io/code-reference/files/woocommerce-includes-gateways-cod-class-wc-gateway-cod.html), [check source](https://woocommerce.github.io/code-reference/files/woocommerce-includes-gateways-cheque-class-wc-gateway-cheque.html), [bank-transfer source](https://woocommerce.github.io/code-reference/files/woocommerce-includes-gateways-bacs-class-wc-gateway-bacs.html).
- **S7 - Monetary integration:** [WC_Cart](https://woocommerce.github.io/code-reference/classes/WC-Cart.html), [cart totals source](https://woocommerce.github.io/code-reference/files/woocommerce-includes-class-wc-cart-totals.html), [order totals source](https://woocommerce.github.io/code-reference/files/woocommerce-includes-abstracts-abstract-wc-order.html).
- **S8 - Existing coupon integration:** [Auto Coupons for WooCommerce](https://wordpress.org/plugins/woo-auto-coupons/).
- **S9 - Offline order status and payment:** [COD processing](https://woocommerce.github.io/code-reference/files/woocommerce-includes-gateways-cod-class-wc-gateway-cod.html), [WC_Order source](https://woocommerce.github.io/code-reference/files/woocommerce-includes-class-wc-order.html).
- **S10 - Order storage abstraction:** [HPOS extension recipe book](https://developer.woocommerce.com/docs/features/orders/high-performance-order-storage/recipe-book/).
- **S11 - Stored order items:** [WC_Order_Item_Product](https://woocommerce.github.io/code-reference/classes/WC-Order-Item-Product.html), [WP Overnight document hooks](https://docs.wpovernight.com/woocommerce-pdf-invoices-packing-slips/pdf-template-filter-hooks/).
- **S12 - Composer:** [Scripts](https://getcomposer.org/doc/articles/scripts.md), [locked dependencies](https://getcomposer.org/doc/01-basic-usage.md).
- **S13 - PHP test compatibility:** [WordPress PHPUnit matrix](https://make.wordpress.org/core/handbook/references/phpunit-compatibility-and-wordpress-versions/), [WooCommerce testing](https://developer.woocommerce.com/docs/contribution/testing/).
- **S14 - Development environment and browsers:** [wp-env](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/), [Playwright languages](https://playwright.dev/docs/languages).
- **S15 - PHP ZIP packaging:** [ZipArchive](https://www.php.net/manual/en/class.ziparchive.php).
- **S16 - Releases:** [GitHub releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases), [GitHub CLI in Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-github-cli), [uploading release assets](https://cli.github.com/manual/gh_release_upload).
- **S17 - Existing storefront extensions:** [Advanced Product Search](https://wordpress.org/plugins/advanced-product-search-for-woo/), [Modal Fly Cart](https://wordpress.org/plugins/woocomm-popup-cart-ajax/), [aThemeArt swatches](https://wordpress.org/plugins/variation-swatches-style/), [CartFlows swatches](https://wordpress.org/plugins/variation-swatches-woo/).
- **S18 - Installed Stripe integration distribution:** [Plugin listing](https://wordpress.org/plugins/checkout-plugins-stripe-woo/), [author support notice](https://wordpress.org/support/topic/weve-made-a-shift-but-youre-fully-supported/).

