# Analysis of Base Proc Query

## 1. Initial Setup and Logging

### Initialization:
- The stored procedure `[CZ_RG].[Refresh_RG_F_Ledger_Session]` begins by declaring the runtime and logging the start time into the `MAINTENANCE.Daily_Proc_Info_CZ` table.

### Parameters:
- The procedure accepts three parameters: 
 - `@from_Date`
 - `@to_Date`
 - `@cutOffDurationSeconds`

## 2. Temporary Table Preparation for Ledger Transactions

### Dropping Existing Temporary Table:
- If a temporary table `#F_Ledger_Txn` exists, it is dropped.

### Creating Temporary Table `#F_Ledger_Txn`:
**Purpose:**
- This table is created to store ledger transactions within the specified date range.

**Joins:**
- `CZ_presentation.FactledgerTransactions`: Main source of ledger transactions.
- `CZ_presentation_views.DimPlayer`: To join player details using `whPlayerID`.
- `prod_views.dim_player_CZ`: To fetch additional player information using `username`.

**Fields Included:**
- `Wh_Player_Id`
- `Src_Player_Id`
- `Wh_Ledger_Transaction_ID`
- `Src_Ledger_Serial_ID`
- `Src_Ledger_Transaction_ID`
- `Src_Game_type`
- `Remote_Transaction_Code`
- `Network_Code`
- `Central_Transaction_Datetime`
- `Central_Transaction_Date`
- `Central_Transaction_Date_ID`
- `Ledger_Transaction_Chapter`
- `Ledger_Transaction_Stats`
- `Ledger_Transaction_Amount`
- `Src_Ticket_CD`

**Conditions:**
- Filters transactions based on `CentralTransactionDatetime` falling within the specified date range (`@from_Date` to `@to_Date`).

## 3. Timezone Adjustment Table

### Dropping Existing Temporary Table:
- If a temporary table `#temp_vwDSTTimeZone` exists, it is dropped.

### Creating Temporary Table `#temp_vwDSTTimeZone`:
**Purpose:**
- This table stores timezone adjustments to convert transaction times to local times.

**Source:**
- Data is fetched from `Audit.vwDSTTimeZone` for the 'Central' timezone.

**Fields Included:**
- All fields from `Audit.vwDSTTimeZone`.

## 4. Processing Transactions with Local Time Adjustments

### Dropping Existing Table:
- If the table `CZ_RG.F_Ledger_Txn_Temp` exists, it is dropped.

### Creating Table `CZ_RG.F_Ledger_Txn_Temp`:
**Purpose:**
- To store processed transactions with local time adjustments.

**Source:**
- Data is selected from `#F_Ledger_Txn`.

**Joins:**
- `#temp_vwDSTTimeZone`: To adjust `Central_Transaction_Datetime` to local time.

**Fields Included:**
- All fields from `#F_Ledger_Txn`.
- Additional calculated fields: 
 - `Central_Transaction_Date_Minute_Id`
 - `Local_Transaction_Datetime`
 - `Local_Transaction_Date`
 - `Local_Transaction_Date_ID`
 - `Local_Transaction_Date_Minute_Id`
 - `Cut_Off_Duration`

**Logic:**
- **Timezone Adjustment:** Adjusts `Central_Transaction_Datetime` to `Local_Transaction_Datetime` using `#temp_vwDSTTimeZone`.
- **Calculations:**
 - `Central_Transaction_Date_Minute_Id`: Minute ID for central transaction date.
 - `Local_Transaction_Date`: Local date derived from the adjusted `Central_Transaction_Datetime`.
 - `Local_Transaction_Date_ID`: Local date ID in the format `YYYYMMDD`.
 - `Local_Transaction_Date_Minute_Id`: Minute ID for local transaction date.
 - `Cut_Off_Duration`: Uses the `@cutOffDurationSeconds` parameter.

**Conditions:**
- Filters transactions based on `Local_Transaction_Date` falling within the specified date range (`@from_Date` to `@to_Date`).

## Summary of Logic and Terms in Query 1

### Initialization:
- Logs the start time of the procedure.

