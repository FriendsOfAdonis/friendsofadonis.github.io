# Checkout

Shopkeeper Stripe also provides support for [Stripe Checkout](https://stripe.com/payments/checkout). Stripe Checkout takes the pain out of implementing custom pages to accept payments by providing a pre-built, hosted payment page.

The following documentation contains information on how to get started using Stripe Checkout with Shopkeeper. To learn more about Stripe Checkout, you should also consider reviewing [Stripe's own documentation on Checkout](https://stripe.com/docs/payments/checkout).

## Product Checkout

You may perform a checkout for an existing product that has been created within your Stripe dashboard using the `checkout` method on a billable model. The `checkout` method will initiate a new Stripe Checkout session. By default, you're required to pass a Stripe Price ID:

```ts
router.get('/product-checkout', async ({ auth, response }) => {
  const user = auth.getUserOrFail()
  const checkout = await user.checkout('price_tshirt')

  response.redirect().status(303).toPath(checkout.session.url)
})
```

If needed, you may also specify a product quantity:

```ts
router.get('/product-checkout', async ({ auth, response }) => {
  const user = auth.getUserOrFail()
  const checkout = await user.checkout({ price_tshirt: 15 })

  response.redirect().status(303).toPath(checkout.session.url)
})
```

When a customer visits this route they will be redirected to Stripe's Checkout page. By default, when a user successfully completes or cancels a purchase they will be redirected to your `home` route location, but you may specify custom callback URLs using the `success_url` and `cancel_url` options:

```ts
router.get('/product-checkout', async ({ auth, response }) => {
  const user = auth.getUserOrFail()
  const checkout = await user.checkout({ price_tshirt: 15 }, {
    success_url: route('checkout.success'),
    cancel_url: route('checkout.cancel'),
  })

  response.redirect().status(303).toPath(checkout.session.url)
})
```

When defining your `success_url` checkout option, you may instruct Stripe to add the checkout session ID as a query string parameter when invoking your URL. To do so, add the literal string `{CHECKOUT_SESSION_ID}` to your `success_url` query string. Stripe will replace this placeholder with the actual checkout session ID:

```ts
router.get('/product-checkout', async ({ auth, response }) => {
  const user = auth.getUserOrFail()
  const checkout = await user.checkout({ price_tshirt: 15 }, {
    success_url: route('checkout.success'),
    cancel_url: route('checkout.cancel'),
  })

  response.redirect().status(303).toPath(checkout.session.url)
})
```

    use Illuminate\Http\Request;
    use Stripe\Checkout\Session;
    use Stripe\Customer;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 1], [
            'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
            'cancel_url' => route('checkout-cancel'),
        ]);
    });

    Route::get('/checkout-success', function (Request $request) {
        $checkoutSession = $request->user()->stripe()->checkout->sessions->retrieve($request->get('session_id'));

        return view('checkout.success', ['checkoutSession' => $checkoutSession]);
    })->name('checkout-success');
