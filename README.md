# OMISE-ANDROID-SDK

[![](https://img.shields.io/maven-central/v/co.omise/omise-android.svg?style=flat-square)](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22omise-android%22) [![](https://img.shields.io/gitter/room/omise/omise-ios.svg?style=flat-square)](https://gitter.im/omise/omise-ios) [![](https://img.shields.io/badge/email-support-yellow.svg?style=flat-square)](mailto:support@omise.co)

Omise is a payment service provider currently operating in Thailand. Omise provides a set of clean APIs that helps merchants of any size accept credit cards online.

Omise Android SDK provides Android bindings for the Omise API as well as components for entering credit card information. Hop into the Gitter chat or email our support team if you have any question regarding this SDK and the functionality it provides.

## Requirements

* Public key. [Register for an Omise account](https://dashboard.omise.co/signup) to obtain your API keys.
* Android 4.4+ (KitKat) target or higher.
* Android Studio and Gradle build system.

## Merchant Compliance

Card data should never transit through your server. We recommend that you follow our guide on how to safely [collection credit information](https://www.omise.co/collecting-card-information).

To be authorized to create tokens server-side you must have a currently valid PCI-DSS Attestation of Compliance (AoC) delivered by a certified QSA Auditor.

## Installation

Adds the following line to your project's build.gradle file inside the `dependencies` block:

```groovy
compile 'co.omise:omise-android:2.0.0'
```

## Usage

#### Credit Card Activity

The simplest way to use this SDK is to integrate the provided `CreditCardActivity` directly into your application. This activity contains a pre-made credit form and will automatically [tokenize credit card information](https://www.omise.co/security-best-practices) for you.

To use it, first declare the availability of the activity in your `AndroidManifest.xml` file as follows:

```xml
<activity
  android:name="co.omise.android.ui.CreditCardActivity"
  android:theme="@style/OmiseSDKTheme" />
```

Then in your activity, declare the method that will start this activity as follows:

```java
private static final String OMISE_PKEY = "pkey_test_123”;
private static final int REQUEST_CC = 100;

private void showCreditCardForm() {
  Intent intent = new Intent(this, CreditCardActivity.class);
  intent.putExtra(CreditCardActivity.EXTRA_PKEY, OMISE_PKEY);
  startActivityForResult(intent, REQUEST_CC);
}
```

Replace the string `pkey_test_123` with the public key obtained from your Omise dashboard.

After the end-user completes entering credit card information, the activity result callback will be called, handle it like so:

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
  switch (requestCode) {
    case REQUEST_CC:
      if (resultCode == CreditCardActivity.RESULT_CANCEL) {
        return;
      }

      Token token = data.getParcelableExtra(CreditCardActivity.EXTRA_TOKEN_OBJECT);
      // process your token here.

    default:
      super.onActivityResult(requestCode, resultCode, data);
  }
}
```

A number of result is returned as an activity result. You can obtain it from the resulting `Intent` from the following keys:

* `data.getStringExtra(CreditCardActivity.EXTRA_TOKEN)` - The string ID of the token. Use this if you only needs the ID and not   the card data.
* `data.getParcelableExtra(CreditCardActivity.EXTRA_TOKEN_OBJECT)` - The full `Token` object returned from the Omise API.
* `data.getParcelableExtra(CreditCardActivity.EXTRA_CARD_OBJECT)` - The `Card` object which is part of the `Token` object returned from the Omise API.

#### Custom Credit Card Form

If you need to build your own credit card form, the components inside `CreditCardActivity` can be used on its own. For example, the `CreditCardEditText` can be used in XML like so:

```xml
<co.omise.android.ui.CreditCardEditText
  android:layout_width="match_parent"
  android:layout_height="wrap_content" />
```

This component provides automatic spacing into groups of 4 digits as the user types. Additionally the following utility classes are available from the SDK:

* `co.omise.android.ui.ExpiryMonthSpinnerAdapter` - This is a [SpinnerAdapter](https://developer.android.com/reference/android/widget/SpinnerAdapter.html) that provide list of months (01-12) for use in a [Spinner](https://developer.android.com/guide/topics/ui/controls/spinner.html) control for selecting expiry dates.
* `co.omise.android.ui.ExpiryYearSpinnerAdapter` - Same as above but lists the current year up to twelve years into the future.
* `co.omise.android.CardNumber` - The `CardNumber` class provides utility methods for validating and formatting credit card numbers.

#### Manual Tokenization

If you have built your own credit card form you can use the SDK to manually tokenizes the card. First build the `Client` and supply your public key like so:

```java
Client client = new Client("pkey_test_123");
```

Then construct the token request with values from your custom form:

```java
TokenRequest request = new TokenRequest();
request.number = "4242424242424242";
request.name = "JOHN SMITH";
request.expirationMonth = 10;
request.expirationYear = 2020;
request.securityCode = "123";
```

And then send the request using the `client` we've constructed earlier:

```java
client.send(request, new TokenRequestListener() {
  @Override
  public void onTokenRequestSucceed(TokenRequest request, Token token) {
      // you've got Token!
  }

  @Override
  public void onTokenRequestFailed(TokenRequest request, Throwable throwable) {
      // something bad happened
  }
});
```

The `Client` class will automatically dispatch the network call to an internal background thread and will call the listener methods on the thread that initially calls the `send` method.

## Contributing

Pull requests and bugfixes are welcome. For larger scope of work, please pop on to our [![](https://img.shields.io/gitter/room/omise/omise-ios.svg?style=flat-square)](https://gitter.im/omise/omise-ios) chatroom to discuss first.

## LICENSE

MIT