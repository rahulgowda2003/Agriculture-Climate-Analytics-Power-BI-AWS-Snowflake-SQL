# Agriculture Climate Analytics – Power BI | AWS | Snowflake | SQL

A multi-page Power BI dashboard analyzing Temperature, Humidity, Rainfall, and Crop Yield across years, seasons, locations, and crop types.

This report was created using AWS, Snowflake and Power BI.

The data used to build the report is not accurate, its a sample data used to make the report.

## Problem Statement

This dashboard provides a comprehensive climate-based analysis to support agricultural decision-making.

It helps users understand:

- How temperature, rainfall, and humidity vary across years, seasons, crops, and locations
- Which agricultural regions receive the highest rainfall
- How climate patterns impact crop yield
- Which crops thrive under certain climate conditions
- Season-wise climate distribution (Kharif, Rabi, Zaid)

This dashboard is useful for farmers, agronomists, researchers, policymakers, and agriculture departments.

### Steps Followed

- step 1 : Created a bucket in Amazon S3 named "powerb", then uploaded the data file which was in csv type to the S3 bucket.
- step 2 : IAM role created in AWS named "powerbi.role" to establish connection between AWS and Snowflake, copy the ARN in the role i.e. arn:aws:iam::825765422200:role/powerbi.role
- step 3 : In snowflake created a new integration.sql file, write query for the intergration object:
  

   CREATE OR REPLACE STORAGE INTEGRATION PBI_Integration  ##PBI_Integration is the name of the integration object
  
  TYPE = EXTERNAL_STAGE
  
  STORAGE_PROVIDER = 'S3'
  
  ENABLED = TRUE
  
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::825765422200:role/powerbi.role'   ##paste the copied ARN from IAM role
  
  STORAGE_ALLOWED_LOCATIONS = ('s3://powerb/')  ##change the name of the S3 bucket to mentioned in step 1
  
  COMMENT = 'Optional Comment'

- step 4 : After creating integratoin object, run query to get the description of the integration object, after the execution of the query copy the code representing in "STORAGE_AWS_IAM_USER_ARN" and "STORAGE_AWS_EXTERNAL_ID" respectively.

  desc integration PBI_Integration;

- step 5 : Open the "powerbi.role" role in IAM, click on Trust relationships followed by edit trust policy, paste the copied ARN code in "AWS" : "   " and External_ID code in "sts:ExternalID" : "   ", then update the policy. Hence integrating the AWS with Snowflake.
- step 6 : Loaded the data from Amamzon S3 bucket "powerb" to snowfalke by creating database "PowerBI", schema "PBI_Data" and table "PBI_Dataset".

CREATE database PowerBI;

create schema PBI_Data;

create table PBI_Dataset 
(
Year int,	Location string,	Area	int,
Rainfall	float, Temperature	float, Soil_type string,
Irrigation	string, yeilds	int,Humidity	float,
Crops	string,price	int,Season string
);

create stage PowerBI.PBI_Data.pbi_stage   

url = 's3://powerb'

storage_integration = PBI_Integration

 
copy into PBI_Dataset 

from @pbi_stage

file_format = (type=csv field_delimiter=',' skip_header=1 )

on_error = 'continue'

##to load data from S3 bucket to snowflake

list @pbi_stage ##to confirm the loaded data

- step 7 : Transforming the data in Snowflake, increasing "rainfall" column by 10%, decreasing "area" column by 10% and adding new column "year_group" to get the new category of the year.

create table agriculture as

select * from pbi_dataset;


update agriculture

set rainfall = 1.1*rainfall;


update agriculture

set area = 0.9*area;


//Year 2004 & 2009 - Y1

//Year 2010 & 2015 - Y2

//Year 2016 & 2019 - Y3


ALTER TABLE Agriculture

add Year_Group String;


select * from agriculture;

//1st update

update agriculture

set year_group = 'Y1'

where year >=2004 and year<=2009


//2nd update

update agriculture

set year_group = 'Y2'

where year >=2010 and year<=2015



//3rd Update

update agriculture

set year_group = 'Y3'

where year >=2016 and year<=2019

