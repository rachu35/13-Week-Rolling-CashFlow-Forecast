### 🪴 Excel Build — Methodology & Key considerations
<br>

**🌱 Simulated data logic**
- AR/AP detail tables simulate raw transaction exports from an ERP system, with fields covering invoice/bill dates, due dates (auto-calculated from payment terms via an IFS formula), amounts, and status
- Customers/vendors are tiered by payment behavior **(Good/Average/Poor for customers & High/Medium/Low Priority for vendors)**, with credit tier driving collection probability and typical delay days, so the data has internal logic rather than being independently randomized

**🌱 Issues identified and fixed during development**
1. **Date plausibility**: The first draft of simulated data had invoice dates spread across the entire year (including future dates), which contradicted the model's "today" reference point. Fixed by constraining invoice/bill dates to roughly the 4 months preceding the as-of date, so the Status logic holds up.
2. **Payment dates shouldn't be random**: In practice, both AP and AR run on fixed batch payment cycles (e.g., the 10th and 25th of each month), not on arbitrary days. Adjusted the model so each customer/vendor is tied to a fixed set of payment-run days.
3. **Field labels not matching actual content**: Caught a case where the Vendor_Code and Vendor_Name column headers were swapped — a reminder to verify each column's actual content against its label rather than assuming the header is correct.<img width="521" height="170" alt="image" src="https://github.com/user-attachments/assets/928205a4-b341-4dd8-bfbf-0f2c7c749bcc" />

4. **Logical consistency checks**: Ensured Status (Open/Collected/Overdue) always lines up with Actual_Collection_Date/Actual_Payment_Date, so there are no contradictions like a row marked "Open" that still shows a collection date.

**🌱 Modeling Assumptions**
- 13-week rolling forecast, calculated forward from the as-of date — consistent with the industry-standard "13-week cash flow" approach
- AR inflows are probability-weighted (Amount × Collection_Probability) to avoid overstating expected cash in; AP outflows are counted in full, since a payment obligation isn't probabilistic — it's a question of timing, not likelihood
- Best/Base/Worst scenarios are driven by three adjustable parameters: collection probability adjustment, customer delay multiplier, and a company-controlled AP payment-stretch multiplier. AP stretching is a deliberate, controllable lever (the company can choose to slow vendor payments to preserve cash), while AR delay is an external risk conceptually different, but both are captured as scenario inputs

**🌱 Known Limitations**
- Data is simulated, not sourced from a real company; collection/payment behavior patterns are based on reasonable business assumptions
- No automated weekly refresh: This is maintained manually. After updating the data each week, a new dated file is saved (e.g., Cash_Flow_Forecast_20260727.xlsx) and sent to leadership. This ensures stakeholders always see the latest 13-week view, and the weekly files themselves double as a natural version history, making it easy to later compare forecast vs. actual for variance analysis. Fully automating this would require integrating directly with the ERP system for scheduled data exports.
