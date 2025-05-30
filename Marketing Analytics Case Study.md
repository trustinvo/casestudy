# Marketing Analytics: Customer Preferences Case Study *(10 min. read)*

## Introduction

With this project, I wanted to experiment and challenge myself with a massive dataset. This dataset had over 5 million observations of publically available data of a bike share business. Within the 5 million observations, the dataset contained columns indicating bike ride information, as well as user information. Ride information included but wasn't limited to: starting/ending location, time/date, and ride type. User information included subscription type each customer was. For context, there are two subscription types in this dataset -- "casual" and "member". Casual subscribers use the service on a pay-as-you-go, low commitment basis. Member subscribers purchased a long-term subscription to the service to enjoy more benefits.

With a detailed analysis, I wanted to answer these questions:

1. How do different subscription user types use the bike share differently?
2. Why would casual riders buy annual memberships?
3. How can the business use digital media to influence casual riders to become members?

## Preparation

Because the data was on a month-to-month basis, I knew I was going to be able to have a large sample size to draw insights from. The bigger the sample size, the more accurate the analysis will be. The available data were monthly datasets, and to analyze the whole year, I was going to have to combine the last 12 months of data. Because of the size of the dataset, I leveraged R, knowing R's capabilities of processing large amounts of data efficiently. With any dataset, it is essential to clean and prepare the data to ensure a clear and accurate analysis.

This script quickly merged all the monthly data .csv files:
````R
# Install and load the dplyr package if not already installed
# 
install.packages("dplyr")
library(dplyr)

# Get a list of all CSV files in the current working directory
csv_files <- list.files(pattern = "\\.csv$")

# Check if there are any CSV files
if (length(csv_files) == 0) {
  stop("No CSV files found in the current directory.")
}

# Read and combine CSV files into one data frame
combined_data <- lapply(csv_files, read.csv) %>%
  bind_rows()

# Save the combined data to a new CSV file
write.csv(combined_data, "all2023_tripdata.csv", row.names = FALSE)
````

This script validated all ride_IDs had the same amount of 16 characters to be used as a primary key in our analysis:
````R
# Check the number of characters in each ride_id
ride_id_lengths <- nchar(combined_data$ride_id)

# Check if all ride_id values have 16 characters
all_16_characters <- all(ride_id_lengths == 16)

# If all_16_characters is TRUE, then all ride_id values have 16 characters
if (all_16_characters) {
  print("All ride_id values have 16 characters.")
} else {
  print("Not all ride_id values have 16 characters.")
}
````

This script removed all blank rows:
````R
# Replace empty strings with NA in combined_data
combined_data[combined_data == ""] <- NA

# Remove rows with missing values
clean_combined_data <- na.omit(combined_data)
````
![alt text](https://github.com/trustinvo/casestudy/blob/main/data%20clean.png)

> We can see that we removed ~1.3 million observations of blank rows, which will be helpful in our analysis as well as data processing load.

This script counted, checked, and printed number of duplicates based on the data's primary key (ride_id):
````R
# Check for duplicates in clean_combined_data based on ride_id
duplicates <- duplicated(clean_combined_data$ride_id)

# Count the number of duplicate ride ids
num_duplicates <- sum(duplicates)

# Print the count of duplicate ride ids
print(num_duplicates)
````
![alt text](https://github.com/trustinvo/casestudy/blob/main/dupe%20validation.png)

> Okay cool, we have no duplicates, so I exported our data to import into Google Cloud to utilize Google BigQuery SQL.
>
> Admittedly, I faced challenges troubleshooting this import since I had never worked with a dataset with 4 million+ observations. But that was part of the challenge I wanted to face, and definitely expanded my knowledge with working with massive datasets.

I wanted my analysis to additionally categorize the ride information by day of the week and ride length. I also wanted to further clean the data. This SQL script below is detailed in its blockquote underneath.

````SQL
SELECT *,
  TIMESTAMP_DIFF(ended_at, started_at, minute) as ride_length_mins,
  CASE
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 1 THEN 'Sunday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 2 THEN 'Monday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 3 THEN 'Tuesday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 4 THEN 'Wednesday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 5 THEN 'Thursday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 6 THEN 'Friday'
    WHEN EXTRACT(DAYOFWEEK FROM started_at) = 7 THEN 'Saturday'
    ELSE 'Unknown' 
  END AS day_of_week
FROM `project.2023clean_tripdata` 
WHERE TIMESTAMP_DIFF(ended_at, started_at, minute)> 0;
````
> 1) Forms ride_length_mins from calculating the difference between started_at and ended_at minutes
> 2) CASE WHEN and EXTRACT statements from TIMESTAMP data format in the started_at column to calculate the day of the week
> 3) Filters out erroneous ride information for rides with less than 1 minute of ride length

With this stage of preparation covered, I was now ready to import into Tableau to analyze, visualize, and quantify my findings to eventually form actionable insights.

## Analysis

I wanted to start my analysis with some preliminary summary statistics to help me and my audience to have some context behind the data. As previously mentioned, I wanted to answer these questions:

1. How do different subscription user types use the bike share differently?
2. Why would casual riders buy annual memberships?
3. How can the business use digital media to influence casual riders to become members?

