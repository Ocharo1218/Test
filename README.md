Worked for 3m 26s

I’ve reviewed your uploaded FY27 R&Q dashboard.xlsx file and recommend building a three-page Power BI report. The aim is to give management a quick overview, highlight engagement risks and control gaps, and retain the full Excel register for detailed review.

The most important design decision is to keep the report simple, formula-driven, and easy to refresh from SharePoint. You do not need lots of separate tables or complicated relationships to achieve this.

### 1\. What I found in your Excel file

Your workbook contains two sheets: `FY27 won jobs` and `Chart Data`.

The main sheet has 53 engagement records across 29 columns, in rows 2–54. The legend is lower down the sheet, so it must not be included as engagement data. Your screenshot also shows an active Excel filter, so the rows currently visible are not necessarily all the records.

Here are the main figures I calculated from the 53 records.

Engagements

### 53

Total engagement value

### USD 3.77M

Distinct clients

### 46

Acceptance status requiring attention

### 3

The acceptance and risk distributions are:

| Measure                            | Records |
| ---------------------------------- | ------- |
| Acceptance status: Approved        | 50      |
| Acceptance status: In-Progress     | 2       |
| Acceptance status: Action Required | 1       |
| Low risk                           | 49      |
| Medium risk                        | 3       |
| High risk                          | 1       |

There are also important control gaps in the data:

| Control indicator                                 | Current count |
| ------------------------------------------------- | ------------- |
| Planning tasks blank or marked No                 | 47            |
| LoE signed-date field blank or marked Not on file | 38            |
| Workspace blank or marked No workspace            | 21            |
| Creation date missing                             | 28            |

These are useful starting points for your control page. They reflect the data currently in the workbook and will need to update dynamically in Power BI.

I also found several data-quality issues:

* Four columns are completely blank: `Pre PAC Date`, `JBR Active (Yes/No/N/A)`, `JBR Third Party eLearn - YES/NO`, and `UAT Application approved - Date`.

* Several date-related fields contain mixtures of actual dates, text dates such as `8th April 2026`, and labels such as `N/A`, `Not on file`, `Withdrawn`, and `G/T`.

* Some cells have special colours based on compliance-related conditions. The colours are not themselves data values that Power BI can automatically analyse.

My recommendation: clean the data in Power Query, use a few dynamic DAX measures, and keep the main engagement table as the central data source.

### 2\. The report I recommend building

### Page 1 — FY27 Won Jobs Overview

Purpose: Answer how many engagements were won, their value and where attention is needed.

Four KPI cards, a monthly engagement trend, and a chart showing value by Engagement Leader.

### Page 2 — Risk & Control Monitoring

Purpose: Highlight missing information, outstanding control actions and risk.

Four control KPI cards, one risk chart, and a compact table of engagements requiring attention.

### Page 3 — Full Engagement Register

Purpose: Let users find and inspect individual records.

One table containing the original 29 fields, slicers, consistent date and currency formatting, and recreated compliance highlighting.

This is enough for a useful management report without overcrowding the pages. Page 2 is the optional page you can retain or remove depending on how much control analysis management needs.

### 3\. Step 1 — Prepare the Excel data

Do this before connecting Power BI to SharePoint.

### A. Convert the main data into an Excel Table

* Open `FY27 R&Q dashboard.xlsx` in Excel.

* Go to the `FY27 won jobs` sheet.

* Click anywhere inside the engagement data.

* Select Insert → Table, or press `Ctrl + T`.

* Set the table range to `A1:AC54`.

* Tick My table has headers and click OK.

* Click inside the new table. Go to Table Design → Table Name and enter:

    `tblFY27WonJobs`

Make sure you include only the header and the 53 engagement records. Do not include the blank rows and the colour legend at the bottom.

Excel Tables automatically expand when new records are added, making them more suitable for a refreshable Power BI report. Microsoft also documents how the Excel connector identifies workbook tables and worksheet data.

