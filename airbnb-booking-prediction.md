Airbnb Booking Rate Prediction
================

# The goal of the project was to predict the highest booking rate for Airbnb listings. Therefore, this report
# summarizes the initial data exploratory findings and the various models implemented and tested to obtain the
# highest accuracy.

## Part 1: Cleaning the data

1)  Loading the data

<!-- end list -->

``` r
airbnbTrain = read_csv("Data/airbnb_train_x.csv")
airbnbTestX <- read_csv("Data/airbnb_test_x.csv")
airbnb_train_y <- read_csv("Data/airbnb_train_y.csv")
airbnb <- rbind(airbnbTrain, airbnbTestX)
```

2)  Cleaning Availability Columns, converting to numeric

<!-- end list -->

``` r
#availibility_365
airbnb$availability_365 <- as.numeric(airbnb$availability_365)

#availibility_30
airbnb$availability_30 <- as.numeric(airbnb$availability_30)
```

    ## Warning: NAs introduced by coercion

``` r
airbnb$availability_30[is.na(airbnb$availability_30)] <- 0

#availibility_60
airbnb$availability_60 <- as.numeric(airbnb$availability_60)
```

    ## Warning: NAs introduced by coercion

``` r
airbnb$availability_60[is.na(airbnb$availability_60)] <- 0

#availibbility_90
airbnb$availability_90 <- as.numeric(airbnb$availability_90)
```

    ## Warning: NAs introduced by coercion

``` r
airbnb$availability_90[is.na(airbnb$availability_90)] <- 0
```

3)  Cleaning bathrooms, bedrooms, bed\_type, bedrooms, beds

<!-- end list -->

``` r
#Bathrooms
airbnb$bathrooms <- as.numeric(airbnb$bathrooms)
```

    ## Warning: NAs introduced by coercion

``` r
airbnb$bathrooms[is.na(airbnb$bathrooms)] <- 0

#Bed_Type
airbnb$bed_type <- as.character(airbnb$bed_type)
airbnb$bed_type[airbnb$bed_type == '100%'] <- 'Airbed'
airbnb$bed_type[airbnb$bed_type == '81%'] <- 'Airbed'
airbnb$bed_type <- as.factor(airbnb$bed_type)

#bedrooms
airbnb$bedrooms <- as.numeric(airbnb$bedrooms)
```

    ## Warning: NAs introduced by coercion

``` r
airbnb$bedrooms[is.na(airbnb$bedrooms)] <- 1.363

#beds
airbnb$beds <- as.numeric(airbnb$beds)
```

    ## Warning: NAs introduced by coercion

``` r
airbnb$beds[is.na(airbnb$beds)] <- 1.89
```

4)  Cleaning host\_identity\_verified, host\_is\_superhost,
    host\_listings\_count

<!-- end list -->

``` r
#host_identity_Verified
airbnb$host_identity_verified <- as.character(airbnb$host_identity_verified)
airbnb$host_identity_verified[is.na(airbnb$host_identity_verified)] <- 'NA'
airbnb$host_identity_verified <- as.factor(airbnb$host_identity_verified)

#host_is_superhost
airbnb$host_is_superhost <- as.character(airbnb$host_is_superhost)
airbnb$host_is_superhost[airbnb$host_is_superhost == 't'] <- TRUE
airbnb$host_is_superhost[airbnb$host_is_superhost == 'f'] <- FALSE
airbnb$host_is_superhost[airbnb$host_is_superhost == 'Bed,Bath&Bike in Sunny Santa Monica'] <- FALSE
airbnb$host_is_superhost[airbnb$host_is_superhost == 'Pristine Mid-Century Modern w 180° Canyon View!'] <- FALSE
airbnb$host_is_superhost <- as.character(airbnb$host_is_superhost)
airbnb$host_is_superhost[is.na(airbnb$host_is_superhost)] <- 'NA'
airbnb$host_is_superhost <- as.factor(airbnb$host_is_superhost)

#host_listings_count
airbnb$host_listings_count <- as.numeric(airbnb$host_listings_count)
```

    ## Warning: NAs introduced by coercion

