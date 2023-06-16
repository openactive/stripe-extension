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

## Errors

A pass-through approach for errors from the Stripe API ensures that the Broker has as much information as possible about any errors that occur, and can display messages to the end-user as appropriate.

### `OpenBookingError` subclasses

A number of `oa:OpenBookingError` subclasses are defined to identify the problem type of various error conditions. These are based on Stripe's own [error types](https://stripe.com/docs/api/errors#errors-type), and pass through any errors received from Stripe. All other errors defined here subclass `stripe:Error`, which subclasses `oa:OpenBookingError`.

| Error Type (`@type`)                        | Status Code (`oa:statusCode`) | Use Case (`schema:name`)                                                                                                                                                     |
|---------------------------------------------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `stripe:Error`                              | 400         | A problem with Stripe was encountered that did not have a known [error type](https://stripe.com/docs/api/errors#errors-type) |
| `stripe:ApiError`                           | 400         | API error from Stripe's servers |
| `stripe:CardError`                          | 400         | The user has provided a card that cannot be charged |
| `stripe:IdempotencyError`                   | 409         | An `Idempotency-Key` was re-used on a request to Stripe that did not match the first request's API endpoint and parameters |
| `stripe:InvalidRequestError`                | 400         | Request to Stripe has invalid parameters |
| `stripe:MissingPaymentIntentDetailsError`   | 400         | `stripe:PaymentIntent` details (e.g. `identifier`) were expected but not set. |
| `stripe:PaymentIntentMismatchError`         | 400         | `stripe:PaymentIntent` `identifier` does not match the `identifier` provided by the C2 response for this `Order`. |

Note that `stripe:IdempotencyError` and `stripe:InvalidRequestError` should not occur if the Booking System has implemented this extension correctly.

Note also that `stripe:MissingPaymentIntentDetailsError` and `stripe:PaymentIntentMismatchError` relate to issues with Broker behaviour, and do not originate from Stripe.

### `stripe:Error` property mapping

Attributes from a [Stripe error](https://stripe.com/docs/api/errors) are mapped to properties in [`OpenBookingError`](https://openactive.io/open-booking-api/EditorsDraft/1.0CR3/#oa-openbookingerror), with additional extension properties defined for use on the `stripe:Error`.

| Property                                                           | Stripe [attribute](https://stripe.com/docs/api/errors)      | Type | Notes |
|--------------------------------------------------------------------|-------------|-|------|
| `@type`                                                            | - | [`schema:Text`](https://schema.org/Text) | `OpenBookingError` |
| [`schema:name`](https://schema.org/name)                           | - | [`schema:Text`](https://schema.org/Text) | A short, human-readable summary of the problem type, that MUST communicate the associated "Use Case" defined above. It SHOULD NOT change from occurrence to occurrence of the problem, except for purposes of localization. |
| [`schema:description`](https://schema.org/description)             | [`message`](https://stripe.com/docs/api/errors#errors-message) | [`schema:Text`](https://schema.org/Text) | A human-readable message providing details about the specific occurrence of the problem. For `stripe:CardError`, these messages can be shown to end users. |
| [`schema:url`](https://schema.org/url)                             | [`doc_url`](https://stripe.com/docs/api/errors#errors-doc_url) | [`schema:URL`](https://schema.org/URL) | A URL to more information about the error code reported. |
| [`stripe:parameter`](https://openactive.io/stripe-extension#parameter)     | [`param`](https://stripe.com/docs/api/errors#errors-param) | [`schema:Text`](https://schema.org/Text) | If the error is parameter-specific, the parameter related to the error. For example, you can use this to display a message near the correct form field. |
| [`stripe:errorCode`](https://openactive.io/stripe-extension#errorCode)     | [`code`](https://stripe.com/docs/api/errors#errors-code) | [`schema:Text`](https://schema.org/Text) | For some errors that could be handled programmatically, a short string indicating the [error code](https://stripe.com/docs/error-codes) reported. |
| [`stripe:declineCode`](https://openactive.io/stripe-extension#declineCode) | [`decline_code`](https://stripe.com/docs/api/errors#errors-decline_code) | [`schema:Text`](https://schema.org/Text) | For card errors resulting from a card issuer decline, a short string indicating the [decline code](https://stripe.com/docs/declines/codes), which is the card issuer’s reason for the decline if they provide one. |
| [`oa:instance`](https://openactive.io/instance)                    | [`request_log_url`](https://stripe.com/docs/api/errors#errors-request_log_url) | [`schema:URL`](https://schema.org/URL) | A URL to the request log entry in the Stripe dashboard. |
| [`oa:statusCode`](https://openactive.io/statusCode)                | [`code`](https://stripe.com/docs/api/errors#errors-code)    | [`schema:Integer`](https://schema.org/Integer) | An integer representing the HTTP status code, that MUST match the associated "Status Code" defined above. |


### `stripe:Error` example

```json
{
  "@context": [
    "https://openactive.io/",
    "https://openactive.io/stripe-extension"
  ],
  "@type": "stripe:CardError",
  "name": "The user has provided a card that cannot be charged",
  "description": "Your card's security code is incorrect. Please try again using the correct CVC.",
  "stripe:parameter": "cvc",
  "stripe:errorCode": "card_declined",
  "stripe:declineCode": "incorrect_cvc",
  "instance": "https://dashboard.stripe.com/test/logs/req_1234567890abcdef",
  "statusCode": "400"
}
```


### Test Interface Action

Test interface actions are defined to simulate user interactions that would ordinarily be performed via on the frontend via Stripe Elements.

#### `stripe:ConfirmPaymentIntentSimulateAction`
This test action simulates the authorisation of a card by confirming the Payment Intent. This is useful for test code that uses [Stripe Test Mode](https://stripe.com/docs/test-mode) to test paid booking on the backend without requiring a frontend instance of Stripe Elements to authorise the card.

```json
{
    "@context": [
        "https://openactive.io/",
        "https://openactive.io/test-interface",
        "https://openactive.io/stripe-extension"
    ],
    "@type": "stripe:ConfirmPaymentIntentSimulateAction",
    "object": {
      "@type": "stripe:PaymentIntent",
      "identifier": "pi_1GPsnyKarmweGdVC5WhNworN" // payment intent ID
    },
    "stripe:paymentMethod": "pm_card_visa"
}
```

For a list of the various test values for `stripe:paymentMethod` (e.g. `pm_card_chargeDeclinedIncorrectCvc`) see the [Stripe documentation](https://stripe.com/docs/testing?testing-method=payment-methods#declined-payments).


# Namespace

