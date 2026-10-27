Worked for 10s

Yes, absolutely! In Power BI, you can colour only the individual cell red without colouring the entire row. You will use conditional formatting on each column separately.

We will set up these two rules on your FY27 Won Jobs Register table:

* Workspace name: colour the cell red when the value is `No workspace`.

* Planning tasks approved: colour the cell red when the value is `No`.

### Step 1 — Create the Workspace colour formula

* Go to Modeling → New measure.

* Copy and paste this DAX formula:

```dax
Workspace Cell Color =
VAR CellText =
    SELECTEDVALUE(
        'FY27 won jobs'[Workspace name],
        ""
    )
RETURN
    IF(
        LOWER(TRIM(CellText)) = "no workspace",
        "#FF0000",
        BLANK()
    )
```

3\. Press Enter.

This measure identifies cells containing `No workspace`, ignoring differences in capitalisation and extra spaces.

### Step 2 — Apply red formatting to the Workspace name cells

### 1

Select your table visual on the Engagement Register page.

### 2

In the visual's field list, find Workspace name among the columns you added to the table.

### 3

Click the dropdown arrow beside Workspace name and select Conditional formatting → Background color.

### 4

In the formatting window, set:

* Format style: Field value

* What field should we base this on?: `Workspace Cell Color`

Then click OK.

Only the background of the Workspace name cell will turn red when its value is `No workspace`. All other cells in that row remain unchanged.

### Step 3 — Create the Planning tasks approved formula

Go to Modeling → New measure again and enter:

```dax
Planning Cell Color =
VAR CellText =
    SELECTEDVALUE(
        'FY27 won jobs'[Planning tasks approved],
        ""
    )
RETURN
    IF(
        LOWER(TRIM(CellText)) = "no",
        "#FF0000",
        BLANK()
    )
```

Press Enter.

### Step 4 — Apply red formatting to Planning tasks approved

* Select the same table visual.

* Find Planning tasks approved in the table's field list.

* Click its dropdown arrow.

* Select Conditional formatting → Background color.

* Set Format style to `Field value`.

* Select `Planning Cell Color` as the field to base the formatting on.

* Click OK.

Now only cells in the Planning tasks approved column containing `No` will turn red.

### Step 5 — Check the result

Your table should behave like this:

Workspace name

No workspace

Project Atlas

Planning tasks approved

No

Yes

Illustration of the expected result. Only the cells matching the conditions are red.

Important: You do not need to add either colour measure as a visible column in your table. Power BI uses them only to determine the background colour.

If your column is still named `Worskpace name` in Power BI because you have not renamed it in Power Query, use that exact name in the first formula instead.

For more details, see [Microsoft's guide to conditional formatting in Power BI tables](https://learn.microsoft.com/en-us/power-bi/desktop-conditional-table-formatting?utm_source=chatgpt.com).