``` r
airbnb$host_listings_count[is.na(airbnb$host_listings_count)] <- 9.617
```

5)  Cleaning maximum\_nights, minimum\_nights, price, security\_deposit

<!-- end list -->

``` r
#maximum_nights
airbnb$maximum_nights <- as.numeric(airbnb$maximum_nights)
airbnb$maximum_nights[airbnb$maximum_nights > 1125] <- 1125
airbnb$maximum_nights[is.na(airbnb$maximum_nights)] <- 668

#minimum_nights
airbnb$minimum_nights<- as.numeric(airbnb$minimum_nights)
airbnb$minimum_nights[is.na(airbnb$minimum_nights)] <- 1
airbnb$minimum_nights[airbnb$minimum_nights == 0] <- 1
airbnb$minimum_nights[airbnb$minimum_nights == 100000000] <- 1

#price
airbnb$price = as.numeric(gsub("[\\$,]", "", airbnb$price))
airbnb$price[is.na(airbnb$price)] <- 0
airbnb$price[airbnb$price == 0] <- 1

#Security_deposit
airbnb$security_deposit = as.numeric(gsub("[\\$,]", "", airbnb$security_deposit))

#PriceMean for 
airbnb$PriceMean <- airbnb$price
airbnb$PriceMean[is.na(airbnb$PriceMean)] <- 154.7
```

6)  Cleaning amenities\_count, host\_verification\_count, host\_age,
    host\_response\_rate, host\_response\_time

<!-- end list -->

``` r
#amenities_count
airbnb$amenities_count<- 
  sapply(airbnb$amenities, function(x) length(unlist(strsplit(as.character(x), ','))))

#host_verifications_count
airbnb$host_verification_count <-
  sapply(airbnb$host_verifications, function(x) length(unlist(strsplit(as.character(x), ','))))

#host_age
airbnb$host_since <- as.Date(airbnb$host_since)
airbnb$host_age <- as.integer(difftime(Sys.Date(), airbnb$host_since, units = 'weeks'))
airbnb$host_age[is.na(airbnb$host_age)] <- 303

#host_responsne_time
airbnb$host_response_time <- as.character(airbnb$host_response_time)
airbnb$host_response_time[airbnb$host_response_time == 'f' ] <- NA
airbnb$host_response_time[airbnb$host_response_time == '' ] <- NA
airbnb$host_response_time <- as.factor(airbnb$host_response_time)

#host_response_rate

airbnb$host_response_rate <- gsub('%', '', airbnb$host_response_rate)
airbnb$host_response_rate <- as.numeric(airbnb$host_response_rate)
```

    ## Warning: NAs introduced by coercion

``` r
airbnb$host_response_rate_binned <- bin(airbnb$host_response_rate, nbins = 3,
                                        labels =c('low','mid','high'), na.omit = FALSE)
```

7)  Cleaning review\_age, accomodates, guests\_included, extra\_people

<!-- end list -->

``` r
#review_age
airbnb$first_review <- as.Date(airbnb$first_review)
airbnb$review_age <- as.integer(difftime(Sys.Date(), airbnb$first_review, units = 'weeks'))
airbnb$review_age[is.na(airbnb$review_age)] <- 219.9

# accomodates
airbnb$accommodates <- as.numeric(airbnb$accommodates)
```

    ## Warning: NAs introduced by coercion

``` r
airbnb$accommodates[is.na(airbnb$accommodates)] <- 3.506

# guests_included 
airbnb$guests_included <- as.numeric(airbnb$guests_included)
airbnb$guests_included[airbnb$guests_included <= 0] <- 1

# extra_people 
airbnb$extra_people = as.numeric(gsub("[$]", "", airbnb$extra_people))
```

