# Render Legal Text

## Vanilla Javascript / HTML 

### Render Legal Text Juxtaposed to Buy Now Button 

```html 
<div id="paypal-legal-container"></div>
 
<script src="https://www.paypal.com/sdk/js?client-id=MOCK_CLIENT_ID&components=legal"></script>
 
<script>
   paypal.Legal({
      fundingSource: paypal.Legal.FUNDING.PAY_UPON_INVOICE
   }).render('#paypal-legal-container');

</script>

```
 
 # Render Legal Text For Errors 
```html
 <div id="paypal-legal-container"></div>
 
<script src="https://www.paypal.com/sdk/js?client-id=MOCK_CLIENT_ID&components=legal"></script>
 
<script>

  paypal.Legal({
      fundingSource: paypal.Legal.FUNDING.PAY_UPON_INVOICE, 
      errorCode: paypal.Legal.ERROR_CODE.PAYMENT_SOURCE_INFO_CANNOT_BE_VERIFIED
   }).render('#paypal-legal-container');
</script>
```
