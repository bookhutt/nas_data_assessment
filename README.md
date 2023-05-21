# nas_data_assessment
Engineering Assessment - Data Import , Analysis, &amp; Comparison

# Data
1.  <a href="https://www.kaggle.com/datasets/muonneutrino/us-census-demographic-data?resource=download" target="_blank">US Census Demographic Data</a>
2.  <a href="https://www.kaggle.com/datasets/nightranger77/covid19-state-data?select=COVID19_state.csv" target="_blank">COVID-19 State Data</a>

# Goals
- import data into a database (postgres)
- ensure data quality and record errors
- Compare and explore data with an emphasis on state population estimates
- Identify differences between data sets
- document process
- Create visualization and written materials for presentation

# Error tracking
Initial data import encountered problems with census_tract_data for 2015 or 2017, likely due to null values in csv source files. 1291 of some odd 74002 rows represents unaccaptable data loss.

# Data Quality Issues
for census_tract_data there's a couple column name issues of concern

--- | --- | ---
*ordinal_position* |  *a.column_name* | *a.column_name*
1 | censustract | tractid
13 | citizen | votingagecitizen

Markdown | Less | Pretty
--- | --- | ---
*Still* | `renders` | **nicely**
1 | 2 | 3

We can reasonably assume, but not be certain that these columns are analogous to eachother, but without the ability to confirm from source we must err on the side of caution on not rely on them for any analytic purpose.