![alt text](https://github.com/trustinvo/casestudy/blob/main/total%20number%20of%20riders%20per%20subscriber%20type.png)

![alt text](https://github.com/trustinvo/casestudy/blob/main/avg%20ride%20length.png)

> These two visuals above tell us an interesting story from our 2023 data. Casual subscribers ride the bikes for a longer ride duration on average than member subscribers, but member subscribers take more rides in total. We can already see how casual and annual subscribers' behavior differ. But let's utillize additional available information to form some more insights and paint a clearer picture.

![alt text](https://github.com/trustinvo/casestudy/blob/main/total%20rides%20per%20day.png)

> With the SQL code I mentioned previously, I created an identifier for the day of the week that a ride took place. Above, the total number of rides from each subscriber type is broken down by the day of the week. From this visual, we can see an inverse relationship -- casual subscribers take less rides on weekdays (and more on weekends) and member subscribers take more rides on weekdays (and less on weekends). The natural assumption is that casual subscribers are visitors/tourists on weekends, while member subscribers are typically locals to the area, utilizing bikes as a form of regular transportation. Let's investigate that further by leveraging geographical information the dataset has available to us.

![alt text](https://github.com/trustinvo/casestudy/blob/main/avg%20ride%20length%20per%20month.png)

> This line chart above displays the different monthly behavior of the two subscription types. Casual subscribers had an influx of riders in the months that are typically warmer (April to September) and gradually tapered off in the following months. Member subscribers' behavior was a lot more constant month-to-month, supporting the idea that member subscribers are typically locals, while casual subscribers are likely visitors/tourists.

![alt text](https://github.com/trustinvo/casestudy/blob/main/starting%20location%20casual.png)

> From this density geographical heat map above, we can see that an overwhelming majority of casual subscribers start their rides at the Chicago Harbor.

![alt text](https://github.com/trustinvo/casestudy/blob/main/starting%20location%20members.png)

> From this density geographical heat map above, we can see that majority of member subscribers started their rides in town, presumably around office/apartment buildings. 

![alt text](https://github.com/trustinvo/casestudy/blob/main/ending%20location%20casual.png)

![alt text](https://github.com/trustinvo/casestudy/blob/main/ending%20location%20members.png)

> Here's the kicker -- the ending locations of both subscriber types are nearly identical. We can conclude that both subscriber types only really vary where they start their rides, but are ending their rides around the same places -- most notably Fulton River District and Logan Square. These neighborhoods are communities known for their vibrant eateries/bars.

## Insights Wrap Up

By observing consumer behavior, we can form conclusions that answer the questions we planned to answer at the beginning of this analysis.

1. How do different subscription user types use the bike share differently?
2. Why would casual riders buy annual memberships?
3. How can the business use digital media to influence casual riders to become members?

Subscription user types differ by:
- Ride duration: casual subscribers tend to ride bikes for longer durations on average compared to member subscribers, while member subscribers take more rides in total.
- Usage patterns: casual subscribers are more active on weekends and during warmer months, suggesting they may be visitors or tourists. In contrast, member subscribers are more active on weekdays and exhibit more consistent month-to-month behavior, indicating they are likely local residents using bikes for regular transportation.
- Behavior: casual subscribers predominantly start their rides at tourist-centric areas like Chicago Harbor, member subscribers primarily start their rides in town, indicating they are likely local residents using bikes for regular transportation.

Despite variations in starting locations, both subscriber types tend to end their rides in similar areas, such as Fulton River District and Logan Square, known for vibrant eateries and bars. This suggests that while their starting points differ, the destinations are relatively consistent, highlighting potential areas for focused marketing efforts.

While casual subscribers are typically visitors/tourists, there is an opportunity to convert the casual subscribers that are local residents into annual subscribers.

Casual, local resident riders might opt for annual memberships due to factors such as cost-effectiveness for frequent users, convenience, or potential incentives offered by the bike share program. Although casual riders typically use the service sporadically, an annual membership could provide them with added benefits like discounted rates, a more convenient rider experience, an additional member-exclusive perks. If casual riders find themselves using the bike share service frequently during their visits to the area, they may see value in investing in an annual membership for long-term savings and convenience if we shine light on what could be valuable to a frequent user. 

There is also value in analyzing/targeting frequent riders who are casual subscribers that would reap the benefits of an annual membership but for whatever reason are not annual members.

Marketing efforts can be used to influence these casual riders to become members by leveraging strategies tailored to address their needs and preferences. 

Some forms of a targeted advertising campaigns could be:
- Why bike sharing is a better form of transportation (ie. carbon footprint "savings" comparison to other forms, flexibility to ride whenever, eliminates the inconvenience of traffic congestion and waiting in lines).
- Benefits of annual memberships, such as cost savings for frequent users, convenience of unlimited rides, and access to member-exclusive perks.
-  Utilize social media platforms to engage with casual riders, share testimonials from satisfied members, and offer limited-time promotions or discounts to incentivize membership sign-ups.
- Providing user-friendly online registration processes and offering flexible membership options can also encourage casual riders to make the switch to becoming long-term members.
- Digital media strategy targeting casual subscribers in their starting locations (ie. offering exclusive promotional pricing if users sign up within a specified timeframe after receiving the notification when in a starting location vicinity).

This project provided an opportunity to explore and analyze a vast dataset from a bike share business, containing over 5 million observations. The dataset featured various columns detailing ride information, such as starting and ending locations, timestamps, and ride types, as well as user information, including subscription types. Notably, the dataset featured two distinct subscription types: "casual" and "member." Casual subscribers utilize the service on a pay-as-you-go basis with minimal commitment, whereas member subscribers have opted for long-term subscriptions, offering them enhanced benefits. Through this analysis, insights were gained into user behavior and subscription preferences within the bike share ecosystem, contributing to a deeper understanding of usage patterns and potential areas for further optimization.

I hope you gained as much knowledge about the data and analysis as I did while working on this project.

*Thank you* for reading!

## Additional Analysis Opportunity

Adding additional user information for each ride to the dataset would enhance a future analysis. This data could provide insights into user behavior, especially focusing on frequent riders who are local residents but yet are only casual subscribers. Studying consumer behavior in this demographic could offer valuable insights and opportunities for increasing annual memberships.















