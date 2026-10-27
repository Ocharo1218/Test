Worked for 33s

Since you only have read permission on the SharePoint Excel file, we will use a slightly different approach. You do not need to edit the workbook, add Excel Tables, or change its formatting. All cleaning, formulas, calculated columns, KPI measures, charts and report formatting will be done inside Power BI Desktop and saved in your own Power BI report file.

Your original Excel workbook will remain unchanged.

### Step 1 — Connect to the SharePoint Excel file

### 1.1 Open Power BI Desktop

* Open Power BI Desktop.

* Select Home → Get data → More.

* Search for `SharePoint folder`.

* Select SharePoint folder, then click Connect.

* Enter your SharePoint site URL, not the direct link to the Excel file.

For example:

`https://yourcompany.sharepoint.com/sites/YourSiteName`

* Click OK.

* When prompted, select Organizational account and sign in with your work account.

* Select Transform Data.

Read-only access is sufficient to read data from a SharePoint source, provided your account has permission to access the relevant site and file.

Microsoft's [SharePoint and Excel connectivity guidance](https://learn.microsoft.com/en-us/power-query/connectors/excel?utm_source=chatgpt.com) explains the supported ways to connect to Excel workbooks.

### 1.2 Select the correct workbook

You should now see a list of files from SharePoint.

* Find the `Name` column.

* Open its filter.

* Select only `FY27 R&Q dashboard.xlsx`.

* Check the `Folder Path` column to confirm that you have the correct file.

* In the `Content` column, open the file's Binary value.

Do not select Combine Files unless you specifically intend to combine multiple workbooks. You need one workbook for this report.

Power Query should display the contents of the workbook, including its worksheets.

* Find the row named `FY27 won jobs`.

* Click the Table link in that row's `Data` column.

You should now be looking at the contents of your engagement worksheet.

### Step 2 — Clean the worksheet without editing Excel

Everything in this section takes place inside the Power Query Editor.

### 2.1 Use the first row as headers

* Select Home → Use First Row as Headers if the column headings have not already been promoted.

* Confirm that your columns include Client Name, Opportunity Name, Engagement Leader, Close Date, Amount, Acceptance Summary Status and Risk rating.

You do not need to edit the original column headings in Excel.

### 2.2 Remove the legend and blank rows

Your worksheet contains the red, orange and blue legend underneath the engagement data. We must exclude that from the report.

* Open the filter on `Client Name`.

* Remove `(null)` or blank values.

* Remove `Legend`.

* Check that the remaining records are actual engagements.

The initial data contains 53 engagement records. The final row count in Power Query should be 53 before any other deliberate filtering.

This approach also avoids counting your compliance legend as a client.

### 2.3 Rename the query and one column

In the Query Settings pane, rename the query to:

`FY27 won jobs`

If your imported column name is still misspelled as `Worskpace name`, right-click it and select Rename. Enter:

`Workspace name`

This change applies only to the Power BI query, not the SharePoint workbook.

### 2.4 Clean the text fields

Select the text columns that commonly contain names or statuses. You can Ctrl-click several columns to select them together.

Then use:

Transform → Format → Trim

This removes unnecessary spaces around text, helping prevent values such as `Approved` and `Approved` from being treated differently.

Retain labels such as `N/A`, `Not on file` and `Withdrawn` where they carry meaning. Do not automatically replace every one with `No`, because they may represent different situations.

### 2.5 Set the data types carefully

Use the following settings as your starting point.

| Column                          | Data type                              |
| ------------------------------- | -------------------------------------- |
| Client Name                     | Text                                   |
| Opportunity Name                | Text                                   |
| Engagement Leader               | Text                                   |
| Engagement Manager              | Text                                   |
| Close Date                      | Date                                   |
| Amount                          | Decimal number or Fixed decimal number |
| Acceptance Summary Status       | Text                                   |
| Risk rating                     | Text                                   |
| Independence Restriction Status | Text                                   |
| AFS                             | Text                                   |
| Workspace name                  | Text                                   |
| Mixed date/status fields        | Text initially                         |

The mixed fields include items such as `LoE (client signed date)`, `CA Approval Expiry`, `EA Approval`, `JBR - Approval date` and `Creation date`.

