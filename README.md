# Sql_scripting_for_projects 
## Airline Industry Studies for Google Data Analytics Capstone: Selecting, Filtering, Joining, Casting, When, Where, Aliasing, etc
## BigQuery for engine; data sources include "T100 Carrier by Domestic Segment" data from Bureau of Transportation Statistics and Aravind Ram Nathan's "List of US Airports"
## READ “T-100 Traffic Reporting Guide” before continuing

### Examine limited number of carriers for study base on sum of Departures Performed, count of Unique Carrier names select by segments served, or whatever metric is of interest
-- examine top 25 carriers by count of segment in original datafile
-- perform for each year of study to determine carriers for study then choose by metric performance over years of dat studied
SELECT 
    UNIQUE_CARRIER_NAME
FROM `database.Airline_Data_DupChk.Unclean_Carrier_2019` 
  GROUP BY UNIQUE_CARRIER_NAME
  ORDER BY 
   COUNT (UNIQUE_CARRIER_NAME) DESC
LIMIT 25

### Shrink data set to necessary data for analysis and begin cleaning
--create simplified table related to specific business question from original data and limited data set to subset 
--of table columns and carriers, change to lowercase, change troublesome names
SELECT
  MONTH AS num_month,
    YEAR AS num_year,
    ORIGIN AS origin,
    DEST AS dest,
    UNIQUE_CARRIER_NAME AS U_carrier_name,
    DEPARTURES_PERFORMED AS num_flights,
    RAMP_TO_RAMP AS ramp_to_ramp,
    AIR_TIME AS air_time
  FROM `database.Airline_Data_DupChk.Unclean_Carrier_2022`
  WHERE UNIQUE_CARRIER_NAME LIKE "Chosen1%"
    OR UNIQUE_CARRIER_NAME LIKE "Chosen2%"
    OR UNIQUE_CARRIER_NAME LIKE "Chosen3%"
    OR UNIQUE_CARRIER_NAME LIKE "Chosen4%"
    OR UNIQUE_CARRIER_NAME LIKE "Chosen5%"
    OR UNIQUE_CARRIER_NAME LIKE "Chosen6%"
###

###  Preserve needed columns but mask Airline names for posting studies since having to use an analog for tarmac delay with questionable accuracy.  Test with internal data if ever an option.
SELECT --create simplified table related to specific business question from original data and mask limited data set to study
    num_month,
    num_year,
    origin,
    dest,
    num_flights,
    ramp_to_ramp,
    air_time, 
      CASE 
      WHEN U_carrier_name = 'Chosen1' THEN 'Airline_A'
      WHEN U_carrier_name = 'Chosen3' THEN 'Airline_E'
      WHEN U_carrier_name = 'Chosen4' THEN 'Airline_C'
      WHEN U_carrier_name = 'Chosen5' THEN 'Airline_F'
      WHEN U_carrier_name = 'Chosen6' THEN 'Airline_D'
      ELSE 'Airline_2'
    END AS U_mask  
  FROM `database.Airline_Data_DupChk.Unclean_2022_ToMask`

### Finish Cleaning, Calc Date to different format to work better for other viz; Calculate analog lag indicator, concat route names from origin and dest
--Be sure to edit date in 2 places below, filename and in concat call

SELECT 
-- first get month into a date with fake day of the month but with datatype= date for scatterplots, calc tarmac lag and routenames to get date as as datatype of date for scatterplots over total time period
  DATE(CONCAT('2022-', LPAD(CAST(num_month AS STRING), 2, '0'), '-01')) AS year_mo_fake01,
  num_month,  -- for scatterplots for seasonal trends
  num_year, -- for scatterplots or bar charts for yearly trends

    --concatenating origin and destination to make route names
    origin,
    dest,
    CONCAT(origin,"_",dest) AS route,
    
    U_mask,--masked airline name
    
    --calculate analog for tarmac that is actually a psuedo lag time due to 
    --USDOT T100 reporting by summing time for each segment by month
    num_flights,
    ramp_to_ramp,
    air_time,
    ((ramp_to_ramp - air_time)/num_flights) AS tarmac_lag_indicator

  FROM `database.Airline_Data_DupChk.Masked22`

  WHERE -- AVOID calculating when time data is not populated or if unrealistically short or to avoid divide by 0
    air_time IS NOT NULL 
    AND ramp_to_ramp IS NOT NULL
    AND air_time >0
    AND ramp_to_ramp >0
    AND num_flights >0

### Vertically join data from tables covering different years
SELECT *
FROM `database.Airline_Data_DupChk.Clean19`
UNION ALL

SELECT *
FROM `database.Airline_Data_DupChk.Clean20`
UNION ALL

SELECT *
FROM `database.Airline_Data_DupChk.Clean21`
UNION ALL

SELECT *
FROM `database.Airline_Data_DupChk.Clean22`

### Join lat longs from a database with lat longs and airport names to carrier segment data with airport names to creat route paths in viz from origin to dest; 2 queries or if careful migh be able to double join; recommend copy past of databse.table.column as I repeatedly had difficulties with that for some reason; some aliasing to simplify

  SELECT 
  a.*,
  b.LATITUDE AS origin_lat,
  b.LONGITUDE AS origin_long
FROM 
  `database.Airline_Data_DupChk.VertJoined_NoLatLongs` AS a
JOIN 
  `database.Airline_Data_DupChk.Airport_Locs` AS b
ON 
  a.origin = b.IATA

### 2nd time through for destination

  SELECT 
  a.*,
  b.LATITUDE AS Dest_Lat,
  b.LONGITUDE AS Dest_Long
FROM 
  database.Airline_Data_DupChk.DupChkCleanCalcOriginLL AS a
JOIN 
  `database.Airline_Data_DupChk.Airport_Locs` AS b
ON 
  a.dest = b.IATA