### Temporary Table for Ledger Transactions:
- **Tables Joined:** `FactledgerTransactions`, `DimPlayer`, `dim_player_CZ`.
- **Purpose:** To collect and store detailed ledger transactions.
- **Fields Included:** Player IDs, transaction IDs, game type, transaction codes, network code, transaction datetime, transaction amounts, etc.
- **Conditions:** Filters transactions by date range.

### Timezone Adjustment Table:
- **Table Used:** `Audit.vwDSTTimeZone`.
- **Purpose:** To adjust transaction times to local time.
- **Fields Included:** All fields from `Audit.vwDSTTimeZone`.

### Processed Transactions Table:
- **Tables Used:** `#F_Ledger_Txn`, `#temp_vwDSTTimeZone`.
- **Purpose:** To store transactions with local time adjustments and additional calculated fields.
- **Fields Included:** All fields from `#F_Ledger_Txn` plus calculated fields for local transaction date, minute IDs, and cut-off duration.

### Logic:
- **Timezone Adjustment:** Adjusts `Central_Transaction_Datetime` to `Local_Transaction_Datetime`.
- **Calculations:** Adds minute IDs and local transaction date fields.
- **Conditions:** Filters transactions by local transaction date.

### Key Points:
- **Joins:** Multiple tables are joined to gather detailed transaction data and player information.
- **Conditions:** Date range filters are applied to ensure only relevant transactions are processed.

### Terms:
- **Central_Transaction_Datetime:** Original transaction time.
- **Local_Transaction_Datetime:** Adjusted transaction time based on timezone.
- **Cut_Off_Duration:** Threshold for identifying session breaks.

## 1. Processing Ledger Transactions to Identify Inactivity and Sessions

- **Dropping Existing Table:** If the table `CZ_RG.Stage_F_Ledger_Player_Session` exists, it is dropped.
- **Creating Table `CZ_RG.Stage_F_Ledger_Player_Session`:**
 - **Purpose:** To identify periods of inactivity and session starts within the transactions.
 - **Source:** Data is selected from `CZ_RG.F_Ledger_Txn_Temp`.

**Logic:**
- **Lag Function:** Uses the `LAG` window function to get the previous transaction time for each player on the same day.
- **Inactivity Flag:** Determines if there is inactivity based on the difference between the current and previous transaction times, compared to the `Cut_Off_Duration`.
- **Conditions:** Filters to include only 'Bet' transactions with specific statuses ('bet', 'bonus_bet').

## 2. Differentiating Between Late Night and Day Play

- **Dropping Existing Table:** If the table `CZ_RG.Stage_F_Player_Session` exists, it is dropped.
- **Creating Table `CZ_RG.Stage_F_Player_Session`:**
 - **Purpose:** To differentiate between late-night play (midnight to 6 AM) and day play (6 AM to 11 PM).
 - **Source:** Data is selected from `CZ_RG.Stage_F_Ledger_Player_Session`.

**Logic:**
- **Late Night Play:** Identifies sessions where transactions occur between midnight and 5 AM.
- **Day Play:** Identifies sessions where transactions occur between 6 AM and 11 PM.
- **Session ID:** Generates a unique session ID based on player ID, date, play type (late night or day), and inactivity count.
- **Conditions:** Filters transactions based on the hour of the day.

## 3. Inserting Player Sessions into Permanent Table

- **Transaction Management:** Begins a transaction to ensure data integrity.
- **Deleting Existing Data:** Deletes existing player sessions for the specified date range in `CZ_RG.F_Player_Session`.
- **Inserting New Data:**
 - **Source:** Data is selected from `CZ_RG.Stage_F_Player_Session`.

**Logic:**
- **Session Duration:** Calculates session duration as the difference between session start and end times.
- **Cut-Off Duration:** Uses the `Cut_Off_Duration` parameter for session duration if it is zero.
- **Late Night Play Flag:** Indicates if the session includes late-night play.
- **Conditions:** Inserts data within the specified date range.

## 4. Preparing Network Table

- **Dropping Existing Temporary Table:** If the temporary table `#Networks` exists, it is dropped.
- **Creating Temporary Table `#Networks`:**
 - **Purpose:** To store distinct network codes and names, excluding specific portal entries.
 - **Source:** Data is selected from `[CZ_DER_ARCHIVE].[NETWORKS]`.