Some of these columns contain dates, while others contain text labels or blanks. Setting them all directly to Date could produce errors.

For `Close Date`, select the column and choose Transform → Data Type → Date.

For `Amount`, select Decimal Number. If the values import as text because they include currency symbols or commas within the stored value, clean those characters first and then convert the column to a number.

You can leave other mixed date/status columns as text until you decide which ones need genuine date calculations.

### 2.6 Apply the changes

Once you have cleaned the records, renamed the query and set the data types, click:

Home → Close & Apply

Power BI will load the cleaned data into your report.

At this point, you have not changed the original Excel file.

### Step 3 — Create your KPI measures using DAX

Next, we will create calculations inside Power BI. These measures will update when users apply slicers.

Select the `FY27 won jobs` table in the Data pane. Go to Modeling → New measure and enter each formula below separately.

### 3.1 Total engagements

```dax
Engagement Count =
COUNTROWS('FY27 won jobs')
```

### 3.2 Total engagement value

```dax
Total Value =
SUM('FY27 won jobs'[Amount])
```

Select this measure and set its format to Currency, using USD if that is the currency represented by your workbook.

### 3.3 Unique clients

```dax
Unique Clients =
DISTINCTCOUNT('FY27 won jobs'[Client Name])
```

### 3.4 Approved engagements

```dax
Approved Engagements =
CALCULATE(
    [Engagement Count],
    'FY27 won jobs'[Acceptance Summary Status] = "Approved"
)
```

### 3.5 Acceptance requiring attention

```dax
Acceptance Attention =
[Engagement Count] - [Approved Engagements]
```

For the workbook you supplied, the expected initial results are:

| Measure              | Expected value                 |
| -------------------- | ------------------------------ |
| Engagement Count     | 53                             |
| Total Value          | Approximately USD 3.77 million |
| Unique Clients       | 46                             |
| Approved Engagements | 50                             |
| Acceptance Attention | 3                              |

These are checks against your starting workbook. If a figure differs, verify that the legend was excluded, the Amount column is numeric and no unintended filters remain.

### Step 4 — Build Page 1: Overview

Rename the first report page `Overview`.

This page should answer four questions: How many engagements have we won? What is their total value? How many clients are represented? Which acceptance records need attention?

### 4.1 Add four KPI cards

Choose the Card visual from the Visualizations pane. Add these measures, one card at a time.

Total engagements

### 53

Total value

### USD 3.77M

Unique clients

### 46

Acceptance attention

### 3

Illustrative layout using the starting workbook figures. The real cards will use your Power BI measures.

Arrange the cards horizontally near the top of the page.

### 4.2 Add engagements by month

* Select Clustered column chart.

* Drag `Close Date` to the X-axis.

* Use the date hierarchy, selecting the Month level and retaining Year where appropriate.

* Drag `Engagement Count` to the Y-axis.

* Rename the visual `Engagements by Close Month`.

This will show how the number of engagements changes across close months.

### 4.3 Add value by Engagement Leader

* Add a Clustered bar chart.

* Drag `Engagement Leader` to the Y-axis.

* Drag `Total Value` to the X-axis.

* Sort by `Total Value`, descending.

* In the visual-level filters, set `Engagement Leader` to Top N, enter 5 and select `Total Value` as the value used for ranking.

Rename it `Top 5 Engagement Leaders by Value`.

### 4.4 Add slicers

Add three slicers:

* `Close Date`

* `Engagement Leader`

* `Acceptance Summary Status`

Arrange them along the side or underneath the report title. Use a date-range slicer for Close Date if you want users to select a period.

Keep the Overview page to four cards, two charts and these three slicers.

### Step 5 — Build Page 2: Risk & Controls

Create a new page using the plus sign at the bottom and rename it `Risk & Controls`.

This page will concentrate on risk and missing information.

### 5.1 Create four additional measures

Choose Modeling → New measure for each formula.

### Planning unresolved

```dax
Planning Unresolved =
CALCULATE(
    [Engagement Count],
    FILTER(
        'FY27 won jobs',
        ISBLANK('FY27 won jobs'[Planning tasks approved])
            || TRIM('FY27 won jobs'[Planning tasks approved]) = ""
            || UPPER(
                TRIM('FY27 won jobs'[Planning tasks approved])
            ) = "NO"
    )
)
```

