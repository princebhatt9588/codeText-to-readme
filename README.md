# Comprehensive Summary of Logic and Terms

## Initialization
 - **Logs the start time** of the procedure.

## Temporary Table for Ledger Transactions
 - **Tables Joined**: FactledgerTransactions, DimPlayer, dim_player_CZ.
 - **Purpose**: To collect and store detailed ledger transactions.
 - **Fields Included**: 
 - Player IDs
 - Transaction IDs
 - Game type
 - Transaction codes
 - Network code
 - Transaction datetime
 - Transaction amounts
 - **Conditions**: Filters transactions by date range.

## Timezone Adjustment Table
 - **Table Used**: Audit.vwDSTTimeZone.
 - **Purpose**: To adjust transaction times to local time.
 - **Fields Included**: All fields from Audit.vwDSTTimeZone.

## Processed Transactions Table
 - **Tables Used**: #F_Ledger_Txn, #temp_vwDSTTimeZone.
 - **Purpose**: To store transactions with local time adjustments and additional calculated fields.
 - **Fields Included**: 
 - All fields from #F_Ledger_Txn
 - Calculated fields for local transaction date
 - Minute IDs
 - Cut-off duration

## Logic
 - **Timezone Adjustment**: Adjusts Central_Transaction_Datetime to Local_Transaction_Datetime.
 - **Calculations**: Adds minute IDs and local transaction date fields.
 - **Conditions**: Filters transactions by local transaction date.

## Identifying Inactivity and Sessions
 - **Table Created**: CZ_RG.Stage_F_Ledger_Player_Session.
 - **Tables Used**: CZ_RG.F_Ledger_Txn_Temp.
 - **Lag Function**: Identifies previous transaction times.
 - **Inactivity Flag**: Marks periods of inactivity based on Cut_Off_Duration.
 - **Conditions**: Filters for 'Bet' transactions with specific statuses.

## Differentiating Late Night and Day Play
 - **Table Created**: CZ_RG.Stage_F_Player_Session.
 - **Tables Used**: CZ_RG.Stage_F_Ledger_Player_Session.
 - **Late Night Play**: Transactions between midnight and 5 AM.
 - **Day Play**: Transactions between 6 AM and 11 PM.
 - **Session ID**: Unique ID based on player ID, date, play type, and inactivity count.
 - **Conditions**: Filters based on transaction times.

## Inserting Player Sessions
 - **Permanent Table**: CZ_RG.F_Player_Session.
 - **Source Table**: CZ_RG.Stage_F_Player_Session.
 - **Session Duration**: Calculated from session start and end times.
 - **Cut-Off Duration**: Used if session duration is zero.
 - **Late Night Play Flag**: Indicates late-night play.
 - **Conditions**: Inserts data within the specified date range.

## Preparing Network Table
 - **Temporary Table**: #Networks.
 - **Source Table**: [CZ_DER_ARCHIVE].[NETWORKS].
 - **Logic**: Excludes specific portal entries.

## Preparing Bet Type Table
 - **Temporary Table**: #Cliso.
 - **Source Table**: CZ_RG.Vw_NVP_Tab_Sazenky.
 - **Logic**: Matches ticket codes from CZ_RG.F_Ledger_Txn_Temp.

## Key Points
 - **Joins**: Multiple tables are joined to gather detailed transaction data, player information, and bet types.
 - **Conditions**: Filters based on transaction statuses, times, and date ranges.
 - **Inactivity Flag**: Marks periods of inactivity based on transaction time differences.
 - **Late Night Play**: Transactions between midnight and 5 AM.
 - **Session ID**: Unique identifier for player sessions.
 - **Cut-Off Duration**: Threshold for identifying session breaks.

## Terms
 - **Central_Transaction_Datetime**: Original transaction time.
 - **Local_Transaction_Datetime**: Adjusted transaction time based on timezone.
 - **Cut_Off_Duration**: Threshold for identifying session breaks.

## Keyword Explanation

### Bet
 Represented by a betslip. Types include:
 - Single (solo)
 - Straight accumulator (AKO)
 - Stepwise accumulator (FALC)
 - System bet (Maxikombi)

In the context of casino gaming, it represents one spin in the game or the amount of money staked in one position.

### Ticket
 A container of a placed bet(s) with all the additional information.

### Session
 A server-side storage of information persisting throughout the user's interaction with the website or web application. Instead of storing large and constantly changing information in the user's browser, only a unique identifier is stored on the client side. Through this identifier, the web application can pair internal data with client-side requests.

### Transaction
 An atomic operation where some information is exchanged and treated as a single unit.

## Conclusion
 The comprehensive process outlined in this document is designed to collect, adjust, and analyze detailed ledger transactions for player sessions. By joining multiple tables, applying date range filters, and adjusting for timezones, the system ensures accurate and relevant transaction data. Identifying periods of inactivity and differentiating between late-night and day play allow for a deeper understanding of player behavior. The final result is a robust dataset that supports detailed analysis and reporting, aiding in the identification of player trends and behaviors for better decision-making and strategy development.
