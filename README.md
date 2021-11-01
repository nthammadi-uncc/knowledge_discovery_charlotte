# Discovering Knowledge for the City of Charlotte, NC
This project is a course requirement for DSBA-6162, Knowledge Discovery in Databases, at UNCC for the Fall semester, 2021.

**Team Members:**

###### Naomi Thammadi
###### Kevin Gharavizadeh
###### Imad Ahmad
###### Dustin Ballentine


## Project Motivation
The city of Charlotte has, like many cities, made large volumes of data open to the public in an online repository. This open data provides substantial potential for the city to benefit from knowledge discovery and insights generated by members of the public, such as ourselves. Service requests dialed to 311 represent a large opportunity to delve into the needs of the population and potentially extract useful trends that could allow the city to improve its service to its citizens. The goal of this project is to explore and uncover exactly those trends in the hope that the knowledge we discover may be used by the city to improve the quality of life of our families, friends, and neighbors.
## Research Question
Do different areas within the city of Charlotte experience higher recurrence of any type of 311 service request than other areas, and do those areas correlate with red-line districts or other known demographic or socioeconomic profiles?
## Data Resources
<ol>
<li> 311 Service Requests. </li> Data was retrieved from the City of Charlotte's
<a href="https://data.charlottenc.gov/datasets/charlotte::service-requests-311/about"> Open Data Portal </a>
on 30 September, 2021. The data set is a collection of the public records requests received by the City of Charlotte from 2017 through March, 2020. It contains multiple descriptive fields related to service request entry, including request type, location (street, lat/long, etc.), date, city department responsible for the request, and point of entry.
<li> Census Household Income Data </li> Census data was taken from <a href="https://data.census.gov/cedsci/table?d=ACS%205-Year%20Estimates%20Data%20Profiles&hidePreview=false&vintage=2018&layer=zcta5&tid=ACSDP5Y2019.DP03">the datasets of census.gov</a>. This data set is a collection of household income statistics, Employment statistics, Details about Families and Non-families, People under poverty line and People commuting to work. The dataset consists of 932 rows and 551 features.
<li> Census Tract Ids and Zip Codes </li> The source of Census Tract Id to Zip code conversion has been taken from <a href="https://www.huduser.gov/portal/datasets/usps_crosswalk.html#data">the dataset portal of huduser.gov</a>. This conversion was necessary to link the census data to 311-Service Requests data. The dataset has 5301 rows and 6 features.
<li> Violent Crime Data </li> Data was retrieved from the City of Charlotte's <a href="https://data.charlottenc.gov/datasets/charlotte::violent-crime-demographics/about"> Open Data Portal </a> on 30 September, 2021. The dataset is a collection of victims of violence, mainly domestic violence, that occured from 2015-2021. The dataset has an "Attribute Count" which is the number of people(victim/offender) involved in a type of violence crime. We use this as a Violence score to link to our Service Requests Dataset

</ol>

## Python libraries to install
Before running any of the Jupyter notebooks mentioned in the repository, the below libraries need to be installed in the local machine using Anaconda:

```
    pip install missingno
    pip install geopandas
    pip install contextily
```

## Data Preprocessing
### 311 Service Requests
From the initial 311 service requests dataset, we dropped 15 features which were either redundant or contained too many missing values. We also dropped just over 99,000 records that contained null values for address, zipcode, and LAT/LON coordinates, since those are the fields of primary interest to our analysis, we have no reasonable way to impute those fields, and that represented only about 5.5% of the data. Finally, we dropped an additional 8,279 records which pertained to COVID-19 protocols, Solid Waste Services administrative functions such as a data upload, and 8 additional call types which had fewer than 10 total occurrences. <br>
Moving on with the 14 remaining features, we transformed two features and engineered another five as discussed below:
##### Transformations
<ol>
  <li> RECEIVED_DATE - Simple datatype transformation from object to datetime </li>
  <li> FULL_ADDRESS - Replaced with a rank-ordered index, with 1 representing the address with the highest call volume, ascending in order inversely to the volume of 311 calls. </li>
</ol>

##### Engineered Features
<ol>
<li> REQUEST_CAT - The original data contained 165 separate types of 311 service calls. Many of the categories contained only a handful of records, and many were clearly related to other call types. We therefore decided to bin the call types into a new feature called request categories. Twenty-three of the types were imported directly as categories--mainly the largest categories or those without a clear connection to other types. The remaining call types were binned into 16 categories, yielding 39 final categories. The breakdown of which types were binned into which categories can be seen in the <a href="https://github.com/DABallentine/knowledge_discovery_charlotte/blob/main/Jupiter%20Notebooks/EDA_and_Preprocessing.ipynb"> EDA and preprocessing notebook </a>.  </li>
  <li> RECEIVED_YEAR - Parsed out the year from the DATE_RECEIVED feature </li>
  <li> RECEIVED_MONTH - Parsed out the month from the DATE_RECEIVED feature </li>
  <li> SEASON - Parsed out the quarter from the DATE_RECEIVED feature, which generally differs from the official season of the year by only 1 week </li>
  <li> TOTAL_CALLS - Summed the total number of calls from each address </li>
  <li> COL_MERGE - Combined RECEIVED_YEAR, RECEIVED_MONTH and NEIGHBORHOOD_PROFILE_AREA for merging 311-Service Requests with Violent Crime dataset </li>
</ol>

