# Plethora — Domain Model v1.0

## 1. Overview

Plethora's domain model represents an individual's personal financial state through accounts, financial transactions, transaction effects, and manual adjustments.

The current account balance is derived rather than directly stored as an independently editable value.

### Balance Formula

Balance = Opening Balance + Sum(Transaction Effects) + Sum(Adjustments)

---

# 2. User

## Definition

A User represents an individual who owns and manages financial accounts within Plethora.

## Attributes

| Attribute | Type | Required | Description |
|---|---|---:|---|
| id | Identifier | Yes | Unique user identity |
| name | Text | Yes | User's name; Unicode supported |
| email | Email | No | Optional contact information |

## Relationships

- A User owns zero or more Accounts.
- A User creates zero or more Transactions.
- A User creates zero or more Adjustments.
- A User configures zero or more AlertConfigurations.

## Responsibilities

- Identify the user.
- Establish ownership of accounts.
- Associate financial records with their creator.

## Invariants

1. A User has a unique identifier.
2. A User has a name.
3. A provided email must satisfy email validation.
4. A User may exist without any Accounts.

## Assumption

Plethora relies on the User to provide truthful and accurate financial information.

---

# 3. Account

## Definition

An Account represents a financial container belonging to a User.

## Attributes

| Attribute | Type | Required | Description |
|---|---|---:|---|
| id | Identifier | Yes | Unique account identity |
| owner | User | Yes | Account owner |
| name | Text | Yes | User-facing account name |
| type | AccountType | Yes | Financial account category |
| openingBalance | Money | Yes | Initial financial state |
| createdAt | Date/time | Yes | Account creation timestamp |
| accountIdentifier | Text | No | IBAN or other external identifier |
| status | AccountStatus | Yes | ACTIVE or INACTIVE |

### Supported Account Types

- Checking
- Savings
- Loan
- Credit Card
- Investment
- Cash

## Derived State

Current balance is derived from:

Balance = Opening Balance + Sum(Transaction Effects) + Sum(Adjustments)

The current balance is not directly editable.

Negative balances are permitted.

## Relationships

- An Account belongs to exactly one User.
- An Account has zero or more Transactions through TransactionEffects.
- An Account has zero or more Adjustments.
- An Account has zero or one AlertConfiguration.

## Responsibilities

- Represent a user's financial account.
- Provide the basis for calculating current balance.
- Maintain account lifecycle state.

## Invariants

1. An Account belongs to exactly one User.
2. An Account has an opening balance.
3. An Account can be ACTIVE or INACTIVE.
4. Historical financial records remain accessible after deactivation.
5. An inactive Account cannot receive a new Transaction.
6. Negative balances are permitted.

---

# 4. Transaction

## Definition

A Transaction represents a financial event recorded by the User that produces one or more financial effects on Accounts.

## Attributes

| Attribute | Type | Required | Description |
|---|---|---:|---|
| id | Identifier | Yes | Unique transaction identity |
| type | TransactionType | Yes | Type of financial event |
| amount | Money | Yes | Positive financial magnitude |
| date | Date | Yes | Date of the financial event |
| time | Time | No | Optional transaction time |
| productName | Text | No | Optional product name |
| merchant | Text | No | Optional merchant/store |
| location | Text | No | Optional location |
| quantity | Number | No | Optional quantity |

### Supported Transaction Types

- Expense
- Income
- Transfer
- Debt Repayment

## Relationships

- A Transaction belongs to one User.
- A Transaction has one or two TransactionEffects in the MVP.
- Each TransactionEffect affects exactly one Account.

## Responsibilities

- Represent a financial event.
- Identify the event's type.
- Maintain the event's financial amount and information.
- Define the financial event from which its effects are derived.

## Invariants

1. Transaction amount must be greater than zero.
2. A Transaction must have at least one TransactionEffect.
3. A Transaction may have at most two TransactionEffects in the MVP.
4. All Accounts affected by a Transaction must belong to the same User.
5. New TransactionEffects cannot affect inactive Accounts.
6. Transactions are editable but cannot be deleted in the MVP.
7. Editing a Transaction must preserve all Transaction invariants.

---

# 5. TransactionEffect

## Definition

A TransactionEffect represents the signed financial impact of a Transaction on one specific Account.

## Attributes

| Attribute | Type | Required | Description |
|---|---|---:|---|
| amount | Money | Yes | Signed financial impact |
| account | Account | Yes | Account affected by the effect |

## Relationships

- A Transaction has one or more TransactionEffects.
- A TransactionEffect belongs to exactly one Transaction.
- A TransactionEffect affects exactly one Account.

## Responsibilities

- Represent the financial impact of a Transaction on an Account.
- Identify the affected Account.
- Identify the signed financial impact.

## Invariants

1. Every TransactionEffect belongs to exactly one Transaction.
2. Every TransactionEffect affects exactly one Account.
3. A TransactionEffect amount cannot be zero.
4. A TransactionEffect cannot be committed against an inactive Account.
5. TransactionEffects are not independently edited; they are changed through their Transaction.

