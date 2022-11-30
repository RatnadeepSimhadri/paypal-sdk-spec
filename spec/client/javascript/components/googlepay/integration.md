# GooglePay 

Adapted from https://developers.google.com/pay/api/web/guides/tutorial#full-example


Load paypal script
 
```javascript
<script src="https://www.paypal.com/sdk/js?client-id=YOUR_CLIENT_ID&currency=USD&merchant-id=MERCHANT_ID&components=googlepay"></script>
```

Load googlepay js

```javascript
<script
  async
  src="https://pay.google.com/gp/p/js/pay.js"
  onload="onGooglePayLoaded()">
</script>
```

```javascript
const googlePay = paypal.GooglePay();

function getGoogleTransactionInfo() {
  return {
    displayItems: [
      {
        label: "Subtotal",
        type: "SUBTOTAL",
        price: "11.00",
      },
      {
        label: "Tax",
        type: "TAX",
        price: "1.00",
      },
    ],
    countryCode: "US",
    currencyCode: "USD",
    totalPriceStatus: "FINAL",
    totalPrice: "12.00",
    totalPriceLabel: "Total",
  };
}

let paymentsClient = null;

async function getGooglePaymentsClient() {
  const { merchantInfo } = await googlePay.getConfig();

  if (!paymentsClient) {
    paymentsClient = new google.payments.api.PaymentsClient({
      environment: "TEST",
      merchantInfo,
      paymentDataCallbacks: {
        onPaymentAuthorized,
        onPaymentDataChanged,
      },
    });
  }

  return paymentsClient;
}

function onPaymentAuthorized(
  paymentData
) {
  return processPayment(paymentData)
    .then(
      () =>
        ({
          transactionState: "SUCCESS",
        })
    )
    .catch(() => {
      return {
        transactionState: "ERROR",
        error: {
          intent: "PAYMENT_AUTHORIZATION",
          message: "Insufficient funds",
          reason: "PAYMENT_DATA_INVALID",
        },
      };
    });
}

async function onPaymentDataChanged({
  shippingAddress,
  shippingOptionData,
  callbackTrigger,
}) {

  switch (callbackTrigger) {
    case "INITIALIZE":
    case "SHIPPING_ADDRESS":
      if (shippingAddress?.administrativeArea === "NJ") {
        return {
          error: {
            reason: "SHIPPING_ADDRESS_UNSERVICEABLE",
            message: "Cannot ship to the selected address",
            intent: "SHIPPING_ADDRESS",
          },
        };
      }

      const newShippingOptionParameters = getGoogleDefaultShippingOptions();

      let selectedShippingOptionId =
        newShippingOptionParameters.defaultSelectedOptionId;

      return {
        newShippingOptionParameters,
        newTransactionInfo: calculateNewTransactionInfo(
          selectedShippingOptionId
        ),
      };

    case "SHIPPING_OPTION":
      return {
        newTransactionInfo: calculateNewTransactionInfo(shippingOptionData?.id),
      };
    default:
  }
}

const baseRequest = {
  apiVersion: 2,
  apiVersionMinor: 0,
};

async function onClick() {
  const { allowedPaymentMethods, merchantInfo } = await googlePay.getConfig();

  const paymentDataRequest = {
    ...baseRequest,
    allowedPaymentMethods,
    transactionInfo: {
      displayItems: [
        {
          label: "Subtotal",
          type: "SUBTOTAL",
          price: "11.00",
        },
        {
          label: "Tax",
          type: "TAX",
          price: "1.00",
        },
      ],
      countryCode: "US",
      currencyCode: "USD",
      totalPriceStatus: "FINAL",
      totalPrice: "12.00",
      totalPriceLabel: "Total",
    },
    merchantInfo,
    callbackIntents: [
      "SHIPPING_ADDRESS",
      "SHIPPING_OPTION",
      "PAYMENT_AUTHORIZATION",
    ],
    shippingAddressRequired: true,
    shippingAddressParameters: {
      allowedCountryCodes: ["US"],
      phoneNumberRequired: true,
    },
    shippingOptionRequired: true,
  };

  const paymentsClient = await getGooglePaymentsClient();

  paymentsClient.loadPaymentData(paymentDataRequest);
}

async function onGooglePayLoaded() {
  const { allowedPaymentMethods } = await googlePay.getConfig();

  const paymentsClient = await getGooglePaymentsClient();

  const { result } = await paymentsClient.isReadyToPay({
    ...baseRequest,
    allowedPaymentMethods,
  });

  if (result) {
    const button = paymentsClient?.createButton({
      onClick,
      allowedPaymentMethods,
    });
    document.getElementById("gpay-btn")?.appendChild(button);
  }
}

function calculateNewTransactionInfo(shippingOptionId) {
  let newTransactionInfo = getGoogleTransactionInfo();

  let shippingCost = {
    "shipping-001": "0.00",
    "shipping-002": "1.99",
    "shipping-003": "10.00",
  }[shippingOptionId];

  newTransactionInfo?.displayItems?.push({
    type: "LINE_ITEM",
    label: "Shipping cost",
    price: shippingCost,
    status: "FINAL",
  });

  let totalPrice = 0.0;

  newTransactionInfo?.displayItems?.forEach(
    (displayItem) => (totalPrice += parseFloat(displayItem.price))
  );
  newTransactionInfo.totalPrice = totalPrice.toString();

  return newTransactionInfo;
}

function getGoogleDefaultShippingOptions() {
  return {
    defaultSelectedOptionId: "shipping-001",
    shippingOptions: [
      {
        id: "shipping-001",
        label: "Free: Standard shipping",
        description: "Free Shipping delivered in 5 business days.",
      },
      {
        id: "shipping-002",
        label: "$1.99: Standard shipping",
        description: "Standard shipping delivered in 3 business days.",
      },
      {
        id: "shipping-003",
        label: "$10: Express shipping",
        description: "Express shipping delivered in 1 business day.",
      },
    ],
  };
}

async function processPayment(paymentData) {

  const { id } = await googlePay.createOrder({
    intent: "CAPTURE",
    purchase_units: [
      {
        amount: {
          currency_code: "USD",
          value: "1.00",
        },
      },
    ],
  });


  /*
   * Approve
   */
  await googlePay.confirmOrder({
    id,
    paymentData: {
      paymentMethodData: paymentData.paymentMethodData,
      shippingOptionData: paymentData.shippingOptionData,
      shippingAddress: paymentData.shippingAddress,
    },
  });

  /*
   * Capture
   */
  await fetch(`/order/${id}/capture`, {
    method: "POST"
  })
}

```
