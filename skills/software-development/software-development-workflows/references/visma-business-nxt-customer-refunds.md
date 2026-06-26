# Visma Business NXT: customer refunds via GraphQL API

Use this note when investigating or implementing refund flows against Visma Business NXT. It captures the durable API pattern discovered from Visma docs; verify exact schema field/argument names with GraphiQL introspection in the target tenant before shipping.

## Sources

- Business NXT API root: https://docs.vismasoftware.no/businessnxtapi/
- How to create/update batch: https://docs.vismasoftware.no/businessnxtapi/howto/batch/
- Open customer entry processings: https://doc.visma.net/userdoc/businessnxt/en/Accounting/Customer-ledger/open_customer_entry_processings.html
- Open customer entry table: https://doc.visma.net/userdoc/businessnxt/en/Accounting/Customer-ledger/open_customer_entry.html
- Voucher table/account type: https://doc.visma.net/userdoc/businessnxt/en/Accounting/Vouchers/voucher.html
- GraphiQL playground: https://docs.business.visma.net/graphiql/

## Key distinction

Customer refunds have two materially different flows:

1. **Business NXT/AutoPay should actually pay money to the customer**
   - Use customer ledger, not supplier ledger.
   - Start from `openCustomerEntry` and run the `Create payment suggestion` processing.
   - The documentation says this processing is for “Direct Debit and for payments to customers”.
   - Expected GraphQL pattern: `openCustomerEntry_processings { createPaymentSuggestion(filter: ..., args: ...) { succeeded ... } }`.
   - Exact `args` names are not fully shown in public docs; confirm by schema introspection.

2. **Money has already been refunded externally (e.g. PSP/Vipps/MobilePay/card), and NXT only needs bookkeeping**
   - Do **not** create a payment suggestion, or the customer may be paid twice.
   - Create a batch/voucher that debits customer ledger and credits bank/settlement GL account, then update the batch.

## Safe API workflow: customer payout from NXT

1. Query `openCustomerEntry` for the specific customer/open entry.
2. Filter to the actual credit/overpayment/open entry to refund.
3. Ensure prerequisites:
   - payment suggestion not already created (`paymentSuggested` not enabled)
   - customer does not have “No payment” in Associate customer options
   - bank account/postal giro/IBAN exists when company setup requires it
   - appropriate bank partner exists
4. Run `Create payment suggestion` with an explicit filter on the target row(s), typically the primary key:
   - `voucherJournalNo`
   - `auditNo`
5. Read back `payment` / `paymentLine` to store `paymentNo`, `lineNo`, status, amount, and bank partner.

Never run the processing without a filter in integrations: docs state that if no open customer entry rows are selected, all rows ready for payment suggestions may be processed.

## Safe API workflow: bookkeeping an external refund

Use `batch_create` with nested `vouchers`, then `batch_processings.updateBatch`.

Account type values from the Voucher table:

| Value | Meaning |
|---:|---|
| 1 | Customer |
| 2 | Supplier |
| 3 | General ledger account |
| 4 | Capital asset |

Typical voucher shape for refund already paid externally:

```graphql
mutation create_customer_refund_voucher(
  $cid: Int!,
  $voucherSeriesNo: Int!,
  $customerNo: Int!,
  $bankAccountNo: Int!,
  $amount: Decimal!,
  $voucherDate: Int!,
  $valueDate: Int!
) {
  useCompany(no: $cid) {
    batch_create(values: [{
      voucherSeriesNo: $voucherSeriesNo
      valueDate: $valueDate
      vouchers: [{
        voucherNo: null
        voucherDate: $voucherDate
        valueDate: $valueDate
        debitAccountType: 1
        debitAccountNo: $customerNo
        customerNo: $customerNo
        creditAccountType: 3
        creditAccountNo: $bankAccountNo
        amountDomestic: $amount
        text: "Refund to customer"
      }]
    }]) {
      affectedRows
      items { batchNo }
    }
  }
}
```

Then post/update the batch:

```graphql
mutation update_refund_batch($cid: Int!, $batchNo: Int!) {
  useCompany(no: $cid) {
    batch_processings {
      updateBatch(filter: { batchNo: { _eq: $batchNo } }) {
        succeeded
        voucherJournalNo
      }
    }
  }
}
```

## Implementation notes

- Store idempotency and external correlation IDs in the application database: refund ID, source payment ID, customerNo, amount, batchNo, voucherNo, voucherJournalNo, auditNo, paymentNo, paymentLineNo, and status.
- Do not use `openSupplierEntry`, `supplierNo`, or account type `2` unless the counterparty is genuinely a supplier.
- Business NXT processings can be long-running; consider async execution for heavy operations and verify backend state after timeouts.
- Business NXT GraphQL mutations are not transactional across a multi-step business flow; implement compensation/cleanup in the client where partial success matters.