## 5. Preparing Bet Type Table

- **Dropping Existing Temporary Table:** If the temporary table `#Cliso` exists, it is dropped.
- **Creating Temporary Table `#Cliso`:**
 - **Purpose:** To store bet type information.
 - **Source:** Data is selected from `CZ_RG.Vw_NVP_Tab_Sazenky` where the ticket code matches those in `CZ_RG.F_Ledger_Txn_Temp`.

## Summary of Logic and Terms

### Identifying Inactivity and Sessions:
- **Table Created:** `CZ_RG.Stage_F_Ledger_Player_Session`.
- **Tables Used:** `CZ_RG.F_Ledger_Txn_Temp`.
- **Lag Function:** Identifies previous transaction times.
- **Inactivity Flag:** Marks periods of inactivity based on `Cut_Off_Duration`.
- **Conditions:** Filters for 'Bet' transactions with specific statuses.

### Differentiating Late Night and Day Play:
- **Table Created:** `CZ_RG.Stage_F_Player_Session`.
- **Tables Used:** `CZ_RG.Stage_F_Ledger_Player_Session`.
- **Late Night Play:** Transactions between midnight and 5 AM.
- **Day Play:** Transactions between 6 AM and 11 PM.
- **Session ID:** Unique ID based on player ID, date, play type, and inactivity count.
- **Conditions:** Filters based on transaction times.

### Inserting Player Sessions:
- **Permanent Table:** `CZ_RG.F_Player_Session`.
- **Source Table:** `CZ_RG.Stage_F_Player_Session`.
- **Session Duration:** Calculated from session start and end times.
- **Cut-Off Duration:** Used if session duration is zero.
- **Late Night Play Flag:** Indicates late-night play.
- **Conditions:** Inserts data within the specified date range.

### Preparing Network Table:
- **Temporary Table:** `#Networks`.
- **Source Table:** `[CZ_DER_ARCHIVE].[NETWORKS]`.
- **Logic:** Excludes specific portal entries.

### Preparing Bet Type Table:
- **Temporary Table:** `#Cliso`.
- **Source Table:** `CZ_RG.Vw_NVP_Tab_Sazenky`.
- **Logic:** Matches ticket codes from `CZ_RG.F_Ledger_Txn_Temp`.

### Key Points:
- **Joins:** Multiple tables are joined to gather detailed transaction data, player information, and bet types.
- **Conditions:** Filters based on transaction statuses, times, and date ranges.
- **Inactivity Flag:** Marks periods of inactivity based on transaction time differences.
- **Late Night Play:** Transactions between midnight and 5 AM.
- **Session ID:** Unique identifier for player sessions.
- **Cut-Off Duration:** Threshold for identifying session breaks.

## 6. Preparing Financial Transactions Data

- **Dropping Existing Temporary Table:** If the table `#LEDGERENTRIESFACTS_Fact` exists, it is dropped.
- **Creating Temporary Table `#LEDGERENTRIESFACTS_Fact`:**
 - **Purpose:** To store financial transaction details for bets and bonus bets.
 - **Source:** Data is selected from `CZ_DER_ARCHIVE.LEDGERENTRIESFACTS`.
 - **Conditions:** Filters transactions where:
 - `WA_CODE` is null.
 - `CHAPTER` is 'Bet'.
 - `STATSINFO` is in ('bet', 'bonus_bet').
 - `EventDate` falls within the specified date range (`@from_Date` to `@to_Date`).

## 7. Adjusting Financial Transaction Times

- **Dropping Existing Temporary Table:** If the table `#LEDGERENTRIESFACTS` exists, it is dropped.
- **Creating Temporary Table `#LEDGERENTRIESFACTS`:**
 - **Purpose:** To store financial transactions with local time adjustments.
 - **Source:** Data is selected from `#LEDGERENTRIESFACTS_Fact` with the following adjustments:
 - **Timezone Adjustment:** `EventDate` is adjusted to local time using the `#temp_vwDSTTimeZone` table.
 - **Additional Fields:**
 - `LocalEventdate`: Local date derived from the adjusted `EventDate`.
 - `whLedgerTransactionID`: A unique identifier generated using a hash of `lwCode`
