# Data Modeling with PostgreSQL

## Introduction
A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analytics team is particularly interested in understanding what songs users are listening to. Currently, they don't have an easy way to query their data, which resides in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.

In this project, we create a Postgres database with tables designed to optimize queries on song play analysis and an ETL pipeline to ingest data into the tables.

## Getting Started
To get started, the first step is to create database and tables. 
    
    python create_tables.py
  
Next, run the ETL pipeline to ingest data into the tables.
    
    python etl.py
    
## Database Schema
Database consists of following tables. 

- songplays: Records in log data associated with song plays
- users: Users in the app
- songs: Songs in music database
- artists: Artists in music database
- time: Timestamps of records in songplays broken down into specific units

It is a star schema with `songplays` as the fact table and `users`, `songs`, `artitsts`, `time` being dimension tables.

![Schema](Schema.png)

## ETL pipeline
ETL pipeline consists of processing two sets of data files - song data and log data available in respective folders. The pipeline processes all the files one-by-one. It reads the JSON records, extracts the relevant fields, transforms them if required, and inserts them into respective tables.

1. Song data
Each file in the song data consists of song details and the artist details. 
    
        {
          "num_songs": 1,
          "artist_id": "ARJIE2Y1187B994AB7",
          "artist_latitude": null,
          "artist_longitude": null,
          "artist_location": "",
          "artist_name": "Line Renaud",
          "song_id": "SOUPIRU12A6D4FA1E1",
          "title": "Der Kleine Dompfaff",
          "duration": 152.92036,
          "year": 0
        }
    
From this file, we extract columns for the `songs` table and `artists` table.

*Table: songs*
|song_id|title|artist_id|year|duration|
|-------|-----|---------|----|--------|
|SOUPIRU12A6D4FA1E1|Der Kleine Dompfaff|ARJIE2Y1187B994AB7|-|152.92036

*Table: artists*
|artist_id|name|location|longitude|latitude|
|---------|----|--------|---------|--------|
|ARJIE2Y1187B994AB7|Line Renaud|-|-|-|

2. Log data
Log files contain events of the song plays by a user.
    
        {
          "artist": "Pavement",
          "auth": "Logged In",
          "firstName": "Sylvie",
          "gender": "F",
          "itemInSession": 0,
          "lastName": "Cruz",
          "length": 99.16036,
          "level": "free",
          "location": "Washington-Arlington-Alexandria, DC-VA-MD-WV",
          "method": "PUT",
          "page": "NextSong",
          "registration": 1540266185796.0,
          "sessionId": 345,
          "song": "Mercy:The Laundromat",
          "status": 200,
          "ts": 1541990258796,
          "userAgent": "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4\"",
          "userId": "10"
        }

From this file, we extract fields for `songplays`, `users` and `time` tables. 

*Table: time*

The `ts` field is transformed into elements of `time` table.

|start_time|hour|day|week|month|year|weekday|
|----------|----|---|----|-----|----|-------|
|2018-11-29| 00:00:57.796000|0|29|48|11|2018|3|

*Table: users*

User details are extracted into this table.

|user_id|first_name|last_name|gender|level|
|-------|----------|---------|------|-----|
|10|Sylvie|Cruz|F|free|

*Table: songplays*

The song ID and artist ID will be retrieved by querying the songs and artists tables to find matches based on song title, artist name, and song duration time. Rest of the fields are extracted from the log data file.

|songplay_id|start_time|user_id|level|song_id|artist_id|session_id|location|user_agent|
|-----------|----------|-------|-----|-------|---------|----------|--------|----------|
|1|2018-11-29 00:00:57.796000|10|free|SOUPIRU12A6D4FA1E1|ARJIE2Y1187B994AB7|345|Washington-Arlington-Alexandria, DC-VA-MD-WV|Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4|
