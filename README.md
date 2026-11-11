# Council data analysis
A data analytics portfolio project exploring spending, budget variance, supplier concentration and payment status across six fictional UK councils.

The project follows a Python → MySQL → Power BI workflow: prepare and reconcile the data in Python, validate and analyse it in SQL, then build an interactive management report in Power BI.
The dataset is synthetic. Supplier concentration and procurement methods are signals for further review and do not establish misconduct or non-compliance.

Project status
- Python: ETL and feature engineering implemented; accepted CSV available.
- MySQL: Staging, validation and reporting phase in progress; SQL scripts are not yet included in this repository.
- Power BI: Dashboard development planned.
Business objectives
The analysis aims to answer:
- Which councils, departments and spending categories have the largest budget variances?
- How do recorded, paid and pending spending change over time?
- How concentrated is each council’s spending among suppliers?
- How does spending vary by procurement method and funding source?
- Where do missing supplier details and other data-quality issues require follow-up?
Tools
Tool	Purpose	Status
Python and pandas	Profiling, cleaning, exception handling and feature engineering	Implemented
Jupyter Notebook	Documenting and running the Python analysis	Included
MySQL and MySQL Workbench	Staging, validation, constraints and reporting views	In progress
Power BI	Data modelling, DAX measures and interactive reporting	Planned


The requirement document references PostgreSQL or SQL Server. This implementation uses MySQL; loading methods and SQL syntax will reflect that choice.
Dataset
The source file contains 50,000 transactions and 14 columns, covering six fictional councils: Southmoor, Northbridge, Greenvale, Rivermouth, Westhaven and Eastford.

The transaction fields include dates, councils, departments, spending categories, suppliers, descriptions, actual spend, budgets, payment statuses, procurement methods and funding sources. Amounts are expressed in GBP.
Repository contents
File	Description
Local_Council_Spending_50000_rows.csv	Original synthetic input dataset
local council.ipynb	Python notebook for data preparation and feature engineering
accepted_data.csv	Accepted dataset containing 49,570 rows and 23 columns
Local_Council_Spending_Project_Requirement.docx	Business requirements and phased deliverables
README.md	Project overview, rules and progress


Rejected/review exports, standalone quality-summary files, SQL scripts and Power BI deliverables will be added as the repository is developed.
Workflow
1. Python: data preparation
The notebook profiles the source data, separates exceptions, standardises payment statuses and creates analytical fields.

The cleaning rules distinguish between:
- Accepted records: transactions suitable for downstream analysis.
- Rejected records: hard validation failures, such as invalid dates or non-GBP currency.
- Review records: negative actual-spend amounts requiring a finance decision; these are kept separately rather than sign-flipped or silently removed.
Missing supplier names are retained. A derived supplier label reports them as Not recorded, and a Boolean flag preserves the missing-data indicator.
2. MySQL: controlled loading and analysis
The next phase loads the accepted CSV into staging, checks the loaded records and promotes validated data to final tables.

Planned controls include:
- Comparing staged and final row counts with Python’s accepted count.
- Checking transaction-key uniqueness, required fields, currency, amounts and status flags.
- Using DATE for dates and DECIMAL(14,2) for currency amounts.
- Recording load counts, timestamps and outcomes in a load-audit table.
- Making reruns duplicate-safe and reconciling SQL measures with Python.
The reporting views will cover spend versus budget, monthly movements, supplier/procurement concentration, and payment status/data completeness.
3. Power BI: reporting
The planned dashboard will connect to validated SQL tables or views and contain four pages:
1. Executive overview.
2. Council and department analysis.
3. Supplier and procurement analysis.
4. Payment status and data quality.
Explicit DAX measures will follow the same inclusion rules used in Python and SQL. Dashboard totals will be reconciled with SQL before publication.
Payment-status inclusion rules
Recorded Spend represents transaction value included under the project’s workflow rule; it is not the same as cash paid.
Standardised status	Recorded Spend	Paid Spend	Pending Spend
paid	Included	Included	Excluded
approved	Included	Excluded	Included
pending	Included	Excluded	Included
under review	Included	Excluded	Included
cancelled	Excluded	Excluded	Excluded


Under Review and Cancelled values can also be reported separately. A transaction with the under review payment status is distinct from an ETL review record routed out because of a negative amount.
Derived columns
The accepted export contains the 14 source fields plus these nine derived columns:
Column	Meaning
derived_supplier_name	Supplier reporting label, including Not recorded for missing suppliers
derived_supplier_flag	Indicates that the original supplier name was missing
derived_payment_status	Standardised payment status
recorded_spend	Flag identifying records included in Recorded Spend
paid_spend	Flag identifying records included in Paid Spend
pending_spend	Flag identifying records included in Pending Spend
Variance_GBP	Actual spend minus budget
Variance_pct	Variance divided by budget, multiplied by 100
Supplier_Share_Of_Council_Pct	Supplier Recorded Spend divided by Council Recorded Spend, multiplied by 100


Positive variance indicates overspend; negative variance indicates underspend. Aggregate variance and budget calculations use the same Recorded Spend inclusion rule. Supplier share also uses Recorded Spend, with Not recorded retained as its own supplier group.

The row-level variance percentage uses a missing result where the budget is zero or missing. SQL ratios will use safe division such as NULLIF.

For MySQL import, Boolean flags are exported as 1 for True and 0 for False. These flag columns identify inclusion; spend values must be calculated from actual_spend_gbp, rather than by summing the flags.
Row-count reconciliation
Category	Rows	Share of input
Accepted	49,570	99.14%
Rejected	180	0.36%
Review	250	0.50%
Total accounted for	50,000	100.00%


49,570 + 180 + 250 = 50,000, matching the input count with a numerical difference of zero. Output categories must also remain mutually exclusive so that each input record is accounted for once.

The quality summary covers row counts by category, exception counts by reason, null counts and mappings applied. Exception counts may exceed affected-row counts when a record has multiple issues.
Current data-quality observations
- 99.14% of input records were retained in the accepted dataset.
- 430 records were separated into rejected or review categories, preserving visibility of exceptions.
- The accepted-data null summary identified 646 missing original supplier names, approximately 1.30% of accepted records. These remain visible through the supplier flag and Not recorded reporting label.
Business findings on spending, budget variance and concentration will be added after the SQL analysis and validation are complete.
Running the Python notebook
1. Clone or download this repository.
2. Open local council.ipynb in Jupyter Notebook or JupyterLab in an environment with pandas installed.
3. Update the notebook’s input and output paths to match your local project folder. The current notebook contains Windows paths.
4. Run the cells in order from a fresh kernel.
5. Confirm the output counts reconcile to 50,000 before exporting or loading data into SQL.
6. Export flags as 1/0 for MySQL import and check the CSV-to-table column mappings.
Preserve the original source CSV when rerunning the workflow. A dependency file and documented package versions are planned for reproducibility.
Next steps
- Add rejected and review CSVs and the exported quality summary.
- Complete and publish MySQL DDL, staging-load, validation and audit scripts.
- Build the four SQL reporting views.
- Develop the Power BI report and metric dictionary.
- Publish quantified business findings, recommendations and reconciliation evidence.