- step 8 : It was observed that in LOCATION column "Banglore" and "Davangere" where having blank values for "SOIL_TYPE", it was filled with "Red" and "Red Sandy" respectively by wrting query.

  //1st Update
  
  upadate agriculture
  
  set SOIL_TYPE = 'Red'
  
  where LOCATION = 'Banglore'
  

  //2nd Update
  
  upadate agriculture
  
  set SOIL_TYPE = 'Red Sandy'
  
  where LOCATION = 'Davangere' and CROPS in ('Cashew', 'Coconut', 'Groundnut')
  

  //3rd Update
  
  upadate agriculture
  
  set SOIL_TYPE = 'Black'
  
  where LOCATION = 'Davangere' and CROPS = 'Blackgram'

- step 9 : Adding "rainfall_group" column and importing data into Power BI Desktop.

  Opened power query editor and in view tab, under Data preview section, check "column distribution", "column quality" and "column profile" options.

  By default, column profile will be opened only for 1000 rows, so you will need to select "column profiling based on entire dataset".

  It was observed that in none of the columns errors and no empty values were present.

  //Rainfall_Groups
  
//Min 255 Max 4103

//rainfall 255 & 1200 - Low

//rainfall 1200 2800 - Medium

//Rainfall 2800 & 4103 - High


alter table agriculture

add rainfall_groups string;


//1st Update

update agriculture

set rainfall_groups = 'Low'

where rainfall>=255 and rainfall<1200


//2nd update

update agriculture

set rainfall_groups = 'Medium'

where rainfall >=1200 and rainfall<2800


//3rd update

update agriculture

set rainfall_groups='High'

where rainfall >=2800

- step 10 : By looking at the RAINFALL column, the values were not accurate, using DAX created a new column named "Actual Rainfall" to adjust the values by dividing it by 3.5 to make it as realistic as possible. Using the Actual Rainfall column make the "Rainfall Analysis" page and the four stacked bar chart.

  Actual Rainfall = 
DIVIDE(AGRICULTURE[RAINFALL],3.5,BLANK())

- step 10 : Creating four page report for the better understanding of the data by extracting the valueable insights namely "Rainfall Analysis", "Temperature Analysis", "Humidity  Analysis" and "Yield Analysis" respectively.
- step 11 : Added four charts for each report.

  a) Rainfall Analysis:
  
  - Average Rainfall by Year
  - Average Rainfall by Season
  - Average Rainfall by Crops
  - Average Rainfall by Location

  b) Temperature Analysis:
  
  - Average Temperature by Year
  - Average Temperature by Season
  - Average Temperature by Crops
  - Average Temperature Ranking by Location

  c) Humidity Analysis

  - Average Humidity by Year
  - Average Humidity by Season
  - Average Humidity by Crops
  - Average Humidity by Location

  d) Yield Analysis

  - Average Yield by Year
  - Average Yield by Season
  - Average Yield by Crops
  - Average Yield by Location


# Report Snapshot (power BI Desktop)

![Dashboard_upload](https://github.com/rahulgowda2003/Agriculture-Analysis-Report/blob/main/Humidity%20Analysis%20Screenshot.png)


![Dashboard_upload](https://github.com/rahulgowda2003/Agriculture-Analysis-Report/blob/main/Rainfall%20Anaysis%20Screenshot.png)


![Dashboard_upload](https://github.com/rahulgowda2003/Agriculture-Analysis-Report/blob/main/Temperature%20Analysis%20Screenshot.png)


![Dashboard_upload](https://github.com/rahulgowda2003/Agriculture-Analysis-Report/blob/main/Yield%20Analysis%20Screenshot.png)



# Insights

a) Dashboard Pages

This report contains four detailed analytical pages:

1] Temperature Analysis
2] Humidity Analysis
3] Rainfall Analysis
4] Yield Analysis

Each page includes breakdowns by Year, Season, Crop, and Location.

b) Temperature Analysis

1] Average Temperature by Year

- Shows temperature trends from 2007–2017
- Yearly temperature ranges between 69°F to 73°F
- Warmer years: 2016, 2017 (73°F)
- Cooler years: 2012, 2007 (69°F)

