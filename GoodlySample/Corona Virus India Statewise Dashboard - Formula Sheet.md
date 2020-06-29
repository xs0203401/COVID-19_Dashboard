# Corona Virus India Statewise Dashboard - Formula Sheet


## Measures
1.`%Active` = DIVIDE( [Active Cases TD] , [Total Cases TD] )


1.`%Deaths` = DIVIDE([Deaths TD] , [Total Cases TD])


1.`%Recovered` = DIVIDE( [Recovered TD] , [Total Cases TD] )


1.`Active Cases` = SUM(Data[Active Added])


1.`Active Cases State Rank` = 
```
RANKX(
    ALL(Data[State]),
    [Active Cases TD],,
    DESC,
    Dense
)
```


1.`Active Cases TD` = 
```
CALCULATE(
    SUM(Data[Active]),
    LASTDATE(Data[Date])
)
```


1.`Active Cases World Rank` = 
```
RANKX(
    ALL(World),
    CALCULATE(SUM(World[Active Cases])),
    CALCULATE(
        [Active Cases TD],
        ALL(Data[State],Data[Date])
    ),
    DESC,
    Dense
)
```


1.`Cases Yesterday` = 
```
CALCULATE(
    [Total Cases TD],
    DATEADD('Calendar'[Date],-1,DAY)
)
```


1.`CF Measure` = 
```
IF(
    MAXX(
        ALL(Data[State]),
        [Δ from yesterday]
    ) = [Δ from yesterday] && [Δ from yesterday] <> 0,
    1
)
```


1.`Compounded Growth% - Last 15 Days` = 
```
VAR DaysofGrowth = 15
VAR LastCaseDate = 
    CALCULATE(
        MAX(Data[Date]),
        ALL('Calendar'[Date])
    )
VAR LastDayValue = 
    CALCULATE(
        [Total Cases TD],
        DATESBETWEEN(
            'Calendar'[Date],
            LastCaseDate,
            LastCaseDate
        )
    )
VAR PrevValue = 
    CALCULATE(
        [Total Cases TD],
        DATESBETWEEN(
            'Calendar'[Date],
            LastCaseDate-DaysofGrowth,
            LastCaseDate-DaysofGrowth
        )
    )
VAR DailyCompoundedGrowth = 
    IFERROR(((LastDayValue/PrevValue)^(1/DaysofGrowth))-1,BLANK())
RETURN
    DailyCompoundedGrowth
```


1.`Deaths` = SUM(Data[Deaths Added])


1.`Deaths TD` = 
```
IF(
    HASONEVALUE(Data[State]),
    MAX(Data[Deaths]),
    CALCULATE(
        SUM(Data[Deaths]),
        LASTDATE(Data[Date])
    )
)
```


1.`Distribution` = 
```
VAR MinNum = MIN('Range Table'[Lower])
VAR MaxNum = MIN('Range Table'[Upper])
VAR CheckState = HASONEVALUE(Data[State])
VAR OverallMaxDate = 
    CALCULATE(
        MAX(Data[Date]),
        ALL(Data[Date])
    )
VAR FilteredTable = 
    FILTER(
        Data,
        VAR CheckthisValue = 
            IF(
                CheckState,
                [Total Cases TD],
                CALCULATE(MAX(Data[Running Total by State]))
            )
        RETURN
            CheckthisValue >= MinNum &&
            CheckthisValue <= MaxNum
    )
VAR MinDate = 
    CALCULATE(
        MIN(Data[Date]),
        FilteredTable
    )
VAR MaxDate = 
    CALCULATE(
        MAX(Data[Date]),
        FilteredTable
    )
VAR Label = 
    IF(
        MaxDate = OverallMaxDate,
        " Days & counting..",
        " Days"
    )
RETURN
    IF(
        MaxDate <> BLANK() && MinDate <> BLANK(),
        "In " & MAX( MaxDate - MinDate + 1 , BLANK()) & Label
    )
```


1.`Growth from Yesterday` = 
```
VAR casestd = [Total Cases TD]
VAR casesyesterday = 
    CALCULATE(
        [Total Cases TD],
        DATEADD('Calendar'[Date],-1,DAY)
    )
RETURN
    IF(
        casestd <> BLANK() && casesyesterday <> BLANK() &&
        HASONEVALUE('Calendar'[Date]),
        DIVIDE(casestd,casesyesterday) -1
    )
```


1.`GrowthRate` = SELECTEDVALUE(GrowthRateTable[Value])/100


1.`Max Cases Added on` = 
```
CONCATENATEX(
    TOPN(
        1,
        SUMMARIZE(
            Data,
            Data[Date]
        ),
        [Total Cases]
    ),
    FORMAT(Data[Date],"d-mmm") & " | " & [Total Cases],
    UNICHAR(10)
)
```


1.`Max Increase State` = 
```
VAR StateTable = 
    FILTER(
        ALL(Data[State]),
        [Δ from yesterday] <> 0
    )
RETURN
"⚫ Max Δ from yesterday" & UNICHAR(10)
&
CONCATENATEX(
    TOPN(
        1,
        StateTable,
        [Δ from yesterday]
    ),
    Data[State],
    ", "
) 
    &
    " | "
    &
MAXX(
    StateTable,
    [Δ from yesterday]
) & " Cases added"
```


