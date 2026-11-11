# DAX Measures

<!-- In Power BI, click each measure in the Data pane and copy its formula from the formula bar. -->

## Sales table

| Measure | What it shows | DAX |
|---|---|---|
| Total Revenue | Sum of Total Spent | `Total Revenue = SUM(Sales[Total Spent])` |
| Avg Transaction Value | Revenue per transaction | `Avg Transaction Value = round(DIVIDE([Total Revenue],[Transactions]),2)` |
| ATV rate | Change in average transaction value vs prior year | `ATV rate = 
VAR _selected_year =
    SELECTEDVALUE(Calendar_Table[Year], 2024)

VAR _current_ATV =
    CALCULATE(
        [Avg Transaction Value],
        REMOVEFILTERS(Calendar_Table),
        Calendar_Table[Year] = _selected_year
    )

VAR _previous_ATV =
    CALCULATE(
        [Avg Transaction Value],
        REMOVEFILTERS(Calendar_Table),
        Calendar_Table[Year] = _selected_year - 1
    )

VAR _growth =
    DIVIDE(
        _current_ATV - _previous_ATV ,
        _previous_ATV,
        0
    )

VAR _format =
    FORMAT(_growth, "0.0%;-0.0%;0.0%")

RETURN
    IF(
        _growth > 0,
        "Vs PY " & UNICHAR(129029) & " " & _format,
        IF(
            _growth < 0,
            "Vs PY " & UNICHAR(129031) & " " & _format,
            "0.0%"
        )
    )
` |
| Growth Rate | Revenue change vs prior year | `Growth Rate = 
VAR _growth =
    DIVIDE(
         [Revenue (Selected Year)]-[Revenue (Previous Year)],
        [Revenue (Previous Year)],
        0
    )

VAR _format =
    FORMAT(_growth, "0.0%;-0.0%;0.0%")

RETURN
    IF(
        _growth > 0,
        "Vs PY " & UNICHAR(129029) & " " & _format,
        IF(
            _growth < 0,
            "Vs PY " & UNICHAR(129031) & " " & _format,
            "0.0%"
        )
    )
` |
| Revenue (Previous Year) | Prior-year revenue for the line chart | `Revenue (Previous Year) = 
VAR SelectedYear =
    SELECTEDVALUE(Calendar_Table[Year], 2024)
RETURN
    CALCULATE(
        [Total Revenue],
        REMOVEFILTERS(Calendar_Table[Year]),
        'Calendar_Table'[Year] = SelectedYear - 1
    )` |
| Online revenue Share% | Online revenue / total revenue | `Online revenue Share% = 
DIVIDE(CALCULATE([Total Revenue],Sales[Channel]="Online"),[Total Revenue])` |
| Discount Flagged% | Share of clean transactions flagged as discounted | `Discount Flagged% = 
DIVIDE(CALCULATE([Transactions],Sales[Discount Applied]="Yes"),[Transactions])` |
| Online Share Color | Conditional-format colour for the share card | `Online Share Color = 
VAR _selected_year =
    SELECTEDVALUE(Calendar_Table[Year], 2024)

VAR _current_online_share =
    CALCULATE(
        [Online revenue Share%],
        REMOVEFILTERS(Calendar_Table),
        Calendar_Table[Year] = _selected_year
    )

VAR _previous_online_share =
    CALCULATE(
        [Online revenue Share%],
        REMOVEFILTERS(Calendar_Table),
        Calendar_Table[Year] = _selected_year - 1
    )

VAR _growth3 =
    DIVIDE(
        _current_online_share - _previous_online_share,
        _previous_online_share
    )

RETURN
    IF(
        _growth3 > 0,
        "#00B050",
        IF(_growth3 < 0, "#FF0000", "#808080")
       ) & ""
` |
| ATV Color | Conditional-format colour for the ATV card | `ATV Color = 
VAR _selected_year =
    SELECTEDVALUE(Calendar_Table[Year], 2024)

VAR _current_ATV =
    CALCULATE(
        [Avg Transaction Value],
        REMOVEFILTERS(Calendar_Table),
        Calendar_Table[Year] = _selected_year
    )

VAR _previous_ATV =
    CALCULATE(
        [Avg Transaction Value],
        REMOVEFILTERS(Calendar_Table),
        Calendar_Table[Year] = _selected_year - 1
    )

VAR _growth2 =
    DIVIDE(_current_ATV - _previous_ATV, _previous_ATV, 0)

RETURN
    IF(
        _growth2 > 0,
        "#00B050",
        IF(_growth2 < 0, "#FF0000", "#808080")
    ) & ""` |

## Model

- `Sales`: transaction facts
- `Calendar_Table`: Date, Month Name, Month Number, Quarter, Weekday, Weekday Number, Year
- `DQ_Audit`: data quality audit table