2] Average Temperature by Season

Season Average Temperature

- Kharif	72°F
- Zaid	72°F
- Rabi	61°F

The Rabi season is significantly cooler than Kharif and Zaid.

3] Average Temperature by Crop

Top crops by temperature requirement:

- Crop Average Temperature
- Ginger	79°F
- Tea	74°F
- Cashew	74°F
- Blackgram	73°F
- Arecanut	70°F

Cooler climate crops include Cardamom, Pepper, Groundnut, Coffee, Coconut, etc.

4] Average Temperature Ranking by Location

Location Average Temperature

- Bangalore
- Davangere
- Raichur
- Gulbarga
- Mandikeri

Bangalore and Davangere show the highest aggregated temperature measures.

“Aggregated temperature measure” is not a standard scientific term — it simply refers to how the temperature values are combined or summarized in your Power BI visual.

c) Humidity Analysis

1] Average Humidity by Year

- Humidity remains fairly stable across years.
- Most years show 56% average humidity.

2] Average Humidity by Season

Season	Average Humidity

- Rabi	56%
- Zaid	56%
- Kharif	56%

Humidity is consistent across all agricultural seasons.

3] Average Humidity by Crop

Crop	Average Humidity

- Cotton 56%
- Pepper 56%
- Coffee 56%
- Blackgram 56%
- Paddy 56%
- Coconut 56%
- Arecanut 55%
- Cardamum 55%
- Tea 55%
- Ginger 55%
- Cashew 55%
- Groundnut 55%
- Cocoa 55%

The average Humidity by crop is around 55.5%.

4] Average Humidity by Location

All locations show humidity values between 55–56%, indicating stable weather patterns across regions.

d) Rainfall Analysis

1] Average Rainfall by Year

- Ranges between 861 mm to 983 mm
- Wettest year: 2018 (983 mm)
- Driest year: 2006 (861 mm)

2] Average Rainfall by Season

Season	Average Rainfall

- Rabi	976 mm
- Kharif	973 mm
- Zaid	965 mm

Season-wise rainfall remains constant.

3] Average Rainfall by Crops

Crop	Average Rainfall

- Paddy	1085 mm
- Arecanut	1042 mm
- Cardamom	1032 mm
- Cashew	1006 mm
- Groundnut	994 mm
- Cocoa 989 mm
- Tea 986 mm
- Coffee 979 mm
- Blackgram 975 mm
- Ginger 971 mm
- Pepper 964 mm
- Coconut 942 mm
- Cotton 906 mm

Coconut has the lowest rainfall requirement 906 mm

4] Average Rainfall by Location

Location	Average Rainfall

- Bangalore	1208 mm
- Raichur	1014 mm
- Kasaragodu 983 mm
- Mangalore	997 mm
- Chikmangaluru 983 mm
- Hassan 971 mm
- Gulbarga 969 mm
- Kodagu 950 mm
- Davangere 943 mm
- Madikeri 936 mm
- Mysuru 905 mm

Bangalore receives the highest average rainfall and Mysuru with lowest average rainfall.

e) Yield Analysis

1] Average Yield by Year

Top Yield Years:

Year	Average Yield

- 2008	28.7K Kg
- 2014	27.8K Kg
- 2018	27.2K Kg

Lowest Yield : 2006 : 22.3K Kg

2] Average Yield by Season

Season	Avg Yield

- Rabi	24.9K Kg
- Zaid	22.0K Kg
- Kharif	20.2K Kg

Rabi has the highest crop productivity.

3] Average Yield by Crop

Crop	Avg Yield

- Cotton	51K Kg
- Coconut	34K Kg
- Ginger	26K Kg
- Tea	23K Kg
- Blackgram	16K Kg

Bottom yield crop : Cashew : 3K Kg

4] Average Yield by Location

Top Yield Locations:

Location	Average Yield

- Kodagu	28.7K Kg
- Mysuru	27.6K Kg
- Madikeri	24.9K Kg
- Kasaragodu	24.4K Kg

Lowest Yield : Davangere : 11.8K Kg
