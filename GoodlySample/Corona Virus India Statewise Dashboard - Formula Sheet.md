# Corona Virus India Statewise Dashboard - Formula Sheet


## Measures
`%Active` = DIVIDE( [Active Cases TD] , [Total Cases TD] )


`%Deaths` = DIVIDE([Deaths TD] , [Total Cases TD])


`%Recovered` = DIVIDE( [Recovered TD] , [Total Cases TD] )


`Active Cases` = SUM(Data[Active Added])


`Active Cases State Rank` = 
RANKX(
    ALL(Data[State]),
    [Active Cases TD],,
    DESC,
    Dense
)


`Active Cases TD` = 
CALCULATE(
    SUM(Data[Active]),
    LASTDATE(Data[Date])
)


`Active Cases World Rank` = 
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


`Cases Yesterday` = 
CALCULATE(
    [Total Cases TD],
    DATEADD('Calendar'[Date],-1,DAY)
)


`CF Measure` = 
IF(
    MAXX(
        ALL(Data[State]),
        [Δ from yesterday]
    ) = [Δ from yesterday] && [Δ from yesterday] <> 0,
    1
)


`Compounded Growth% - Last 15 Days` = 
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


`Deaths` = SUM(Data[Deaths Added])


`Deaths TD` = 
IF(
    HASONEVALUE(Data[State]),
    MAX(Data[Deaths]),
    CALCULATE(
        SUM(Data[Deaths]),
        LASTDATE(Data[Date])
    )
)


`Distribution` = 
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


`Growth from Yesterday` = 
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


`GrowthRate` = SELECTEDVALUE(GrowthRateTable[Value])/100


`Max Cases Added on` = 
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


`Max Increase State` = 
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


`Projected Cases Date Tag` = 
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


`Projection Cases` = 
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


`Projection Cases Label` = 
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


`Recovered` = SUM(Data[Recovered Added])


`Recovered TD` = 
IF(
    HASONEVALUE(Data[State]),
    MAX(Data[Recovered]),
    CALCULATE(
        SUM(Data[Recovered]),
        LASTDATE(Data[Date])
    )
)


`Refreshed` = 
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


`Total Cases` = SUM(Data[Total Added])


`Total Cases TD` = 
IF(
    HASONEVALUE(Data[State]),
    MAX(Data[Total]),
    CALCULATE(
        SUM(Data[Total]),
        LASTDATE(Data[Date])
    )
)


`World Active Cases` = SUM(World[Active Cases])


`Δ from yesterday` = 
[Total Cases TD] - 
CALCULATE(
    [Total Cases TD],
    DATESBETWEEN(
        'Calendar'[Date],
        LASTDATE(Data[Date])-1,
        LASTDATE(Data[Date])-1
    )
)