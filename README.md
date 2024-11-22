### Detailed Analysis of Markers and Corresponding Tables:

1. **Marker 3 – Frequency of Play Increase**
 - **Configuration Table:** `CZ_RG.MOH_Scores_Execution_Config`
 - **Calculation Table:** `CZ_RG.Player_MOH_Calculated_Metrics`

2. **Marker 9 – Count of Deposits vs Average in the L180 Days**
 - **Configuration Table:** `CZ_RG.MOH_Scores_Execution_Config`
 - **Calculation Table:** `CZ_RG.Player_MOH_Calculated_Metrics`

3. **Marker 10 – Net Deposits vs Last 180 Days Net Deposits**
 - **Configuration Table:** `CZ_RG.MOH_Scores_Execution_Config`
 - **Calculation Table:** `CZ_RG.Player_MOH_Calculated_Metrics`
 - **Risk Bucket Table:** `CZ_RG.FD_Player_MOH_Value_Risk_Bucket`

4. **Marker 11 – Time on Site vs Average time on site**
 - **Configuration Table:** `CZ_RG.MOH_Scores_Execution_Config`
 - **Calculation Table:** `CZ_RG.Player_MOH_Calculated_Metrics`

5. **Marker 13 – Theoretical win vs average theoretical win in L180 days**
 - **Configuration Table:** `CZ_RG.MOH_Scores_Execution_Config`
 - **Calculation Table:** `CZ_RG.Player_MOH_Calculated_Metrics`

6. **Marker 14 – Player loss vs average player loss in the last 180 days**
 - **Configuration Table:** `CZ_RG.MOH_Scores_Execution_Config`
 - **Calculation Table:** `CZ_RG.Player_MOH_Calculated_Metrics`

7. **Marker 17 – Game Tile Play Increase on day**
 - **Configuration Table:** `CZ_RG.MOH_Scores_Execution_Config`
 - **Calculation Table:** `CZ_RG.Player_MOH_Calculated_Metrics`

### Summary:

- **Configuration Table `CZ_RG.MOH_Scores_Execution_Config`**: This table is used for all listed markers (3, 9, 10, 11, 13, 14, 17) to fetch the query and execution order.
 - **Calculation Table `CZ_RG.Player_MOH_Calculated_Metrics`**: This table is used for calculating the MOH values for markers (3, 9, 10, 11, 13, 14, 17).
 - **Risk Bucket Table `CZ_RG.FD_Player_MOH_Value_Risk_Bucket`**: Specifically used for Marker 10 to determine the risk bucket for players based on net deposits.
