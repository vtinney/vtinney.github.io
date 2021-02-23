---
title: 'Using Tidycensus package to extract ACS 5-year estimates'
date: 2021-02-22
permalink: /posts/2021/02/tidycensus/

---


`#install.packages('tidycensus')
library(tidycensus)
library(tidyverse)
library(data.table)

census_api_key("cf7ba23c01a84fe015a7669aa755f04bd2fcb4e2")

#2001, 2004 - dicennial census (2000)
#2006, 2008 - ACS 2005-2009
#2011, 2013, 2016  - ACS Five year

years <- c(2016,2013,2011,2009)

us <- c("AL","AK","AZ","AR","CA","CO","CT","DE","FL","GA","HI","ID",
        "IL","IN","IA","KS","KY","LA","ME","MD","MA","MI","MN","MS","MO",
        "MT","NE","NV","NH","NJ","NM","NY","NC","ND","OH","OK","OR","PA",
        "RI","SC","SD","TN","TX","UT","VT","VA","WA","WV","WI","WY")


for(k in 1:length(years)){
  lists <- list()

  for (i in 1:length(us)){
    acs_df <- get_acs(geography = "tract", 
                       variables = c("B02001_001","B02001_002","B02001_003","B02001_004",
                                     "B02001_005","B02001_006","B02001_007","B03001_001","B03001_003",
                                     "B17001_002"),
                 state=paste(us[i]),
                 year = years[k])
    lists[[i]] <- as.data.frame(acs_df)
    print(paste('list_',us[i],sep=''))
}

all_us_df <- rbindlist(lists)

all_us_df$variable[all_us_df$variable == 'B02001_001'] <- 'Total race'
all_us_df$variable[all_us_df$variable == 'B02001_002'] <- 'White'
all_us_df$variable[all_us_df$variable == 'B02001_003'] <- 'Black'
all_us_df$variable[all_us_df$variable == 'B02001_004'] <- 'AIAN'
all_us_df$variable[all_us_df$variable == 'B02001_005'] <- 'Asian'
all_us_df$variable[all_us_df$variable == 'B02001_006'] <- 'NAPI'
all_us_df$variable[all_us_df$variable == 'B02001_007'] <- 'Other'
all_us_df$variable[all_us_df$variable == 'B03001_001'] <- 'Total ethnicity'
all_us_df$variable[all_us_df$variable == 'B03001_003'] <- 'Hispanic'
all_us_df$variable[all_us_df$variable == 'B17001_002'] <- 'Income less poverty level'

write.csv(all_us_df, paste(years[k],'_acs_df.csv',sep=''))
print(paste('fileout',years[k],sep=''))
rm(all_us_df)
rm(lists)
}`