### Census Income Data
<ol>
  <li>GEO_ID - The original data from the GEO_ID column is in the format 1400000US37119000100. This data has been parsed to only include the 11-digit Tract Id after 'US'.</li>
  <li>INCOME AND BENEFITS_Households with Income < $15,000 - Feature created by adding up the Income and Benefits of households < $10,000 and between $10,000 to $14,999</li>
  <li>INCOME AND BENEFITS_Households with Income > $150,000 - Feature created by adding up the Income and Benefits of households between $150,000 to $199,999 and > $200,000</li>
  <li>INCOME AND BENEFITS_Family Income < $15,000 - Feature created by adding up the Income and Benefits of families < $10,000 and between $10,000 to $14,999</li>
  <li>INCOME AND BENEFITS_Family Income > $150,000 - Feature created by adding up the Income and Benefits of families between $150,000 to $199,999 and > $200,000</li>
  <li>COMMUTING TO WORK_By Car - Feature created by adding up the commuting to work alone in car and commuting to work by carpool</li>
  <li>ZIP_YEAR - Combined ZIP_CODE and YEAR column to map the data accordingly to the Service Requests dataset</li>
</ol>
The dataset consisted of 551 features that were reduced to 44 features as part of the data preprocessing which can be seen in the <a href="https://github.com/DABallentine/knowledge_discovery_charlotte/blob/main/Jupiter%20Notebooks/EDA_and_Preprocessing.ipynb"> EDA and preprocessing notebook </a>.  

### Violent Crime Data
<ol>
  <li>COL_MERGE - Combined CALENDAR_YEAR, CALENDAR_MONTH and GEOGRAPHY_ID for merging Violent Crime dataset with 311-Service Requests</li>
</ol>
## Data Understanding and Exploration
### Overview
##### Categories
Non-Recyclable Items make up over half of the requests and Recyclable Items comprise another 13%. Binning into the 39 categories created a much more even balance through categories 3-16, with each comprising from 7% down to 1% of total requests, as opposed to the inital proportions which were mostly much less than 1%. Note that the scale on the graph below is logarithmic to compensate for the imbalance between categories.
![image](https://user-images.githubusercontent.com/78170609/139606719-8fd9a565-d720-4926-acdf-b705106adb1b.png)

### Time Series and Seasonality
Plotting the requests over various time intervals seems to show a steady increase in call volume by year (2016 and 2021 are only partially represented in the data), with 2020 perhaps breaking the trend due to the COVID-19 pandemic. The summer months have the highest volume of requests, particularly in June and July, and there exists a clear trend for the most calls on Monday each week, and then tapering off as the week goes on. Daily, calls are fairly uncommon in the mornings, which was somewhat surprising, peaking in the early afternoon and then decreasing drastically after dinner time.
![image](https://user-images.githubusercontent.com/78170609/139607530-862f1398-e667-4266-9db6-6e5bc99e65fd.png)

We do observe some differences by category when plotting over different timeframes. For example, plotting by season across all categories reveals several categories with peaks during the winter months. Though not as extreme, each of the other seasons does also have one category somewhat above the even 25% split.
![image](https://user-images.githubusercontent.com/78170609/139609372-6f7c2050-60f9-4b0e-9f53-381fa8bd7090.png)
![image](https://user-images.githubusercontent.com/78170609/139610261-0550828e-91cf-4047-b74f-985c6873624b.png)

The category with the greatest disparity, SW ONLY - DOOR HANGER LEFT, seems to represent a note left on someone's door by the Solid Waste department. Possible reasons for this trend include a large proportion of the population traveling during the winter months. A closer look reveals that the peak actually occurs in February and March, so it does not seem to correlate to the Christmas or New Year holidays. Without additional data going deeper into the nature of this category, it is difficult to surmise what the trend may represent. FIELD OBSERVED PROBLEMs, almost 50% of which occur during Winter, also occur at a similarly high rate into April, totaling 73% of the year's records just in the first 4 months. This trend could provide a starting point for future research. What are the nature of those field-observed problems? Are there simply more city employees driving around during the early months of the year, or is there some seasonal effect pertaining to winter weather perhaps? We could perhaps answer those questions with the right data added to this analysis.
Several of the other categories align with common sense. For example, in Spring people return to the city parks, and issues which have cropped up unnoticed over the Winter months are identified and reported. Likewise, as the weather warms and the plants grow, people trim hedges, clean out and re-landscape old areas, and thus have more yard waste than at other times during the year.
![image](https://user-images.githubusercontent.com/78170609/139609399-f1cf45e5-3be6-4aa8-8609-2950fb97cd0b.png)

The monthly top categories by proportion reveal a few other imbalances across categories. 

### Customer Observations
By using each unique address as a "customer", we see one address that is 75% higher than the next highest address by call volume, and thereafter a fairly steady decrease across all addresses. The number one address corresponds to the Sharon Lakes Comdominiums complex located in Starmount Forest, in south Charlotte. The next few addresses also appear to be townhome developments or apartment complexes at various locations across the city. However, the #4 address is in close proximity to the #1 address, which may represent an area of high call volume worth investigating.
![image](https://user-images.githubusercontent.com/78170609/139608857-f3ceb422-2855-4372-ba61-d9d996cdb6a6.png)

### Geographic Analysis
Since our 311 Call data contained geospatial data, we decided to map a few data points onto the Charlotte map. A few variables we have explored are Recycle and Non Recycle requests (comparing 2016, 2018, 2020). 
The Geopandas notebook can be found here: <a href="https://github.com/DABallentine/knowledge_discovery_charlotte/blob/main/EDA_and_Preprocessing_with_MAPS.ipynb"> EDA and preprocessing with MAPS notebook </a>

## Data Preparation for Modeling
## Modeling
## Evaluation
## Results
## Future Work
## Possible future work may include:
<ol>
  <li>Explore correlations between weather and 311 service requests</li>
  <li>Predict high call volume days or periods to alleviate workload and wait time concerns</li>
  <li>Explore correlations between location and specific service requests' response times</li>
</ol>
