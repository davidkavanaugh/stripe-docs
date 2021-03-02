### Update Subscription Plan
To let your customers change their subscription, collect the price ID of the option they want to change to. Then send the new price ID from the frontend to a backend endpoint (//update-subscription). This example also passes the subscription ID, but you can retrieve it from your database for your logged in user.
- You should receive a customer.subscription.updated event.

#### Preview a Price Change
When your customer changes their subscription, there’s often an adjustment to the amount they owe, known as a proration. You can use the upcoming invoice endpoint to display the adjusted amount to your customers.

On the frontend, pass the upcoming invoice details to a backend endpoint (/retrieve-upcoming-invoice)
