
#### 6. Preparing Financial Transactions Data
 - **Dropping Existing Temporary Table**: If the table `#LEDGERENTRIESFACTS_Fact` exists, it is dropped.
 - **Creating Temporary Table `#LEDGERENTRIESFACTS_Fact**:
 - **Purpose**: To store financial transaction details for bets and bonus bets.
 - **Source**: Data is selected from `CZ_DER_ARCHIVE.LEDGERENTRIESFACTS`.
 - **Conditions**: Filters transactions where:
 - `WA_CODE` is null.
 - `CHAPTER` is 'Bet'.
 - `STATSINFO` is in ('bet', 'bonus_bet').
 - `EventDate` falls within the specified date range (`@from_Date` to `@to_Date`).

#### 7. Adjusting Financial Transaction Times
 - **Dropping Existing Temporary Table**: If the table `#LEDGERENTRIESFACTS` exists, it is dropped.
 - **Creating Temporary Table `#LEDGERENTRIESFACTS`**:
 - **Purpose**: To store financial transactions with local time adjustments.
 - **Source**: Data is selected from `#LEDGERENTRIESFACTS_Fact` with the following adjustments:
 - **Timezone Adjustment**: `EventDate` is adjusted to local time using the `#temp_vwDSTTimeZone` table.
 - **Additional Fields**: 
 - `LocalEventdate`: Local date derived from the adjusted `EventDate`.
 - `whLedgerTransactionID`: A unique identifier generated using a hash of `lwCode`, `code`, and `eventDate`.

#### 8. Preparing Wallet Game Action Facts
 - **Dropping Existing Temporary Table**: If the table `#WalletGameActionFacts` exists, it is dropped.
 - **Creating Temporary Table `#WalletGameActionFacts`**:
 - **Purpose**: To store wallet game action facts related to financial transactions.
 - **Source**: Data is selected from `#LEDGERENTRIESFACTS` and joined with:
 - `CZ_DER_ARChive.Walletgameactionfacts`: To get game codes using `WGA_CODE`.
 - `Prod_views.Dim_Player_CZ`: To get player IDs using `Playercode`.
 - **Additional Fields**:
 - `Gamecode`: Derived from `REMOTESESSIONCODE` or `NGS_Gamecode`.
 - `whLedgerTransactionID`: Carried over from `#LEDGERENTRIESFACTS`.
 - **Conditions**: Filters transactions where `EventDate` falls within the specified date range.

#### 9. Preparing Financial Transactions Summary
 - **Dropping Existing Temporary Table**: If the table `#F_Financial_Txn` exists, it is dropped.
 - **Creating Temporary Table `#F_Financial_Txn`**:
 - **Purpose**: To store summarized financial transactions.
 - **Source**: Data is selected from `CZ_presentation.FactFinancialTransactions` and joined with:
 - `DimPlayer`: To get player details using `whPlayerID`.
 - `dim_player_CZ`: To get additional player information using `username`.
 - `#temp_vwDSTTimeZone`: To adjust transaction times to local time.
 - **Additional Fields**:
 - `Summary_Date`: Local date derived from the adjusted `CentralTransactionDatetime`.

#### 10. Aggregating Financial Data
 - **Dropping Existing Temporary Table**: If the table `#Fact_Financial_Agg` exists, it is dropped.
 - **Creating Temporary Table `#Fact_Financial_Agg`**:
 - **Purpose**: To aggregate financial data for each player.
 - **Source**: Data is selected from `#F_Financial_Txn` and joined with:
 - `#F_Ledger_Session_RG_CZ`: To get session IDs.
 - `vw_DimDWHFeedToControlReportMapping`: To map reporting system KPIs using `whglentryid`.
 - `DimPaymentMethod`: To get payment method details.
 - `Dimstore`: To get store details.
 - `Dim_Cash_Register_CZ`: To get cash register details.
 - `#WalletGameActionFacts`: To get game codes using `whLedgerTransactionID`.
 - **Aggregations**:
 - **Stakes**:
 - `Cash_Stake_Amt`: Sum of cash stake amounts.
 - `Bonus_Stake_Amt`: Sum of bonus stake amounts.
 - `Pending_Winning_Stake_Amt`: Sum of pending winning stake amounts.
 - **Payouts**:
 - `Cash_Payout_Amt`: Sum of cash payout amounts.
 - `Bonus_Payout_Amt`: Sum of bonus payout amounts.
 - `Player_Bonus_Win_To_Pending_Winnings_Amt`: Sum of bonus win to pending winnings amounts.
 - **Bonuses**:
 - `Player_Bonus_Added_Amt`: Sum of player bonus added amounts.
 - `Player_Bonus_Removed_Amt`: Sum of player bonus removed amounts.
 - `Player_Pending_Winnings_Removed_Amt`: Sum of player pending winnings removed amounts.
 - `Player_Immediately_Redeemed_Bonus_Amt`: Sum of immediately redeemed bonus amounts.
 - `Player_Redeemed_Afterwager_Amt`: Sum of redeemed after wager amounts.
 - `Player_Redeemed_Bonus_Amt`: Sum of redeemed bonus amounts.
 - `Player_Redeemed_Casino_Bonus_Initial_Amt`: Sum of redeemed casino bonus initial amounts.
 - `Player_Redeemed_Casino_Bonus_Pending_Winnings_Amt`: Sum of redeemed casino bonus pending winnings amounts.
 - **Costs**:
 - `Handling_Fee_Amt`: Sum of handling fee amounts.
 - `Sportsbook_Bonus_Cost_Amt`: Sum of sportsbook bonus cost amounts.
 - **Deposits and Withdrawals**:
 - `Deposit_Amt`: Sum of deposit amounts.
 - `Deposit_Count`: Count of distinct deposit transactions.
 - `Withdrawal_Amt`: Sum of withdrawal amounts.
 - `Withdrawal_Count`: Count of distinct withdrawal transactions.
 - `Declined_Withdrawal_Amt`: Sum of declined withdrawal amounts.
 - `Declined_Withdrawal_Count`: Count of distinct declined withdrawal transactions.
 - **Bets**:
 - `Bet_Count`: Count of distinct bets.
 - **Conditions**: Filters transactions within the specified date range.

