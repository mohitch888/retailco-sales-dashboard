# Data Cleaning

Raw rows: **12,575** | Clean rows: **11,971** | Retained: **95.2%**

## Rules applied (in order)

1. **Missing Total Spent (604 rows): exclude.**
   These rows also lack Quantity, so the amount cannot be rebuilt.
2. **Missing Price Per Unit (609 rows): recover.**
   `Price Per Unit = Total Spent / Quantity`. Validation: Price x Quantity = Total Spent on every complete row.
3. **Missing Item (1,213 rows): recover where possible.**
   Look up the item from Category + Price Per Unit. Validation: all 200 Category + Price pairs are unique, so no ambiguity.
   609 were recovered; the other 604 were already excluded in rule 1.
4. **Missing Discount Applied (4,199 rows): label "Unknown".**
   The flag cannot be inferred. 3,988 remain in clean data as "Unknown"; 211 were in excluded rows.
5. **Types:** Transaction Date set to Date, Discount Applied to Yes / No / Unknown text for reporting.

## Audit table

| Issue | Raw rows affected | Clean rows affected | Action |
|---|---|---|---|
| Missing Total Spent | 604 | 0 | Rows excluded |
| Missing Price Per Unit | 609 | 0 | Recovered |
| Missing Item | 1,213 | 0 | 609 recovered; 604 in excluded rows |
| Missing Discount Flag | 4,199 | 3,988 | Labelled Unknown |
| Total rows | 12,575 | 11,971 | 604 excluded |

The same audit is stored in the `DQ_Audit` table of the Power BI model.

## Power Query steps

<!-- Paste the M code from Transform data > Advanced Editor here -->

