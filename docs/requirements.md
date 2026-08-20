# Plethora — MVP Requirements

## 1. Problem Statement

Plethora provides an individual with a reliable way to record, retrieve, and analyze their financial activity in order to understand their current financial situation.

## 2. Target User

An individual managing their personal finances.

## 3. MVP Scope

The MVP is a single-user personal finance application.

### Included

- User profile
- Financial accounts
- Opening balances
- Financial transactions
- Transaction history
- Derived account balances
- Account adjustments
- Account activation/deactivation
- Low-balance alert configuration
- Basic financial analysis

### Excluded from MVP

- Multiple users
- Authentication and authorization
- Parent/child financial education
- Virtual currency
- Advanced forecasting
- Automated financial advice
- Bank synchronization
- External financial institution APIs
- AI-powered analysis
- Financial goals
- Automated recurring-transaction detection
- Birthday functionality
- Debt repayment

Debt repayment is deferred because the MVP does not yet define the domain behavior required to accurately represent liability reduction, including the relationship between repayment, debt principal, and interest.

## 4. User

A user represents an individual who owns and manages financial accounts within Plethora.

Required:
- Unique identifier
- Name

Optional:
- Email

Names must support Unicode characters.

## 5. Account

An account represents a financial container belonging to a user.

An account has:

- Unique identifier
- Owner
- Name
- Type
- Opening balance
- Creation timestamp
- Optional external account identifier
- Status
- Optional alert configuration

Supported account types for the MVP include:

- Checking
- Savings
- Loan
- Credit card
- Investment
- Cash

Account status is limited to:

- ACTIVE
- INACTIVE

Inactive accounts remain visible and retain their historical financial information.

### Opening Balance Modification

An account's opening balance may be directly modified only while the account has no TransactionEffects and no Adjustments.

Once a TransactionEffect or Adjustment exists, the opening balance cannot be directly modified. Corrections to the initial financial state must instead be represented through an Adjustment.

This preserves the distinction between the account's initial state, financial events, and corrections to recorded financial state.

## 6. Balance

The current account balance is derived from:

Balance = Opening Balance + Transaction Effects + Adjustments

The balance is not directly editable.

Negative balances are permitted.

## 7. Transactions

A transaction represents a financial event recorded by the user.

Supported transaction types:

- Expense
- Income
- Transfer

Required information:

- Transaction type
- Amount
- Date
- At least one affected account

Optional information:

- Time
- Product name
- Merchant/store
- Location
- Quantity

Transaction amounts must be greater than zero.

Transactions are editable but cannot be deleted in the MVP.

### Transaction Amount and Direction

The transaction amount represents the positive financial magnitude of the event.

The direction of the financial impact is represented by the associated TransactionEffect(s), rather than by the Transaction amount itself.

For example, an expense of €70 is represented as:

Transaction:
- Type: Expense
- Amount: €70

TransactionEffect:
- Account: Checking
- Amount: -€70

## 8. Transaction Effects

A transaction effect represents the signed financial impact of a transaction on one specific account.

A transaction may have:

- One effect for an expense
- One effect for an income
- Two effects for a transfer
- One effect for a debt repayment

Transaction effects cannot affect inactive accounts.

All accounts affected by a transaction must belong to the same user.

For a transfer:

- One effect is negative
- One effect is positive
- Both effects have equal absolute values
- The net effect is zero

## 9. Adjustments

An adjustment represents a user-authorized correction to an account's financial state that does not represent a transaction.

An adjustment requires:

- Unique identifier
- Signed amount
- Creation timestamp
- Reason
- Associated account
- Creating user

An adjustment amount cannot be zero.

An adjustment affects exactly one account.

Adjustments may affect inactive accounts.

Adjustments are editable but cannot be deleted in the MVP.

## 10. Account Lifecycle

An account can be:

- ACTIVE
- INACTIVE

Active accounts may receive new transactions.

Inactive accounts cannot receive new transactions.

Historical transactions and adjustments remain accessible.

Existing transactions may be edited provided the resulting transaction remains valid.

Adjustments may be made to inactive accounts.

### Inactive Account Transaction Rules

An inactive account cannot be affected by new transactions.

Existing transactions associated with an inactive account remain editable so that the user can correct historical financial records.

Editing an existing transaction may recalculate its TransactionEffects and the account's derived balance.

Adjustments may affect inactive accounts because they represent corrections to recorded financial state.

## 11. Alert Configuration

An alert configuration defines a balance threshold for an account.

An account may have at most one alert configuration in the MVP.

The threshold may be:

- Positive
- Zero
- Negative

The alert condition is satisfied when:

Balance <= Threshold

An inactive account does not actively evaluate its alert configuration.

Reactivating the account restores its existing alert configuration.

## 12. Financial Analysis

The MVP focuses primarily on descriptive and basic diagnostic analysis.

Examples include:

- Current account balance
- Transaction history
- Income and expense summaries
- Account-level financial summaries
- Threshold-based alerts

Predictive and prescriptive financial recommendations are outside the MVP.

## 13. Financial Goals

Financial goals are part of the future product vision but are not included in the MVP.

## 14. Future Product Direction

Plethora may eventually include a financial education environment for children, allowing parents to teach financial concepts through simulated money, allowances, savings, spending, scarcity, and financial goals.