# Open Booking API Stripe Extension

This is the repository of the specification and JSON-LD namespace for an extension for the [Open Booking API](https://openactive.io/open-booking-api/EditorsDraft/1.0CR3/) to handle [Stripe](https://stripe.com/) integrated payments.

> Status: Draft | [Please Provide Feedback via GitHub](https://github.com/openactive/stripe-extension/issues)

## Status

This specification and namespace is in a draft state.

## Overview

This specification includes changes to C2 and B endpoints necessary to integrate the [Stripe Payment Intents API](https://stripe.com/docs/payments/payment-intents) with the Open Booking API.

It is based on the [Integrated Payments example](https://openactive.io/open-booking-api/EditorsDraft/1.0CR3/#extension-example-integrated-payments) in the Open Booking API specification, and Stripe's own [best practices](https://stripe.com/docs/payments/payment-intents#best-practices).

Please note:
- As per the Open Booking API specification, this extension should be implemented as an optional feature for Brokers that require it. Full conformance to the Open Booking API specification cannot be reached by a Booking System if all Brokers MUST use the extension to interact with the API. This is because enforcing a single payment provider severely limits the available use cases and business models within the OpenActive ecosystem.
- This extension cannot be integrated into the Open Booking API specification itself as it is vendor-specific, and hence it is not appropriate within the core of an open standard.

Additionally note that [Payment reconciliation detail validation](https://openactive.io/open-booking-api/EditorsDraft/1.0CR3/#payment-reconciliation-detail-validation) cannot be used in conjunction with this Stripe extension, as reconciliation is handled entirely by the Booking System in this case.

## Namespace

The namespace MUST be referenced using the URL `"https://openactive.io/stripe-extension"` (which will return the [JSON-LD definition](https://openactive.io/stripe-extension/stripe-extension.jsonld) if the `Accept` header contains `application/ld+json`), and is designed to be used in conjunction with the `"https://openactive.io/"` namespace.

## Context

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

### C2 request

The Broker uses a `"stripe:PaymentIntent"` to indicate an intention to use Stripe for integrated payments, as it would with Payment reconciliation detail validation. The Broker must specify the web page or app on which Stripe will be used. This allows the booking system to authorise and track the usage of Stripe, which will aid PCI-DSS compliance.

```json
"payment": {
  "@type": "stripe:PaymentIntent",
  "stripe:paymentPageUrl": "https://example.com/checkout"
}
```

### C2 response

The C2 response provides enough information for the Broker to Authorise the Payment Intent (invoking 3D Secure as necessary). It also reflects back the provided `payment` data.

```json
"stripe:paymentRequest": {
  "@type": "stripe:PaymentIntent",
  "identifier": "pi_1GPsnyKarmweGdVC5WhNworN", // payment intent ID
  "stripe:clientSecret": "pi_1GPsnyKarmweGdVC5WhNworN_secret_LgKKeZNVv4XlayiahHTt3YjHA",
  "stripe:publishableKey": "pk_test_4JnvQX1ZfhZacZh3ZiLOrAXq",
  "stripe:connectAccountIdentifier": "acct_24BFMpJ1svR5A89k"
},
"payment": {
  "@type": "stripe:PaymentIntent",
  "stripe:paymentPageUrl": "https://example.com/checkout"
}
```

### B request and response

The Broker must include the Payment Intent identifier in the request, and the Booking System must reflect it back in the response. This helps the Booking System to verify that the correct Payment Intent has been used by the Broker to accept a payment for this Order.

#### B request
```json
"payment": {
  "@type": "stripe:PaymentIntent",
  "stripe:paymentPageUrl": "https://example.com/checkout",
  "identifier": "pi_1GPsnyKarmweGdVC5WhNworN" // payment intent ID
}
```

#### B response
```json
"payment": {
  "@type": "stripe:PaymentIntent",
  "stripe:paymentPageUrl": "https://example.com/checkout",
  "identifier": "pi_1GPsnyKarmweGdVC5WhNworN" // payment intent ID
}
```

### Test Interface Action

Test interface actions are defined to simulate user interactions that would ordinarily be performed via on the frontend via Stripe Elements.

#### AuthorisePaymentIntentSimulateAction
This test action simulates the authorisation of a card by confirming the Payment Intent. This is useful for [Stripe Test Mode](https://stripe.com/docs/test-mode), to test paid booking without requiring Stripe Elements to authorise the card.

```json
{
    "@context": [
        "https://openactive.io/",
        "https://openactive.io/test-interface",
        "https://openactive.io/stripe-extension"
    ],
    "@type": "stripe:AuthorisePaymentIntentSimulateAction",
    "object": {
      "@type": "stripe:PaymentIntent",
      "identifier": "pi_1GPsnyKarmweGdVC5WhNworN" // payment intent ID
    }
}
```


# Namespace

