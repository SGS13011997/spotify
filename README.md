# spotify
![spotify_logo](https://github.com/user-attachments/assets/b5930f43-82a5-4d00-a4e3-349ac8a2a67d)
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);

-- EDA

Select count(*) From spotify;

Select Count(Distinct artist) From spotify;

Select Distinct album_type From spotify;

Select Max(duration_min) From spotify;

Select Min(duration_min) From spotify;

Select * From spotify 
Where duration_min = 0;

Delete From spotify
Where duration_min = 0;

Select * From spotify 
Where duration_min = 0;

Select Distinct channel From spotify;

Select Distinct most_played_on From spotify;

----------------------------------
-- Data Analysis - Easy Category
----------------------------------

-- Retrieve the names of all tracks that have more than 1 billion streams.

Select * 
From spotify
Where stream > 1000000000;

-- List all albums along with their respective artists.

Select Distinct album , artist
From spotify
Order by 1;

-- Get the total number of comments for tracks where licensed = TRUE.

Select Sum(comments) as Total_Comments
From spotify
Where licensed = 'true';

-- Find all tracks that belong to the album type single.

Select track
From spotify
Where album_type = 'single';

-- Count the total number of tracks by each artist.

Select artist , Count(track) as total_tracks
From spotify
Group by artist 
Order by total_tracks Desc;

------------------------------------
-- Data Analysis - Medium Category
------------------------------------

-- Calculate the average danceability of tracks in each album.

Select album , Avg(danceability) as avg_danceability
From spotify
Group by album
Order by avg_danceability desc;

-- Find the top 5 tracks with the highest energy values.

Select track , Max(energy) highest_energy
From spotify
Group by track
Order by highest_energy desc
Limit 5;

-- List all tracks along with their views and likes where official_video = TRUE.

Select track , Sum(views) as total_views , Sum(likes) as total_likes
From spotify 
Where official_video = 'true'
Group by track
Order by total_views desc;

-- For each album, calculate the total views of all associated tracks.

Select album , track , Sum(views) as total_views
From spotify
Group by album , track
Order by total_views desc;

-- Retrieve the track names that have been streamed on Spotify more than YouTube.

Select * 
From (Select track ,
	Coalesce(Sum(Case when most_played_on = 'Youtube' then stream end),0) as youtube_stream , 
	Coalesce(Sum(Case when most_played_on = 'Spotify' then stream end),0) as spotify_stream
From spotify
Group by track)x
Where spotify_stream > youtube_stream
And 
youtube_stream <> 0;

------------------------------------
-- Data Analysis - Advance Category
------------------------------------

-- Find the top 3 most-viewed tracks for each artist using window functions.

Select artist , track , total_views 
From (Select artist , track ,  sum(views) as total_views , 
Dense_Rank() Over(Partition by artist Order by sum(views) desc) as rn
From spotify
Group by artist , track) x
Where rn <=3;

-- Write a query to find tracks where the liveness score is above the average.

Select track , artist , liveness
From spotify
Where liveness > (select avg(liveness) From spotify);

-- Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.

With Cte 
as (
Select album , Max(energy) as highest_energy_value , Min(energy) as lowest_energy_value
From spotify
Group by album
)
Select album , (highest_energy_value - lowest_energy_value) as energy_difference 
From Cte
Order by energy_difference desc;

-- Find tracks where the energy-to-liveness ratio is greater than 1.2.

SELECT artist , track , ROUND((energy / NULLIF(liveness, 0))::NUMERIC, 2) AS energy_liveness_ratio
FROM spotify
WHERE (energy / NULLIF(liveness, 0)) > 1.2;

-- Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.

SELECT 
    artist,
    track,
    album,
    views,
    likes,
    SUM(likes) OVER (ORDER BY views DESC) AS cumulative_likes
FROM 
    spotify;
