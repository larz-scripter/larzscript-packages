# lz-invoice

Build and format invoices from Money line items.
`larzscript larzpkg.lz install invoice`.

```
import "invoice" as invoice
let inv = invoice.new("INV-001", "Acme Co")
invoice.add(inv, "Widget", 3, $9.99)
print(invoice.render(inv))
```
