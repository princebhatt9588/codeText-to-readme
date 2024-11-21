
#### 1. Processing Ledger Transactions to Identify Inactivity and Sessions
 - **Dropping Existing Table**: If the table `CZ_RG.Stage_F_Ledger_Player_Session` exists, it is dropped.
 - **Creating Table `CZ_RG.Stage_F_Ledger_Player_Session`**:
 - **Purpose**: To identify periods of inactivity and session starts within the transactions.
 - **Source**: Data is selected from `CZ_RG.F_Ledger_Txn_Temp`.
 - **Logic**:
 - **Lag Function**: Uses the `LAG` window function to get the previous transaction time for each player on the same day.
 - **Inactivity Flag**: Determines if there is inactivity based on the difference between the current and previous transaction times, compared to the `Cut_Off_Duration`.
 - **Conditions**: Filters to include only 'Bet' transactions with specific statuses ('bet', 'bonus_bet').

#### 2. Differentiating Between Late Night and Day Play
 - **Dropping Existing Table**: If the table `CZ_RG.Stage_F_Player_Session` exists, it is dropped.
 - **Creating Table `CZ_RG.Stage_F_Player_Session`**:
 - **Purpose**: To differentiate between late-night play (midnight to 6 AM) and day play (6 AM to 11 PM).
 - **Source**: Data is selected from `CZ_RG.Stage_F_Ledger_Player_Session`.
 - **Logic**:
 - **Late Night Play**: Identifies sessions where transactions occur between midnight and 5 AM.
 - **Day Play**: Identifies sessions where transactions occur between 6 AM and 11 PM.
 - **Session ID**: Generates a unique session ID based on player ID, date, play type (late night or day), and inactivity count.
 - **Conditions**: Filters transactions based on the hour of the day.

#### 3. Inserting Player Sessions into Permanent Table
 - **Transaction Management**: Begins a transaction to ensure data integrity.
 - **Deleting Existing Data**: Deletes existing player sessions for the specified date range in `CZ_RG.F_Player_Session`.
 - **Inserting New Data**:
 - **Source**: Data is selected from `CZ_RG.Stage_F_Player_Session`.
 - **Logic**:
 - **Session Duration**: Calculates session duration as the difference between session start and end times.
 - **Cut-Off Duration**: Uses the `Cut_Off_Duration` parameter for session duration if it is zero.
 - **Late Night Play Flag**: Indicates if the session includes late-night play.
 - **Conditions**: Inserts data within the specified date range.

#### 4. Preparing Network Table
 - **Dropping Existing Temporary Table**: If the temporary table `#Networks` exists, it is dropped.
 - **Creating Temporary Table `#Networks`**:
 - **Purpose**: To store distinct network codes and names, excluding specific portal entries.
 - **Source**: Data is selected from `[CZ_DER_ARCHIVE].[NETWORKS]`.

#### 5. Preparing Bet Type Table
 - **Dropping Existing Temporary Table**: If the temporary table `#Cliso` exists, it is dropped.
 - **Creating Temporary Table `#Cliso`**:
 - **Purpose**: To store bet type information.
 - **Source**: Data is selected from `CZ_RG.Vw_NVP_Tab_Sazenky` where the ticket code matches those in `CZ_RG.F_Ledger_Txn_Temp`.

### Summary of Logic and Terms in Query 2

1. **Identifying Inactivity and Sessions**:
 - **Table Created**: `CZ_RG.Stage_F_Ledger_Player_Session`.
 - **Tables Used**: `CZ_RG.F_Ledger_Txn_Temp`.
 - **Logic**:
 - **Lag Function**: Identifies previous transaction times.
 - **Inactivity Flag**: Marks periods of inactivity based on `Cut_Off_Duration`.
 - **Conditions**: Filters for 'Bet' transactions with specific statuses.

2. **Differentiating Late Night and Day Play**:
 - **Table Created**: `CZ_RG.Stage_F_Player_Session`.
 - **Tables Used**: `CZ_RG.Stage_F_Ledger_Player_Session`.
 - **Logic**:
 - **Late Night Play**: Transactions between midnight and 5 AM.
 - **Day Play**: Transactions between 6 AM and 11 PM.
 - **Session ID**: Unique ID based on player ID, date, play type, and inactivity count.
 - **Conditions**: Filters based on transaction times.

3. **Inserting Player Sessions**:
 - **Permanent Table**: `CZ_RG.F_Player_Session`.
 - **Source Table**: `CZ_RG.Stage_F_Player_Session`.
 - **Logic**:
 - **Session Duration**: Calculated from session start and end times.
 - **Cut-Off Duration**: Used if session duration is zero.
 - **Late Night Play Flag**: Indicates late-night play.
 - **Conditions**: Inserts data within the specified date range.

4. **Preparing Network Table**:
 - **Temporary Table**: `#Networks`.
 - **Source Table**: `[CZ_DER_ARCHIVE].[NETWORKS]`.
 - **Logic**: Excludes specific portal entries.

5. **Preparing Bet Type Table**:
 - **Temporary Table**: `#Cliso`.
 - **Source Table**: `CZ_RG.Vw_NVP_Tab_Sazenky`.
 - **Logic**: Matches ticket codes from `CZ_RG.F_Ledger_Txn_Temp`.

### Key Points:
 - **Joins**: Multiple tables are joined to gather detailed transaction data, player information, and bet types.
 - **Conditions**: Filters based on transaction statuses, times, and date ranges.
 - **Terms**:
 - **Inactivity Flag**: Marks periods of inactivity based on transaction time differences.
 - **Late Night Play**: Transactions between midnight and 5 AM.
 - **Session ID**: Unique identifier for player sessions.
 - **Cut-Off Duration**: Threshold for identifying session breaks.