8)  Property Type

<!-- end list -->

``` r
# property_type

## Cleans property_type
airbnb$property_type <- toupper(airbnb$property_type)
airbnb$property_type <- gsub('BED AND BREAKFAST', 'BED & BREAKFAST', airbnb$property_type)

## Creates separate dataframe for property_type and frequencies
property_typeDF <- data.frame(sort(table(airbnb$property_type), ascending = TRUE))
names(property_typeDF)[1] <- "Property"
property_typeDF$Property <- as.character(property_typeDF$Property)

# Creates a list of the properties that should be grouped in miscellaneous 
# (properties that have < 300 instn)
misc_properties = c()
for (i in 1:nrow(property_typeDF)) {
  if (property_typeDF$Freq[i] < 300) {
    misc_properties = c(misc_properties, property_typeDF$Property[i])
  }
}

# Fills in "MISCELLANEOUS" for those property types 
group <- function(property) {
  if (property %in% misc_properties) {
    property = 'MISCELLANEOUS'
  }
  return (property)
}
airbnb$property_type <- sapply(airbnb$property_type, group)

# Checks the grouping
sort(table(airbnb$property_type))
```

    ## 
    ##     GUEST SUITE        BUNGALOW BED & BREAKFAST           OTHER      GUESTHOUSE 
    ##             628             720             784             938            1001 
    ##            LOFT   MISCELLANEOUS       TOWNHOUSE     CONDOMINIUM           HOUSE 
    ##            2090            2096            3090            4144           35352 
    ##       APARTMENT 
    ##           61342

``` r
airbnb$property_type <- as.factor(airbnb$property_type)
```

8)  Cleaning cancellation\_policy, instant\_bookable,
    is\_location\_exact, requires\_licence

<!-- end list -->

``` r
# cancellation_policy 
airbnb$cancellation_policy <- as.character(airbnb$cancellation_policy)
airbnb$cancellation_policy[airbnb$cancellation_policy == '1.0' ] <- NA
airbnb$cancellation_policy[airbnb$cancellation_policy == '5.0' ] <- NA
airbnb$cancellation_policy[airbnb$cancellation_policy == '2.0' ] <- NA
airbnb$cancellation_policy <- as.character(airbnb$cancellation_policy)
airbnb$cancellation_policy[is.na(airbnb$cancellation_policy)] <- 'NA'
airbnb$cancellation_policy <- as.factor(airbnb$cancellation_policy)

# instant_bookable
airbnb$instant_bookable[airbnb$instant_bookable == 'f'] <- FALSE
airbnb$instant_bookable[airbnb$instant_bookable == 't'] <- FALSE
airbnb$instant_bookable[airbnb$instant_bookable == '$150.00'] <- FALSE
airbnb$instant_bookable <- as.factor(airbnb$instant_bookable)

#is_location_exact and 
airbnb$is_location_exact <- as.factor(airbnb$is_location_exact)
airbnb$is_location_exact[is.na(airbnb$is_location_exact)] <- TRUE

#requires_licence
airbnb$requires_license <- as.character(airbnb$requires_license)
airbnb$requires_license[airbnb$requires_license==""] <- "f"
airbnb$requires_license <- as.factor(airbnb$requires_license)
airbnb$requires_license[is.na(airbnb$requires_license)] <- FALSE
```

9)  Cleaning cleaning\_fee

<!-- end list -->

``` r
#cleaning Fees
airbnb$cleaning_fee = as.numeric(gsub("[\\$,]", "", airbnb$cleaning_fee))
```

    ## Warning: NAs introduced by coercion

``` r
#cleaning_fee_zero
airbnb$cleaning_fee_zero <- airbnb$cleaning_fee
airbnb$cleaning_fee_zero[is.na(airbnb$cleaning_fee_zero)] <- 0
```

10) Cleaning the market variable

<!-- end list -->

