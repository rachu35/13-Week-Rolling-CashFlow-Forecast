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
  - `Due_Date`: Same IFS-formula approach based on `Payment_Terms`
  - `Priority`: Pulled via VLOOKUP from Vendor_list, based on the vendor's priority tier
  - `Status`: Open (not yet due) / Paid (payment made) / Overdue (past due, not yet paid)
  - `Expected_Payment_Date`: Uses the actual payment date if already paid; the due date if not yet due; and for overdue bills, estimates "due date + vendor's typical delay days × scenario multiplier"
  - Screenshot
    <img width="1202" height="193" alt="image" src="https://github.com/user-attachments/assets/10d073d1-e6da-4ef1-aadb-28be9cd63136" />

- AP_detail: Simulates an ERP accounts payable export
- Customer_list: Customer master table that drives the collection assumptions in AR_detail
- Vendor_list: Vendor master table that drives the payment scheduling assumptions in AP_detail




<br>

**🌱 Step 3: Building & reviewing simulated data**  
Generated the AR/AP detail rows and reviewed them closely rather than accepting the first draft. This review caught several logic issues that were fixed iteratively:

Invoice/bill dates initially spanned the full year, including future dates — inconsistent with a fixed "as-of" reference date
Payment dates were randomly distributed, when in practice AP/AR run on fixed batch cycles
A few field labels didn't match their actual column content
<br>

**🌱 Step 4: Core formula logic**  
Built the Due_Date calculation (IFS formula based on payment terms), Collection_Probability, and status logic (Open/Collected/Overdue) with cross-sheet lookups back to the Customer_list/Vendor_list tables.
<br>

**🌱 Step 5: Weekly rollup**  
Built the Weekly_Cash_Flow_Summary sheet: 13-week buckets, SUMPRODUCT-based AR inflow (probability-weighted) and AP outflow calculations, and a rolling Ending Balance with a below-threshold flag.
<br>

**🌱 Step 6: Scenario modeling**  
Added a Best/Base/Worst scenario toggle, driven by three parameters — collection probability adjustment, AR delay multiplier, and AP payment-stretch multiplier — so the forecast can be stress-tested without editing any underlying formulas.
<br>

**🌱 Step 7: Finalizing check**
Reduced the dataset to a manageable row count for a portfolio-scale demo, then re-validated that all formulas and logic still held after the trim.
<br>
