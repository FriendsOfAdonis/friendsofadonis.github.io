# Subscription Trials

## With Payment Method Up Front

If you would like to offer trial periods to your customers while still collecting payment method information up front, you should use the `trialDays` method when creating your subscriptions:

```ts
// file: start/routes.ts
import router from '@adonisjs/core/services/router'

router.get('/user/subscribe', async ({ auth, request }) => {
  const user = auth.getUserOrFail()
  const paymentMethodId = request.get('paymentMethodId')

  await user
    .newSubscription('default', 'price_monthly')
    .trialDays(10)
    .create(paymentMethodId)

  // ...
})
```

This method will set the trial period ending date on the subscription record within the database and instruct Stripe to not begin billing the customer until after this date. When using the `trialDays` method, Shopkeeper will overwrite any default trial period configured for the price in Stripe.

:::warning

If the customer's subscription is not canceled before the trial ending date they will be charged as soon as the trial expires, so you should be sure to notify your users of their trial ending date.

:::

The `trialUntil` method allows you to provide a `DateTime` instance that specifies when the trial period should end:

```ts
await user
  .newSubscription('default', 'price_monthly')
  .trialUntil(DateTime.now().plus({ days: 10 }))
  .create(paymentMethod)
```

You may determine if a user is within their trial period using either the `onTrial` method of the user instance or the `onTrial` method of the subscription instance. The two examples below are equivalent:

```ts
if (await user.onTrial()) {
  // ...
}

if (subscription.onTrial()) {
  // ...
}
```

You may use the `endTrial` method to immediately end a subscription trial:

```ts
await subscription.endTrial()
```

To determine if an existing trial has expired, you may use the `hasExpiredTrial` methods:

```ts
if (await user.hasExpiredTrial()) {
  // ...
}

if (subscription.hasExpiredTrial()) {
  // ...
}
```

### Defining Trial Days in Stripe / Shopkeeper

You may choose to define how many trial days your price's receive in the Stripe dashboard or always pass them explicitly using Shopkeeper. If you choose to define your price's trial days in Stripe you should be aware that new subscriptions, including new subscriptions for a customer that had a subscription in the past, will always receive a trial period unless you explicitly call the `skipTrial()` method.

## Without Payment Method Up Front

If you would like to offer trial periods without collecting the user's payment method information up front, you may set the `trialEndsAt` column on the user record to your desired trial ending date. This is typically done during user registration:

```ts
await User.create({
  trialEndsAt: DateTime.now().plus({ days: 10 })
})
```

Shopkeeper refers to this type of trial as a "generic trial", since it is not attached to any existing subscription. The `onTrial` method on the billable model instance will return `true` if the current date is not past the value of `trialEndsAt`:

```ts
if (user.onTrial()) {
  // User is within their trial period...
}
```

Once you are ready to create an actual subscription for the user, you may use the `newSubscription` method as usual:

```ts
await user.newSubscription('default', 'price_monthly').create(paymentMethod)
```

To retrieve the user's trial ending date, you may use the `trialEndsAt` method. This method will return a DateTime instance if a user is on a trial or `null` if they aren't. You may also pass an optional subscription type parameter if you would like to get the trial ending date for a specific subscription other than the default one:

```ts
if (user.onTrial()) {
  const trialEndsAt = await user.trialEndsAt('main')
}
```

You may also use the `onGenericTrial` method if you wish to know specifically that the user is within their "generic" trial period and has not yet created an actual subscription:

```ts
if (user.onGenericTrial()) {
  // User is within their "generic" trial period...
}
```

## Extending Trials

The `extendTrial` method allows you to extend the trial period of a subscription after the subscription has been created. If the trial has already expired and the customer is already being billed for the subscription, you can still offer them an extended trial. The time spent within the trial period will be deducted from the customer's next invoice:

```ts
const subscription = await user.subscription('default')

// End the trial 7 days from now...
await subscription.extendTrial(
  DateTime.now().plus({ days: 7 })
)

// Add an additional 5 days to the trial...
await subscription.extendTrial(
  user.trialEndsAt.plus({ days: 5 })
)
```
