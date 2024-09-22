# Subscriptions

## Creating Subscriptions

To create a subscription, first retrieve an instance of your billable model, which typically will be an instance of `App\Models\User`. Once you have retrieved the model instance, you may use the `newSubscription` method to create the model's subscription:

```ts
// title: start/routes.ts
router.get('/update-payment-method', ({ request}) => {
  const paymentMethodId = request.get('paymentMethodId')

  const subscription = await user
    .newSubscription('default', 'price_monthly')
    .create(paymentMethodId)

  // ...
})
```

The first argument passed to the `newSubscription` method should be the internal type of the subscription. If your application only offers a single subscription, you might call this `default` or `primary`. This subscription type is only for internal application usage and is not meant to be shown to users. In addition, it should not contain spaces and it should never be changed after creating the subscription. The second argument is the specific price the user is subscribing to. This value should correspond to the price's identifier in Stripe.

The `create` method, which accepts [a Stripe payment method identifier](./payment_methods.md#storing-payment-methods) or Stripe `PaymentMethod` object, will begin the subscription as well as update your database with the billable model's Stripe customer ID and other relevant billing information.

> [!WARNING]  
> Passing a payment method identifier directly to the `create` subscription method will also automatically add it to the user's stored payment methods.

### Collecting Recurring Payments via Invoice Emails

Instead of collecting a customer's recurring payments automatically, you may instruct Stripe to email an invoice to the customer each time their recurring payment is due. Then, the customer may manually pay the invoice once they receive it. The customer does not need to provide a payment method up front when collecting recurring payments via invoices:

```ts
await user
  .newSubscription('default', 'price_monthly')
  .createAndSendInvoice()
```

The amount of time a customer has to pay their invoice before their subscription is canceled is determined by the `days_until_due` option. By default, this is 30 days; however, you may provide a specific value for this option if you wish:

```ts
await user
  .newSubscription('default', 'price_monthly')
  .createAndSendInvoice([], {
    days_until_due: 30
  })
```

### Quantities

If you would like to set a specific [quantity](https://stripe.com/docs/billing/subscriptions/quantities) for the price when creating the subscription, you should invoke the `quantity` method on the subscription builder before creating the subscription:

```ts
await user
  .newSubscription('default', 'price_monthly')
  .quantity(5)
  .create(paymentMethod)
```

### Additional Details

If you would like to specify additional [customer](https://stripe.com/docs/api/customers/create) or [subscription](https://stripe.com/docs/api/subscriptions/create) options supported by Stripe, you may do so by passing them as the second and third arguments to the `create` method:

```ts
await user
  .newSubscription('default', 'price_monthly')
  .quantity(5)
  .create(paymentMethod, { email }, { metadata })
```

### Coupons

If you would like to apply a coupon when creating the subscription, you may use the `withCoupon` method:

```ts
await user
  .newSubscription('default', 'price_monthly')
  .withCoupon('code')
  .create(paymentMethod)
```

Or, if you would like to apply a [Stripe promotion code](https://stripe.com/docs/billing/subscriptions/discounts/codes), you may use the `withPromotionCode` method:

```ts
await user
  .newSubscription('default', 'price_monthly')
  .withPromotionCode('promo_code_id')
  .create(paymentMethod)
```

The given promotion code ID should be the Stripe API ID assigned to the promotion code and not the customer facing promotion code. If you need to find a promotion code ID based on a given customer facing promotion code, you may use the `findPromotionCode` method:

```ts
// Find a promotion code ID by its customer facing code...
const promotionCode = await user.findPromotionCode('SUMMERSALE')

// Find an active promotion code ID by its customer facing code...
const promotionCode = await user.findActivePromotionCode('SUMMERSALE')
```

In the example above, the returned `promotionCode` object is an instance of Shopkeeper's `PromotionCode`. This class decorates an underlying `Stripe.PromotionCode` object. You can retrieve the coupon related to the promotion code by invoking the `coupon` method:

```ts
const promotionCode = await user.findPromotionCode('SUMMERSALE')
const coupon = promotionCode.coupon()
```

The coupon instance allows you to determine the discount amount and whether the coupon represents a fixed discount or percentage based discount:

```ts
if (coupon.isPercentage()) {
  return `${coupon.percentOff(}%` // 21.5%
} else {
  return `${coupon.amountOff(}` // $5.99
}
```

You can also retrieve the discounts that are currently applied to a customer or subscription:

```ts
const discount = await user.discount()
const discount = await subscription.discount()
```

The returned Shopkeeper's `Discount` instances decorate an underlying `Stripe.Discount` object instance. You may retrieve the coupon related to this discount by invoking the `coupon` method:

```ts
const discount = await subscription.discount()
const coupon = discount.coupon()
```

If you would like to apply a new coupon or promotion code to a customer or subscription, you may do so via the `applyCoupon` or `applyPromotionCode` methods:

```ts
await user.applyCoupon('coupon_id')
await user.applyPromotionCode('coupon_id')

await subscription.applyCoupon('coupon_id')
await subscription.applyPromotionCode('coupon_id')
```

Remember, you should use the Stripe API ID assigned to the promotion code and not the customer facing promotion code. Only one coupon or promotion code can be applied to a customer or subscription at a given time.

For more info on this subject, please consult the Stripe documentation regarding [coupons](https://stripe.com/docs/billing/subscriptions/coupons) and [promotion codes](https://stripe.com/docs/billing/subscriptions/coupons/codes).

### Adding Subscriptions

If you would like to add a subscription to a customer who already has a default payment method you may invoke the `add` method on the subscription builder:

```ts
const user = await User.find(1)

await user.newSubscription('default', 'price_monthly').add()
```

### Creating Subscriptions From the Stripe Dashboard

You may also create subscriptions from the Stripe dashboard itself. When doing so, Cashier will sync newly added subscriptions and assign them a type of `default`. To customize the subscription type that is assigned to dashboard created subscriptions, [define webhook event handlers](./defining-webhook-event-handlers).

In addition, you may only create one type of subscription via the Stripe dashboard. If your application offers multiple subscriptions that use different types, only one type of subscription may be added through the Stripe dashboard.

Finally, you should always make sure to only add one active subscription per type of subscription offered by your application. If a customer has two `default` subscriptions, only the most recently added subscription will be used by Cashier even though both would be synced with your application's database.

## Checking Subscription Status

Once a customer is subscribed to your application, you may easily check their subscription status using a variety of convenient methods. First, the `subscribed` method returns `true` if the customer has an active subscription, even if the subscription is currently within its trial period. The `subscribed` method accepts the type of the subscription as its first argument:

```ts
if (await user.subscribed('default'))
  // ...
}
```

The `subscribed` method also makes a great candidate for a [route middleware](https://docs.adonisjs.com/guides/basics/middleware), allowing you to filter access to routes and controllers based on the user's subscription status:

```ts
// title: app/middlewares/subscribe_middleware.ts
import { HttpContext } from '@adonisjs/core/http'
import { NextFn } from '@adonisjs/core/types/http'

export default class SubscribedMiddleware {
  async handle({ auth, response }: HttpContext, next: NextFn) {
    const user = auth.getUser()
    const subscribed = await user?.subscribed()

    if (!subscribed) {
      response.redirect().toRoute('home')
    } else {
      await next()
    }
  }
}
```

If you would like to determine if a user is still within their trial period, you may use the `onTrial` method. This method can be useful for determining if you should display a warning to the user that they are still on their trial period:

```ts
const subscription = await user.subscription('default')
if (subscription.onTrial())
  // ...
}
```

The `subscribedToProduct` method may be used to determine if the user is subscribed to a given product based on a given Stripe product's identifier. In Stripe, products are collections of prices. In this example, we will determine if the user's `default` subscription is actively subscribed to the application's "premium" product. The given Stripe product identifier should correspond to one of your product's identifiers in the Stripe dashboard:

```ts
if (await user.subscribedToProduct('prod_premium', 'default'))
  // ...
}
```

By passing an array to the `subscribedToProduct` method, you may determine if the user's `default` subscription is actively subscribed to the application's "basic" or "premium" product:

```ts
if (await user.subscribedToProduct(['prod_basic', 'prod_premium'], 'default'))
  // ...
}
```

The `subscribedToPrice` method may be used to determine if a customer's subscription corresponds to a given price ID:

```ts
if (await user.subscribedToPrice('price_basic_monthly', 'default'))
  // ...
}
```

The `recurring` method may be used to determine if the user is currently subscribed and is no longer within their trial period:

```ts
const subscription = await user.subscription('default')
if (subscription.recurring())
  // ...
}
```

:::warning

If a user has two subscriptions with the same type, the most recent subscription will always be returned by the `subscription` method. For example, a user might have two subscription records with the type of `default`; however, one of the subscriptions may be an old, expired subscription, while the other is the current, active subscription. The most recent subscription will always be returned while older subscriptions are kept in the database for historical review.

:::

### Canceled Subscription Status

To determine if the user was once an active subscriber but has canceled their subscription, you may use the `canceled` method:

```ts
const subscription = await user.subscription('default')
if (subscription.canceled()) {
  // ...
}
```

You may also determine if a user has canceled their subscription but are still on their "grace period" until the subscription fully expires. For example, if a user cancels a subscription on March 5th that was originally scheduled to expire on March 10th, the user is on their "grace period" until March 10th. Note that the `subscribed` method still returns `true` during this time:

```ts
const subscription = await user.subscription('default')
if (subscription.onGracePeriod()) {
  // ...
}
```

To determine if the user has canceled their subscription and is no longer within their "grace period", you may use the `ended` method:

```ts
const subscription = await user.subscription('default')
if (subscription.ended()) {
  // ...
}
```

### Incomplete and Past Due Status

If a subscription requires a secondary payment action after creation the subscription will be marked as `incomplete`. Subscription statuses are stored in the `stripe_status` column of Cashier's `subscriptions` database table.

Similarly, if a secondary payment action is required when swapping prices the subscription will be marked as `past_due`. When your subscription is in either of these states it will not be active until the customer has confirmed their payment. Determining if a subscription has an incomplete payment may be accomplished using the `hasIncompletePayment` method on the billable model or a subscription instance:

```ts
if (await user.hasIncompletePayment('default')) {
  // ...
}

const subscription = await user.subscription('default')
if (subscription.hasIncompletePayment()) {
  // ...
}
```

When a subscription has an incomplete payment, you should direct the user to Shopkeeper's payment confirmation page, passing the `latestPayment` identifier. You may use the `latestPayment` method available on subscription instance to retrieve this identifier:

```edge
<a href="{{ route('shopkeeper.payment', await subscription.latestPayment().then(lp => id)) }}">
    Please confirm your payment.
</a>
```

If you would like the subscription to still be considered active when it's in a `past_due` or `incomplete` state, you may set the `keepPastDueSubscriptionsActive` and `keepIncompleteSubscriptionsActive` options in your Shopkeeper's configuration:

```ts
// title: config/shopkeeper.ts
import { defineConfig } from 'adonis-shopkeeper'

export default defineConfig({
  keepPastDueSubscriptionsActive: true,
  keepIncompleteSubscriptionsActive: true,
  ...
})
```

:::warning

When a subscription is in an `incomplete` state it cannot be changed until the payment is confirmed. Therefore, the `swap` and `updateQuantity` methods will throw an exception when the subscription is in an `incomplete` state.

:::

## Changing prices

After a customer is subscribed to your application, they may occasionally want to change to a new subscription price. To swap a customer to a new price, pass the Stripe price's identifier to the `swap` method. When swapping prices, it is assumed that the user would like to re-activate their subscription if it was previously canceled. The given price identifier should correspond to a Stripe price identifier available in the Stripe dashboard:

```ts
const user = await User.find(1)
const subscription = await user.subscription('default')

await subscription.swap('price_yearly')
```

If the customer is on trial, the trial period will be maintained. Additionally, if a "quantity" exists for the subscription, that quantity will also be maintained.

If you would like to swap prices and cancel any trial period the customer is currently on, you may invoke the `skipTrial` method:

```ts
const subscription = await user.subscription('default')

await subscriptions
  .skipTrial()
  .swap('price_yearly')
```

If you would like to swap prices and immediately invoice the customer instead of waiting for their next billing cycle, you may use the `swapAndInvoice` method:

```ts
const subscription = await user.subscription('default')

await subscriptions.swapAndInvoice('price_yearly')
```

### Prorations

By default, Stripe prorates charges when swapping between prices. The `noProrate` method may be used to update the subscription's price without prorating the charges:

```ts
const subscription = await user.subscription('default')

await subscription
  .noProrate()
  .swap('price_yearly')
```

For more information on subscription proration, consult the [Stripe documentation](https://stripe.com/docs/billing/subscriptions/prorations).

:::warning

Executing the `noProrate` method before the `swapAndInvoice` method will have no effect on proration. An invoice will always be issued.

:::

## Subscription Quantity

Sometimes subscriptions are affected by "quantity". For example, a project management application might charge $10 per month per project. You may use the `incrementQuantity` and `decrementQuantity` methods to easily increment or decrement your subscription quantity:

```ts
const user = await User.find(1)
const subscription = await user.subscription('default')

await subscription.incrementQuantity()
await subscription.incrementQuantity(5)

await subscription.decrementQuantity()
await subscription.decrementQuantity(5)
```

Alternatively, you may set a specific quantity using the `updateQuantity` method:

```ts
await subscription.updateQuantity(5)
```

The `noProrate` method may be used to update the subscription's quantity without prorating the charges:

```ts
await subscription
  .noProrate()
  .updateQuantity(5)
```

For more information on subscription quantities, consult the [Stripe documentation](https://stripe.com/docs/subscriptions/quantities).

### Quantities for Subscriptions With Multiple Products

If your subscription is a [subscription with multiple products](#subscriptions-with-multiple-products), you should pass the ID of the price whose quantity you wish to increment or decrement as the second argument to the increment / decrement methods:

```ts
await subscription.incrementQuantity(1, 'price_chat')
```

## Subscriptions With Multiple Products

[Subscription with multiple products](https://stripe.com/docs/billing/subscriptions/multiple-products) allow you to assign multiple billing products to a single subscription. For example, imagine you are building a customer service "helpdesk" application that has a base subscription price of $10 per month but offers a live chat add-on product for an additional $15 per month. Information for subscriptions with multiple products is stored in Shopkeeper's `subscription_items` database table.

You may specify multiple products for a given subscription by passing an array of prices as the second argument to the `newSubscription` method:

```ts
// file: start/routes.ts
import router from '@adonisjs/core/services/router'

router.get('/user/subscribe', async ({ auth, request }) => {
  const user = auth.getUserOrFail()
  const paymentMethodId = request.get('paymentMethodId')

  await user
    .newSubscription('default', ['price_monthly', 'price_chat'])
    .create(paymentMethodId)
})
```

In the example above, the customer will have two prices attached to their `default` subscription. Both prices will be charged on their respective billing intervals. If necessary, you may use the `quantity` method to indicate a specific quantity for each price:

```ts
await user
  .newSubscription('default', ['price_monthly', 'price_chat'])
  .quantity(5, 'price_chat')
  .create(paymentMethodId)
```

If you would like to add another price to an existing subscription, you may invoke the subscription's `addPrice` method:

```ts
await user
  .newSubscription('default')
  .addPrice('price_chat')
  .create(paymentMethodId)
```

The example above will add the new price and the customer will be billed for it on their next billing cycle. If you would like to bill the customer immediately you may use the `addPriceAndInvoice` method:

```ts
await user
  .newSubscription('default')
  .addPriceAndInvoice('price_chat')
```

If you would like to add a price with a specific quantity, you can pass the quantity as the second argument of the `addPrice` or `addPriceAndInvoice` methods:

```ts
const user = await User.find(1)
const subscription = await user.subscription('default')

await subscription.addPrice('price_chat', 5)
```

You may remove prices from subscriptions using the `removePrice` method:

```ts
await subscription.removePrice('price_chat')
```

:::warning

You may not remove the last price on a subscription. Instead, you should simply cancel the subscription.

:::

### Swapping Prices

You may also change the prices attached to a subscription with multiple products. For example, imagine a customer has a `price_basic` subscription with a `price_chat` add-on product and you want to upgrade the customer from the `price_basic` to the `price_pro` price:

```ts
const user = await User.find(1)
const subscription = await user.subscription('default')

await subscription.swap(['price_pro', 'price_chat'])
```

When executing the example above, the underlying subscription item with the `price_basic` is deleted and the one with the `price_chat` is preserved. Additionally, a new subscription item for the `price_pro` is created.

You can also specify subscription item options by passing an array of key / value pairs to the `swap` method. For example, you may need to specify the subscription price quantities:

```ts
const user = await User.find(1)
const subscription = await user.subscription('default')

await subscription.swap({
  price_pro: { quantity: 5 },
  price_chat: {},
})
```

If you want to swap a single price on a subscription, you may do so using the `swap` method on the subscription item itself. This approach is particularly useful if you would like to preserve all of the existing metadata on the subscription's other prices:

```ts
const user = await User.find(1)
const subscription = await user.subscription('default')

const item = await subscription.findItemOrFail('price_basic')

await item.swap('price_pro')

```

    $user = User::find(1);

    $user->subscription('default')
            ->findItemOrFail('price_basic')
            ->swap('price_pro');

### Proration

By default, Stripe will prorate charges when adding or removing prices from a subscription with multiple products. If you would like to make a price adjustment without proration, you should chain the `noProrate` method onto your price operation:

```ts
await subscription
  .noProrate()
  .addPrice('price_chat')
```

### Quantities

If you would like to update quantities on individual subscription prices, you may do so using the [existing quantity methods](#subscription-quantity) by passing the ID of the price as an additional argument to the method:

```ts
const subscription = await user.subscription('default')

await subscription.incrementQuantity(5, 'price_chat')
await subscription.decrementQuantity(5, 'price_chat')
await subscription.updateQuantity(5, 'price_chat')
```

:::warning

When a subscription has multiple prices the `stripe_price` and `quantity` attributes on the `Subscription` model will be `null`. To access the individual price attributes, you should use the `items` relationship available on the `Subscription` model.

:::

### Subscription Items

When a subscription has multiple prices, it will have multiple subscription "items" stored in your database's `subscription_items` table. You may access these via the `items` relationship on the subscription:

```ts
const subscription = await user.subscription('default')
await subscription.load('items') // Load relationship

const subscriptionItem = subs.items[0]

const stripePrice = subscriptionItem.stripePrice
const quantity = subscriptionItem.quantity

```

You can also retrieve a specific price using the `findItemOrFail` method:

```ts
const subscription = await user.subscription('default')
const subscriptionItem = await subscription.findItemOrFail('price_chat')
```

## Multiple Subscriptions

Stripe allows your customers to have multiple subscriptions simultaneously. For example, you may run a gym that offers a swimming subscription and a weight-lifting subscription, and each subscription may have different pricing. Of course, customers should be able to subscribe to either or both plans.

When your application creates subscriptions, you may provide the type of the subscription to the `newSubscription` method. The type may be any string that represents the type of subscription the user is initiating:

```ts
import router from '@adonisjs/core/services/router'

router.get('/swimming/subscribe', async ({ auth, request }) => {
  const user = auth.getUserOrFail()
  const paymentMethodId = request.get('paymentMethodId')

  await user.newSubscription('swimming')
    .price('price_swimming_monthly')
    .create(paymentMethodId)

  // ...
})
```