### B. Standardise the headers

I recommend renaming the following field in the source table:

`Worskpace name` → `Workspace name`

You can also standardise the extra spaces in `JBR Third Party eLearn - YES/NO`.

Keep the remaining field names unless there is a good reason to change them. Clear any unnecessary Excel filters when checking record counts, although the presence of a filter does not mean the hidden records should be excluded from your Power BI report.

### C. Decide how to handle the colours

Your red, orange and blue legend means:

| Colour | Meaning         |
| ------ | --------------- |
| Red    | Non compliance  |
| Orange | Due for renewal |
| Blue   | No information  |

Power BI normally imports the cell values, not the business meaning of their Excel fill colours. You therefore need to represent the meaning explicitly in the data model before reproducing the colours in visuals.

For example, create fields such as:

* `Workspace Control Status`

* `Planning Control Status`

* `Expiry Control Status`

You can derive workspace and planning statuses from their actual values. For the expiry status, use your organisation's agreed renewal window. For manually coloured exceptions, enter the corresponding text category in a dedicated status field rather than relying on cell fill alone.

This is particularly important because the Excel colours apply to individual control cells, not necessarily to an entire engagement. A single overall flag could hide the fact that an engagement has several separate control issues.

### 4\. Step 2 — Connect Power BI to the SharePoint workbook

Use Power BI Desktop.

### A. Connect to SharePoint

* Open Power BI Desktop.

* Select Home → Get data → More.

* Search for SharePoint folder.

* Select the connector and click Connect.

* Enter your SharePoint site URL, not the URL of the individual Excel file.

For example:

`https://yourcompany.sharepoint.com/sites/YourSiteName`

* Sign in using your organisation's account.

* Click Transform Data.

### B. Find the Excel workbook

The SharePoint Folder preview may show many files.

* Find the `Name` column.

* Filter it to `FY27 R&Q dashboard.xlsx`.

* Check `Folder Path` to confirm you selected the correct SharePoint folder.

* Use the workbook's `Content`/combine navigation to open its Excel objects.

* Select `tblFY27WonJobs` if it is available.

If you have not created the Excel Table, select the `FY27 won jobs` worksheet instead, and remove the extra blank and legend rows in Power Query.

Important: select only the main engagement data for this query. Do not load the `Chart Data` sheet as a second production table just to get your charts. That sheet contains summary values stored as fixed values rather than formulas, so they will not automatically recalculate when the underlying records change.

### 5\. Step 3 — Clean the data in Power Query

You should now be in the Power Query Editor.

Power Query is where you clean and prepare the data before building charts.

### A. Filter out empty records

If you connected to the Excel Table, the legend should already be excluded.

If you selected the worksheet:

* Open the filter on `Client Name`.

* Remove blank values.

* Confirm the legend row is gone.

* Check that the query has 53 engagement rows.

### B. Clean text fields

Select the relevant text columns and use Transform → Format → Trim. This removes unnecessary leading and trailing spaces.

You should retain text such as `N/A`, `Not on file`, and `Withdrawn` where it represents the original business status. Do not change every such value to `No`, because that would confuse missing information with a negative status.

### C. Set sensible data types

Use the following rules:

| Fields                                                   | Recommended treatment                                     |
| -------------------------------------------------------- | --------------------------------------------------------- |
| Amount                                                   | Fixed decimal number; format as USD currency in the model |
| Close Date                                               | Date                                                      |
| Client Name, Opportunity Name                            | Text                                                      |
| Engagement Leader, Engagement Manager                    | Text                                                      |
| Acceptance Summary Status, Risk rating                   | Text                                                      |
| Independence Restriction Status, AFS                     | Text                                                      |
| Workspace name, Planning tasks approved                  | Text                                                      |
| LoE, approval, expiry and other mixed date/status fields | Preserve the original values as text initially            |

