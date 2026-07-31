**🌱 Week_Table**
```dax
Week_Table = 
VAR AsOf = MAX(Model_Parameters[As of Date])
RETURN
ADDCOLUMNS(
    GENERATESERIES(1, 13, 1),
    "Week_Start", AsOf + (([Value]-1)*7),
    "Week_End", AsOf + (([Value]-1)*7) + 6
)
```
**🌱 Inflow (AR)**
```dax
Inflow (AR) = 
VAR SelectedProbAdj = SELECTEDVALUE(Scenario_Assumptions[Prob Adj], 0)
VAR SelectedARDelayMult = SELECTEDVALUE(Scenario_Assumptions[AR Delay Mult], 1)
VAR WkStart = SELECTEDVALUE(Week_Table[Week_Start])
VAR WkEnd = SELECTEDVALUE(Week_Table[Week_End])
RETURN
SUMX(
    AR_detail,
    VAR AdjProb = MIN(1, MAX(0, AR_detail[Collection_Probability] + SelectedProbAdj))
    VAR ExpectedDate = 
        SWITCH(
            TRUE(),
            AR_detail[Status] = "Collected", AR_detail[Autual_Collection_Date],
            AR_detail[Status] = "Open", AR_detail[Due_Date],
            AR_detail[Due_Date] + RELATED(Customer_list[Typical_Delay_Days]) * SelectedARDelayMult
        )
    RETURN
        IF(
            ExpectedDate >= WkStart && ExpectedDate <= WkEnd && AR_detail[Status] <> "Collected",
            AR_detail[Amount] * AdjProb,
            0
        )
)
```
**🌱 Outflow (AP)**
```dax
Outflow (AP) = 
VAR SelectedAPStretchMult = SELECTEDVALUE(Scenario_Assumptions[AP Stretch Mult], 1)
VAR WkStart = SELECTEDVALUE(Week_Table[Week_Start])
VAR WkEnd = SELECTEDVALUE(Week_Table[Week_End])
RETURN
SUMX(
    AP_detail,
    VAR ExpectedDate = 
        SWITCH(
            TRUE(),
            AP_detail[Status] = "Paid", AP_detail[Actual_Payment_Date],
            AP_detail[Status] = "Open", AP_detail[Due_Date],
            AP_detail[Due_Date] + RELATED(Vendor_list[Typical_Delay_Days]) * SelectedAPStretchMult
        )
    RETURN
        IF(
            ExpectedDate >= WkStart && ExpectedDate <= WkEnd && AP_detail[Status] <> "Paid",
            AP_detail[Amount],
            0
        )
)
```
**🌱 Ending Balance**
```dax
Ending Balance = 
VAR CurrentWeek = SELECTEDVALUE(Week_Table[Week_Number])
VAR BeginningCash = SELECTEDVALUE(Model_Parameters[Beginning Cash])
VAR CumulativeNet = 
    SUMX(
        FILTER(ALL(Week_Table), Week_Table[Week_Number] <= CurrentWeek),
        CALCULATE([Net Cash Flow])
    )
RETURN
    BeginningCash + CumulativeNet
```
**🌱 Net Cash Flow**
```dax
Net Cash Flow = [Inflow (AR)] - [Outflow (AP)]
```
**🌱 Below Threshold?**
```dax
Below Threshold? = 
VAR EndBal = [Ending Balance]
VAR MinThreshold = SELECTEDVALUE(Model_Parameters[Min Cash Threshold])
RETURN
    IF(EndBal < MinThreshold, "WARNING", "OK")
```
**🌱 AR_Risk_Label**
```dax
AR_Risk_Label = AR_detail[Customer_Name] & " (" & AR_detail[Invoice_ID] & ")"
```
**🌱 AP_Risk_Label**
```dax
AP_Risk_Label = AP_detail[Vendor_Name] & " (" & AP_detail[Bill_ID] & ")"
```
**🌱 Semantic Model**  
<img width="1323" height="652" alt="image" src="https://github.com/user-attachments/assets/fcffafe0-61dc-4ab6-a6ec-a7ab98453198" />
**🌱 Different Scenerioes** 

<img width="773" height="573" alt="image" src="https://github.com/user-attachments/assets/761b0cd1-e6c0-4e1e-93f5-754a50a97c78" />
<img width="773" height="573" alt="image" src="https://github.com/user-attachments/assets/f838b279-0bef-4230-83ba-07c0c3391825" />
<img width="773" height="573" alt="image" src="https://github.com/user-attachments/assets/4c293472-a7e1-4c09-a602-f2f0a2792f87" />


