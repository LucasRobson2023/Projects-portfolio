Chicago crime data analysis

I am looking at analysising a large dataset (1.1 million records) containing data around crime in Chicago. I chose this because there are plenty of data points to look at and to tie in with other datasets. Some of these other datasets I will look at come from the Chicago Data Portal. This contains information around community areas, demographics and businesses. 

To start off with I looked at the chicago crime data dataset by itself. As this is a huge dataset, querying needs to be smart, considering efficency.
To start off with I had a look at 'battery' cases after 2020-09-29.

<img width="706" height="230" alt="Screenshot 2025-08-21 at 16 07 45" src="https://github.com/user-attachments/assets/e7cb466c-e934-4b6b-b21e-1dc58c4d89bc" />

|    |   latitude |   longitude | primary_type   | date                      |
|---:|-----------:|------------:|:---------------|:--------------------------|
|  0 |    41.8838 |    -87.6262 | BATTERY        | 2021-05-23 00:39:00+00:00 |
|  1 |    41.8855 |    -87.6279 | BATTERY        | 2024-06-25 08:00:00+00:00 |
|  2 |    41.8868 |    -87.6284 | BATTERY        | 2024-06-29 15:30:00+00:00 |
|  3 |    41.8854 |    -87.6263 | BATTERY        | 2021-09-10 21:00:00+00:00 |
|  4 |    41.8838 |    -87.6262 | BATTERY        | 2023-10-26 16:10:00+00:00 |

I then wanted to map this out, but after doing so I realised that outliers existed within the dataset, causing the map to be shifted outside of Chicago's bounds.
So to start with I removed outliers based on the latitude and longitude.

This plot shows the result of all of the battery cases across Chicago. However, its quite difficult to see where most of the crime happens and within what communities.

<img width="494" height="556" alt="Screenshot 2025-08-20 at 09 17 42" src="https://github.com/user-attachments/assets/d1742f55-1f8e-4f15-93c5-3009b4f99266" />

I started off with merging the communities area dataset with the crimes dataset based on the longitude and latitude. The community area dataset provides multipolygons for each community, which allows us to check whether a specific location (long, lat) is within that area.