### LoE evidence gap

```dax
LoE Evidence Gap =
CALCULATE(
    [Engagement Count],
    FILTER(
        'FY27 won jobs',
        ISBLANK('FY27 won jobs'[LoE (client signed date)])
            || TRIM('FY27 won jobs'[LoE (client signed date)]) = ""
            || UPPER(
                TRIM('FY27 won jobs'[LoE (client signed date)])
            ) = "NOT ON FILE"
    )
)
```

### Workspace absent

```dax
Workspace Absent =
CALCULATE(
    [Engagement Count],
    FILTER(
        'FY27 won jobs',
        ISBLANK('FY27 won jobs'[Workspace name])
            || TRIM('FY27 won jobs'[Workspace name]) = ""
            || UPPER(
                TRIM('FY27 won jobs'[Workspace name])
            ) = "NO WORKSPACE"
    )
)
```

### Creation date missing

```dax
Creation Date Missing =
CALCULATE(
    [Engagement Count],
    FILTER(
        'FY27 won jobs',
        ISBLANK('FY27 won jobs'[Creation date])
    )
)
```

These formulas assume the relevant source fields have been set to Text in Power Query. The creation-date measure above counts true blanks; if the field also uses text such as `N/A`, you can expand the definition to include those labels as well.

The initial workbook review indicated approximately 47 planning records unresolved, 38 LoE evidence gaps, 21 missing workspaces and 28 missing creation dates. Check the measures against the imported values, because the final results depend on how the original labels are stored.

### 5.2 Add the control KPI cards

Add the four measures as separate Card visuals across the top.

Label them clearly. For example, `Workspace Absent` is more informative than simply labelling the card `Workspace`.

Remember that a blank field can indicate missing information rather than a confirmed compliance failure. Keep these concepts distinct.

### 5.3 Add risk distribution

* Add a Clustered bar chart.

* Drag `Risk rating` to the Y-axis.

* Drag `Engagement Count` to the X-axis.

* Sort by count in descending order.

* Title the chart `Engagements by Risk Rating`.

The expected initial split is 49 Low, 3 Medium and 1 High.

### 5.4 Add the exception table

Create a compact Table visual with:

* Client Name

* Opportunity Name

* Engagement Leader

* Acceptance Summary Status

* Risk rating

* Workspace name

* Planning tasks approved

* LoE (client signed date)

Use the filters pane to focus this visual on records requiring attention. This table will give management enough information to identify the engagement without displaying all 29 fields on this page.

### Step 6 — Reproduce the red, orange and blue compliance highlighting

This is the main limitation to account for because you cannot edit the source workbook.

Power BI imports the underlying worksheet values, but the standard Excel/Power Query connection does not automatically convert cell background colours into data fields.

Your existing legend means:

| Colour | Meaning         |
| ------ | --------------- |
| Red    | Non compliance  |
| Orange | Due for renewal |
| Blue   | No information  |

Power BI cannot automatically determine that a particular cell was red, orange or blue if that information exists only in its Excel fill colour.

You can still reproduce the colouring without modifying the SharePoint workbook, using one of two approaches.

### Option A — Derive the status from the cell value

This is the best approach whenever the text already tells you the status.

For example, select `Workspace name`, then use Add Column → Custom Column in Power Query.

Name the new column `Workspace Control Status` and enter:

```powerquery
if [Workspace name] = null then
    "No information"
else if Text.Trim([Workspace name]) = "" then
    "No information"
else if Text.Upper(Text.Trim([Workspace name])) = "NO WORKSPACE" then
    "Non compliance"
else
    "Recorded"
```

You can do the same for `Planning tasks approved`, mapping blanks to `No information` and a recorded `No` to `Non compliance`.

For expiry dates, create an appropriate renewal flag only after confirming the organisation's actual renewal window. For example, do not assume that renewal is due within 30 days unless that is the agreed rule.

### Option B — Store the existing colour classifications inside Power BI

Where the background colour is the only indication of status, you will need to maintain that information separately.

You can use Home → Enter data to create a small mapping table in Power BI, containing fields such as:

