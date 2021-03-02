### Display Customer Pmt Method
Displaying the brand and last 4 digits of your customer’s card can help them know which card is being charged, or if they need to update their payment method.

- On the frontend, send the payment method ID to a backend endpoint (/retrieve-customer-payment-method) that retrieves the payment method details.
- Stripe recommends that you save the paymentMethod.id and last4 in your database, e.g., paymentMethod.id as stripeCustomerPaymentMethodId in your users collection or table. You can optionally store exp_month, exp_year, fingerprint, billing_details as needed. This is to limit the number of calls you make to Stripe, for performance efficiency and to avoid possible rate limiting.