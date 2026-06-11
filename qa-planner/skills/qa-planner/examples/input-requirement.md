# Sample Requirement: Checkout Promo Code

Project: Retail Web
Feature: Promo code validation during checkout

## Business Requirement

Customers can apply one promo code during checkout before payment. A valid promo code applies a discount to the order summary. An invalid, expired, or already used promo code must show a clear error message and must not change the payable total.

## Acceptance Criteria

- Valid promo code `SAVE10` applies 10 percent discount before payment.
- Expired promo code `SPRINGOLD` shows `Promo code has expired`.
- Already used promo code `ONCEONLY` shows `Promo code has already been used`.
- Empty promo code submission shows `Enter a promo code`.
- Removing an applied promo code restores the original total.
- Only one promo code can be active at a time.

## Constraints

- Payment provider sandbox is available.
- Promo configuration API endpoint is not documented.
- UI selector names are not confirmed.