# Raw Sales
```m
let
    Source = Csv.Document(File.Contents("C:\Users\Mohit Chaudhary\OneDrive\Desktop\Github Projects\POwerbi project\archive (3)\retail_store_sales.csv"),[Delimiter=",", Columns=11, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Transaction ID", type text}, {"Customer ID", type text}, {"Category", type text}, {"Item", type text}, {"Price Per Unit", type number}, {"Quantity", Int64.Type}, {"Total Spent", type number}, {"Payment Method", type text}, {"Location", type text}, {"Transaction Date", type date}, {"Discount Applied", type logical}}),
    #"Replaced Value" = Table.ReplaceValue(#"Changed Type","Electric household essentials","Electric essentials",Replacer.ReplaceText,{"Category"})
in
    #"Replaced Value"
```
# Stg_Sales
```m
let
    Source = Raw_Sales,
    #"Set Data Type" = Table.TransformColumnTypes(Source,{{"Discount Applied", type text}}),
    #"Trim Text Columns" = Table.TransformColumns(#"Set Data Type",{{"Transaction ID", Text.Trim, type text}, {"Customer ID", Text.Trim, type text}, {"Category", Text.Trim, type text}, {"Item", Text.Trim, type text}, {"Payment Method", Text.Trim, type text}, {"Location", Text.Trim, type text}, {"Discount Applied", Text.Trim, type text}}),
    #"Cleaned Text Columns" = Table.TransformColumns(#"Trim Text Columns",{{"Transaction ID", Text.Clean, type text}, {"Customer ID", Text.Clean, type text}, {"Category", Text.Clean, type text}, {"Item", Text.Clean, type text}, {"Payment Method", Text.Clean, type text}, {"Location", Text.Clean, type text}, {"Discount Applied", Text.Clean, type text}}),
    #"Blank Item to null" = Table.ReplaceValue(#"Cleaned Text Columns","",null,Replacer.ReplaceValue,{"Item"}),
    #"Inserted Division" = Table.AddColumn(#"Blank Item to null", "Division", each [Total Spent] / [Quantity], type number),
    #"Added Custom column" = Table.AddColumn(#"Inserted Division", "Custom", each if [Price Per Unit] = null then [Division] else [Price Per Unit]),
    #"Remove Original Price per Unit and Division" = Table.RemoveColumns(#"Added Custom column",{"Price Per Unit", "Division"}),
    #"Rename custom column to Price Per Unit" = Table.RenameColumns(#"Remove Original Price per Unit and Division",{{"Custom", "Price Per Unit"}}),
    #"Change type of Price per unit to Decimal" = Table.TransformColumnTypes(#"Rename custom column to Price Per Unit",{{"Price Per Unit", type number}}),
    #"Reordered Columns" = Table.ReorderColumns(#"Change type of Price per unit to Decimal",{"Transaction ID", "Customer ID", "Category", "Item", "Price Per Unit", "Quantity", "Total Spent", "Payment Method", "Location", "Transaction Date", "Discount Applied"}),
    #"Replaced Value" = Table.ReplaceValue(#"Reordered Columns","true","Yes",Replacer.ReplaceText,{"Discount Applied"}),
    #"Replaced Value1" = Table.ReplaceValue(#"Replaced Value","false","No",Replacer.ReplaceText,{"Discount Applied"})
in
    #"Replaced Value1"
```
# Sales
```m
let
    Source = Stg_Sales,
    #"Merged Queries" = Table.NestedJoin(Source, {"Category", "Price Per Unit"}, Item_Lookup, {"Category", "Price Per Unit"}, "Item_Lookup", JoinKind.LeftOuter),
    #"Expanded Item_Lookup" = Table.ExpandTableColumn(#"Merged Queries", "Item_Lookup", {"Item Found"}, {"Item_Lookup.Item Found"}),
    #"Added Conditional Column" = Table.AddColumn(#"Expanded Item_Lookup", "Item Fixed", each if [Item] = null then [Item_Lookup.Item Found] else [Item]),
    #"Removed Columns" = Table.RemoveColumns(#"Added Conditional Column",{"Item", "Item_Lookup.Item Found"}),
    #"Renamed Columns" = Table.RenameColumns(#"Removed Columns",{{"Item Fixed", "Item"}}),
    #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns",{{"Item", type text}}),
    #"Reordered Columns" = Table.ReorderColumns(#"Changed Type",{"Transaction ID", "Customer ID", "Category", "Item", "Price Per Unit", "Quantity", "Total Spent", "Payment Method", "Location", "Transaction Date", "Discount Applied"}),
    #"Remove Unfrecoverable Rows" = Table.SelectRows(#"Reordered Columns", each ([Quantity] <> null)),
    #"Replaced Value" = Table.ReplaceValue(#"Remove Unfrecoverable Rows",null,"Unknown",Replacer.ReplaceValue,{"Discount Applied"}),
    #"Inserted Year" = Table.AddColumn(#"Replaced Value", "Year", each Date.Year([Transaction Date]), Int64.Type),
    #"Inserted Month" = Table.AddColumn(#"Inserted Year", "Month", each Date.Month([Transaction Date]), Int64.Type),
    #"Inserted Month Name" = Table.AddColumn(#"Inserted Month", "Month Name", each Date.MonthName([Transaction Date]), type text),
    #"Inserted Start of Month" = Table.AddColumn(#"Inserted Month Name", "Start of Month", each Date.StartOfMonth([Transaction Date]), type date),
    #"Inserted Quarter" = Table.AddColumn(#"Inserted Start of Month", "Quarter", each Date.QuarterOfYear([Transaction Date]), Int64.Type),
    #"Inserted Day Name" = Table.AddColumn(#"Inserted Quarter", "Day Name", each Date.DayOfWeekName([Transaction Date]), type text),
    #"Added Conditional Column1" = Table.AddColumn(#"Inserted Day Name", "Data Period", each if [Year] = 2025 then "Partial (to 18 Jan 2025)" else "Full Year"),
    #"Changed Type1" = Table.TransformColumnTypes(#"Added Conditional Column1",{{"Data Period", type text}}),
    #"Renamed Columns1" = Table.RenameColumns(#"Changed Type1",{{"Location", "Channel"}}),
    #"Replaced Value1" = Table.ReplaceValue(#"Renamed Columns1","Electric household essentials","Electric essentials",Replacer.ReplaceText,{"Category"})
in
    #"Replaced Value1"
```
# item_Lookup
```m
let
    Source = Stg_Sales,
    #"Removed Other Columns" = Table.SelectColumns(Source,{"Category", "Price Per Unit", "Item"}),
    #"Filtered Rows" = Table.SelectRows(#"Removed Other Columns", each ([Item] <> null)),
    #"Removed Duplicates" = Table.Distinct(#"Filtered Rows"),
    #"Renamed Columns" = Table.RenameColumns(#"Removed Duplicates",{{"Item", "Item Found"}})
in
    #"Renamed Columns"
```
# Excluded_Rows
```m
let
    Source = Sales,
    #"Filtered Rows" = Table.SelectRows(Source, each ([Quantity] = null))
in
    #"Filtered Rows"
```
# DQ_Audit
```m
let
    Source = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("ZZFBawIxEIX/yhAotFRas2p18SSWgoeCtXpSD2N2lLDpRJJZrf++WRdLraeBycz33pssl2oSY0WqpWZ4hJk/Rhhtt2SEitQbO0K+6Y6MWM8wx5JYrVtL9W5jtLyDidBXeteZ7qTSrqFk/IECFfDSzuFgEcYotPPhBI8wDdYQOO/Laj9MA12IKGAZ6Nu4qkhLISlfKTQrUwqwYCtJIGFvpDDC3As6+NwTy/NHhSxWTlegPwNnSvdCqbNe9O8XHBoqbhw9XAFebTS+YoE3h7u02NX5xUknHwzA4YZcMrNSCy7ZH3ml6mjmfNECBYeQaf0vLTVpG3O1lfM1e/1eXXXe143X35UW5L2n7A4CCVpOn7Ne/wA=", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [Column1 = _t, Column2 = _t, Column3 = _t, Column4 = _t]),
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Column1", type text}, {"Column2", type text}, {"Column3", type text}, {"Column4", type text}}),
    #"Promoted Headers" = Table.PromoteHeaders(#"Changed Type", [PromoteAllScalars=true]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers",{{"Issue", type text}, {"Raw Rows Affected", Int64.Type}, {"Clean Rows Affected", Int64.Type}, {"Action Taken", type text}})
in
    #"Changed Type1"
```