``` r
# Market
airbnb$market <- as.character(airbnb$market)
majorCities <- c("South Bay, CA", "Malibu", "Other (Domestic)", "NA's", "Monterey Region" , 
                 "North Carolina Mountains", "East Bay, CA", "Seattle", "Denver", 
                 "Boston", "Portland",
                 "San Diego", "San Francisco", "Chicago" , 
                 "Nashville" , "New Orleans" ,"D.C.", "Austin",
                 "Los Angeles", "New York")

m <- c()
for(i in 1:112208){
  if(airbnb$market[i] %in% majorCities){
    m <- c(m, airbnb$market[i])
  }
  else{
    m <- c(m,'Other')
  }
}
airbnb$marketCheck <- m
airbnb$marketCheck <- as.factor(airbnb$marketCheck)
```

11) Text Mining on access variable

<!-- end list -->

``` r
#using access string
airbnb$access <- as.character(airbnb$access)
matrix <- create_matrix(airbnb$access, language="english", removeSparseTerms = 0.90,
                        removeStopwords=TRUE, removeNumbers=TRUE, stemWords=TRUE, 
                        stripWhitespace=TRUE, toLower=TRUE)
mat <- as.matrix(matrix)
airbnb <- cbind(airbnb, mat)
```

12) Adding a description variable

<!-- end list -->

``` r
#length of decription / avg length of each word
airbnb$description <- as.character(airbnb$description)

airbnb$descriptionLength <- 
  sapply(airbnb$description, function(x) length(unlist(strsplit(as.character(x), ' '))))

words <- strsplit(airbnb$description, ' ')
word_lengths <- lapply(words, str_length)
avgLength <- lapply(word_lengths, mean)

airbnb$descriptionLength <- as.numeric(airbnb$descriptionLength) / as.numeric(avgLength)
```

13) Cleaning interaction

<!-- end list -->

``` r
#interaction
airbnb$interaction <- as.character(airbnb$interaction)
interactionMatrix <- create_matrix(airbnb$interaction, language="english", removeSparseTerms = 0.90,
                        removeStopwords=TRUE, removeNumbers=TRUE, stemWords=TRUE, 
                        stripWhitespace=TRUE, toLower=TRUE)
interactionMat <- as.matrix(interactionMatrix)
airbnb <- cbind(airbnb, interactionMat)
```

14) Neighbourhood\_overview variable

<!-- end list -->

``` r
#neighbourhood_overview
airbnb$neighborhood_overview <- as.character(airbnb$neighborhood_overview)
nMatrix <- (as.matrix(create_matrix(airbnb$neighborhood_overview, language="english", 
                                removeSparseTerms = 0.90,removeStopwords=TRUE,
                                removeNumbers=TRUE,stemWords=TRUE, 
                        stripWhitespace=TRUE, toLower=TRUE)))
airbnb <- cbind(airbnb, nMatrix)
```

15) Transit variable

<!-- end list -->

``` r
#transit
airbnb$transit <- as.character(airbnb$transit)
tansitMatrix <- as.matrix(create_matrix(airbnb$transit, language="english", removeSparseTerms = 0.90,
                        removeStopwords=TRUE, removeNumbers=TRUE, stemWords=TRUE, 
                        stripWhitespace=TRUE, toLower=TRUE))
airbnb <- cbind(airbnb, tansitMatrix)
```

16) Cleaning security\_deposit

<!-- end list -->

``` r
#security_deposit_zero
airbnb$security_deposit_zero <- airbnb$security_deposit
airbnb$security_deposit_zero[is.na(airbnb$security_deposit_zero)] <- 0

#SecurityMean using mean to replace security_deposit values
airbnb$securityMean <- airbnb$security_deposit
airbnb$securityMean[is.na(airbnb$securityMean)] <- 307.1
```

17) CLeaning security\_deposit

<!-- end list -->

