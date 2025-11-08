# Mortgage Calculator Component

## Overview

A fully reactive mortgage calculator built with Svelte 5 runes that calculates monthly mortgage payments and total costs.

## Features

- **Reactive Inputs**: All calculations update automatically when any input changes
- **Linked Down Payment**: Down payment amount and percentage are synced - changing one updates the other
- **Standard Mortgage Formula**: Uses the correct formula `M = P[r(1+r)^n]/[(1+r)^n-1]`
- **Formatted Currency**: All currency values displayed with $ and proper comma formatting
- **Responsive Design**: Works on desktop and mobile devices

## Usage

```svelte
<script>
  import MortgageCalculator from '$lib/components/MortgageCalculator.svelte';
</script>

<MortgageCalculator />
```

## Default Values

- House Price: $500,000
- Down Payment: 20%
- Interest Rate: 7.0% (realistic Colorado rate)
- Loan Term: 30 years

## Calculations Displayed

1. **Monthly Payment** - Prominently displayed using mortgage formula
2. **Loan Amount** - House price minus down payment
3. **Total Interest Paid** - Total interest over the life of the loan
4. **Total Amount Paid** - Principal + interest over loan term

## Technical Details

### Svelte 5 Runes Used

- `$state` - For user input values (house price, interest rate, etc.)
- `$derived` - For simple calculated values (down payment amount, loan amount)
- `$derived.by()` - For complex calculations requiring logic (monthly payment, totals)

### Key Features

#### Synced Down Payment

The down payment amount and percentage are kept in sync:
- Changing the percentage recalculates the amount
- Changing the amount recalculates the percentage
- Both update automatically when house price changes

#### Reactive Calculations

All derived values automatically update when dependencies change:
- `downPaymentAmount` updates when `housePrice` or `downPaymentPercent` changes
- `loanAmount` updates when down payment changes
- `monthlyPayment` recalculates when loan amount or rate changes
- All totals update when monthly payment changes

## Example Calculation

For a $500,000 house with 20% down at 7% for 30 years:
- Down Payment: $100,000
- Loan Amount: $400,000
- Monthly Payment: $2,661.21
- Total Interest: $558,035.59
- Total Paid: $958,035.59