|    | the_geom                     |   AREA_NUMBE | COMMUNITY      |   AREA_NUM_1 | SHAPE_AREA      | SHAPE_LEN      |   Community Area | Community Area Name   |   Below Poverty Level |   Crowded Housing |   Dependency |   No High School Diploma | Per Capita Income   |   Unemployment | geometry                     |
|---:|:-----------------------------|-------------:|:---------------|-------------:|:----------------|:---------------|-----------------:|:----------------------|----------------------:|------------------:|-------------:|-------------------------:|:--------------------|---------------:|:-----------------------------|
|  0 | MULTIPOLYGON (((-87.65455... |            1 | ROGERS PARK    |            1 | 51,259,902.4506 | 34,052.3975757 |                1 | Rogers Park           |                  22.7 |               7.9 |         28.8 |                     18.1 | 23,714              |            7.5 | MULTIPOLYGON (((-87.65455... |
|  1 | MULTIPOLYGON (((-87.68465... |            2 | WEST RIDGE     |            2 | 98,429,094.8621 | 43,020.6894583 |                2 | West Ridge            |                  15.1 |               7   |         38.3 |                     19.6 | 21,375              |            7.9 | MULTIPOLYGON (((-87.68465... |
|  2 | MULTIPOLYGON (((-87.64102... |            3 | UPTOWN         |            3 | 65,095,642.7289 | 46,972.7945549 |                3 | Uptown                |                  22.7 |               4.6 |         22.2 |                     13.6 | 32,355              |            7.7 | MULTIPOLYGON (((-87.64102... |
|  3 | MULTIPOLYGON (((-87.67440... |            4 | LINCOLN SQUARE |            4 | 71,352,328.2399 | 36,624.6030848 |                4 | Lincoln Square        |                   9.5 |               3.1 |         25.6 |                     12.5 | 35,503              |            6.8 | MULTIPOLYGON (((-87.67440... |
|  4 | MULTIPOLYGON (((-87.67336... |            5 | NORTH CENTER   |            5 | 57,054,167.85   | 31,391.6697542 |                5 | North Center          |                   7.1 |               0.2 |         25.5 |                      5.4 | 51,615              |            4.5 | MULTIPOLYGON (((-87.67336... |

Once this is mapped out, it now provides the community area boundaries and names. However, it's still quite difficult to see where it is mostly concentrated.

<img width="578" height="702" alt="Screenshot 2025-08-20 at 09 35 19" src="https://github.com/user-attachments/assets/7d9dcfeb-421d-46cf-b249-d7a2d0cc62cb" />

To make it more clear, I grouped the crimes by community area and summed the crimes for each one. Once plotted this is what it reveals.

<img width="690" height="721" alt="Screenshot 2025-08-20 at 09 58 07" src="https://github.com/user-attachments/assets/9b785418-bcdd-4894-9c49-216c4669b7ec" />

Another dataset I looked at was poverty, and how it affected crime. To do this I firstly merged the previous dataset containing crime count by community area, with the the poverty dataset which was done by community.

|    | COMMUNITY      |   num_crimes |   Below Poverty Level |   Crowded Housing |   Dependency |   No High School Diploma | Per Capita Income   |   Unemployment | geometry                     |
|---:|:---------------|-------------:|----------------------:|------------------:|-------------:|-------------------------:|:--------------------|---------------:|:-----------------------------|
|  0 | ALBANY PARK    |         1952 |                  17.1 |              11.2 |         32.1 |                     34.9 | 20,355              |            9   | MULTIPOLYGON (((-87.70403... |
|  1 | ARCHER HEIGHTS |          679 |                  13   |               8.5 |         40.5 |                     36.4 | 16,145              |           14.2 | MULTIPOLYGON (((-87.71436... |
|  2 | ARMOUR SQUARE  |          825 |                  35.8 |               5.9 |         37.9 |                     37.5 | 16,942              |           11.6 | MULTIPOLYGON (((-87.62916... |
|  3 | ASHBURN        |         1629 |                   9.5 |               4.2 |         36.7 |                     18.3 | 22,078              |            8.8 | MULTIPOLYGON (((-87.71254... |
|  4 | AUBURN GRESHAM |         6320 |                  24.5 |               4.1 |         42.1 |                     19.5 | 16,022              |           24.2 | MULTIPOLYGON (((-87.63990... |


This allowed me to plot the poverty level by number of crimes, as seen below. However, this graph is not telling the complete truth, as some of these communities are much larger than others, and so will have a much larger crime count.

<img width="784" height="530" alt="Screenshot 2025-08-20 at 09 59 15" src="https://github.com/user-attachments/assets/3338e0ea-a1f3-4b08-80f3-3ff386a90e70" />

To fix this issue, I normalised the crime rate by crime per 1000 people. Once replotted it tells a slightly different story, showing a stronger correlation and highlighting communities where this affects the most. 

<img width="803" height="527" alt="Screenshot 2025-08-20 at 09 59 52" src="https://github.com/user-attachments/assets/c97c4bba-e2c0-4783-a424-f4c6ea7abfa0" />

To explore other details around what might be affecting crime I explored the data by plotting various relevant data onto the chicago map. I firstly continued with finding the community with the worst poverty.

<img width="680" height="726" alt="Screenshot 2025-08-20 at 10 00 49" src="https://github.com/user-attachments/assets/5408efb0-4ad3-40c9-ae23-59a8cca18022" />

I then moved onto exploring the demographics in each community, for example the break down of ages and gender.

<img width="665" height="723" alt="Screenshot 2025-08-20 at 10 03 45" src="https://github.com/user-attachments/assets/8e088880-787b-44a4-a706-c8b52ccd246a" />



<img width="680" height="723" alt="Screenshot 2025-08-20 at 10 04 39" src="https://github.com/user-attachments/assets/b2ca403d-9078-45e8-a311-7ec2298e7c2c" />


<img width="704" height="706" alt="Screenshot 2025-08-20 at 10 05 05" src="https://github.com/user-attachments/assets/a1561749-12fd-4520-a7ae-9977da287993" />

<img width="664" height="716" alt="Screenshot 2025-08-20 at 10 05 33" src="https://github.com/user-attachments/assets/dc25406c-6bba-4d59-ac1d-59eee453c96f" />


I then wanted to move on to exploring more of the crime dataset, but this time including all of the cases from 2020.


This time I wanted to explore location, specifically where the crime happened - as the crime dataset includes information on the surroundings of where the crime happened. As seen below, it provides location descriptions such as 'Abandoned Building' or 'Alley'.



I then grouped by crime type and summed up for each location, to find the top 5 location for each crime. I used a simple bar plot for this.



I also wanted to check what crimes were likely to be one incident. To do this I used DBSCAN, providing it both the location and the time. Because of how DBSCAN functions, I converted the latitude and longitude into radians and the time into date time format. 
This returned clusters across all of the crimes. At first it was clustering too many of the crimes into one, as seen below where crimes which have a 2 hours difference were clustered.


I fixed this by ammending the eps (way of identifying two samples of being neighbours).


I then had a look at the top pairs of crimes within the clusters. Summing up occurances of the same two crimes across all clusters.



These clusters were also plotted on the chicago map, to find where these big crimes happened.


I then wanted to bring in the businesses dataset to have a look at whether certain business have more crime occuring around them, or less.
I initially plotted all the businesses on the map with inividual crimes also plotted. This wasn't very useful and only showed concentration of businesses in chicago. 



To make this more accurate I wanted to cluster businesses and crimes. Rather than using a clustering algorithm, this time I based it upon the spacial location and filtered the crimes based on when they happened, along with when the business first started operating.


Once plotted it showed the crimes and the businesses which they were around. However, this doesn't tell me much - other than crimes do happen to certain businesses (but we don't know to which businesses and what sector)



To start with I wanted to find which businesses had the most reoccuring crimes. I then plotted this on the map, showing a few businesses having a significantly higher occurence. 


I also wanted to find what sectors have the most crimes. However, the identifier for this within the database was through 'business activity'. These were manually written descriptions and often included misspellings and similar businesses did not have the same description. 


I started of with trying to use regex for word matching into categories. However, this didn't heed the best results, with many of the descriptions having misspellings.


Next I attempted to use embeddings and similarity cosine, which would allow me to match strongly linked words within the descriptions and match it to the categories.
I used Qwen 3.5 0.6 billion parameter model and 4 billion parameter model invidually.




While this did result in better matches, there were still some cases such as McDonalds' descriptions of 'retail of perishable foods' which did not match any of the categories - even after casting a wider net of categories.


Instead I tried using SentenceBERT. This finally resulted in better results.