1.`Projected Cases Date Tag` = 
```
VAR MaxDataDate = 
    CALCULATE(
        MAX(Data[Date]),
        ALL('Calendar'[Date])
    )
VAR MaxCalDate = MAX('Calendar'[Date])
RETURN
"by " & 
FORMAT(
    IF(
        MaxCalDate > MaxDataDate,
        MIN(
            EDATE(MaxDataDate,1),
            MaxCalDate
        )
    ),
    "dd-mmm"
)
```


1.`Projection Cases` = 
```
VAR ProjectionOnOff = SELECTEDVALUE('On/Off Table'[Label],"Off")
VAR GrowthRate = 
    IF(
        SELECTEDVALUE(GrowthRateTable[Tag]) = "15 Days Compounding",
        [Compounded Growth% - Last 15 Days],
        SELECTEDVALUE(GrowthRateTable[Value])/100
    )
VAR AsonCases = 
    CALCULATE(
        [Total Cases],
        ALL('Calendar'[Date])
    )
VAR MaxDate = 
    CALCULATE(
        MAX(Data[Date]),
        ALL('Calendar'[Date])
    )
VAR ProjectedTable = 
    CALCULATETABLE(
        'Calendar',
        DATESBETWEEN(
            'Calendar'[Date],
            MaxDate + 1,
            EDATE(MaxDate + 1, 1)
        )
    )
RETURN
IF( 
    MAX('Calendar'[Date]) <> BLANK()&&
    MAX('Calendar'[Date]) <= EDATE(MaxDate,1) && 
    ProjectionOnOff = "On",
    INT(
        MAXX(
            CALCULATETABLE(
                'Calendar',
                VAR OneMonthTable = 
                    DATESBETWEEN(
                        'Calendar'[Date],
                        MaxDate + 1,
                        MIN(MAX('Calendar'[Date]),EDATE(MaxDate,1))
                    )
                RETURN
                    OneMonthTable
            ),
            AsonCases * (1 + GrowthRate) ^ VALUE('Calendar'[Date] - MaxDate)
        )
    )
)
```


1.`Projection Cases Label` = 
```
VAR ProjectionOnOff = SELECTEDVALUE('On/Off Table'[Label],"Off")
VAR GrowthRate = 
    IF(
        SELECTEDVALUE(GrowthRateTable[Tag]) = "15 Days Compounding",
        [Compounded Growth% - Last 15 Days],
        SELECTEDVALUE(GrowthRateTable[Value])/100
    )
VAR AsonCases = 
    CALCULATE(
        [Total Cases],
        ALL('Calendar'[Date])
    )
VAR MaxDate = 
    CALCULATE(
        MAX(Data[Date]),
        ALL('Calendar'[Date])
    )
VAR ProjectedTable = 
    CALCULATETABLE(
        'Calendar',
        DATESBETWEEN(
            'Calendar'[Date],
            MaxDate + 1,
            EDATE(MaxDate + 1, 1)
        )
    )
RETURN
IF( 
    ProjectionOnOff = "On",
    INT(
        MAXX(
            CALCULATETABLE(
                'Calendar',
                VAR OneMonthTable = 
                    DATESBETWEEN(
                        'Calendar'[Date],
                        MaxDate + 1,
                        MIN(MAX('Calendar'[Date]),EDATE(MaxDate,1))
                    )
                RETURN
                    OneMonthTable
            ),
            AsonCases * (1 + GrowthRate) ^ VALUE('Calendar'[Date] - MaxDate)
        )
    )
)
```


1.`Recovered` = SUM(Data[Recovered Added])


1.`Recovered TD` = 
```
IF(
    HASONEVALUE(Data[State]),
    MAX(Data[Recovered]),
    CALCULATE(
        SUM(Data[Recovered]),
        LASTDATE(Data[Date])
    )
)
```


1.`Refreshed` = 
```
"Data Until " & 
FORMAT(
    CALCULATE(
        MAX(Data[Date]),
        ALL('Calendar')
    ),
    "dd-mmm"
 ) &
" | Refreshed " & 
FORMAT(
    MAX('Refresh'[Refresh]),
    "d-mmm hh:mm AM/PM"
)
```


1.`Total Cases` = SUM(Data[Total Added])


1.`Total Cases TD` = 
```
IF(
    HASONEVALUE(Data[State]),
    MAX(Data[Total]),
    CALCULATE(
        SUM(Data[Total]),
        LASTDATE(Data[Date])
    )
)
```


1.`World Active Cases` = SUM(World[Active Cases])


1.`Δ from yesterday` = 
```
[Total Cases TD] - 
CALCULATE(
    [Total Cases TD],
    DATESBETWEEN(
        'Calendar'[Date],
        LASTDATE(Data[Date])-1,
        LASTDATE(Data[Date])-1
    )
)
```