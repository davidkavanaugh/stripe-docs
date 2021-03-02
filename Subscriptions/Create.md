### Create Subscription
#### API Object Relationships
![API Objects Relationships Chart](https://raw.githubusercontent.com/davidkavanaugh/stripe-docs/7d71eafb2a8e868a3215d13922d03b520236ad56/Diagrams/subscription_objects_fixed_price.svg)

#### Testing and Monitoring
A production-ready integration should monitor events automatically. You can call the Stripe API and check specific values in the response (polling), or you can set up a webhook endpoint and let Stripe push events to your integration. Webhook monitoring is simpler and more efficient, especially for asynchronous integrations like subscriptions. The Stripe CLI provides a listen command for testing event monitoring during development. [More Info](https://stripe.com/docs/billing/subscriptions/fixed-price#testing)

#### Create Business Model with Products and Prices
- You create your products and their pricing options in the Dashboard or with the Stripe CLI. 
- A fixed-price service needs a product and a price for each option.
- Use the product IDs to create a price for each product.
- [Products API Documentation](https://stripe.com/docs/api/products)
- [Prices API Documentation](https://stripe.com/docs/api/prices)

#### Create Stripe Customer
Customer objects allow you to perform recurring charges, and to track multiple charges, that are associated with the same customer. The API allows you to create, delete, and update your customers. You can retrieve individual customers as well as a list of all your customers. [Customers API Documentation](https://stripe.com/docs/api/customers)
- per [these Stripe Docs](https://stripe.com/docs/billing/subscriptions/fixed-price#create-customer), Stripe recommends creating a customer by passing customer email to a backend endpoint

#### Collection Payment Information
Let your customer choose a plan and provide payment information. 
- Test with card number 4242 4242 4242 4242, any three-digit CVC, any future expiration date, and any five-digit ZIP code.

#### Save Payment Details
On the frontend, save the payment details you just collected to a [payment method](https://stripe.com/docs/api/payment_methods), passing the ID of the customer you already created:

#### Create Subscription
Send the customer ID, payment method ID, and price ID to a backend endpoint (/create-subscription). 
- The backend updates the customer with the payment method, and then passes the customer ID to the subscription. 
- The payment method is also assigned as the default payment method for the subscription invoices.
- Verify initial payment status with the API response by checking the value of subscription.latest_invoice.payment_intent.status. The invoice tracks payment status for the subscription; the payment intent tracks the status of the provided payment method. To retrieve subscription.latest_invoice.payment_intent.status, you expand the latest_invoice child object of the response.
- You should also receive an invoice.paid event. You can disregard this event for initial payment, but monitor it for subsequent payments. The invoice.paid event type corresponds to the payment_intent.status of succeeded, so payment is complete, and the subscription status is active.

#### Provision Access to your Service
To give the customer access to your service:
- Verify the subscription status is active.
- Check the product the customer subscribed to and grant access to your service. Checking the product instead of the price gives you more flexibility if you need to change the pricing or billing interval.
- Store the product.id and subscription.id in your database along with the customer.id you already saved.

On the frontend, you can implement these steps in the success callback after the subscription is created.

- It’s possible for a user to leave your application after payment is made and before this function is called. Continue to monitor the invoice.paid event on your webhook endpoint to verify that the payment succeeded and that you should provision the subscription.

- This is also good practice because during the lifecycle of the subscription, you need to keep provisioning in sync with subscription status. Otherwise, customers might be able to access your service even if their payments fail.

#### Manage Payment Authentication
- If you support payment methods that require customer authentication with 3D Secure, the value of latest_invoice.payment_intent.status is initially requires_action.

- In production, you’d monitor the invoice.payment_action_required event type.

- To handle this scenario, on the frontend notify the customer that authentication is required to complete payment and start the subscription. Retrieve the client secret for the payment intent, and pass it in a call to POST /v1/payment_intents/:id/confirm

- Make sure to monitor the invoice.paid event on your webhook endpoint to verify that the payment succeeded. It’s possible for users to leave your application before confirmCardPayment() finishes, so verifying whether the payment succeeded allows you to correctly provision your product.

#### Manage Subscription Payment Failure
If the value of subscription.latest_invoice.payment_intent.status is requires_payment_method, the card was processed when the customer first provided card details, but payment then failed later--in a production scenario, if a customer’s card was stolen or canceled after the subscription was set up, for example. 
- monitor invoice.payment_failed webhook event for payments after initial payment success.
- Catch the error, let your customer know their card was declined, and return them to the payment form to try a different card.
- On the frontend, define the function to attach the new card to the customer and update the invoice settings. Pass the customer ID, new payment method ID, invoice ID, and price ID to a backend endpoint (/retry-invoice)
- The backend code updates the customer with the new payment method, and assigns it as the new default payment method for subscription invoices.

#### Test Integration
To make sure your integration is ready for production, you can work with the [following test cards](https://stripe.com/docs/billing/subscriptions/fixed-price#test). Use them with any CVC, postal code, and future expiration date.
[More Info](https://stripe.com/docs/billing/testing)