# Bike Share Company Data Analysis: Case Study

## Introduction

With this project, I wanted to experiment and challenge myself with a massive dataset. This dataset had over 5 million observations of publically available data of a bike share business. Within the 5 million observations, the dataset contained columns indicating bike ride information, as well as user information. Ride information included but wasn't limited to: starting/ending location, time/date, and __________. User information included subscription type each customer was. For context, there are two subscription types in this dataset -- "casual" and "member".

With a detailed analysis, I wanted to answer these questions:

1. How do different subscription user types use the bike share differently?
2. Why would casual riders buy annual memberships?
3. How can the business use digital media to influence casual riders to become members?

## Preparation

Because the data was on a month-to-month basis, I knew I was going to be able to have a large sample size to draw insights from. The bigger the sample size, the more accurate the analysis will be. The available data were monthly datasets, and to analyze the whole year, I was going to have to combine the last 12 months of data. Because of the size of the dataset, I leveraged R, knowing R's capabilities of processing large amounts of data efficiently. With any dataset, it is essential to clean and prepare the data to ensure a clear and accurate analysis.

This code chunk quickly merged all the monthly data .csv files:
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

This code chunk validated all ride_IDs had the same amount of 16 characters to be used as a primary key in our analysis:
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

This code chunk removed all blank rows:
````R
# Replace empty strings with NA in combined_data
combined_data[combined_data == ""] <- NA

# Remove rows with missing values
clean_combined_data <- na.omit(combined_data)
````
![alt text](https://github.com/trustinvo/casestudy/blob/main/data%20clean.png)

We can see that we removed ~1.3 million observations of blank rows, which will be helpful in our analysis as well as data processing load.

This code chunk counted, checked, and printed number of duplicates:
````R
# Check for duplicates in clean_combined_data
duplicates <- duplicated(clean_combined_data)

# Count the number of duplicate rows
num_duplicates <- sum(duplicates)

# Print the count of duplicate rows
print(num_duplicates)

# Export dataframe into csv
write.csv(clean_combined_data, file = "2023clean_tripdata.csv", row.names = FALSE )
````
![alt text](https://github.com/trustinvo/casestudy/blob/main/duplicates%20validation.png)

We have no duplicates, so I exported our data to import into Google Cloud to utilize Google BigQuery SQL. Admittedly, I faced challenges troubleshooting this import since I had never worked with a dataset with 4 million+ observations. But that was part of the challenge, and definitely expanded my knowledge with working with such a massive dataset.

I wanted my analysis to additionally categorize the ride information by day of the week and ride length. I also wanted to further clean the data, so this code chunk below leverages CASE WHEN and EXTRACT statements from TIMESTAMP data format in the started_at column and filters out erroneous ride information with 0 ride minutes:

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
FROM `cyclistic-trustin.cyclistic.2023clean_tripdata` 
WHERE TIMESTAMP_DIFF(ended_at, started_at, minute)> 0;
````

