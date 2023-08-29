# Data Analytics Project: Detailed Explanation

## Data Manipulation Tasks

### Task 1: Filtering and Sorting Data

#### Sub-Task: Fetching and Filtering Data

1. **Creation of Sub-Sheet:** A new sub-sheet was created to manage the data.

2. **Query Function for Data Fetching:** The `QUERY` function was employed to extract data from the "Data Set" sheet.

3. **Conditions for Filtering:** The data was filtered to include only rows where PE Ratio is not blank and PE Ratio is less than 100.

4. **Sorting:** The filtered data was then sorted in descending order of Market Cap.

**Formula Used:**
```excel
=QUERY('Data Set'!A:J, "SELECT * WHERE D IS NOT NULL AND D < 100 ORDER BY B DESC", 1)
```

### Task 2: Adding Industry and NSE/BSE Code

#### Sub-Task: Fetching Additional Data with `VLOOKUP`

1. **Additional Columns:** New columns were added for Industry and NSE/BSE Code.

2. **Array Formula with `VLOOKUP`:** The `ARRAYFORMULA` along with `VLOOKUP` was utilized to retrieve data from the second sheet.

3. **Conditional Application:** The formula was configured to apply only where data is present.

**Formula Used for NSE/BSE Code:**
```excel
=ARRAYFORMULA(IF(A2:A <> "", IFERROR(VLOOKUP(A2:A, 'Sheet2'!A:B, 2, FALSE)), ""))
```

### Task 3: Bear Mode Conditions

#### Sub-Task: Applying Conditional Logic

1. **New Columns:** "Bear Mode 1" and "Bear Mode 2" columns were introduced.

2. **Dynamically Applied IF Conditions:** `ARRAYFORMULA` was combined with `IF` conditions to evaluate if % Change from 52 Week High and Low meet specific criteria.

**Formula Used for Bear Mode 1:**
```excel
=ARRAYFORMULA(IF(F2:F > 30, "Yes", "No"))
```

**Formula Used for Bear Mode 2:**
```excel
=ARRAYFORMULA(IF(H2:H < 30, "Yes", "No"))
```

### Task 4: Stock Status Classification

#### Sub-Task: Stock Status Classification based on Conditions

1. **Column Creation:** A "Stock Status" column was generated.

2. **Dynamic IF Conditions:** `ARRAYFORMULA` with `IF` conditions were employed to assess whether PE Ratio, Bear Mode 1, and Bear Mode 2 satisfy specific criteria.

**Formula Used:**
```excel
=ARRAYFORMULA(IF(AND(D2:D < 65, K2:K = "No", L2:L = "No"), "Good Stock", ""))
```

---

## Report Dashboarding

### Task 1: Creating Google Data Studio Dashboard

#### Sub-Task: Creating an Interactive Dashboard

1. **Opening Google Data Studio:** Google Data Studio was accessed to initiate the dashboard creation.

2. **Data Source Connection:** The "Final Organized Data" sheet was linked as the data source.

3. **Dashboard Design:** The dashboard was designed with filters, scorecards, various charts, and tables to present data insights.

### Task 2: Meaningful Analysis and Visualization

#### Sub-Task: Data Visualization for Analysis

1. **Heading Inclusion:** A title heading was incorporated into the dashboard.

2. **Filter Implementation:** Filters for Industry, Stock Status, and Cap Size were added for interactive data analysis.

3. **Visual Components:** The dashboard integrated scorecards, bar charts, pie charts, line charts, and tables for diverse data representation.

4. **Customization:** Layout and appearance were customized to enhance the visual appeal and effectiveness of data communication.

---

## Google Apps Script Tasks

### Task 1: Copying Data with Google Apps Script

#### Sub-Task: Data Copying Automation

1. **Sheet Duplication:** A duplicate sheet was created to facilitate manipulation.

2. **Access to Script Editor:** The Google Apps Script editor was accessed within the copied Google Sheet.

3. **Script Crafting:** A script was authored to copy data from the source to a target sheet.

4. **Successful Execution:** The script was executed successfully, with necessary permissions granted for its operation.

**Script Used:**
```javascript
function copyDataToAnotherSheet() {
  var sourceSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("SourceSheetName");
  var targetSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("TargetSheetName");

  var sourceData = sourceSheet.getDataRange().getValues();

  // Exclude the header row
  sourceData.shift();

  targetSheet.getRange(targetSheet.getLastRow() + 1, 1, sourceData.length, sourceData[0].length).setValues(sourceData);
}


```
