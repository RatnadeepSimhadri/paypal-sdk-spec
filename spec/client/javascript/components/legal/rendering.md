# Render Legal Text

## Vanilla Javascript / HTML 

```html 
<div id="paypal-legal-container"></div>
 
<script src="https://www.paypal.com/sdk/js?client-id=MOCK_CLIENT_ID&components=legal"></script>
 
<script>
   paypal.Legal.render({
      legalLocale: 'de_DE',
      buyerCountry: 'DE',
      fundingSource: paypal.FUNDING.PUI
   }, '#paypal-legal-container');

</script>

```
 