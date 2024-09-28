# Single Charges

## Simple Charge

If you would like to make a one-time charge against a customer, you may use the `charge` method on a billable model instance. You will need to [provide a payment method identifier](#payment-methods-for-single-charges) as the second argument to the `charge` method:

```ts
router.post('/purchase', async ({ request, auth }) => {
  const user = auth.getUserOrFail()
  const paymentMethodId = request.get('paymentMethodId')

  const stripeCharge = await user.charge(100, paymentMethodId)

  // ...
})
```

The `charge` method accepts an array as its third argument, allowing you to pass any options you wish to the underlying Stripe charge creation. More information regarding the options available to you when creating charges may be found in the [Stripe documentation](https://stripe.com/docs/api/charges/create):

```ts
await user.charge(100, paymentMethod, {
  custom_option: value
})
```

You may also use the `charge` method without an underlying customer or user. To accomplish this, invoke the `charge` method on a new instance of your application's billable model:

```ts
await new User().charge(100, paymentMethod)
```

The `charge` method will throw an exception if the charge fails. If the charge is successful, an instance of Shopkeeper's `Payment` will be returned from the method:

```ts
try {
  const payment = await user.charge(100, paymentMethod)
} catch (e) {
  // ...
}
```

:::warning

The `charge` method accepts the payment amount in the lowest denominator of the currency used by your application. For example, if customers are paying in United States Dollars, amounts should be specified in pennies.

:::

## Charge With Invoice

Sometimes you may need to make a one-time charge and offer a PDF invoice to your customer. The `invoicePrice` method lets you do just that. For example, let's invoice a customer for five new shirts:

```ts
await user.invoicePrice('price_tshirt', 5)
```

The invoice will be immediately charged against the user's default payment method. The `invoicePrice` method also accepts an array as its third argument. This array contains the billing options for the invoice item. The fourth argument accepted by the method is also an array which should contain the billing options for the invoice itself:

```ts
await user.invoicePrice('price_tshirt', 5, {
  discounts: [
    { coupon: 'SUMMER21SALE' }
  ]
}, { default_tax_rates: ['txr_id']})
```

Similarly to `invoicePrice`, you may use the `tabPrice` method to create a one-time charge for multiple items (up to 250 items per invoice) by adding them to the customer's "tab" and then invoicing the customer. For example, we may invoice a customer for five shirts and two mugs:

```ts
await user.tabPrice('price_tshirt', 5)
await user.tabPrice('price_mug', 2)
await user.invoice()
```

Alternatively, you may use the `invoiceFor` method to make a "one-off" charge against the customer's default payment method:

```ts
await user.invoiceFor('One Time Fee', 500)
```

Although the `invoiceFor` method is available for you to use, it is recommended that you use the `invoicePrice` and `tabPrice` methods with pre-defined prices. By doing so, you will have access to better analytics and data within your Stripe dashboard regarding your sales on a per-product basis.

:::warning

The `invoice`, `invoicePrice`, and `invoiceFor` methods will create a Stripe invoice which will retry failed billing attempts. If you do not want invoices to retry failed charges, you will need to close them using the Stripe API after the first failed charge.

:::

## Creating Payment Intents

You can create a new Stripe payment intent by invoking the `pay` method on a billable model instance. Calling this method will create a payment intent that is wrapped in a Shopkeeper's `Payment` instance:

```ts
router.post('/pay', async ({ request, auth }) => {
  const user = auth.getUserOrFail()
  const amount = request.get('amount')

  const payment = await user.pay(amount)

  return payment.clientSecret()
})
```

After creating the payment intent, you can return the client secret to your application's frontend so that the user can complete the payment in their browser. To read more about building entire payment flows using Stripe payment intents, please consult the [Stripe documentation](https://stripe.com/docs/payments/accept-a-payment?platform=web).

When using the `pay` method, the default payment methods that are enabled within your Stripe dashboard will be available to the customer. Alternatively, if you only want to allow for some specific payment methods to be used, you may use the `payWith` method:

```ts
router.post('/pay', async ({ request, auth }) => {
  const user = auth.getUserOrFail()
  const amount = request.get('amount')

  const payment = await user.pay(amount, ['card', 'bancontact'])

  return payment.clientSecret()
})
```

:::warning

The `pay` and `payWith` methods accept the payment amount in the lowest denominator of the currency used by your application. For example, if customers are paying in United States Dollars, amounts should be specified in pennies.

:::

## Refunding Charges

If you need to refund a Stripe charge, you may use the `refund` method. This method accepts the Stripe [payment intent ID](#payment-methods-for-single-charges) as its first argument:

```ts
const payment = await user.charge(100, paymentMethodId)

await user.refund(payment.id)
```
