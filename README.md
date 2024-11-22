Yes, the **Central_Transaction_Datetime** is indeed converted into **Local_Transaction_Datetime** during the process. Here is a detailed explanation based on the provided details:

## Conversion of Central Transaction Time to Local Time

### Process Overview
 1. **Temporary Table for Ledger Transactions**:
 - Transaction data is initially collected with the central transaction times (Central_Transaction_Datetime).

2. **Timezone Adjustment Table**:
 - The table `Audit.vwDSTTimeZone` is used specifically for adjusting transaction times to the local time.

3. **Processed Transactions Table**:
 - The tables `#F_Ledger_Txn` and `#temp_vwDSTTimeZone` are used to store transactions with local time adjustments.
 - **Fields Included**: All fields from `#F_Ledger_Txn` plus calculated fields for local transaction date, minute IDs, and cut-off duration.

### Logic for Time Conversion
 - **Timezone Adjustment**:
 - The logic involves adjusting the `Central_Transaction_Datetime` to `Local_Transaction_Datetime` using the data from the timezone adjustment table `Audit.vwDSTTimeZone`.
 - This ensures that transaction times reflect the local time zone of the player, allowing for accurate tracking and analysis of player activity based on their local time.

### Conclusion
 - **Yes**, the central transaction time (Central_Transaction_Datetime) is converted to local time (Local_Transaction_Datetime) as part of the processing logic. This adjustment is crucial for accurate session tracking, identifying inactivity periods, and differentiating between late-night and day play.

## Keyword Explanation

### Transaction
 - **Transaction**: An atomic operation where some information is exchanged and treated as a single unit.

This time conversion ensures that all subsequent analyses and reports are based on the local transaction times, providing a more accurate and relevant understanding of player behavior and activity patterns.
