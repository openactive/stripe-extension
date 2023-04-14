# Open Booking API Stripe Extension

This is the repository of the specification and JSON-LD namespace for an extension for the [Open Booking API](https://openactive.io/open-booking-api/EditorsDraft/1.0CR3/) to handle Stripe integrated payments.

> Status: Draft | [Please Provide Feedback via GitHub](https://github.com/openactive/stripe-extension/issues)

## Status

This specification and namespace is in a draft state.

## Overview

This specification includes changes to C2 and B endpoints necessary to integrate the [Stripe Payment Intents API](https://stripe.com/docs/payments/payment-intents) with the Open Booking API.

It is based on the [Integrated Payments example](https://openactive.io/open-booking-api/EditorsDraft/1.0CR3/#extension-example-integrated-payments) in the Open Booking API specification, and Stripe's own [best practices](https://stripe.com/docs/payments/payment-intents#best-practices).

Please note:
- As per the Open Booking API specification, this extension should be implemented as an optional feature for Brokers that require it. Full conformance to the Open Booking API specification cannot be reached if all Brokers MUST use the extension to interact with the API. This is because enforcing a single payment provider severely limits the available use cases and business models within the OpenActive ecosystem.
- This extension cannot be integrated into the Open Booking API as it is vendor-specific, and hence it is not appropriate within the core of an open standard.

Additionally note that [Payment reconciliation detail validation](https://openactive.io/open-booking-api/EditorsDraft/1.0CR3/#payment-reconciliation-detail-validation) cannot be used in conjunction with this Stripe extension, as reconciliation is handled entirely by the Booking System in this case.

## Namespace

The namespace MUST be referenced using the URL `"https://openactive.io/stripe-extension"` (which will return the [JSON-LD definition](https://openactive.io/stripe-extension/stripe-extension.jsonld) if the `Accept` header contains `application/ld+json`), and is designed to be used in conjunction with the `"https://openactive.io/"` namespace.

## `@context`

When this extension is in use, the `"@context"` of all requests and responses to the Open Booking API should include the namespace reference as follows:

```json
{
  "@context": [
    "https://openactive.io/",
    "https://openactive.io/stripe-extension"
  ],
  "@type": "Order",
  ...
}
```

Note as above, in order to conform to the Open Booking API, a Booking System must respond to standard Open Booking API requests (that do not include this additional namespace in the `"@context"`) as per the Open Booking API specification.

## Requests and responses

### C2 response

The C2 response provides enough information for the Broker to Authorise the Payment Intent (invoking 3D Secure as necessary)

```json
"payment": {
  "@type": "stripe:PaymentIntent",
  "identifier": "pi_123456789", // payment intent ID
  "stripe:clientSecret": "sk_test_26PHem9AhJZvU623DfE1x4sd",
  "stripe:publishableKey": "pk_test_TYooMQauvdEDq54NiTphI7jx"
}
```

Note `payment` MUST NOT be supplied by the C1 or C2 request.

### B request and response

The Broker must include the Payment Intent identifier in the request, and the Booking System must reflect it back in the response:

#### B request
```json
"payment": {
  "@type": "stripe:PaymentIntent",
  "identifier": "pi_123456789", // payment intent ID
}
```

#### B response
```json
"payment": {
  "@type": "stripe:PaymentIntent",
  "identifier": "pi_123456789", // payment intent ID
}
```



# Namespace

