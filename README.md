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
Initial data import encountered problems with census_tract_data for 2015 or 2017, likely due to censustract/tractid values counted as an integer rather than a uuid or string. There's a significant number of counties with multiple null values in csv source files. 0 values in some cases, such as the case for Yuma, Arizona listed as having a population of 0 in 2015 for several of the census tracts. This is imporant to keep in mind and will require consideration for aggregate functions. 1291 uploaded of some odd 74002 in the initial insert of rows represents unaccaptable data loss.

# Data Quality Issues
for census_tract_data there's a couple column name issues of concern

census_tract_data
*ordinal_position* |  *a.column_name* | *a.column_name*
--- | --- | ---
1 | censustract | tractid
13 | citizen | votingagecitizen

county_data
*ordinal_position* |  *a.column_name* | *a.column_name*
--- | --- | ---
1 | censusid | countyid
13 | citizen | votingagecitizen

We can reasonably assume, but not be certain that these columns are analogous to each other, but without the ability to confirm from source we must err on the side of caution on not rely on them for any analytic purpose until the underlying disparity can be addressed.

Handleing of special characters in source csv files uneven at best, if county level analysis neccesarry a detailed CASE statement ought to be used to correct. Does not seem to be an issue once data is inserted

Example:
AÃ±asco or MayagÃ¼ez counties, Puerto Rico
