

### Analysis of Base Proc Query

#### 1. Initial Setup and Logging
 - **Initialization**: The stored procedure `[CZ_RG].[Refresh_RG_F_Ledger_Session]` begins by declaring the runtime and logging the start time into the `MAINTENANCE.Daily_Proc_Info_CZ` table.
 - **Parameters**: The procedure accepts three parameters: `@from_Date`, `@to_Date`, and `@cutOffDurationSeconds`.

#### 2. Temporary Table Preparation
 - **Dropping Existing Temporary Table**: If a temporary table `#F_Ledger_Txn` exists, it is dropped.
 - **Creating Temporary Table `#F_Ledger_Txn`**:
 - **Purpose**: This table is created to store ledger transactions within the specified date range.
 - **Joins**:
 - `CZ_presentation.FactledgerTransactions`: Main source of ledger transactions.
 - `CZ_presentation_views.DimPlayer`: To join player details using `whPlayerID`.
 - `prod_views.dim_player_CZ`: To fetch additional player information using `username`.
 - **Conditions**: Filters transactions based on `CentralTransactionDatetime` falling within the specified date range (`@from_Date` to `@to_Date`).

#### 3. Timezone Adjustment Table
 - **Dropping Existing Temporary Table**: If a temporary table `#temp_vwDSTTimeZone` exists, it is dropped.
 - **Creating Temporary Table `#temp_vwDSTTimeZone`**:
 - **Purpose**: This table stores timezone adjustments to convert transaction times to local times.
 - **Source**: Data is fetched from `Audit.vwDSTTimeZone` for the 'Central' timezone.

#### 4. Processing Transactions
 - **Dropping Existing Table**: If the table `CZ_RG.F_Ledger_Txn_Temp` exists, it is dropped.
 - **Creating Table `CZ_RG.F_Ledger_Txn_Temp`**:
 - **Purpose**: To store processed transactions with local time adjustments.
 - **Source**: Data is selected from `#F_Ledger_Txn` with additional calculations and joins:
 - **Timezone Adjustment**: Uses `#temp_vwDSTTimeZone` to adjust `Central_Transaction_Datetime` to local time.
 - **Additional Fields**: Calculates local transaction date, minute ID, and a cut-off duration.
 - **Conditions**: Filters transactions based on `Local_Transaction_Date` falling within the specified date range (`@from_Date` to `@to_Date`).

### Summary of Logic and Terms in Query 1

1. **Initialization**:
 - Logs the start time of the procedure.

2. **Temporary Table for Ledger Transactions**:
 - **Tables Joined**: `FactledgerTransactions`, `DimPlayer`, `dim_player_CZ`.
 - **Purpose**: To collect and store detailed ledger transactions.
 - **Conditions**: Filters transactions by date range.

3. **Timezone Adjustment Table**:
 - **Table Used**: `Audit.vwDSTTimeZone`.
 - **Purpose**: To adjust transaction times to local time.

4. **Processed Transactions Table**:
 - **Tables Used**: `#F_Ledger_Txn`, `#temp_vwDSTTimeZone`.
 - **Purpose**: To store transactions with local time adjustments and additional calculated fields.
 - **Conditions**: Filters transactions by local transaction date.

### Key Points:
 - **Joins**: Multiple tables are joined to gather detailed transaction data and player information.
 - **Conditions**: Date range filters are applied to ensure only relevant transactions are processed.
 - **Terms**:
 - **Central_Transaction_Datetime**: Original transaction time.
 - **Local_Transaction_Datetime**: Adjusted transaction time based on timezone.
 - **Cut_Off_Duration**: Threshold for identifying session breaks.

This analysis should provide a clear understanding of the logic and flow within Query 1 of the stored procedure. We'll continue with Query 2 and Query 3 similarly, breaking them down in detail.