The last row matters: your file contains text and numbers mixed within some date-related columns. Converting all these columns directly to Date could generate errors or lose labels that need to be investigated.

For the `Close Date` column, select the column and choose Transform → Data Type → Date.

For other date columns that you later want to analyse, create separate clean date fields. If you use Change Type → Using Locale, select an appropriate locale such as English (United Kingdom) for the written dates. Microsoft's Power Query guidance explains why locale affects how text is interpreted as a date.

### D. Create a status using a formula

For example, to distinguish missing workspace information from a recorded workspace, select Add Column → Custom Column.

Name the column `Workspace Control Status` and use this Power Query formula:

```powerquery
if [Workspace name] = null
    or Text.Trim([Workspace name]) = ""
then "No information"
else if Text.Upper(Text.Trim([Workspace name])) = "NO WORKSPACE"
then "Non compliance"
else "Recorded"
```

Similarly, create a `Planning Control Status` column:

```powerquery
if [Planning tasks approved] = null
    or Text.Trim([Planning tasks approved]) = ""
then "No information"
else if Text.Upper(Text.Trim([Planning tasks approved])) = "NO"
then "Non compliance"
else "Compliant"
```

These formulas create text categories that Power BI can filter, count and colour. They are also preferable to trying to extract meaning from cell formatting.

For expiry-related colours, do not invent a renewal threshold. Agree the actual renewal window with the R&Q process owner first, or explicitly encode the existing classifications in the source data.

### E. Finish the cleaning

* Rename the query to `FY27 won jobs` in the Query Settings pane.

* Check that `Amount` is numeric and `Close Date` is a date.

* Keep the original status fields so the full register remains auditable.

* Select Home → Close & Apply.

### 6\. Step 4 — Create the basic measures

Measures are calculations that respond to slicers and filters. This is exactly what you need for KPI cards and charts. Microsoft explains how measures work in Power BI and how they are used in reports.

Go to Report view or Data view, select the `FY27 won jobs` table, and choose Modeling → New measure.

Create the following measures one at a time.

### Measure 1 — Engagement count

```dax
Engagement Count =
COUNTROWS('FY27 won jobs')
```

Expected current result: 53.

### Measure 2 — Total value

```dax
Total Value =
SUM('FY27 won jobs'[Amount])
```

Expected current result: approximately USD 3.77 million.

Select this measure and set its format to Currency. Use USD if that is the agreed currency for your source data.

### Measure 3 — Unique clients

```dax
Unique Clients =
DISTINCTCOUNT('FY27 won jobs'[Client Name])
```

Expected current result: 46.

### Measure 4 — Approved engagements

```dax
Approved Engagements =
CALCULATE(
    [Engagement Count],
    'FY27 won jobs'[Acceptance Summary Status] = "Approved"
)
```

Expected current result: 50.

### Measure 5 — Acceptance requiring attention

```dax
Acceptance Attention =
[Engagement Count] - [Approved Engagements]
```

Expected current result: 3.

These five measures will cover all the overview KPI cards.

### Create four control measures for Page 2

Planning unresolved

```dax
Planning Unresolved =
CALCULATE(
    [Engagement Count],
    FILTER(
        'FY27 won jobs',
        ISBLANK('FY27 won jobs'[Planning tasks approved])
            || 'FY27 won jobs'[Planning tasks approved] = "No"
    )
)
```

LoE evidence gap

```dax
LoE Evidence Gap =
CALCULATE(
    [Engagement Count],
    FILTER(
        'FY27 won jobs',
        ISBLANK('FY27 won jobs'[LoE (client signed date)])
            || 'FY27 won jobs'[LoE (client signed date)] = ""
            || 'FY27 won jobs'[LoE (client signed date)] = "Not on file"
    )
)
```

Keep that original LoE field as text for this measure, since it contains both dates and labels.

Workspace absent

