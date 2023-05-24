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
Initial data import encountered problems with census_tract_data for 2015 or 2017, likely due to censustract/tractid values counted as an integer rather than a uuid or string. Quick solve by reassigning id fields to BIGINT

There's a significant number of counties with multiple null values in csv source files. 0 values in some cases, such as the case for Yuma, Arizona listed as having a population of 0 in 2015 for several of the census tracts. This is imporant to keep in mind and will require consideration for aggregate functions.  

1291 uploaded of some odd 74002 in the initial insert of rows represents unaccaptable data loss.

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

We can reasonably assume, but not be certain that these columns are analogous to each other, but without the ability to confirm from source we must err on the side of caution on not rely on them for any analytic purpose until the underlying disparity can be addressed. This is probably a non-issue given the fact that there do not appear to be any cases where, for example, a censustract value exists without a cooresponding tractid value.

ex:
```
SELECT * FROM acs2017_census_tract_data a
LEFT JOIN acs2015_census_tract_data b
ON a.tractid = b.censustract
WHERE b.censustract IS NULL;
```
returns no values, nor does it's inversion.

Handleing of special characters in source csv files uneven at best, if county level analysis neccesarry a detailed CASE statement ought to be used to correct. Does not seem to be an issue once data is inserted

Example:
AÃ±asco or MayagÃ¼ez counties, Puerto Rico

# General Observations
covid19_state seems to be a fairly comprehensive, if general summation of census data and Covid 19 information (testing, deaths, etc)

A quick look at the data therin can be used to make some interesting observations such as pollution levels per state. The following examples are three or more standard deviations (1.457) from the mean pollution score (7.413)
*state* |  *pollution*
--- | --- 
California | 12.8

California in this case is a true outlier either as a result of genuinely high pollution levels, or due to how this data was collected. Some cursory research suggests that the former is the case.

This example is derived from a view combining the 2017 county data and covid 19 state data. Saved here as state_summary_cp.

*state* |  *avg_mean_commute*
--- | --- 
Alaska | 11.2

```
CREATE VIEW state_summary_cp AS
SELECT
    c.state,
    avg(c.childpoverty) AS avg_child_poverty,
    avg(c.meancommute) AS avg_mean_commute,
    s.population,
    s.infected
FROM acs2017_county_data c
JOIN covid_19_state s ON c.state = s.state
GROUP BY c.state, s.population, s.infected;
```
# Population Comparison

A helpful view for side-by-side comparisons
```
CREATE VIEW state_population_comparison AS
SELECT state,
       SUM(population) AS covid_19_state_population,
       (SELECT SUM(totalpop) FROM acs2015_county_data WHERE acs2015_county_data.state = covid_19_state.state) AS county_population_2015,
       (SELECT SUM(totalpop) FROM acs2017_county_data WHERE acs2017_county_data.state = covid_19_state.state) AS county_population_2017,
       (SELECT SUM(totalpop) FROM acs2015_census_tract_data WHERE acs2015_census_tract_data.state = covid_19_state.state) AS tract_population_2015,
       (SELECT SUM(totalpop) FROM acs2017_census_tract_data WHERE acs2017_census_tract_data.state = covid_19_state.state) AS tract_population_2017
FROM covid_19_state
GROUP BY state;
```

It seems that the population sums grouped by state from both the tracts and county sources are equal in all cases and hence can be thought of as two, rather than four points of comparison. The following query returns no results. 

```
WITH cte AS (
  SELECT state,
         (county_population_2015 + tract_population_2015)/2 AS avg_population_2015,
         (county_population_2017 + tract_population_2017)/2 AS avg_population_2017,
         county_population_2015,
         county_population_2017
  FROM state_population_comparison
)
SELECT *
FROM cte
WHERE avg_population_2015 <> county_population_2015
   OR avg_population_2017 <> county_population_2017;
```
FROM queries like the following, we can see general trends like population growth (or loss) over time
```
WITH cte AS (
  SELECT state,
         (county_population_2015) AS population_2015,
         (county_population_2017) AS population_2017,
         (covid_19_state_population) as population_2020
  FROM state_population_comparison
)
SELECT state,
       (population_2017 - population_2015) AS population_change_2017,
       (population_2020 - population_2017) AS population_change_2020,
       (population_2020 - population_2015) AS total_population_change
FROM cte;
```
