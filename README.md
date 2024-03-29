# Thailand Domestic Tourism 2019 to 2023

![pexels-pixabay-460376](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/cb792545-d03a-48e2-87d9-7bdd27921f08)
_Photo by Pixabay from [Pexels](https://www.pexels.com/photo/gray-pointed-concrete-structure-460376/)_

## Introduction
This is a data analysis on Thailand Domestic Tourism using a [dataset](https://www.kaggle.com/datasets/thaweewatboy/thailand-domestic-tourism-statistics) from **January 2019 to February 2023**. I chose to explore this data as I had recently traveled back to Thailand and was also there right before the Covid lockdowns. I am curious to see the changes in Thai domestic tourism from prior to the Covid outbreak to post Covid outbreak. There are four aspects in this dataset that will be analysed for trends and insights including total revenue, number of tourists, number of occupied hotel rooms and occupancy rate. The insights and recommendations will be based on the results of my analysis combined with my general knowledge as a Thai person.

_**Note:** Since the dataset only contains 2 months for the year 2023, the results of this analysis will not show the full picture for 2023 but it is still a great indication for what is likely to happen and a useful source for understanding the trends and changes between 2019 and 2023._

### My analytical workflow
1. Import and prepare data
2. Process and clean my data
3. Calculate and analyse my data
4. Findings and recommendations

## Import and prepare data
First, I will download this [dataset](https://www.kaggle.com/datasets/thaweewatboy/thailand-domestic-tourism-statistics) from Kaggle and import it into MySQL to view the dataset. This dataset was sourced from raw data provided by the [Official Ministry of Tourism and Sports Statistics](http://www.mots.go.th/news/category/411). This dataset is from January 2019 to February 2023. 

`region_thai` and `province_thai` columns (the Thai version of the province and region columns) will be dropped as they are not required in this analysis.

![unnamed](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/82823d4a-d0d9-4772-ab37-c7aa2874fa1c)

### Columns in the dataset
* `province_eng`: the english version of the name of provinces in Thailand.

* `region_eng`: the english version of the name of regions in Thailand.

* `variable`: *8 different types* of data being recorded. This includes the following:

  `no_tourist_all`: the number of domestic tourists that visited the province.
  
  `no_tourist_foreign`: the number of foreign tourists that visited the province.

  `no_tourist_thai`: the number of Thai tourists that visited the province.

  `no_tourist_stay`: the number of occupied hotel rooms in the province.

  `ratio_tourist_stay`: the percentage of occupied travel accomodation in the province.

  `revenue_all`: the revenue generated by the tourism industry, in Thai Baht (฿).

  `revenue_foreign`: the revenue generated from foreign tourists in the province, in in Thai Baht (฿).

  `revenue_thai`: the revenue generated from Thai tourists in the province, in in Thai Baht (฿).

## Process and clean my data
As the `variable` column has 8 unique variables, tidying up data by creating new columns with different variables will make it easier for analysis and understading the dataset. I will also create an `id` column and a column for the country name (solely because without the country name, I have encontoured an issue in filling in color for a map while using Tableau before).

### Create a new table
First, I will create a new table using the below query.
```
/* Create a new table */
CREATE TABLE thailand_domestic_tourism (
	id INT NOT NULL AUTO_INCREMENT,
	date DATE,
	province VARCHAR(255),
	region VARCHAR(255),
	country VARCHAR(255),
	occupancy_rate FLOAT,
	num_occupied_room FLOAT,
	num_tourist_all FLOAT,
	num_tourist_thai FLOAT,
	num_tourist_foreign FLOAT,
	revenue_all FLOAT,
	revenue_thai FLOAT,
	revenue_foreign FLOAT,
	PRIMARY KEY (id)
);
```

After creating the new table, I will insert the data for the following columns: `date`, `province`, `region`, `country` and `occupancy_rate`. I will not insert the data for other columns at this stage.
```
/* Insert data */
INSERT INTO thailand_domestic_tourism (date, province, region, country, occupancy_rate)
SELECT
	STR_TO_DATE(CONVERT(date using utf8), '%Y-%m-%d'),
	TRIM(province_eng),
	TRIM(region_eng),
	"thailand",
	value
FROM
	thailand_domestic_tourism_2019_2023_ver2
WHERE
	variable = 'ratio_tourist_stay';
```

I will now insert the correspending values for each new column one by one starting with `num_occupied_room`
```
/* Fill in the rest of the columns with data */
UPDATE
	thailand_domestic_tourism AS new_table
	INNER JOIN (
		SELECT
			*
		FROM
			thailand_domestic_tourism_2019_2023_ver2
		WHERE
			variable = 'no_tourist_stay') AS original_table ON STR_TO_DATE(CONVERT(original_table. `date`
				USING utf8), '%Y-%m-%d') = new_table. `date`
		AND TRIM(original_table.province_eng) = new_table.province SET new_table.num_occupied_room = original_table.value;
```

The above query will be used to fill in the data for the rest of the columns and the only thing I need to change in this query is just the name of the column from the `orignal_table` and the new table I've just created.

I then check whether the name of the province has a correct corresponding region using the following query.
```
SELECT DISTINCT
	(province),
	region
FROM
	thailand_domestic_tourism;
```

As `Sisaket` is a province in the northeastern region, not the southern region, I will fix this error using the following query.
```
UPDATE thailand_domestic_tourism
SET region = 'east_northeast' WHERE province = 'Sisaket';
```

## Calculate and analyse my data
Now I will start by comparing the numbers and percentages that each type of tourist (Thai tourists and foreign tourists) contribute to the overall revenue and number of tourists from all years combined. I will then look at the dataset through 3 different aspects: _total revenue, number of tourists_ and _number of occupied hotel rooms_(_the occupancy rate_ is also considered along side this aspect). For each aspect, I will analyse the dataset by ranking the years, months and regions and calculating the top 10 provinces.
```
/*See the big picture of all years combined */
SELECT
	SUM(revenue_foreign) AS sum_revenue_foreign,
	SUM(revenue_thai) AS sum_revenue_thai,
	SUM(revenue_all) AS total_revenue,
	CONCAT(ROUND(SUM(revenue_foreign)/SUM(revenue_all)*100),'%') AS foreigner_percentage,
	CONCAT(ROUND(SUM(revenue_thai)/SUM(revenue_all)*100),'%') AS thai_percentage,
	SUM(num_tourist_foreign) AS total_num_foreign,
	SUM(num_tourist_thai) AS total_num_thai,
	SUM(num_tourist_all) AS total_num,
	CONCAT(ROUND(SUM(num_tourist_foreign) / SUM(num_tourist_all) * 100), '%') AS foreigner_percentage,
	CONCAT(ROUND(SUM(num_tourist_thai) / SUM(num_tourist_all) * 100), '%') AS thai_percentage
FROM
	thailand_domestic_tourism
ORDER BY
	total_revenue DESC;  
```

### Total Revenue
#### Check for MAX, MIN and AVG of every year.
```
SELECT
	YEAR(date) AS year,
	AVG(revenue_all) AS avg_revenue,
	MAX(revenue_all) AS max_revenue,
	MIN(revenue_all) AS min_revenue
FROM
	thailand_domestic_tourism
GROUP BY
	year;
```

#### Check for total revenue for each year
```
SELECT
	YEAR(date) AS year,
	SUM(revenue_foreign) AS sum_revenue_foreign,
	SUM(revenue_thai) AS sum_revenue_thai,
	SUM(revenue_all) AS total_revenue,
	CONCAT(ROUND(SUM(revenue_foreign)/SUM(revenue_all)*100),'%') AS foreigner_percentage,
	CONCAT(ROUND(SUM(revenue_thai)/SUM(revenue_all)*100),'%') AS thai_percentage 
FROM
	thailand_domestic_tourism
GROUP BY
	year 
ORDER BY
	total_revenue DESC;
```

#### Check for top 10 provinces with the most total revenue
```
SELECT
	province,
	region,
	SUM(revenue_foreign) AS sum_revenue_foreign,
	SUM(revenue_thai) AS sum_revenue_thai,
	SUM(revenue_all) AS total_revenue,
	CONCAT(ROUND(SUM(revenue_foreign) / SUM(revenue_all) * 100), '%') AS foreigner_percentage,
	CONCAT(ROUND(SUM(revenue_thai) / SUM(revenue_all) * 100), '%') AS thai_percentage
FROM
	thailand_domestic_tourism
GROUP BY
	province,
	region
ORDER BY
	total_revenue DESC
LIMIT 10;      
```
#### Rank the provinces
Now I will look at top 10 provinces based on the total revenue generated from each type of tourist to find out in which provinces the different types of tourists spent their money in the most and whether there is any correlation between the different types of tourists.

**Thai tourists** 
```
/* For Thai: Rank top 10 provinces with the most total revenue from Thai tourists */ 
SELECT
	province,
	SUM(revenue_thai) AS sum_revenue_thai
FROM
	thailand_domestic_tourism
GROUP BY
	province
ORDER BY
	sum_revenue_thai DESC
LIMIT 10;
```

**Foreign tourists**
```
/* For Foreigners: Rank top 10 provinces with the most total revenue from Foreign tourists */ 
SELECT
	province,
	SUM(revenue_foreign) AS sum_revenue_foreign
FROM
	thailand_domestic_tourism
GROUP BY
	province
ORDER BY
	sum_revenue_foreign DESC
LIMIT 10;
```

#### Rank the regions
#### Rank all regions based on total revenue
```
SELECT
	region,
	SUM(revenue_foreign) AS sum_revenue_foreign,
	SUM(revenue_thai) AS sum_revenue_thai,
	SUM(revenue_all) AS total_revenue,
	CONCAT(ROUND(SUM(revenue_foreign) / SUM(revenue_all) * 100), '%') AS foreigner_percentage,
	CONCAT(ROUND(SUM(revenue_thai) / SUM(revenue_all) * 100), '%') AS thai_percentage
FROM
	thailand_domestic_tourism
GROUP BY
	region
ORDER BY
	total_revenue DESC;
```

Now I will rank all the regions based on the total revenue generated from each type of tourist to find out if there is any correlation or differences between the rank of provinces and regions for different types of tourists.

**Thai tourists**
```
SELECT
	region,
	SUM(revenue_thai) AS sum_revenue_thai
FROM
	thailand_domestic_tourism
GROUP BY
	region
ORDER BY
	sum_revenue_thai DESC;
```

**Foreign tourists**
```
SELECT
	region,
	SUM(revenue_foreign) AS sum_revenue_foreign
FROM
	thailand_domestic_tourism
GROUP BY
	region
ORDER BY
	sum_revenue_foreign DESC;  
```

#### Rank the months
Now I will rank the months based on the total revenue to see if there are any trends throughout a year
```
SELECT
	MONTHNAME(date) AS month,
	SUM(revenue_foreign) AS sum_revenue_foreign,
	SUM(revenue_thai) AS sum_revenue_thai,
	SUM(revenue_all) AS total_revenue,
	CONCAT(ROUND(SUM(revenue_foreign)/SUM(revenue_all)*100),'%') AS foreigner_percentage,
	CONCAT(ROUND(SUM(revenue_thai)/SUM(revenue_all)*100),'%') AS thai_percentage 
FROM
	thailand_domestic_tourism
GROUP BY
	month
ORDER BY
	total_revenue DESC;  
```

**For Thai tourists**
```
/* For Thai: Check total revenue from Thai tourists based on months */
SELECT
	MONTHNAME(date) AS month,
	SUM(revenue_thai) AS sum_revenue_thai
FROM
	thailand_domestic_tourism
GROUP BY
	month
ORDER BY
	sum_revenue_thai DESC;
```

**For foreign tourists**
```
SELECT
	MONTHNAME(date) AS month,
	SUM(revenue_foreign) AS sum_revenue_foreign
FROM
	thailand_domestic_tourism
GROUP BY
	month
ORDER BY
	sum_revenue_foreign DESC;
```

### Number of tourists
As I have already calculated and compared the the total number of Thai and foreign tourists at the begining of the analysis stage, I will now rank the years based on the total number of tourists, I will also look at the number and percentage that each tourist type contributes.
```
/* Rank years based on total number of tourists */
SELECT
	YEAR(date) AS year,
	SUM(num_tourist_foreign) AS total_num_foreign,
	SUM(num_tourist_thai) AS total_num_thai,
	SUM(num_tourist_all) AS total_num,
	CONCAT(ROUND(SUM(num_tourist_foreign) / SUM(num_tourist_all) * 100), '%') AS foreigner_percentage,
	CONCAT(ROUND(SUM(num_tourist_thai) / SUM(num_tourist_all) * 100), '%') AS thai_percentage
FROM
	thailand_domestic_tourism
GROUP BY
	year
ORDER BY
	total_num DESC; 
```

#### Top 10 most visited provinces
I will now look at the top 10 provinces based on the total number of tourists to find out what the top 10 most visisted provinces are among all tourists. 
```
/* Rank provinces by total number of tourists */
SELECT
	province,
	region,
	SUM(num_tourist_foreign) AS total_num_foreign,
	SUM(num_tourist_thai) AS total_num_thai,
	SUM(num_tourist_all) AS total_num,
	CONCAT(ROUND(SUM(num_tourist_foreign) / SUM(num_tourist_all) * 100), '%') AS foreigner_percentage,
	CONCAT(ROUND(SUM(num_tourist_thai) / SUM(num_tourist_all) * 100), '%') AS thai_percentage
FROM
	thailand_domestic_tourism
GROUP BY
	province,
	region 
ORDER BY 
	total_num DESC LIMIT 10;  
```

**For Thai tourists**
```
SELECT
	province,
	region,
	SUM(num_tourist_thai) AS total_num_thai,
	SUM(num_tourist_all) AS total_num
FROM
	thailand_domestic_tourism
GROUP BY
	province,
	region 
ORDER BY 
	total_num_thai DESC LIMIT 10;
```

**For foreign tourists**
```
SELECT
	province,
	region,
	SUM(num_tourist_foreign) AS total_num_foreign,
	SUM(num_tourist_all) AS total_num
FROM
	thailand_domestic_tourism
GROUP BY
	province,
	region 
ORDER BY 
	total_num_foreign DESC LIMIT 10;    
```

#### Rank the regions
#### Rank all regions based on number of all tourists
```
SELECT
	region,
	SUM(num_tourist_foreign) AS total_num_foreign,
	SUM(num_tourist_thai) AS total_num_thai,
	SUM(num_tourist_all) AS total_num,
	CONCAT(ROUND(SUM(num_tourist_foreign) / SUM(num_tourist_all) * 100), '%') AS foreigner_percentage,
	CONCAT(ROUND(SUM(num_tourist_thai) / SUM(num_tourist_all) * 100), '%') AS thai_percentage
FROM
	thailand_domestic_tourism
GROUP BY
	region 
ORDER BY 
	total_num DESC;
```

**For Thai tourists**
```
SELECT
	region,
	SUM(num_tourist_thai) AS total_num_thai,
	SUM(num_tourist_all) AS total_num
FROM 
	thailand_domestic_tourism
GROUP BY
	region
ORDER BY
	total_num_thai DESC; 
```

**For foreign tourists**
```
SELECT
	region,
	SUM(num_tourist_foreign) AS total_num_foreign,
	SUM(num_tourist_all) AS total_num
FROM 
	thailand_domestic_tourism
GROUP BY
	region
ORDER BY
	total_num_foreign DESC;  
```

#### Rank months by number of tourists
**All tourists**
```
SELECT
	MONTHNAME(date) AS month,
	SUM(num_tourist_foreign) AS total_num_foreign,
	SUM(num_tourist_thai) AS total_num_thai,
	SUM(num_tourist_all) AS total_num,
	CONCAT(ROUND(SUM(num_tourist_foreign) / SUM(num_tourist_all) * 100), '%') AS foreigner_percentage,
	CONCAT(ROUND(SUM(num_tourist_thai) / SUM(num_tourist_all) * 100), '%') AS thai_percentage
FROM
	thailand_domestic_tourism
GROUP BY
	month
ORDER BY 
	total_num DESC;
```

**For Thai tourists**
```
/* Thai: Rank months by total number of Thai tourist */
SELECT
	MONTHNAME(date) AS month,
	SUM(num_tourist_thai) AS total_num_thai,
	SUM(num_tourist_all) AS total_num
FROM
	thailand_domestic_tourism
GROUP BY
	month
ORDER BY 
	total_num_thai DESC;
```

**For foreign tourists**
```
/* Foreign: Rank months by total number of Foreign tourist */
SELECT
	MONTHNAME(date) AS month,
	SUM(num_tourist_foreign) AS total_num_foreign,
	SUM(num_tourist_all) AS total_num
FROM
	thailand_domestic_tourism
GROUP BY
	month
ORDER BY
	total_num_foreign DESC;  
```

### Number of occupied hotel rooms and occupancy rate
As the data for hotel rooms and occupancy rate are recorded from all tourists and are not sepearated by different tourist types, I will look at the overall picture based on year, month, region and provinces only. I will also include the occupancy rate for all the below analysis to see if there is any correlation between the two variables.

#### Rank years based on total number of occupied rooms
```
SELECT
	YEAR(date) AS year,
	ROUND(SUM(num_occupied_room)) AS total_occupied_room,
	CONCAT(ROUND(AVG(occupancy_rate)), '%') AS avg_occupancy_rate
FROM
	thailand_domestic_tourism
GROUP BY
	year
ORDER BY
	total_occupied_room DESC; 
```

#### Rank months by total number of occupied rooms
```
SELECT
	MONTHNAME(date) AS month,
	ROUND(SUM(num_occupied_room)) AS total_occupied_room,
	CONCAT(ROUND(AVG(occupancy_rate)), '%') AS avg_occupancy_rate
FROM
	thailand_domestic_tourism
GROUP BY
	month
ORDER BY
	total_occupied_room DESC;   
```

#### Top 10 provinces by total number of occupied rooms 
```
SELECT
	province,
	SUM(num_occupied_room) AS total_occupied_room,
  	CONCAT(ROUND(AVG(occupancy_rate)), '%') AS avg_occupancy_rate
FROM
	thailand_domestic_tourism
GROUP BY
	province
ORDER BY
	total_occupied_room DESC LIMIT 10;
```

#### Top 10 provinces ranked by occupancy rate 
As there are 77 provinces, it will be difficult to understand whether the correlation between occopancy rate and the total number of occupied rooms without ranking provinces by occupancy rate. As a result, I am also ranking top 10 provinces by occupancy rate.
```
SELECT
	province,
	CONCAT(ROUND(AVG(occupancy_rate)), '%') AS avg_occupancy_rate
FROM
	thailand_domestic_tourism
GROUP BY
	province
ORDER BY
	avg_occupancy_rate DESC LIMIT 10;  
```

#### Rank regions by total number of occupied rooms
```
SELECT
	region,
	SUM(num_occupied_room) AS total_occupied_room,
  	CONCAT(ROUND(AVG(occupancy_rate)), '%') AS avg_occupancy_rate
FROM
	thailand_domestic_tourism
GROUP BY
	region 
ORDER BY
	total_occupied_room DESC;
```

## Findings and recommendations
For this part of the project, I use **Tableau** as a tool for visualising my findings. I create a dynamic dashboard containing 4 pages: overview, total revenue, number of tourists and total occupied hotel rooms. I include important findings next to the visulisation for each section to summarise any interesting trends or discovery. 

To view the full dashboard, please click [here](https://public.tableau.com/views/ThailandDomesticTravel/Dashboard1?:language=en-GB&:display_count=n&:origin=viz_share_link).

## Overall Findings
### Total revenue vs number of tourists

![image](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/48444482-eb88-4ffa-8d80-20293750c154)

51% of the total revenue accumulated from all years are from foreign tourists while 49% are from Thai tourists. This may not seem like a big difference but when we look at the number of tourists, 84% of all tourists are Thai while 16% are foreigners. This shows that Thai tourists tend to spend much less than foreign tourists when they travel domestically in Thailand.

### Total revenue vs number of tourists vs occupied hotel rooms by year
![Dashboard 5](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/1119627c-57ea-47be-bc55-002785293182)


When comparing the years based on total revenue, total number of tourists and total number of hotel rooms occupied, we can see that 2019 comes in first for all categories, followed by 2022 and 2020. This is highly because 2019 was the year before covid outbreak and 2022 was when border restrictions and healthcare measurement were eased in Thailand. 

Although 2020 was the year that covid started spreading in Thailand, the country was not fully shut down until the second half of the year and that could be why the total revenue and number of tourist and hotel rooms occupied are more than 2021. As we do not have the full-year data for 2023, it makes sense that this year ranks last or the second last for all three categories.

### 1. Total revenue
**By province**

![Dashboard 6 (1)](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/65db2094-eb4a-4f73-9c92-af10caf9f652)

Bangkok ranks first as the province with the highest total revenue across all years, followed by Phuket, Chonburi, Chiang Mai and Krabi. For Thai tourists, Chiang mai ranks second in terms of total revenue while for foreign tourists, Chiang mai does not make it into the top 5. 

Besides Bangkok, the foreign tourist’s top 5 consists of the provinces that are famous for beaches and islands such as Phuket, Chonburi(Pattaya is part of this province), Surat Thani and Krabi. Half of the provinces that made it to the Thai tourists’ top 10 have no beaches at all.

It is worth noting that foreign tourists spend over 7 times more money in Phuket than Thai tourists while Thai tourists spend almost 3 times more money than foreign tourists in Chiang Mai. Thai tourists also tend to spend at least 5 times more in the provinces that are in the last top 10 including Prachuap Khiri Khan, Chiang Rai and Phetchaburi.

**By region**

![Dashboard 7](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/6155be3c-29d0-4d51-8909-05a9edf2e8e1)

The central region ranks first based on the total revenue generated, followed by the South, East, North and Northeast regions. Bangkok is likely to play a big part in the result as Bangkok is the number one province with the most total revenue from all tourist types and it is located in the central part of Thailand. 

The southern region and the eastern region rank second and third respectively for the total revenue as foreign tourists tend to spend more money on their trip and provinces with beaches in the southern and eastern regions are popular among foreign tourists. 

In contrast, the northern region rank which is in line with how Chiang Mai is also the province where Thai tourists spend most money in.

**By month**

![Dashboard 8 (1)](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/3c70a8af-aac3-4284-9d5f-0b7e199428e1)

Thailand accumulated its tourism revenue the most in January, February, December, November and October respectively. This is highly because the end of the year and the beginning of the year are during international holiday seasons, therefore, people travel and spend money more during these times.  

May and June generated the least total revenue. This is likely due to the fact that there are not any international holidays during the months and the weather is hot and unpredictable during these months.

### 2. Number of tourists

**By province**

![Screen Shot 2024-01-31 at 11 46 14 AM](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/2bfb2ccb-56e6-43f5-bbd8-1928d613d1c1)

Bangkok is the most visited province from 2019 to 2023 followed by Chonburi which is a province of Pattaya city. Surprisingly, Kanchanaburi was the third most visited province.
As this province does not make it to the top 10 for total revenue and 98% of all tourists were Thai, it might be worth looking into the reasoning behind this. There could be some events in that province that attracted Thai tourists or perhaps there are some hidden gems known among Thai people. Similarly, there are also a few provinces in the top 10 list with over 94% being Thai tourists with not much revenue generated such as Phetchaburi, Nakhon Ratchasima, Prachuap Khiri Khan and Rayong. 

Phra Nakhon Si Ayutthaya is also a similar case as 88% of all tourists visited are Thai but this province is not one of the top 10 provinces with the most total revenue. These provinces are also worth looking at and exploring further.

Chiang Mai and Phuket also make it to the top 10 most visited provinces, this aligns with the fact that they are also in the top 10 provinces that tourists spend most money in.

**By region**

![Dashboard 10](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/f1c081dd-1297-4555-9f5f-9255bba2e22f)

The central region is the most visited by all tourists as expected since Bangkok is located in this region. Surprisingly, the northeast region is the second most visited with 97% of the tourists being Thai, although this region has the least total revenue. 

As Thai tourists significantly outnumbered foreign tourists, it makes sense that the rank most visited regions are hugely influenced by Thai tourists’ preferences and the result would be different than the rank of regions based on the total revenue where Thai tourists spend less money in general.

The northern regions are more popular among Thai tourists whereas foreign tourists prefer to visit the southern region more. The northeastern regions are also the least popular regions for foreign tourists.

**By month**

![Dashboard 9](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/89081003-f045-44e5-be67-5f764936a53c)

When we look at the number of visitors per month, it is evident that the result is also in line with the total revenue of each month. People tend to travel more during holiday seasons ( October - Jan) with January being the most popular month and they travel less in May which is the hottest month in Thailand and June which is during the rainy season.

### 3. Occupancy rate and total hotel reooms occupied

**By province**

![Dashboard 11](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/4bea796d-e9bc-44bd-990d-6ac0501958cf)

Bangkok has the highest number of hotel rooms occupied followed by Chonburi, Phuket and Chiang Mai which is in accordance with the fact these provinces also have high total revenues. Tourists also tend to stay in the provinces that they visit but do not spend much money in such as Nakhon Ratchasima, Prachuap Khiri Khan, Phetchaburi and Kanchanaburi. 

![Dashboard 11 (1)](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/5429bd4d-7418-4639-85c7-be637ad75f76)

In terms of occupancy rates, Nan has the highest rate at 52% followed by Chiang Mai, Phetchhaburi and Kanchanaburi. These provinces are quite far from Bangkok with the Nan Chiang Mai situation in a different region, this may be one of the reasons why the occupancy rate is high. Some provinces also have less accommodation than the bigger provinces causing the occupancy rate to be higher even though the number of the occupied rooms are so much less.

**By month**

![Dashboard 12](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/87f0bd3e-60c3-453e-8c1d-a82487ab298a)

Hotel rooms were the most occupied in January and highly occupied during September to December. April to June was the least occupied period. This finding is in line with what we discovered for total revenue and total tourists.

**By region**

![Dashboard 13](https://github.com/ksadangrit/thailand_domestic_tourism/assets/156267785/de8197fd-1d62-4838-b23e-bb0139a3e1e0)

The central region has the highest number of occupied hotel rooms, followed by the south, east, north and northeast. This is in line with the earlier finding regarding the ranking of the most visited regions and total revenue across all years.

### Recommendations
1. The Ministry of Tourism and Sports should continue to elevate the tourism supply and sustainable standards for popular provinces such as Bangkok, Chonburi and Chiang Mai.
2. The Ministry should also consider a campaign or advertisement of provinces that are only popular among Thai tourists to be more appealing towards foreigners as foreign tourists tend to spend more money while traveling in Thailand. The provinces with high number of Thai tourists seems to not generate that much revenue so it is important to make sure that there are fun activities or interesting things to do in those provinces.
3. The Ministry should continue to ensure that Thailand has the capacity to handle high volumes of tourists during popular months (October to February).
 
