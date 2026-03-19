# Bank Reserves Calculator

This tool queries the **FDIC Bank Data API** (BankFind Suite – Financials endpoint) to calculate and display bank liquidity and reserve-style ratios.  
All data comes directly from FDIC Call Report filings, which insured institutions are legally required to submit quarterly.

---

## Development Setup

### Prerequisites
- Node.js (v18+)
- npm

### Install Dependencies
```bash
npm install
```

### Run the Dev Server
```bash
npm run dev
```
The app starts at [http://localhost:3005](http://localhost:3005).

### Build for Production
```bash
npm run build
```

### Lint
```bash
npm run lint
```

### Deployment
This project deploys to **Cloudflare Pages** via GitHub Actions.

---

## 📊 Methodology

### 1. Data Source
- **Provider:** Federal Deposit Insurance Corporation (FDIC)  
- **API:** [FDIC BankFind Suite – Financials Endpoint](https://api.fdic.gov/banks/docs/)  
- **Dataset:** Financials alias dataset (standardized reporting fields).  
- **Frequency:** Quarterly Call Report filings (`REPDTE`).  
- **Access:** Requires a valid FDIC API key.  

---

### 2. Variables Used
From FDIC’s **Reference Variables & Definitions**:

- **`CHBAL`** – *Cash and Balances Due from Depository Institutions*  
  > Total cash and balances due from depository institutions, including both interest-bearing and non-interest-bearing balances.  
  > Includes balances at Federal Reserve Banks, other U.S. banks, foreign banks, and cash items in process of collection.  

- **`CHFRB`** – *Balances Due from Federal Reserve Banks*  
  > Subcomponent of CHBAL representing funds held in accounts at Federal Reserve Banks.  

- **`LIAB`** – *Total Liabilities*  
  > Aggregate liabilities of the institution as reported in its Call Report.  

- **`REPDTE`** – *Report Date* (quarter-end)  
- **`CERT`** – *FDIC Certificate Number* (unique bank identifier)

---

### 3. Ratios Computed

#### Liquidity Ratio
**Formula:**  
`Liquidity Ratio = CHBAL ÷ LIAB × 100`

- Broad measure of how much of a bank’s liabilities are covered by cash and due balances.  
- Includes balances at other banks and collection items.  
- Higher ratios imply stronger liquidity coverage.

#### Reserve-Style Ratio
**Formula:**  
`Reserve-Style Ratio = CHFRB ÷ LIAB × 100`

- Narrower measure focused only on balances at the Federal Reserve.  
- Closest available proxy for the **pre-2020 reserve requirement definition** (vault cash + Fed balances).  
- Since vault cash is not separately exposed in the alias dataset, this ratio captures only Fed balances.

---

### 4. Importance
- **Liquidity Risk Monitoring** – Gauges ability to meet short-term obligations with liquid assets.  
- **Transparency** – Standardized, comparable ratios across all FDIC-insured institutions.  
- **Regulatory Context** – While reserve requirements are currently 0% (as of March 26, 2020), monitoring cash and Fed balances relative to liabilities provides insight into financial resilience.  

---

### 5. Limitations
1. **Vault Cash** is not exposed in the alias dataset. True historical reserves = Vault Cash + Fed Balances.  
2. **CHBAL** includes balances at other banks and collection items, which are liquid but not regulatory reserves.  
3. **Quarterly Data** – Results represent point-in-time values at quarter-end. Intraday and monthly liquidity positions are not captured.  
4. **Context** – Ratios should be used alongside other regulatory liquidity measures (e.g., Liquidity Coverage Ratio under Basel III).

---

### 6. Replication
To verify results, query the FDIC API directly. Example for **Bank of America (CERT 3510):**

```bash
curl "https://banks.data.fdic.gov/api/financials?filters=CERT:3510&fields=CHBAL,CHFRB,LIAB,REPDTE,NAME&sort_by=REPDTE&sort_order=desc&limit=1&format=json&api_key=YOUR_KEY"