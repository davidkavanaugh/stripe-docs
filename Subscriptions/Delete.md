### Cancel Subscription
- collect the subscription ID on the frontend, but you will most likely get this information from your database for your logged in user.
- On the backend, call DELETE v1/subscriptions/:id 
- You should receive a customer.subscription.deleted event.
- After the subscription is canceled, update your database to remove the Stripe subscription ID you previously stored, and limit access to your service.
- When a subscription is canceled, it cannot be reactivated. Instead, collect updated billing information from your customer, update their default payment method, and create a new subscription with their existing customer record.