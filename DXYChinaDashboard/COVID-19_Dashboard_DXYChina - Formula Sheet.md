# COVID-19_Dashboard_DXYChina - Formula Sheet


## Measures
ActiveTD = [ConfirmedTD] - [DeadTD] - [CuredTD]


ConfirmedGrowthRate_15D% = 
VAR LastTime = MAX(DXYArea_province[updateTime])
VAR LastConfirmed =
CALCULATE(
    [ConfirmedTD],
    DXYArea_province[updateTime] = LastTime
)
VAR LastTime_15 = CALCULATE(MAX([updateTime]), FILTER(DXYArea_province, [updateTime]<LastTime-15))
VAR LastConfirmed_15 =
CALCULATE(
    [ConfirmedTD],
    DXYArea_province[updateTime] = LastTime_15
)
RETURN IFERROR(((LastConfirmed/LastConfirmed_15)^(1/15))-1,BLANK())


ConfirmedTD = 
CALCULATE(
    SUM(DXYArea_province[province_confirmedIncr]),
    FILTER(
        ALL(DXYArea_province[updateTime]),
        DXYArea_province[updateTime] < MAX(DXYArea_province[updateTime])
    )
)


ConfirmedTotal = SUM(DXYArea_province[province_confirmedIncr])


CuredTD = 
CALCULATE(
    SUM(DXYArea_province[province_curedIncr]),
    FILTER(
        ALL(DXYArea_province[updateTime]),
        DXYArea_province[updateTime] < MAX(DXYArea_province[updateTime])
    )
)


DeadTD = 
CALCULATE(
    SUM(DXYArea_province[province_deadIncr]),
    FILTER(
        ALL(DXYArea_province[updateTime]),
        DXYArea_province[updateTime] < MAX(DXYArea_province[updateTime])
    )
)


DeadTotal = SUM(DXYArea_province[province_deadIncr])


DeathRateTD% = DIVIDE([DeadTD],[ConfirmedTD])


DeathRateTotal% = DIVIDE([DeadTotal],[ConfirmedTotal])


ProvinceName = 
VAR myConcatMode = IF(COUNTAX(RELATEDTABLE(DXYArea_provinceZipMap), DXYArea_provinceZipMap[province_zipCode])>3,1,0)
VAR myText = IF(
    myConcatMode=1, 
    CONCATENATE(
        CONCATENATEX(
            TOPN(
                3,
                RELATEDTABLE(DXYArea_provinceZipMap),
                DXYArea_province[ConfirmedTD],
                DESC
            ),
            DXYArea_provinceZipMap[provinceEnglishName],
            ", "
        ),
        ", etc."
    ),
    CONCATENATEX(RELATEDTABLE(DXYArea_provinceZipMap),DXYArea_provinceZipMap[provinceEnglishName],", ")
)
RETURN myText


RecoveryRateTD% = DIVIDE([CuredTD],[ConfirmedTD])



## Calculated Columns
lag_updateTime = 
VAR myID = DXYArea_province[province_zipCode]
VAR myLastDate= MAXX(FILTER(DXYArea_province,DXYArea_province[province_zipCode] = myID && DXYArea_province[updateTime] < EARLIER(DXYArea_province[updateTime])),[updateTime])
RETURN MAXX(FILTER(DXYArea_province,DXYArea_province[province_zipCode] = myID && DXYArea_province[updateTime] = myLastDate), [updateTime])


province_confirmedIncr = 
VAR myID = DXYArea_province[province_zipCode]
VAR myLagConfirmed = SUMX(FILTER(DXYArea_province, DXYArea_province[province_zipCode] = myID && DXYArea_province[updateTime] = EARLIER(DXYArea_province[lag_updateTime])), [province_confirmedCount])
RETURN (DXYArea_province[province_confirmedCount] - myLagConfirmed)


province_curedIncr = 
VAR myID = DXYArea_province[province_zipCode]
VAR myLagCured = SUMX(FILTER(DXYArea_province, DXYArea_province[province_zipCode] = myID && DXYArea_province[updateTime] = EARLIER(DXYArea_province[lag_updateTime])), [province_curedCount])
RETURN (DXYArea_province[province_curedCount] - myLagCured)


province_deadIncr = 
VAR myID = DXYArea_province[province_zipCode]
VAR myLagDead = SUMX(FILTER(DXYArea_province, DXYArea_province[province_zipCode] = myID && DXYArea_province[updateTime] = EARLIER(DXYArea_province[lag_updateTime])), [province_deadCount])
RETURN (DXYArea_province[province_deadCount] - myLagDead)


province_suspectedIncr = 
VAR myID = DXYArea_province[province_zipCode]
VAR myLagSuspected = SUMX(FILTER(DXYArea_province, DXYArea_province[province_zipCode] = myID && DXYArea_province[updateTime] = EARLIER(DXYArea_province[lag_updateTime])), [province_suspectedCount])
RETURN (DXYArea_province[province_suspectedCount] - myLagSuspected)