```dax
Workspace Absent =
CALCULATE(
    [Engagement Count],
    FILTER(
        'FY27 won jobs',
        ISBLANK('FY27 won jobs'[Workspace name])
            || 'FY27 won jobs'[Workspace name] = ""
            || 'FY27 won jobs'[Workspace name] = "No workspace"
    )
)
```

Creation date missing

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

Expected results from the supplied data are 47, 38, 21 and 28, respectively. Validate these when you finish Power Query cleaning. If you later convert the original fields to dates, preserve the original text or create separate clean date columns so these checks still work.

### 7\. Step 5 — Build Page 1: FY27 Won Jobs Overview

Create a report page and rename it `Overview`.

### A. Add four KPI cards

In the Visualizations pane, choose the Card visual. Create four cards using:

| Card title           | Measure              |
| -------------------- | -------------------- |
| Total engagements    | Engagement Count     |
| Total value          | Total Value          |
| Unique clients       | Unique Clients       |
| Acceptance attention | Acceptance Attention |

Arrange the cards in a single row near the top of the page.

### B. Add the monthly engagement chart

* Select an empty area of the page.

* Choose a Clustered column chart.

* Drag `Close Date` to the X-axis.

* Select the date hierarchy and use the Month level, with Year retained if necessary.

* Drag `Engagement Count` to the Y-axis.

* Title the chart `Engagements by Close Month`.

This will show the number of engagements closed in each month. The current records cover July to October 2026, with August having the highest count.

### C. Add the value-by-leader chart

* Add a Clustered bar chart.

* Drag `Engagement Leader` to the Y-axis.

* Drag `Total Value` to the X-axis.

* Sort the chart by `Total Value` in descending order.

* In the visual-level filters, apply a Top N filter to `Engagement Leader`, set it to Top 5, and use `Total Value` as the value.

Title it `Top 5 Engagement Leaders by Value`.

This focuses attention on the largest engagement values rather than trying to show all 16 leaders on one crowded chart.

### D. Add three slicers

Add these slicers in a compact row beneath the title or beside the charts:

* `Close Date`

* `Engagement Leader`

* `Acceptance Summary Status`

For `Close Date`, use the Between slicer if you want users to select a date range.

Keep this page to four cards and two charts. The risk distribution and detailed compliance checks are more useful on Page 2.

### 8\. Step 6 — Build Page 2: Risk & Control Monitoring

Create a new page and rename it `Risk & Controls`.

### A. Add four control KPI cards

Use the four measures you created:

| Card                  | Measure               | Current expected result |
| --------------------- | --------------------- | ----------------------- |
| Planning unresolved   | Planning Unresolved   | 47                      |
| LoE evidence gap      | LoE Evidence Gap      | 38                      |
| Workspace absent      | Workspace Absent      | 21                      |
| Creation date missing | Creation Date Missing | 28                      |

Label these cards carefully. For example, a blank planning field means not recorded, not necessarily that the engagement has failed compliance.

### B. Add the risk chart

* Add a Clustered bar chart.

* Put `Risk rating` on the Y-axis.

* Put `Engagement Count` on the X-axis.

* Sort by the count, descending.

* Title the chart `Engagements by Risk Rating`.

It should show 49 Low, 3 Medium and 1 High for the current dataset.

### C. Add an exception-review table

Add a compact Table visual containing only these fields:

* Client Name

* Opportunity Name

* Engagement Leader

* Acceptance Summary Status

* Risk rating

* Workspace name

* Planning tasks approved

* LoE (client signed date)

This gives users enough context to investigate the controls without putting all 29 columns on the overview page.

Use the relevant status fields and visual filters to focus the table on missing information or exceptions. As you build out further control flags, you can filter for `Non compliance` and `No information` explicitly.

### D. Recreate the red, orange and blue formatting

After adding the relevant status fields, you can colour the control columns in Power BI:

* Select the table visual.

* Open the dropdown beside the relevant status field in the visual's field list.

* Choose Conditional formatting → Background color.

* Use Rules or Field value, depending on how you represent the colours.

