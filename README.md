# Stance detection - vaccination in Poland - geotwitter

## 1. Downloading data 
For this purpose, we used 2 scraped data sources: 
- stweet;
- Twitter API (tweepy).

From the 'stweet' we downloaded all tweets that would allow us to define the stance for vaccinations among Poles.

From the Twitter API, we collected information about users who liked tweets.

The process of retrieving this data was initiated during the implementation of the AMC task.

## 2. Preprocessing data

From the raw data, we created a table per user with his attitude towards vaccination 