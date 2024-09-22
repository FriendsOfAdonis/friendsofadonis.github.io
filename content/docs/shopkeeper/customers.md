# Customers

## Retrieving Customers

You can retrieve a customer by their Stripe ID using the `shopkeeper.findBillable` method. This method will return an instance of the billable model:

```ts
import shopkeeper from 'adonis-shopkeeper/services/shopkeeper'

const user = await shopkeeper.findBillable(stripeId)
```

## Creating Customers

Occasionally, you may wish to create a Stripe customer without beginning a subscription. You may accomplish this using the `createAsStripeCustomer` method:

```ts
const stripeCustomer = await user.createAsStripeCustomer()
```

Once the customer has been created in Stripe, you may begin a subscription at a later date. You may provide an optional `params` argument to pass in any additional [customer creation parameters that are supported by the Stripe API](https://stripe.com/docs/api/customers/create):

```ts
const stripeCustomer = await user.createAsStripeCustomer(params)
```

You may use the `asStripeCustomer` method if you want to return the Stripe customer object for a billable model:

```ts
const stripeCustomer = await user.asStripeCustomer()
```

The `createOrGetStripeCustomer` method may be used if you would like to retrieve the Stripe customer object for a given billable model but are not sure whether the billable model is already a customer within Stripe. This method will create a new customer in Stripe if one does not already exist:

```ts
const stripeCustomer = await user.createOrGetStripeCustomer()
```

## Updating Customers

Occasionally, you may wish to update the Stripe customer directly with additional information. You may accomplish this using the `updateStripeCustomer` method. This method accepts as an argument the [customer update options supported by the Stripe API](https://stripe.com/docs/api/customers/update):

```ts
const stripeCustomer = await user.updateStripeCustomer(params)
```

## Balances

Stripe allows you to credit or debit a customer's "balance". Later, this balance will be credited or debited on new invoices. To check the customer's total balance you may use the `balance` method that is available on your billable model. The `balance` method will return a formatted string representation of the balance in the customer's currency:

```ts
const balance = await user.balance()
```

To credit a customer's balance, you may provide a value to the `creditBalance` method. If you wish, you may also provide a description:

```ts
await user.creditBalance(500, 'Premium customer top-up')
```

Providing a value to the `debitBalance` method will debit the customer's balance:

```ts
await user.debitBalance(500, 'Premium customer top-up')
```

The `applyBalance` method will create new customer balance transactions for the customer. You may retrieve these transaction records using the `balanceTransactions` method, which may be useful in order to provide a log of credits and debits for the customer to review:

```ts
// Retrieve all transactions...
const transactions = await user.balanceTransactions()

for (const transaction of transactions) {
  const amount = transaction.amount() // $2.31

  // Retrieve the related invoice when available...
  const invoice = await transaction.invoice()
}
```

## Tax IDs

Cashier offers an easy way to manage a customer's tax IDs. For example, the `taxIds` method may be used to retrieve all of the [tax IDs](https://stripe.com/docs/api/customer_tax_ids/object) that are assigned to a customer as a collection:

```ts
const taxIds = await user.taxIds()
```

You can also retrieve a specific tax ID for a customer by its identifier:

```ts
const taxId = await user.findTaxId('txi_belgium')
```

You may create a new Tax ID by providing a valid [type](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type) and value to the `createTaxId` method:

```ts
const taxId = await user.createTaxId('eu_vat', 'BE0123456789')
```

The `createTaxId` method will immediately add the VAT ID to the customer's account. [Verification of VAT IDs is also done by Stripe](https://stripe.com/docs/invoicing/customer/tax-ids#validation); however, this is an asynchronous process. You can be notified of verification updates by subscribing to the `customer.tax_id.updated` webhook event and inspecting [the VAT IDs `verification` parameter](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification). For more information on handling webhooks, please consult the [documentation on defining webhook handlers](./handling-stripe-webhooks).

You may delete a tax ID using the `deleteTaxId` method:

```ts
await user.deleteTaxId('eu_vat')
```

## Syncing Customer Data With Stripe

Typically, when your application's users update their name, email address, or other information that is also stored by Stripe, you should inform Stripe of the updates. By doing so, Stripe's copy of the information will be in sync with your application's.

To automate this, you may define an event listener on your billable model that reacts to the model's `updated` event. Then, within your event listener, you may invoke the `syncStripeCustomerDetails` method on the model:

```ts
// title: app/models/user.ts
import { BaseModel, afterCreate } from '@adonisjs/lucid/orm'

export default class User extends compose(BaseModel, Billable) {
  @afterUpdate()
  static synchWithStripe(user: User) {
    user.syncStripeCustomerDetails()
  }
}
```

Now, every time your customer model is updated, its information will be synced with Stripe. For convenience, Cashier will automatically sync your customer's information with Stripe on the initial creation of the customer.

You may customize the columns used for syncing customer information to Stripe by overriding a variety of methods provided by Cashier. For example, you may override the `stripeName` method to customize the attribute that should be considered the customer's "name" when Cashier syncs customer information to Stripe:

```ts
// title: app/models/user.ts
import { BaseModel, afterCreate } from '@adonisjs/lucid/orm'

export default class User extends compose(BaseModel, Billable) {
  stripeName(): string {
    return `${this.firstName} ${this.lastName}`
  }
}
```

Similarly, you may override the `stripeEmail`, `stripePhone`, `stripeAddress`, and `stripePreferredLocales` methods. These methods will sync information to their corresponding customer parameters when [updating the Stripe customer object](https://stripe.com/docs/api/customers/update). If you wish to take total control over the customer information sync process, you may override the `syncStripeCustomerDetails` method.

## Billing Portal

Stripe offers [an easy way to set up a billing portal](https://stripe.com/docs/billing/subscriptions/customer-portal) so that your customer can manage their subscription, payment methods, and view their billing history. You can generate URLs for your users to the billing portal by invoking the `billingPortalUrl` method on the billable model from a controller or route:

You may provide a custom URL that the user should return to by passing the URL as an argument to the `billingPortalUrl` method:

```ts
// title: start/routes.ts
import router from '@adonisjs/core/services/router'

router
  .get('/billing', ({ auth, response }) => {
    const user = auth.getUserOrFail()
    const billingPortalUrl = await user.billingPortalUrl(route('dashboard'))

    response.redirect().toPath(billingPortalUrl)
  })
  .middleware(middleware.auth())
  .name('billing')
```
