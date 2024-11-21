# Analysis of Risk Bucket 

Proc : CZ_RG.Refresh_FD_Player_MOH_Value_Risk_Bucket

## Step-by-Step Breakdown

1. **Procedure Declaration:**
 - The procedure `[CZ_RG].[Refresh_FD_Player_MOH_Value_Risk_Bucket]` takes three parameters: `@FROM_DATE`, `@To_Date`, and `@Moh_Id`.
 - It initializes a runtime variable to the current date and time.

2. **Logging Start:**
 - Inserts a record into `MAINTENANCE.Daily_Proc_Info_CZ` to log the start of the procedure.

3. **Initialization:**
 - Sets the `@Run_Date` variable to `@FROM_DATE`.

4. **While Loop:**
 - The procedure iterates through each date from `@FROM_DATE` to `@To_Date`.

5. **Configuration and Cleanup:**
 - Based on the `@Moh_Id` parameter, it fetches the maximum execution order from `CZ_RG.MOH_Scores_Execution_Config`.
 - If `@Moh_Id` is specified, it deletes old records from the target table for the current `@Run_Date`.

6. **Execution of Configured Queries:**
 - Within the while loop, the procedure iterates through each execution order (`@seq` to `@maxseq`) and executes the corresponding query from `CZ_RG.MOH_Scores_Execution_Config`.

7. **Calculation of MOH Values:**
 - Creates a table `CZ_RG.Player_MOH_Calculated_Metrics` to store calculated metrics for each player.
 - The calculations involve dividing numerator values by denominator values for specified MOH IDs.
 - The MOH IDs used in this calculation are: 3, 9, 10, 11, 12, 13, 14, and 17.

8. **Loading MOH Values:**
 - Deletes existing records for the current `@Run_Date` from `CZ_RG.FD_Player_MOH_Value`.
 - Inserts new calculated MOH values into `CZ_RG.FD_Player_MOH_Value`.

9. **Risk Bucket Calculation:**
 - Deletes existing records for the current `@Run_Date` from `CZ_RG.FD_Player_MOH_Value_Risk_Bucket`.
 - Inserts new risk bucket values into `CZ_RG.FD_Player_MOH_Value_Risk_Bucket`.
 - Uses the following logic for the insertion:
 - Calculates the overall MOH score for each player.
 - Determines the highest MOH score and its corresponding MOH ID.
 - Looks up the risk bucket and RG group based on the overall MOH score and highest MOH ID.
 - Uses a special condition for players with no risk based on their net deposits.

10. **Duplicate Check:**
 - Checks for duplicate records in `CZ_RG.FD_Player_MOH_Base_Metrics`, `CZ_RG.Player_MOH_Calculated_Metrics`, `CZ_RG.FD_Player_MOH_Value`, and `CZ_RG.FD_Player_MOH_Value_Risk_Bucket`.
 - Throws an error if duplicates are found.

11. **Logging Completion:**
 - Inserts a record into `MAINTENANCE.Daily_Proc_Info_CZ` to log the completion of the procedure.

## Terms, Logic, and Formulas

1. **Run Date (`@Run_Date`):**
 - The date for which the calculations are being performed.

2. **MOH (Marker of Harm):**
 - A metric used to assess the risk level of players based on their behavior.

3. **Execution Order (`Execution_Order`):**
 - The sequence in which the queries are executed.

4. **Numerator and Denominator:**
 - Used in the calculation of MOH values.
 - **Formula:** `MOH_Value = Numerator_Value / Denominator_Value`
 - If the denominator is zero or null, the MOH value is set to null.

5. **Risk Bucket:**
 - Categorizes players into different risk levels based on their overall MOH score.
 - Lookup from `CZ_RG.MOH_Risk_Bucket_Lookup_Config`.

6. **RG Group:**
 - Responsible Gambling group assigned to players based on their risk bucket and highest MOH ID.
 - Lookup from `CZ_RG.MOH_RG_Group_Lookup_Config`.

7. **Player Base Rules:**
 - Filters players based on their activity, net loss, and the number of active days in the last 180 days.

8. **Markers:**
 - Specific metrics used to calculate MOH values.
 - Examples include "Net Deposits vs Last 180 Days Net Deposits" (Marker 10), "Frequency of Play Increase" (Marker 3), and others.

## Example Formulas

1. **Frequency of Play Increase (Marker 3):**
 - `MOH_Value = Session_Duration_Hours_LastWeek / Avg_Weekly_Session_Duration_Hours_L180Days`

2. **Net Deposits vs Last 180 Days Net Deposits (Marker 10):**
 - `MOH_Value = NetDeposit_YesterDay / Avg_NetDeposit_L180Days`

3. **Count of Deposits vs Average in the L180 Days (Marker 9):**
 - `MOH_Value = Deposit_Count / Avg_Deposit_Frequency_L180`

## Summary

The query follows a structured approach to calculate and categorize player risk levels based on various markers. It performs the following key actions:

1. Iterates through specified dates.
 2. Executes configured queries to load base metrics.
 3. Calculates MOH values for each player.
 4. Assigns risk buckets and RG groups based on overall MOH scores.
 5. Checks and prevents duplicate records.
 6. Logs the start and completion of the procedure.

---

**Note:**

1. **Player Base Rules:**
 - The query is filtering players based on the following conditions:
 - `Total_Stake_Amt > 0 or Deposit_Amt > 0.0` (Rule 1)
 - `((GrossWin - BonusCost) >= 0)` (Rule 2)
 - The players should have at least 10 APDs in the last 180 days (Rule 3)

## Markers Analysis