#### 11. Inserting Aggregated Financial Data into Permanent Table
 - **Transaction Management**: Begins a transaction to ensure data integrity.
 - **Deleting Existing Data**: Deletes existing financial transactions for the specified date range in `CZ_RG.FD_Player_Financial_Transactions`.
 - **Inserting New Data**:
 - **Source**: Data is selected from `#Fact_Financial_Agg`.
 - **Logic**:
 - **Aggregations**: Inserts aggregated financial metrics for each player.
 - **Conditions**: Inserts data where the sum of absolute values of various metrics is greater than zero.

#### 12. Final Logging
 - **Logging Completion**: Logs the completion time of the procedure into `MAINTENANCE.Daily_Proc_Info_CZ`.

### Summary of Logic and Terms in Query 3

1. **Preparing Financial Transactions Data**:
 - **Table Created**: `#LEDGERENTRIESFACTS_Fact`.
 - **Source Table**: `CZ_DER_ARCHIVE.LEDGERENTRIESFACTS`.
 - **Conditions**: Filters for bets and bonus bets within the specified date range.

2. **Adjusting Financial Transaction Times**:
 - **Table Created**: `#LEDGERENTRIESFACTS`.
 - **Source Table**: `#LEDGERENTRIESFACTS_Fact`.
 - **Logic**: Adjusts event dates to local time using `#temp_vwDSTTimeZone`.

3. **Preparing Wallet Game Action Facts**:
 - **Table Created**: `#WalletGameActionFacts`.
 - **Tables Used**: `#LEDGERENTRIESFACTS`, `CZ_DER_ARChive.Walletgameactionfacts`, `Prod_views.Dim_Player_CZ`.
 - **Logic**: Joins to get game codes and player IDs.
 - **Conditions**: Filters transactions within the specified date range.

4. **Preparing Financial Transactions Summary**:
 - **Table Created**: `#F_Financial_Txn`.
 - **Tables Used**: `CZ_presentation.FactFinancialTransactions`, `DimPlayer`, `dim_player_CZ`, `#temp_vwDSTTimeZone`.
 - **Logic**: Joins to get player details and adjust transaction times.
 - **Conditions**: Filters transactions within the specified date range.

5. **Aggregating Financial Data**:
 - **Table Created**: `#Fact_Financial_Agg`.
 - **Tables Used**: `#F_Financial_Txn`, `#F_Ledger_Session_RG_CZ`, `vw_DimDWHFeedToControlReportMapping`, `DimPaymentMethod`, `Dimstore`, `#WalletGameActionFacts`.
 - **Logic**: Aggregates various financial metrics for each player.
 - **Conditions**: Filters transactions within the specified date range.

6. **Inserting Aggregated Financial Data**:
 - **Permanent Table**: `CZ_RG.FD_Player_Financial_Transactions`.
 - **Source Table**: `#Fact_Financial_Agg`.
 - **Logic**: Inserts aggregated financial metrics.
 - **Conditions**: Inserts data where the sum of absolute values of metrics is greater than zero.

7. **Final Logging**:
 - Logs the completion time into `MAINTENANCE.Daily_Proc_Info_CZ`.

### Key Points:
 - **Joins**: Multiple tables are joined to gather detailed financial transaction data, game action facts, and player information.
 - **Conditions**: Filters based on transaction types, statuses, and date ranges.
 - **EventDate**: Original event time.
 - **LocalEventdate**: Adjusted event time based on timezone.
 - **Aggregations**: Sums and counts of various financial metrics.
 - **Cut-Off Duration**: Threshold for identifying session breaks.