| Engagement key               | Control field | Status          |
| ---------------------------- | ------------- | --------------- |
| Unique engagement identifier | Workspace     | Non compliance  |
| Unique engagement identifier | AFS           | Due for renewal |
| Unique engagement identifier | Planning      | No information  |

Populate this mapping using the affected records and the classifications shown in Excel. You can then connect the mapping to the main engagement data and use the statuses for filtering and conditional formatting.

This requires manual maintenance when the source classifications change or new engagements are added. If the file owner can eventually provide explicit status fields, that would be a more reliable long-term option.

### Apply the colours to a Power BI table

Once the status fields exist:

* Select the relevant Table visual.

* Open the dropdown beside the relevant field.

* Select Conditional formatting → Background color.

* Choose Rules or Field value.

* Set the mapping: Non compliance to red, Due for renewal to orange, and No information to blue.

Repeat for the other applicable control fields. See Microsoft's [conditional formatting instructions](https://learn.microsoft.com/en-us/power-bi/desktop-conditional-table-formatting?utm_source=chatgpt.com) for details.

### Step 7 — Build Page 3: Full Engagement Register

Create another report page and rename it `Engagement Register`.

* Add a Table visual.

* Add all 29 fields from `FY27 won jobs`.

* Arrange the columns in the same order as the original Excel register.

* Format `Close Date` as a date and `Amount` as currency.

* Apply conditional formatting to the relevant status fields.

* Adjust the widths so the key columns are readable.

The table will require horizontal scrolling. That is expected with 29 fields; avoid shrinking the text excessively just to fit everything at once.

Add three slicers above the table:

* Engagement Leader

* Acceptance Summary Status

* Risk rating

You can keep the page dedicated mostly to the register rather than adding charts that repeat information already shown on the other pages.

### Synchronise the common slicers

If you want selections to carry across the report:

* Select a slicer on the Overview page.

* Open View → Sync slicers.

* Enable synchronization for the other pages where the same filter should apply.

* Repeat for the other relevant slicers.

### Step 8 — Make future refreshes and additional sheets manageable

Because you cannot create Excel Tables in the source workbook, connect directly to the worksheet as described in Step 1.

There are two different scenarios.

When more rows are added to `FY27 won jobs`: refresh the Power BI data and check that the new records appear. Since you are importing a worksheet rather than an Excel Table, test this after the first few rows are added. If the connection does not pick up new rows correctly, the more robust fix would be for the workbook owner to convert the range to an Excel Table.

When an additional worksheet is added: Power BI will not automatically create a new data model table and report page for it. You will need to add the new worksheet as another Power Query query.

Use this repeatable process:

* Open Home → Transform data.

* Create another query from the same SharePoint workbook connection.

* Select the newly added worksheet.

* Promote headers, clean its values and set data types in Power Query.

* Rename the query to describe its contents.

* Select Close & Apply.

* Create a new report page and build visuals using that query's fields.

If the new worksheet is independent, keep it as a standalone table. You do not need to create a relationship to `FY27 won jobs` unless the analysis genuinely requires one.

If several future worksheets contain the same kind of data and have the same column structure, you can consider appending them into a single query instead of creating multiple separate tables.

The key point is that your Power BI report and transformations are separate from the SharePoint workbook. You can develop the model and visuals without write permission on the source.

### Step 9 — Refresh, save and publish

* Save your work as `FY27 RQ Dashboard.pbix` on your computer.

* Select Home → Refresh and check that the figures still reconcile with the Excel workbook.

* Confirm the 53 initial records are being counted correctly.

* Publish the report to your Power BI workspace when ready.

* In Power BI Service, confirm the data-source credentials are configured using your organisational account.

* Test a refresh to ensure Power BI can continue reading the SharePoint file.

Your account's read permission must remain valid for subsequent refreshes. Publishing the report does not grant other users access to the original SharePoint workbook.

### The order I recommend

Start with Step 1 through Step 4 to create the connection, clean the data and finish the Overview page. Next, build the Risk & Controls page and create the missing-information measures. Finish with the full register and compliance colours.

That gives you a usable first report without editing the source Excel file or loading unnecessary helper tables.
