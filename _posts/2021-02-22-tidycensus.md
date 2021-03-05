---
title: 'Using Tidycensus package to extract ACS 5-year estimates'
date: 2021-02-22
permalink: /posts/2021/02/tidycensus/

---

I have a new appreciation for studies using ACS data! 

I'm using ACS data in my third dissertation and was finding downloading all the ACS data from Census.gov to be a big pain. Bring in the Tidycensus package to the rescue. Along with some other packages below, I was able to download multiple years for all states and multiple variables.

```library(tidycensus)
library(tidyverse)
library(data.table)
library(sf)```

First you are going to need a free Census API key (just google that). Load it into R like so:
```census_api_key("your key number here")```

To see what variables I wanted to include, I use the load_variables command

```v16 <- load_variables(2016, "acs5", cache = TRUE)
View(v16)```

Now, I had some trouble exporting the geometries needed to create a shapefile. However, this simple command seemed to fix that:
```options(tigris_use_cache = FALSE)```

I then set about creating a massive Do-loop, because I am old school. First I set my vectors:

```years <- c(2009,2011,2013,2016)

us <- c("AL","AK","AZ","AR","CA","CO","CT","DE","FL","GA","HI","ID",
        "IL","IN","IA","KS","KY","LA","ME","MD","MA","MI","MN","MS","MO",
        "MT","NE","NV","NH","NJ","NM","NY","NC","ND","OH","OK","OR","PA",
        "RI","SC","SD","TN","TX","UT","VT","VA","WA","WV","WI","WY")```

Now for the do-loop. Creating a list allows me to store the results in a list for each state within the year I'm running. I'll then merge all the lists to create one shapefile.

```for(k in 1:length(years)){
  lists <- list()
 for (i in 1:length(us)){
    acs_df <- get_acs(geography = "tract", 
                       variables = c("B02001_001","B02001_002","B02001_003","B02001_004"), # all my variables of interest
                                     "B02001_005","B02001_006","B02001_007","B03001_001"),
                                     "B03001_003","B17001_002"),
                 state=paste(us[i]),
                 geometry=TRUE, # I want the geometries so I can export to a shapefile
                 year = years[k])
    acs_df$geometry <- st_cast(acs_df$geometry,"MULTIPOLYGON") # for some reason some polygons were single and some were multi, so here I make them all multi
    lists[[i]] <- as.data.frame(acs_df) # save it to my list
    
    print(paste('list_',us[i],sep='')) # so i have something to know its working
}
all_us_df <- rbindlist(lists) # bind all the states together into one SPDF

all_us_df$variable[all_us_df$variable == 'B02001_001'] <- 'Total race' # re-name all the variables
all_us_df$variable[all_us_df$variable == 'B02001_002'] <- 'White'
all_us_df$variable[all_us_df$variable == 'B02001_003'] <- 'Black'
all_us_df$variable[all_us_df$variable == 'B02001_004'] <- 'AIAN'
all_us_df$variable[all_us_df$variable == 'B02001_005'] <- 'Asian'
all_us_df$variable[all_us_df$variable == 'B02001_006'] <- 'NAPI'
all_us_df$variable[all_us_df$variable == 'B02001_007'] <- 'Other'
all_us_df$variable[all_us_df$variable == 'B03001_001'] <- 'Total ethnicity'
all_us_df$variable[all_us_df$variable == 'B03001_003'] <- 'Hispanic'
all_us_df$variable[all_us_df$variable == 'B17001_002'] <- 'Income less poverty level'

all_us_df <- all_us_df[,c(1:4,6)] # remove unnecessary columns

write.csv(all_us_df, paste(years[k],'_acs_df.csv',sep=''))
st_write(all_us_df, paste(years[k],'_acs_2.shp',sep='')) # export the shapefiles
print(paste('fileout',years[k],sep=''))
rm(all_us_df) # remove to save space on my computer
rm(lists)
}```

