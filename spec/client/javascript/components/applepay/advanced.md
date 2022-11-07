# Applepay Advanced 

## Examples

Load the script
 
```javascript
<script src="https://www.paypal.com/sdk/js?client-id=YOUR_CLIENT_ID&currency=USD&merchant-id=MERCHANT_ID&components=applepay"></script>
```


```javascript
function onApplePayButtonClick() {
  const applePayPaymentRequest = {
    countryCode: window.applePayConfig.countryCode,
    currencyCode: window.applePayConfig.currencyCode,
    merchantCapabilities: window.applePayConfig.merchantCapabilities,
    supportedNetworks: window.applePayConfig.supportedNetworks,
    requiredShippingContactFields: ["name", "phone", "email", "postalAddress"],
    total: {
      label: "Demo",
      type: "final",
      amount: "99.99",
    },
  };

  var session = new ApplePaySession(4, applePayPaymentRequest);

  session.onvalidatemerchant = (event) => {
    paypal.applePay
      .validateMerchant({
        validationUrl: event.validationURL,
      })
      .then((merchantSession) => {
        const session = atob(merchantSession.session);
        session.completeMerchantValidation(merchantSession);
      })
      .catch(() => {
        session.abort();
      });
  };

  session.onshippingcontactselected = (event) => {
    const shippingContactUpdate = {};
    session.completeShippingContactSelection(shippingContactUpdate);
  };

  session.onshippingmethodselected = (event) => {
    console.log("Your shipping method is:", event.shippingMethod);
    // Update payment details.
    var shippingMethodUpdate = {}; // https://developer.apple.com/documentation/apple_pay_on_the_web/applepayshippingmethodupdate
    session.completeShippingMethodSelection(shippingMethodUpdate); // Set shippingMethodUpdate=null if there are no updates.
  };

  session.onpaymentauthorized = async (event) => {
    try {
      const res = await fetch("https://your-server.com/orders", {
        method: "POST",
        body: JSON.stringify({
          purchase_units: [
            {
              amount: {
                currency_code: "USD",
                value: "99.99",
              },
            },
          ],
        }),
      })
      if(!res.ok){
        throw new Error("order create failed")
      }

      const { id: orderId } = await res.json()

      await paypal.applePay.confirmOrder({
        orderId,
        token: event.payment.token,
        billingContact: event.payment.billingContact, // or provide billing address collected on merchant page.
        shippingContact: event.payment.shippingContact,
      });

      session.completePayment(ApplePaySession.STATUS_SUCCESS);

      //Submit approval to the server and authorize or capture the order.
      await fetch(`https://your-server.com/capture/${orderId}`, {
        method: "post",
      });
    } catch (err) {
      session.completePayment(ApplePaySession.STATUS_FAILURE);
    }
  };

  // Open Apple sheet.
  session.begin();
}

if (
  window.ApplePaySession &&
  ApplePaySession.supportsVersion(4) &&
  ApplePaySession.canMakePayments()
) {
  (async () => {
    try {
      window.applePayConfig = await paypal.applePay.getConfiguration();

      if (!window.applePayConfig.isApplePayEligible) {
        return;
      }

      //  render the ApplePay button directly into the page

      const applePayButton = document.getElementById("applepay-btn");
      applePayButton.innerHTML =
        '<apple-pay-button buttonstyle="black" type="buy" locale="el-GR">';

      applePayButton.addEventListener("click", onApplePayButtonClick);
    } catch (err) {
      console.error(err);
    }
  })();
}

```
