### 🪴 Build Process  
<br>

**🌱 Step 1: Problem framing**  
- Started by identifying the real pain point observed in a previous finance role, rather than picking a generic textbook exercise
- When I rotated into the AP function, my manager would sometimes ask which payments could be postponed. Answering that meant exporting the ERP dataset and manually reorganizing it each time. This project is designed to eliminate that repetitive manual work and replace it with a system that shows the cash position in time
- Defined the problem statement before touching any data, to keep the project grounded in an actual business need

  <img width="800" height="160" alt="image" src="https://github.com/user-attachments/assets/80c1fc07-fbfa-4e9d-80b6-0e79f1e60ff1" />

<br>

**🌱 Step 2: Data schema design**  
- Designed **AR_detail** and **AP_detail** table structures to mirror what a real ERP export would look like (invoice/bill ID, customer/vendor reference, dates, payment terms, amount,  status)  
- Built **Customer_list** and **Vendor_list** as separate lookup tables, with payment behavior tiers (Good/Average/Poor, High/Medium/Low Priority) to give the transaction data internal consistency rather than pure randomness  

*💧 Explanation of each sheet*

- AR_detail: Simulates an ERP accounts receivable export
  - `Due_Date`: Auto-calculated from `Invoice_Date` using an IFS formula based on `Payment_Terms` (different logic for EM vs. OA terms)
  - `Collection_Probability`: Pulled via VLOOKUP from Customer_list, based on the customer's payment tier
  - `Status`: Open (not yet due) / Collected (payment received) / Overdue (past due, not yet collected)
  - `Expected_Collection_Date`: Uses the actual collection date if already collected; the due date if not yet due; and for overdue invoices, estimates "due date + customer's typical delay days × scenario multiplier"
  - `Scenario_Adjusted_Probability`: Base collection probability plus a scenario adjustment, reflecting how collection likelihood shifts under Best/Base/Worst scenarios
  - Screenshot
    <img width="1202" height="193" alt="image" src="https://github.com/user-attachments/assets/10d073d1-e6da-4ef1-aadb-28be9cd63136" />

- AP_detail: Simulates an ERP accounts payable export
  - `Due_Date`: Same IFS-formula approach based on `Payment_Terms`
  - `Priority`: Pulled via VLOOKUP from Vendor_list, based on the vendor's priority tier
  - `Status`: Open (not yet due) / Paid (payment made) / Overdue (past due, not yet paid)
  - `Expected_Payment_Date`: Uses the actual payment date if already paid; the due date if not yet due; and for overdue bills, estimates "due date + vendor's typical delay days × scenario multiplier"
  - Screenshot
    <img width="1047" height="184" alt="image" src="https://github.com/user-attachments/assets/cf4050e6-8583-4e4a-ada1-729137fbac56" />

- Customer_list: Customer master table that drives the collection assumptions in AR_detail
  - `Payment Habit` (Good/Average/Poor): Tiered by historical payment behavior
  - `Collection_Probability` / `Typical_Delay_Days`: Auto-populated from a tier lookup table (Habit → Base_Probability → Typical_Delay_Days), ensuring customers in the same tier are treated consistently rather than being set independently
  - Screenshot
    <img width="1024" height="100" alt="image" src="https://github.com/user-attachments/assets/1ca9ee43-7817-4ad9-8d9f-0b06c4824ae8" />

- Vendor_list: Vendor master table that drives the payment scheduling assumptions in AP_detail
  - `Priority` (High/Medium/Low): Tiered by how critical the vendor is to operations (e.g., logistics and core raw-material suppliers are tiered High)
  - `Typical_Delay_Days`: Auto-populated from a priority lookup table (Priority → Typical_Delay_Days). Low-priority vendors carry longer typical delays, reflecting which payments the company would deprioritize first under cash pressure
  - Screenshot  
    <img width="727" height="94" alt="image" src="https://github.com/user-attachments/assets/2f5266f2-bed0-48b9-96c2-c4c9350606e5" />

<br>

**🌱 Step 3: Core formula logic**  
- Not every column in AR_detail/AP_detail is calculated — some are sourced directly from the ERP export (e.g., Invoice_ID, Amount, Status), reflecting actual transaction records. Others are analytical fields layered on top to support forecasting (e.g., Due_Date when not already provided, Collection_Probability, Expected_Collection_Date). The logic below covers the calculated fields.
- Due_Date calculation
  - If the ERP export doesn't already include this column, calculate it using an `IFS` formula based on payment terms:
    ```excel
    =IFS(G2="EM15",EOMONTH(E2,0)+15, G2="EM30",EOMONTH(E2,0)+30, G2="EM45",EOMONTH(E2,0)+45, G2="EM60",EOMONTH(E2,0)+60, G2="OA15",E2+15, G2="OA30",E2+30, G2="OA60",E2+60 )
    ```
    <img width="1257" height="102" alt="image" src="https://github.com/user-attachments/assets/908edde7-ed58-4ef7-9298-9920c04de502" />
   - If the ERP export already includes this column, just copy and paste it directly.

- Collection_Probability
  - This applies only to AR. Collection probability estimates how likely we are to receive payment, helping flag which invoices carry higher bad-debt risk.
    ```excel
    =INDEX(Customer_list!$G$2:$G$21,MATCH(B2,Customer_list!$B$2:$B$21,0))
    ```
    <img width="1224" height="117" alt="image" src="https://github.com/user-attachments/assets/5b8e054b-692f-4e30-be14-a6e8afd5f32c" />

- Status logic (Open/Collected/Overdue) is also sourced directly from the ERP export, same as Invoice_ID and Amount — it reflects the actual transaction state rather than being a calculated field.
<br>

**🌱 Step 4: Weekly rollup**  
Built the Weekly_Cash_Flow_Summary sheet: 13-week buckets, SUMPRODUCT-based AR inflow (probability-weighted) and AP outflow calculations, and a rolling Ending Balance with a below-threshold flag.
<br>

**🌱 Step 5: Scenario modeling**  
Added a Best/Base/Worst scenario toggle, driven by three parameters — collection probability adjustment, AR delay multiplier, and AP payment-stretch multiplier — so the forecast can be stress-tested without editing any underlying formulas.
<br>

**🌱 Step 6: Finalizing check**
Reduced the dataset to a manageable row count for a portfolio-scale demo, then re-validated that all formulas and logic still held after the trim.
<br>
