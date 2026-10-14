# Qatar Banking Sector & QNB Performance Dashboard

A banking analytics project that combines **Qatar Central Bank (QCB) sector data** with **QNB Group's published financial results** to track balance-sheet growth, credit concentration, asset quality, capital adequacy, and liquidity.

**Tools:** Power BI, DAX, Power Query, SQL, star-schema modelling
 

---

## Business questions

1. How large is Qatar's commercial banking sector, and how fast are assets, deposits, and credit growing?
2. Where is domestic credit concentrated by sector?
3. How is the split between public-sector and private-sector deposits changing?
4. How healthy is Qatar's largest bank (QNB) on asset quality, capital, and liquidity?
5. How did QNB's key ratios move between reporting periods?

---

## Data sources

| Source | Content | Period |
|---|---|---|
| Qatar Central Bank (monthly banking data, as reported by Gulf Times) | Commercial bank assets, deposits, domestic credit by sector, credit outside Qatar | June 2026 |
| QNB Group financial results release | Net profit, EPS, cost-to-income, NPL ratio, coverage, CAR, LCR, NSFR | FY ended 31 Dec 2025 |
| QNB Group nine-month results (as reported by Gulf Times) | Total assets, loans, deposits, equity, operating income, ratios | 30 Sep 2025 |

 
 
 

## Data model (star schema)

```
            Dim_Date
               |
Dim_Bank --- Fact_BankRatios --- Dim_Ratio
               
Dim_Date
   |
Fact_SectorCredit --- Dim_Sector
   |
Fact_SectorBalance --- Dim_Category (assets / public deposits / private deposits)
```

| Table | Type | Grain |
|---|---|---|
| `Fact_SectorCredit` | Fact | Month x credit sector |
| `Fact_SectorBalance` | Fact | Month x balance-sheet category |
| `Fact_BankRatios` | Fact | Bank x reporting period x ratio |
| `Dim_Date` | Dimension | Date, month, quarter, year |
| `Dim_Bank` | Dimension | Bank name, group, Islamic/conventional flag |
| `Dim_Sector` | Dimension | Credit sector, public/private flag |
| `Dim_Ratio` | Dimension | Ratio name, category (asset quality, capital, liquidity, efficiency) |

Relationships are one-to-many with single-direction filtering from dimension to fact.

---

## KPI definitions

| KPI | Definition |
|---|---|
| NPL ratio | Non-performing loans ÷ gross loans |
| Loan-loss coverage | Loan-loss provisions ÷ non-performing loans |
| Capital adequacy ratio (CAR) | Regulatory capital ÷ risk-weighted assets |
| Cost-to-income | Operating expenses ÷ operating income |
| Liquidity coverage ratio (LCR) | High-quality liquid assets ÷ net cash outflows over 30 days |
| Net stable funding ratio (NSFR) | Available stable funding ÷ required stable funding |
| Loan-to-deposit ratio | Loans and advances ÷ customer deposits (derived; about 104% for QNB at 30 Sep 2025) |
| Sector credit share | Sector credit ÷ total domestic credit across listed sectors |
| YoY growth | (Current period ÷ same period prior year) − 1 |

DAX:

```dax
Loan to Deposit Ratio =
DIVIDE ( [Total Loans], [Total Customer Deposits] )

Sector Credit Share =
DIVIDE (
    SUM ( Fact_SectorCredit[Credit_QR_bn] ),
    CALCULATE ( SUM ( Fact_SectorCredit[Credit_QR_bn] ), ALL ( Dim_Sector ) )
)

YoY Growth % =
VAR Prior =
    CALCULATE ( SUM ( Fact_SectorBalance[Value_QR_bn] ), SAMEPERIODLASTYEAR ( Dim_Date[Date] ) )
RETURN
    DIVIDE ( SUM ( Fact_SectorBalance[Value_QR_bn] ) - Prior, Prior )
```

---

## Dashboard pages

1. **Sector snapshot:** assets, domestic credit, public vs. private deposits, YoY growth.
2. **Credit concentration:** domestic credit by sector with share of total.
3. **QNB scorecard:** net profit, EPS, cost-to-income, NPL ratio, CAR.
4. **Risk & liquidity:** NPL, CAR, LCR, and NSFR across reporting periods.
5. **Balance sheet:** assets, loans, deposits, equity, and loan-to-deposit ratio.

 

 

- Aggregate and bank-level data only; no customer or transaction-level analysis.
- Single-period snapshot for the sector view until the full QCB time series is loaded.
- Ratios come from company-reported results and may differ in methodology between banks.
