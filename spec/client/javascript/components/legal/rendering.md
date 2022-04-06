# Render Legal Text

## Vanilla Javascript / HTML 

```html 
<div id="paypal-legal-container"></div>
 
<script src="https://www.paypal.com/sdk/js?client-id=MOCK_CLIENT_ID&components=legal"></script>
 
<script>
   paypal.Legal({
      fundingSource: paypal.FUNDING.PUI
   }).render('#paypal-legal-container');

</script>

```
 