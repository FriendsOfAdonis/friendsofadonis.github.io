# Invoices

## Retrieving Invoices

You may easily retrieve an array of a billable model's invoices using the `invoices` method. The `invoices` method returns a collection of Shopkeeper's `Invoice` instances:

```ts
const invoices = await user.invoices()
```

If you would like to include pending invoices in the results, you may use the `invoicesIncludingPending` method:

```ts
const invoice = await user.invoicesIncludingPending()
```

You may use the `findInvoice` method to retrieve a specific invoice by its ID:

```ts
const invoice = await user.findInvoice(invoiceId)
```

### Displaying Invoice Information

When listing the invoices for the customer, you may use the invoice's methods to display the relevant invoice information. For example, you may wish to list every invoice in a table, allowing the user to easily download any of them:

```html
<table>
    @each(invoice in invoices)
        <tr>
            <td>{{ invoice->date()->format() }}</td>
            <td>{{ invoice->total() }}</td>
            <td><a href="/user/invoice/{{ invoice->id }}">Download</a></td>
        </tr>
    @end
</table>
```

## Upcoming Invoices

To retrieve the upcoming invoice for a customer, you may use the `upcomingInvoice` method:

```ts
const invoice = await user.upcomingInvoice()
```

Similarly, if the customer has multiple subscriptions, you can also retrieve the upcoming invoice for a specific subscription:

```ts
const subscription = await user.subscription('default')
const invoice = await subscription.upcomingInvoice()
```

## Previewing subscription Invoices

Using the `previewInvoice` method, you can preview an invoice before making price changes. This will allow you to determine what your customer's invoice will look like when a given price change is made:

```ts
const subscription = await user.subscription('default')
const invoice = await subscription.previewInvoice('price_yearly')
```

You may pass an array of prices to the `previewInvoice` method in order to preview invoices with multiple new prices:

```ts
const subscription = await user.subscription('default')
const invoice = await subscription.previewInvoice(['price_yearly', 'price_metered'])
```

## Generating Invoice PDFs

:::warning

This feature is not developed yet.

:::
