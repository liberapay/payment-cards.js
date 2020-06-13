payment-cards.js provides formatting, validation, and brand detection for payment card forms.

Dependencies: only jQuery.

Compatibility: obsolete browsers are not supported. For example IE ≤ 8 are not, because they don't have [addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener).

Maturity: payment-cards.js is used in production.

Maintenance: this project is maintained by the Liberapay team, you can [donate](https://liberapay.com/Liberapay/donate) to fund this work.

## Demo

You can test payment-cards.js on <https://liberapay.github.io/payment-cards.js/>.

## Accuracy Warning

Due to the lack of accurate and comprehensive public information, the validation of card data and the detection of a card's brand should not be blindly trusted.

Validity checks are only meant to help detect input errors early: you should warn your users when the data they have input appears to be invalid, but you should not prevent them from submitting that data.

## API

Terminology:

- PAN = Primary Account Number - the main number visible on the card, commonly called "the card number" - https://en.wikipedia.org/wiki/ISO/IEC_7812
- CVN = Card Verification Number - https://en.wikipedia.org/wiki/Card_security_code
- IIN = Issuer Identification Number - the first 6 digits of a PAN - https://en.wikipedia.org/wiki/Issuer_identification_number

### PaymentCards.Form(panInput, expiryInput, cvnInput)

This is the class that glues everything together, it's the most convenient way to use this library.

```js
var cardForm = new PaymentCards.Form(
    document.querySelector('input#pan'),
    document.querySelector('input#expiry'),
    document.querySelector('input#cvn')
);
```

If you want to do something with the inputs, like attaching extra event callbacks, you can access them through the `inputs` attribute: `cardForm.inputs.pan`, `cardForm.inputs.expiry`, `cardForm.inputs.cvn`.

When you want to check the card data, call the `check()` method:

```js
var card = cardForm.check();
```

The returned object has the following structure:

```js
{
    pan: Object  // the value and validation status of the PAN
    expiry: Object  // the value and validation status of the expiration date
    cvn: Object  // the value and validation status of the CVN
    range: Object or null  // the detected range of the card, or null if the IIN is unknown, see 'rangesArray' for details
    brand: String or null  // shortcut for 'range.brand'
}
```

The `pan`, `expiry`, and `cvn` objects all share the same structure:

```js
{
    value: String  // the non-formatted value of the field
    status: String or null
    description: String or undefined
    subfield: String or undefined
}
```

`status` is one of:

- `'valid'`: the value appears to be valid
- `'empty'`: the `<input>` doesn't contain any digits
- `'abnormal'`: the value appears to be invalid, it will probably be rejected if you attempt a payment
- `'invalid'`: the value is invalid, it will almost certainly be rejected if you attempt a payment
- `null`: the validity is unclear, because the IIN is unknown

`description` only appears when `status` is `'abnormal'` or `'invalid'`, it tells you in what way the data is wrong. Possible values include:

- `"too short"`, e.g. a CVN of less than 3 digits
- `"too long"`,  e.g. a PAN of more than 19 digits
- `"bad length"`, e.g. a PAN of 11 digits (to our knowledge no institution issues such numbers)
- `"luhn check failure"`, the PAN's last digit failed the [Luhn check](https://en.wikipedia.org/wiki/Luhn_algorithm)
- `"in the past"`, for the expiration date only

Obviously these are not meant to be shown to the user directly.

`subfield` only appears when a specific part of the expiration date is invalid, it's either `'month'` or `'year'`.

### PaymentCards.rangesArray

`rangesArray` contains the data used to guess a card's brand. Each item has the following structure:

```js
{
    brand: String  // the brand's name, see below for the values
    pattern: RegExp  // the regular expression matching the IINs of this range, e.g. /^4/ for Visa
    spacing: Array  // a list of integers indicating where to put spaces when formatting a PAN
    panLengths: Array  // the list of normal PAN lengths for this range
    cvnLengths: Array  // the list of normal CVN lengths for this range
}
```

The current `brand` strings are:

- `American Express`
- `Diners Club`
- `Discover`
- `JCB`
- `Maestro`
- `MasterCard`
- `UnionPay`
- `Visa`

We currently have two `spacing` arrays:

- the default one is `[4, 8, 12]`, i.e. spaces after the 4th, 8th, and 12th digits
  - example: 4242424242424242 → 4242 4242 4242 4242
- the other one is `[4, 10]`, for American Express and Diners Club cards
  - example: 34343434343434 → 3434 343434 3434

If you've seen a card with more than 16 digits or with different spacing, let us know.

### Lower-level Functions

- `PaymentCards.addSeparators(string, positions, separator)`
- `PaymentCards.checkCard(pan, expiry, cvn)`
- `PaymentCards.checkExpiry(expiry)`
- `PaymentCards.formatInputs(panInput, expiryInput, cvnInput)`
- `PaymentCards.getSpacing(cardNumber)`
- `PaymentCards.getRange(cardNumber)`
- `PaymentCards.luhnCheck(num)`
- `PaymentCards.restrictNumeric(input, maxLength, formatter)`

Note: unlike some other implementations we've seen our `restrictNumeric` function does not prevent the user from pasting into the restricted `<input>`, nor from using text selection to modify its content.

## License

[CC0 Public Domain Dedication](http://creativecommons.org/publicdomain/zero/1.0/)

## Alternatives

- https://github.com/jessepollak/card
  - "Make your credit card form better in one line of code"
  - This cool project attempts to show the user a realistic facsimile of their physical card, using the IIN to guess its brand and thus its appearance.
- https://github.com/jessepollak/payment
  - "A jQuery-free general purpose library for building credit card forms, validating inputs and formatting numbers"
  - This is the code underlying the above project, by the same author.
- https://github.com/PawelDecowski/jquery-creditcardvalidator
  - "jQuery credit card validation and detection plugin"
  - If you prefer a jQuery plugin this one looks okay (we haven't tried it).

All three of these projects use CoffeeScript, and jQuery or the minimal jQuery replica [QJ](https://github.com/jessepollak/qj). On the other hand `payment-cards.js` is written in light standard JavaScript.
