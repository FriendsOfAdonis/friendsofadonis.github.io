# Handling Stripe Webhooks

:::tip

You may use [the Stripe CLI](https://stripe.com/docs/stripe-cli) to help test webhooks during local development.

:::

Stripe can notify your application of a variety of events via webhooks. By default, a route that points to Shopkeeper's webhook controller is automatically registered by the Shopkeeper service provider. This controller will handle all incoming webhook requests.

By default, the Shopkeeper webhook controller will automatically handle cancelling subscriptions that have too many failed charges (as defined by your Stripe settings), customer updates, customer deletions, subscription updates, and payment method changes; however, as we'll soon discover, you can extend this controller to handle any Stripe webhook event you like.

To ensure your application can handle Stripe webhooks, be sure to configure the webhook URL in the Stripe control panel. By default, Shopkeeper's webhook controller responds to the `/stripe/webhook` URL path. The full list of all webhooks you should enable in the Stripe control panel are:

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `payment_method.automatically_updated`
- `invoice.payment_action_required`
- `invoice.payment_succeeded`

For convenience, Shopkeeper includes a `shopkeeper:webhook` Ace command. This command will create a webhook in Stripe that listens to all of the events required by Shopkeeper:

```shell
node ace shopkeeper:webhook
```

By default, the created webhook will point to the URL defined by the `APP_URL` environment variable and the `shopkeeper.webhook` route that is included with Shopkeeper. You may provide the `--url` option when invoking the command if you would like to use a different URL:

```shell
node ace shopkeeper:webhook --url "https://example.com/stripe/webhook"
```

The webhook that is created will use the Stripe API version that your version of Shopkeeper is compatible with. If you would like to use a different Stripe version, you may provide the `--api-version` option:

```shell
node ace shopkeeper:webhook --api-version="2019-12-03"
```

After creation, the webhook will be immediately active. If you wish to create the webhook but have it disabled until you're ready, you may provide the `--disabled` option when invoking the command:

```shell
node ace shopkeeper:webhook --disabled
```

Make sure you protect incoming Stripe webhook requests with Shopkeeper's included [webhook signature verification](#verifying-webhook-signatures) middleware.

## Defining Webhook Event Handlers

Shopkeeper automatically handles subscription cancellations for failed charges and other common Stripe webhook events. However, if you have additional webhook events you would like to handle, you may do so by listening to the event using the [Event Emitter](https://docs.adonisjs.com/guides/digging-deeper/emitter#event-emitter).

The event names follow the semantic `stripe:<eventType>` or `stripe:<eventType>:handled`.

```ts
// title: start/events.ts
import emitter from '@adonisjs/core/services/emitter'
import Stripe from 'stripe'

emitter.on(
  'stripe:customer.subscription.created', 
  (event: Stripe.CustomerSubscriptionCreatedEvent) => {
    // ...
  }
)

// Emitted after the webhook has been handled by Shopkeeper
emitter.on(
  'stripe:customer.subscription.created:handled',
  (event: Stripe.CustomerSubscriptionCreatedEvent) => {
    // ...
  }
)
```

## Verifying Webhook Signatures

To secure your webhooks, you may use [Stripe's webhook signatures](https://stripe.com/docs/webhooks/signatures). For convenience, Shopkeeper automatically includes a middleware which validates that the incoming Stripe webhook request is valid.

To enable webhook verification, ensure that the `STRIPE_WEBHOOK_SECRET` environment variable is set in your application's `.env` file. The webhook `secret` may be retrieved from your Stripe account dashboard.