## Transaction Type Rules

### Expense

Exactly one effect:

Effect = -Transaction Amount

### Income

Exactly one effect:

Effect = +Transaction Amount

### Transfer

Exactly two effects:

- One negative effect
- One positive effect
- Equal absolute values
- Net effect equals zero

### Debt Repayment

Exactly one negative effect.

---

# 6. Adjustment

## Definition

An Adjustment represents a user-authorized correction to an Account's financial state that does not represent a Transaction.

## Attributes

| Attribute | Type | Required | Description |
|---|---|---:|---|
| id | Identifier | Yes | Unique adjustment identity |
| amount | Money | Yes | Signed financial impact |
| createdAt | Date/time | Yes | Automatically generated creation timestamp |
| reason | Text | Yes | Reason for the correction |

## Relationships

- An Adjustment belongs to one User.
- An Adjustment affects exactly one Account.
- An Adjustment is independent of Transactions and TransactionEffects.

## Responsibilities

- Represent a correction to recorded account state.
- Record the signed financial impact.
- Record the reason for the correction.
- Contribute to the Account's derived balance.

## Invariants

1. An Adjustment must have a reason.
2. Adjustment amount cannot be zero.
3. An Adjustment affects exactly one Account.
4. An Adjustment belongs to the User who created it.
5. An Adjustment may affect an inactive Account.
6. An Adjustment is not a Transaction.
7. An Adjustment is not a TransactionEffect.
8. Adjustments are editable but cannot be deleted in the MVP.

---

# 7. AlertConfiguration

## Definition

An AlertConfiguration defines a balance threshold for an Account. When the Account's balance reaches or falls below that threshold, the alert condition is satisfied.

## Attributes

| Attribute | Type | Required | Description |
|---|---|---:|---|
| id | Identifier | Yes | Unique configuration identity |
| account | Account | Yes | Account being monitored |
| threshold | Money | Yes | Balance threshold |

## Relationships

- An AlertConfiguration belongs to exactly one Account.
- An Account may have zero or one AlertConfiguration in the MVP.
- A User configures AlertConfigurations through their Accounts.

## Responsibilities

- Define the Account's balance threshold.
- Determine whether the Account's balance satisfies the alert condition.

## Invariants

1. Threshold is required.
2. Threshold may be positive, zero, or negative.
3. An AlertConfiguration belongs to exactly one Account.
4. An Account has at most one AlertConfiguration in the MVP.
5. An inactive Account does not actively evaluate its AlertConfiguration.

## Alert Condition

An alert condition is satisfied when:

Balance <= Threshold

---

# 8. Account Lifecycle

Accounts have two states in the MVP:

- ACTIVE
- INACTIVE

### ACTIVE

- New transactions may be created.
- Existing transactions may be edited.
- Adjustments may be created.
- Alert configuration is evaluated.

### INACTIVE

- New transactions cannot be created.
- Historical transactions remain accessible.
- Existing transactions may be edited if the resulting state remains valid.
- Adjustments may still be created.
- Alert configuration is not actively evaluated.

---

# 9. Record Integrity Rules

Plethora distinguishes between financial events and corrections.

| Record | Editable | Deletable | Correction Mechanism |
|---|---:|---:|---|
| Opening Balance | Yes, before financial activity | N/A | Adjustment after activity |
| Transaction | Yes | No | Edit Transaction |
| TransactionEffect | Through Transaction | No | Edit Transaction |
| Adjustment | Yes | No | Edit Adjustment |
| AlertConfiguration | Yes | Yes | Reconfigure |
| Account | Limited | Not in MVP | Lifecycle changes |

## Opening Balance Rule

An Opening Balance may be edited before an Account has financial activity.

Once financial activity exists, changing the original opening state must be represented through an Adjustment.

This preserves the distinction between:

- Initial account state
- Financial events
- Corrections to recorded state

---

# 10. Core Domain Invariants

The following rules apply across the domain:

1. A Transaction may only affect Accounts belonging to the same User.
2. New TransactionEffects cannot affect inactive Accounts.
3. Adjustments may affect inactive Accounts.
4. Negative Account balances are valid.
5. Account balance is derived and not directly editable.
6. Any operation that changes an Account's financial state must result in a valid recalculation of its balance.
7. Alert conditions are evaluated against the resulting Account balance.
8. Editing a Transaction or Adjustment must preserve all applicable domain invariants.
9. A Transaction represents a financial event.
10. A TransactionEffect represents how that event affects a particular Account.
11. An Adjustment represents a correction to recorded financial state and is independent of Transactions.

---

# 11. MVP Scope Constraint

Plethora intentionally does not support split transactions in the MVP.

Therefore:

- Expenses affect exactly one Account.
- Income affects exactly one Account.
- Transfers affect exactly two Accounts.
- Debt repayments affect exactly one Account.

More complex transaction distributions may be considered in a future version.