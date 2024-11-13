# Dayton-OH-REALTORS-Real-Estate-Agents-Scraper


![download](https://github.com/user-attachments/assets/71aac3a5-3377-449a-ba39-bb2720ac808f)


# Dayton, OH REALTORS® & Real Estate Agents Scraper

This project provides an R-based script to scrape data on REALTORS® and Real Estate Agents in Dayton, OH, from Realtor.com. It collects data such as agent names, company names, contact numbers, number of properties for sale, properties sold, and price range activity. This data can be useful for real estate analysis, local market research, and building a directory of agents.

## Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Output](#output)
- [Contributing](#contributing)
- [License](#license)

## Project Overview

Realtor.com hosts listings for real estate agents and REALTORS® in various locations. This script targets Dayton, OH, scraping the first five pages of listed agents. The data is extracted using the `rvest` package and stored in a structured format in both CSV and Excel files for easy access and analysis.

## Features

- **Automated Web Scraping**: Scrapes real estate agent data from Realtor.com.
- **Data Extraction**: Collects key information, including:
  - Agent's Name
  - Company Name
  - Phone Number
  - Number of Properties for Sale
  - Properties Sold
  - Activity Price Range
- **Multi-Page Navigation**: Automatically retrieves data from multiple pages.
- **Data Output**: Exports data to CSV and Excel formats.

## Installation

### Prerequisites

- R (version 4.0 or later recommended)
- Required R libraries: `rvest`, `dplyr`, `purrr`, and `writexl`

### Install Required Packages

To install the required packages, run the following in your R console:

```r
install.packages("rvest")
install.packages("dplyr")
install.packages("purrr")
install.packages("writexl")  # For Excel export


library(rvest)
library(dplyr)
library(purrr)
library(writexl)  # For writing Excel files

# Base URL for Dayton, OH REALTORS® & Real Estate Agents
base_url <- "https://www.realtor.com/realestateagents/dayton_oh/pg-"

# Initialize an empty data frame to store results
agents_data <- data.frame(Name = character(),
                          Company = character(),
                          Phone = character(),
                          PropertiesForSale = character(),
                          PropertiesSold = character(),
                          ActivityRange = character(),
                          stringsAsFactors = FALSE)

# Loop through the first 5 pages
for (page_num in 1:5) {
  # Construct the URL for each page
  url <- paste0(base_url, page_num)
  
  # Read the HTML content of the page
  page <- read_html(url)
  
  # Helper function to safely extract and pad missing values with NA
  safe_extract <- function(selector) {
    result <- page %>% html_nodes(selector) %>% html_text(trim = TRUE)
    if (length(result) == 0) return(NA)  # Return NA if no data found
    return(result)
  }
  
  # Extract data fields with padding for missing values
  agent_names <- safe_extract(".jsx-3873707352 .text-bold")
  company_names <- safe_extract(".jsx-3873707352 .agent-group")
  phone_numbers <- safe_extract(".jsx-3873707352 .agent-phone")
  properties_for_sale <- safe_extract(".jsx-3873707352 .agent-detail-item .price")
  properties_sold <- safe_extract(".jsx-3873707352 .agent-detail-item.pl-1 .price")
  activity_range <- safe_extract(".jsx-3873707352 .agent-detail-item .bold-text")
  
  # Ensure all columns have the same length by padding with NAs where necessary
  max_length <- max(length(agent_names), length(company_names), length(phone_numbers), 
                    length(properties_for_sale), length(properties_sold), length(activity_range))
  
  agent_names <- c(agent_names, rep(NA, max_length - length(agent_names)))
  company_names <- c(company_names, rep(NA, max_length - length(company_names)))
  phone_numbers <- c(phone_numbers, rep(NA, max_length - length(phone_numbers)))
  properties_for_sale <- c(properties_for_sale, rep(NA, max_length - length(properties_for_sale)))
  properties_sold <- c(properties_sold, rep(NA, max_length - length(properties_sold)))
  activity_range <- c(activity_range, rep(NA, max_length - length(activity_range)))
  
  # Combine the extracted data into a data frame
  page_data <- data.frame(
    Name = agent_names,
    Company = company_names,
    Phone = phone_numbers,
    PropertiesForSale = properties_for_sale,
    PropertiesSold = properties_sold,
    ActivityRange = activity_range,
    stringsAsFactors = FALSE
  )
  
  # Append the page data to the main data frame
  agents_data <- bind_rows(agents_data, page_data)
  
  # Pause between requests to avoid overloading the server
  Sys.sleep(2)
}

# Display the collected data
print(agents_data)

# Save data to a CSV file
write.csv(agents_data, "dayton_real_estate_agents.csv", row.names = FALSE)

# Save data to an Excel file
write_xlsx(agents_data, "dayton_real_estate_agents.xlsx")

```

![list](https://github.com/user-attachments/assets/2f8fa879-b131-4d6c-9577-ba27a51b02bb)



# Output
**The output files dayton_real_estate_agents.csv and dayton_real_estate_agents.xlsx contain the following columns:**

- **Name:** Name of the real estate agent
- **Company:** Company name associated with the agent
- **Phone:** Contact phone number for the agent
- **PropertiesForSale:** Number of properties currently listed by the agent
- **PropertiesSold:** Number of properties sold by the agent
- **ActivityRange:** Price range of properties managed by the agent

# License
- This project is licensed under the MIT License. See the LICENSE file for details.
