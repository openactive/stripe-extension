# Open Booking API Stripe Extension
This is the repository of the specification and JSON-LD namespace for an extension for the [Open Booking API](https://openactive.io/open-booking-api/EditorsDraft/1.0CR3/) to handle Stripe integrated payments.

It is based on the [Integrated Payments example](https://openactive.io/open-booking-api/EditorsDraft/1.0CR3/#extension-example-integrated-payments) in the Open Booking API specification, and Stripe's own [best practices](https://stripe.com/docs/payments/payment-intents#best-practices).

Please note:
- As per the Open Booking API specification, this extension should be implemented as an optional feature for Brokers that require it. Full conformance to the Open Booking API specification cannot be reached if all Brokers MUST use the extension to interact with the API. This is because enforcing a single payment provider severely limits the available use cases and business models within the OpenActive ecosystem.
- This extension cannot be integrated into the Open Booking API as it is vendor-specific, and hence it is not appropriate within the core of an open standard.

Please find documentation relating to this specification and namespace at https://openactive.io/stripe-extension.

## Contribution

Please create a pull request amending the file `stripe-extension.jsonld` with the relevant terms, which includes the rationale for such proposed amendments. Documentation is automatically generated from this file when the PR is merged into the `master` branch.

## Tooling dependencies

The OpenActive validator (https://validator.openactive.io) will pick up changes to the namespace immediately.
