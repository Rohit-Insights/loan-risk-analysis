# loan-risk-analysis
Loan Risk &amp; Lending Performance Analysis using SQL and Power BI
Business Problem
Banks need to monitor loan performance and identify high-risk borrowers to reduce defaults and improve profitability. This project analyzes lending data to uncover patterns in borrower risk, loan performance, and repayment behavior — enabling better credit decision-making.
Dataset
•	38,576 loan records  ·  Full year 2021
•	$435.8M total funded  ·  $473.0M total received
•	13.8% charge-off rate — 5,333 defaulted loans
•	$28.2M permanent net loss — $65.5M lent to defaulters, $37.3M recovered (56.9% recovery rate)
Key Insights
•	Grade B & C are the real loss driver, not Grade G. They account for 49% of all defaults — high volume at moderate risk outweighs low volume at extreme risk.
•	60-month loans default at 2.1× the rate of 36-month loans. All 1,098 currently open loans are 60-month and still developing; average loan size is 1.6× larger, amplifying dollar loss per default.
•	Business-purpose loans carry structural risk beyond credit grade. Default rate is 11.4% even at Grade A — 2× all other purposes at the same grade.
•	Largest dollar risk: Grade E + Debt Management — $31.8M at 24.8% default rate. Invisible from individual charts; surfaces only via cross-filtering the detail page.
•	Verified borrowers default more than unverified ones (15.7% vs 12.2%). Verified borrowers receive ~2× larger loans on average — verification flags risk but doesn't limit exposure.
Business Impact
•	Reframed risk focus — Grade B & C concentration is a bigger priority than high-grade outliers
•	Surfaced term-based risk — 60-month loans warrant tighter approval controls and adjusted pricing
•	Uncovered $28.2M in permanent losses not visible from the charge-off count alone
•	Built evidence for grade × purpose × term segmentation to inform approval policy and pricing strategy
Dashboard Overview
4-page Power BI dashboard, each page built around a single business question:

•	Page 1 — Summary: Is the portfolio healthy?  —  KPI strip with MTD/MoM metrics, loan status breakdown, grade distribution, verification analysis, income band split
•	Page 2 — Risk View: Where is the risk?  —  Dual-axis trend (volume vs charge-off rate), purpose chart colored by SQL risk tiers, term comparison, geographic map
•	Page 3 — Detail: Show me individual loans.  —  14-column row-level grid with Risk Tier on every loan, recovery panel locked to portfolio totals, 6 slicers
•	Page 4 — Insights: What does it mean?  —  8 insight cards, 5 recommendations, each traceable to a specific visual or filter combination
Data Modeling (SQL)
Normalized a 24-column flat CSV into a star schema — 1 fact table, 4 dimension tables:

•	Fact Table: fact_loan — loan-level metrics with surrogate keys to all four dimensions
•	Dimensions:
◦	dim_grade  —  Grades A–G enriched with risk_label (Very Low Risk → Extreme Risk), surfaced in dashboard tooltips
◦	dim_purpose  —  Loan purpose with risk_tier derived via SQL CTE from actual charge-off rates — High >18%, Medium 11–18%, Low <11%
◦	dim_date  —  Full 365-day calendar via stored procedure; source had only 65 sparse dates, insufficient for time intelligence
◦	dim_region  —  State codes mapped to US Census Bureau groups (Northeast / South / Midwest / West)

•	dim_borrower evaluated and rejected — one row per loan = same granularity as the fact table; no repetition means no dimensional value
•	risk_tier flows end-to-end: SQL CTE → dim_purpose → bar chart colors → Risk Tier column on every row in the detail grid
Technical Highlights
•	SQL · Risk classification: CTE calculates charge-off rate per purpose and assigns tiers dynamically — no hardcoded labels
•	SQL · Key debugging: purpose_id misaligned across all 14 purposes — dim_purpose loaded in charge-off order, fact_loan in source order; diagnosed, corrected, and validated
•	DAX · Time intelligence: CALCULATE + DATESMTD used instead of TOTALMTD for averages — TOTALMTD is additive-only; average of monthly averages ≠ true period average
•	DAX · Bar colors from data: A SWITCH measure returns hex strings from dim_purpose[risk_tier] — chart colors driven by the SQL model, not manual formatting
•	DAX · Recovery panel lock: CALCULATE with a hardcoded 'Charged Off' filter preserves portfolio-level recovery totals regardless of active slicers
•	Validation: Every KPI cross-checked against SQL ground truth before sign-off; rounding differences between 1dp and 2dp display documented
Notable Analytical Decisions
•	Recovery panel — self-initiated: Not in the original scope. The $28.2M net loss figure is more meaningful than a raw charge-off count and reframes how portfolio health is interpreted
•	Current loans shown separately in amber: Not grouped with Fully Paid as 'Good Loans' — they haven't completed repayment and could still default. Silent assumptions don't belong in an analytical dashboard
What This Project Demonstrates
•	End-to-end ownership: raw data → star schema → validated SQL → Power BI dashboard → business insights
•	SQL & data modeling: schema design, CTE pipelines, stored procedures, surrogate key management
•	DAX proficiency: time intelligence, filter context control, data-driven formatting
•	Analytical reasoning: modeling decisions documented with logic — choices explained, not just executed
•	Business communication: insights framed around decisions and financial impact, not just data observations