``` r
# security_deposit
airbnb$security_deposit_binned <- airbnb$security_deposit
## Cleans security deposit 
# table(airbnb$security_deposit)
airbnb$security_deposit_binned = as.numeric(gsub("[\\$,]", "", airbnb$security_deposit_binned))

# bin function 
bin <- function(value) {
  if (!(is.na(value))){
    if (value <= 99) {
      value = "very low"
    } else if (value <= 100) {
      value = "low"
    } else if (value <= 200) {
      value = "medium"
    } else if (value <= 450) {
      value = "high"
    } else {
      value = "very high"
    }
  }
  else {
    value = "NA"
  }
  return (value)
}

# Bins 
airbnb$security_deposit_binned <- as.numeric(airbnb$security_deposit_binned)
airbnb$security_deposit_binned <- sapply(airbnb$security_deposit_binned, bin)
# Checks the binning
airbnb$security_deposit_binned <- as.character(airbnb$security_deposit_binned)
airbnb$security_deposit_binned <- as.factor(airbnb$security_deposit_binned)
sort(table(airbnb$security_deposit_binned))
```

    ## 
    ##       low  very low    medium very high      high        NA 
    ##     10694     11354     14164     14712     15502     45782

18) Cleaning weekly\_price, monthly\_price, weekly\_available,
    monthly\_available

<!-- end list -->

``` r
#weekly price
airbnb$weekly_price = as.numeric(gsub("[\\$,]", "", airbnb$weekly_price))

#monthly price
airbnb$monthly_price = as.numeric(gsub("[\\$,]", "", airbnb$monthly_price))

#weely_available
airbnb$weekly_available <- ifelse(is.na(airbnb$weekly_price), FALSE, TRUE)
airbnb$weekly_available <- as.factor(airbnb$weekly_available)

#monthly_available
airbnb$monthly_available <- ifelse(is.na(airbnb$monthly_price), FALSE, TRUE)
airbnb$monthly_available <- as.factor(airbnb$monthly_available)
```

## Part 2: Modelling

``` r
airbnbTrain <- airbnb[1:100000,]
airbnbTestX <- airbnb[100001:112208,]
```

``` r
airbnbTrain$high_booking_rate <- airbnb_train_y$high_booking_rate
airbnbTrain$high_booking_rate <- as.factor(airbnbTrain$high_booking_rate)
```

``` r
valid_instn = sample(nrow(airbnbTrain), 0.25*nrow(airbnbTrain))
airbnbValidDF <- airbnbTrain[valid_instn,]
airbnbTrainDF <- airbnbTrain[-valid_instn,]
```

Random
Forest

``` r
rfMod <- randomForest(high_booking_rate~ accommodates + availability_365 + availability_60 + 
                      bathrooms + availability_90+ availability_30 + bedrooms + bed_type + 
                        host_identity_verified + host_is_superhost +
                        host_listings_count + maximum_nights +
               minimum_nights + amenities_count + host_verification_count + host_age + review_age + price + guests_included + extra_people + property_type + instant_bookable + cancellation_policy + is_location_exact + requires_license + cleaning_fee_zero + price/guests_included + bedrooms/guests_included + bathrooms/guests_included + host_response_rate_binned + marketCheck + help + need + phone + questions + available + around + block + blocks + easy + minute + minutes + subway + walk + walking + private + use + will + bathroom + entire + kitchen + living  +  PriceMean + security_deposit_zero/price + weekly_available + monthly_available + cleaning_fee_zero/price + security_deposit_binned + host_response_time, 
                      data=airbnbTrainDF,
                      mtry=3,
                      ntree=1000,
                      importance=TRUE,
               na.action = na.roughfix
               )

rfPredsValid <- predict(rfMod,airbnbValidDF)
print(class_performance(table(rfPredsValid, airbnbValidDF$high_booking_rate))[1])
```

    ## [1] 0.8057441

The accuracy on validation data is 80.7%.