* Map `Non compliance` to red, `Due for renewal` to orange, and `No information` to blue.

Repeat this for the relevant control columns.

This reproduces the legend as a report feature based on actual values instead of relying on Excel's background colours.

### 9\. Step 7 — Build Page 3: Full Engagement Register

Create another page named `Engagement Register`.

Your goal here is to reproduce the Excel register as closely as is practical, while making it easier to search and filter.

### A. Create the table

* Add a Table visual.

* Add all 29 original fields in the same order as the Excel sheet.

* Keep the columns in the order shown in the source workbook, from `Client Name` through `RSD`.

* Apply currency formatting to `Amount` and date formatting to `Close Date`.

* Resize the columns so that key identifiers are easy to read.

Because there are 29 fields, the table will require horizontal scrolling. This is preferable to making each column so narrow that nobody can read it.

Keep the page mostly dedicated to the table. Put three compact slicers above it:

* Engagement Leader

* Acceptance Summary Status

* Risk rating

Use the same control-status colours you established on Page 2 where appropriate.

### B. Synchronise common slicers

To keep filtering consistent across pages:

* Select a slicer on the Overview page.

* Go to View → Sync slicers.

* In the Sync slicers pane, enable syncing to the other relevant report pages.

* Repeat for the slicers you want to share.

This way, a user selecting a particular engagement leader on the Overview page can continue to work with the same selection on the other pages.

### 10\. Step 8 — Make future refreshes and additional sheets easier

This is especially important because you mentioned that more worksheets will be added to the Excel file later.

There are two different situations.

Situation A: More rows are added to `FY27 won jobs`.

Since the data is stored in `tblFY27WonJobs`, new engagement rows will be included when the query refreshes. The existing measures, charts and slicers will continue to work without rebuilding the page.

Situation B: A new worksheet is added to the workbook.

Power BI will not automatically create a new data model table and report page for an arbitrary new worksheet. You will need to add a query for that worksheet and decide how it should be visualised.

Use this repeatable process for each future worksheet:

* In Excel, turn the new sheet's data range into its own Excel Table with a unique name, proper column headers and no legend or notes inside the data.

* In Power BI Desktop, open Transform Data and create a query for that new table from the existing workbook connection.

* Give the new query a meaningful name.

* Clean the new table's data types and fields.

* If the new sheet is standalone, keep it as its own table. Do not create relationships simply because it lives in the same workbook.

* Duplicate an existing report page as a starting point, replace the old visual fields with fields from the new table, and rename the page.

* If two future sheets have the same column structure and represent the same kind of information, consider appending them into one table instead of creating separate tables.

The goal is to have one reusable method for loading new worksheet tables, while keeping the model simple. You do not need to load every worksheet into Power BI immediately.

### 11\. Step 9 — Check the report before publishing

Before you publish, compare Power BI against Excel.

Use the following checklist.

Final validation

0 of 12

Engagement Count shows 53

Total Value shows approximately USD 3.77 million

Unique Clients shows 46

Approved engagements shows 50

Acceptance Attention shows 3

Planning Unresolved shows 47

LoE Evidence Gap shows 38

Workspace Absent shows 21

Creation Date Missing shows 28

The legend and blank rows are not included as engagements

The original date/status text is not lost during cleaning

Red, orange and blue indicators are represented by data fields

Once this works, select Home → Publish and publish the report to your Power BI workspace. Configure the semantic model's data-source credentials and test a refresh so the SharePoint source continues to update.

### My final recommendation

Start with Page 1 and Page 3 first. They will give you a working executive overview and a usable engagement register. Then build Page 2 using the four control measures and risk chart.

One final caution: the current Excel workbook contains a mix of true dates and textual status values in several date-related columns. Preserve those original fields, and only create clean date columns where you need calculations such as expiry or renewal analysis. That will make the report more reliable and reduce the risk of hiding real exceptions during data cleaning.
