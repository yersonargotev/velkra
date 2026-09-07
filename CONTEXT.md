# Velkra

Velkra supports the day-to-day operation of a small appliance and furniture retail store. Its first product boundary is operational control rather than formal accounting or tax compliance.

## Language

**Sale**:
A completed exchange in which the store provides one or more products and records the money received or owed.
_Avoid_: Invoice, transaction

**Credit Sale**:
A Sale in which the customer receives the product before paying the full amount and owes the remaining balance to the Store.
_Avoid_: Layaway, cash sale

**Sale Line**:
One product and quantity within a Sale, recording the unit price actually agreed with the customer.
_Avoid_: Product price, discount record

**Reference Price**:
The usual selling price shown in the catalog when a Sale is made. A Sale Line may preserve both this value and a different price agreed with the customer.
_Avoid_: Final price

**Layaway**:
An agreement locally called a "separado" in which a customer pays for a reserved product over time and receives it only after completing payment.
_Avoid_: Credit sale, installment sale

**Installment**:
A partial payment made toward a Layaway's outstanding balance.
_Avoid_: Deposit, Sale

**Customer**:
A person who buys, reserves, or owes payment for products. The first release records their name, address, phone number, and identity document.
_Avoid_: Account, buyer

**Stock**:
The quantity of a product the store currently has available to sell.
_Avoid_: Inventory count

**Reserved Stock**:
A product physically held by the Store for an active Layaway and unavailable for another Sale.
_Avoid_: Available Stock, sold Stock

**Stock Adjustment**:
A reasoned correction to recorded Stock when the physical quantity and the recorded quantity differ.
_Avoid_: Edit, overwrite

**Cash Movement**:
Money entering or leaving the business, including sales, expenses, and later corrections.
_Avoid_: Accounting entry

**Daily Close**:
The end-of-day reconciliation that explains the day's recorded sales, other cash movements, and resulting balance.
_Avoid_: Financial statement, accounting close

**Store**:
The single business location covered by the first Velkra release.
_Avoid_: Branch, tenant

**Register**:
The single point of sale whose daily balance is reconciled in the first Velkra release.
_Avoid_: Account

**Administrator**:
A user who may view business results and perform restricted corrections or Stock adjustments. The store owner fills this role in the first release.
_Avoid_: Owner

**Seller**:
A user who records routine store operations but cannot perform restricted administrative actions.
_Avoid_: Cashier, employee

**Supplier**:
A person or business from which the Store acquires products.
_Avoid_: Vendor

**Payable**:
An amount the Store owes to a Supplier for products it has acquired but not fully paid for.
_Avoid_: Expense, loan

**Purchase**:
An acquisition of products from a Supplier that records product costs, payments, due date, and any remaining Payable.
_Avoid_: Stock entry, expense