### Marker 10 – Net Deposits vs Last 180 Days Net Deposits
 - The logic for this marker is used in the insertion into `CZ_RG.FD_Player_MOH_Value_Risk_Bucket`.
 - The query checks for the players' net deposits and evaluates their risk bucket.

```sql
 insert into CZ_RG.FD_Player_MOH_Value_Risk_Bucket
 SELECT 
 Summary_Date AS Run_Date,
 Wh_Player_ID,
 0 AS Overall_Moh_Score,
 10 AS Highest_MOH_ID,
 'Marker 10 - Net Deposits vs Last 180 Days Net Deposits' AS Highest_Marker_Name,
 'Depositing' AS Highest_Marker_Type,
 0 AS Max_Moh_Score,
 GETDATE() AS ETL_Record_Load_Datetime,
 'No Risk' AS Risk_Bucket,
 1 AS RG_Group
 FROM (
 SELECT 
 Summary_Date,
 a.Wh_Player_ID
 FROM (
 SELECT 
 a.Wh_Player_ID,
 Summary_Date,
 SUM(COALESCE(Total_Stake_Amt, 0)) AS Total_Stake_Amt,
 SUM(COALESCE(Deposit_Amt, 0)) AS Deposit_Amt,
 SUM(COALESCE(Total_Stake_Amt, 0) + COALESCE(Handling_Fee_Amt, 0) - COALESCE(Total_Payout_Amt, 0)) AS GrossWin,
 SUM(COALESCE(Casino_Bonus_Cost, 0) + COALESCE(Sportsbook_Bonus_Cost_Amt, 0)) AS BonusCost 
 FROM CZ_RG.FD_Player_Financial_Transactions a
 JOIN prod_views.Dim_player_CZ b 
 ON a.Wh_Player_ID = b.Wh_Player_ID 
 AND b.Internal_Player_YN = 'N'
 JOIN (
 SELECT distinct Wh_Player_ID 
 FROM (
 SELECT 
 Wh_Player_ID,
 COUNT(Summary_Date) AS Count 
 FROM CZ_RG.FD_Player_Financial_Transactions 
 WHERE Summary_Date BETWEEN DATEADD(day, -180, @Run_Date) AND @Run_Date and Cash_Stake_Amt>0
 GROUP BY Wh_Player_ID
 HAVING COUNT(Summary_Date) <= 10
 ) a
 ) c 
 ON a.Wh_Player_ID = c.Wh_Player_ID
 WHERE Summary_Date = @Run_Date 
 GROUP BY a.Wh_Player_ID, Summary_Date
 ) a
 WHERE (GrossWin - BonusCost) < 0
 ) a
 ```

### Marker 3 – Frequency of Play Increase
 - This marker's logic is present in the table `CZ_RG.FD_Player_MOH_Base_Metrics` and is used in the calculation of `MOH_Value`.

```sql
 when MOH in (3, 9, 10, 11, 12, 13, 14, 17) 
 then 
 case when Denominator_Value =0 or Denominator_Value is null then null
 else Numerator_Value/Denominator_Value end
 ```

### Marker 9 – Count of Deposits vs Average in the L180 Days
 - This marker's logic is used in the calculation for `Deposit_Count / Avg_Deposit_Frequency_L180`.

```sql
 when MOH in (3, 9, 10, 11, 12, 13, 14, 17) 
 then 
 case when Denominator_Value =0 or Denominator_Value is null then null
 else Numerator_Value/Denominator_Value end
 ```

### Marker 11 – Time on Site vs Average time on site
 - This marker's logic is used in the calculation for `Session_Duration_YesterDay / Avg_Session_Duration_L180Days`.

```sql
 when MOH in (3, 9, 10, 11, 12, 13, 14, 17) 
 then 
 case when Denominator_Value =0 or Denominator_Value is null then null
 else Numerator_Value/Denominator_Value end
 ```

### Marker 13 – Theoretical win vs average theoretical win in L180 days
 - This marker's logic is used in the calculation for `Theo_Win_Yesterday / Avg_Theo_Win_L180`.

```sql
 when MOH in (3, 9, 10, 11, 12, 13, 14, 17) 
 then 
 case when Denominator_Value =0 or Denominator_Value is null then null
 else Numerator_Value/Denominator_Value end
 ```

### Marker 14 – Player loss vs average player loss in the last 180 days
 - This marker's logic is used in the calculation for `Net_Loss_Yesterday / Avg_Net_Loss_L180`.

```sql
 when MOH in (3, 9, 10, 11, 12, 13, 14, 17) 
 then 
 case when Denominator_Value =0 or Denominator_Value is null then null
 else Numerator_Value/Denominator_Value end
 ```

### Marker 17 – Game Tile Play Increase on day
 - This marker's logic is used in the calculation for `Game_Tile_YesterDay / Game_Tiles_L180Days`.

```sql
 when MOH in (3, 9, 10, 11, 12, 13, 14, 17) 
 then 
 case when Denominator_Value =0 or Denominator_Value is null then null
 else Numerator_Value/Denominator_Value end
 ```

## Summary

The following markers are used in the query:

1. **Marker 3 – Frequency of Play Increase**
 2. **Marker 9 – Count of Deposits vs Average in the L180 Days**
 3. **Marker 10 – Net Deposits vs Last 180 Days Net Deposits**
 4. **Marker 11 – Time on Site vs Average time on site**
 5. **Marker 13 – Theoretical win vs average theoretical win in L180 days**
 6. **Marker 14 – Player loss vs average player loss in the last 180 days**
 7. **Marker 17 – Game Tile Play Increase on day**
