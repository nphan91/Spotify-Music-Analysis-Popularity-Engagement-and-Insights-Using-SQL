# Spotify Music Analysis: Popularity Engagement and Insights Using SQL
![Spotify Logo](https://github.com/nphan91/Spotify-Music-Analysis-Popularity-Engagement-and-Insights-Using-SQL/blob/main/Spotify%20Logo.png)

## Overview and Purpose
This project focuses on conducting a comprehensive analysis of music tracks, albums, and artist performance using data-driven insights. By leveraging SQL queries and analytics, the project aims to uncover patterns and trends in audience engagement, track popularity, and artist productivity. The results will guide marketing, playlist curation, and content strategies tailored to enhance user satisfaction and maximize platform success.
## SQL Queries
## Database Link: https://github.com/nphan91/Spotify-Music-Analysis-Popularity-Engagement-and-Insights-Using-SQL/blob/main/cleaned_dataset.csv
## Review data in the table
```sql
SELECT COUNT(*)  
FROM [dbo]. [spotify];
```sql
SELECT COUNT(DISTINCT [artist]) 
FROM  [dbo]. [spotify];
```sql
SELECT DISTINCT [album_type] 
FROM  [dbo]. [spotify];
```sql
SELECT MAX([duration_min])  
FROM  [dbo]. [spotify];
```sql
SELECT MIN([duration_min])  
FROM  [dbo]. [spotify];
## There are songs with 0 minutes in the table, delete these data from the table 
```sql
SELECT *  
FROM  [dbo]. [spotify]
WHERE [duration_min] = 0;

DELETE FROM  [dbo]. [spotify]
WHERE [duration_min] = 0;
SELECT *  
FROM  [dbo]. [spotify]
WHERE [duration_min] = 0;
```
# 1. Analyzing Popular Tracks
-	Query: Retrieve the names of all tracks with more than 1 billion streams.
-	Purpose: Identify highly popular tracks to understand trends and user preferences for better marketing and playlist curation.
```sql
SELECT *
FROM  [dbo]. [spotify]
WHERE [stream]>1000000000
```
# 2. Cataloging Albums and Artists
-	Query: List all albums along with their respective artists.
-	Purpose: Create a comprehensive view of the music library for reporting or cataloging purposes.
```sql
SELECT DISTINCT [album], [artist]
FROM  [dbo]. [spotify]
```
# 3. Licensed Tracks' Engagement
-	Query: Get the total number of comments for tracks where licensed = TRUE.
-	Purpose: Measure engagement on licensed tracks to assess audience interaction and licensing performance.
```sql
SELECT COUNT([Comments]) AS 'Total_comments'
FROM  [dbo]. [spotify]
WHERE [licensed] = 'TRUE';
```
# 4. Identifying Singles
-	Query: Find all tracks that belong to the album type "single."
-	Purpose: Separate singles from albums for focused analysis on track performance.
```sql
SELECT  *
FROM  [dbo]. [spotify]
WHERE [album_type] LIKE 'single';
```
# 5. Artist Contribution Analysis
-	Query: Count the total number of tracks by each artist.
-	Purpose: Understand artist productivity and catalog size for insights into their contribution to the platform.
```sql
SELECT  [artist], COUNT([Track]) AS 'Total_number_of_tracks'
FROM  [dbo]. [spotify]
GROUP BY  [artist];
```
# 6. Album Danceability Insights
-	Query: Calculate the average danceability of tracks in each album.
-	Purpose: Determine the rhythmic appeal of albums to target dance-focused audiences.
```sql
SELECT  
	[album],
	AVG ([danceability]) AS 'Avg_danceability'
FROM  [dbo]. [spotify]
GROUP BY  [album]
ORDER BY AVG([danceability]) DESC;

```
# 7. Most Energetic Tracks
-	Query: Find the top 5 tracks with the highest energy values.
-	Purpose: Highlight high-energy tracks for playlists, promotions, or workout-related content.
```sql
SELECT TOP 5
	[track],
	MAX ([energy]) AS 'Max_energy'
FROM  [dbo]. [spotify]
GROUP BY  [track]
ORDER BY MAX ([energy]) DESC;
```
# 8. Official Video Engagement
-	Query: List all tracks along with their views and likes where official_video = TRUE.
-	Purpose: Analyze the performance of tracks with official videos to assess visual content's impact on engagement.
```sql
SELECT  
	[track],
	SUM([views]) AS 'total_views', 
	SUM([likes]) AS 'total_likes'
FROM  [dbo]. [spotify]
WHERE [official_video] LIKE 'TRUE'
GROUP BY  [track]
ORDER BY SUM([views]) DESC;
```
# 9. Aggregated Album Views
-	Query: For each album, calculate the total views of all associated tracks.
-	Purpose: Measure overall album popularity by aggregating track-level views.
```sql
SELECT  
	[Album],
	[Track],
	SUM([Views]) AS 'total_views' 
FROM  [dbo]. [spotify]
GROUP BY [Album], [Track]
ORDER BY SUM([Views]) DESC;
```
# 10. Platform Preference Analysis
-	Query: Retrieve the track names that have been streamed on Spotify more than YouTube.
-	Purpose: Compare platform-specific streaming performance to guide marketing and distribution strategies.
```sql
SELECT  
   	[Track],
    SUM(CASE WHEN [most_playedon] = 'Spotify' THEN [Stream] ELSE 0 END) AS 'spotify_streams',
    SUM(CASE WHEN [most_playedon]= 'YouTube' THEN [Stream] ELSE 0 END) AS 'youtube_streams'
FROM  
    [dbo].[spotify]
GROUP BY  
   [Track]
HAVING  
    SUM(CASE WHEN [most_playedon] = 'Spotify' THEN [Stream] ELSE 0 END) > 
	SUM(CASE WHEN [most_playedon]= 'YouTube' THEN [Stream] ELSE 0 END)
ORDER BY  
    spotify_streams DESC;
```
# 11. Artist-Specific Top Tracks 
-	Query: Find the top 3 most-viewed tracks for each artist using window functions.
-	Purpose: Identify an artist's most successful tracks to prioritize in playlists or promotions.
```sql
WITH RankedTracks AS (
    SELECT  
        artist,
        track,
        views,
        RANK() OVER (PARTITION BY artist ORDER BY views DESC) AS rank
    FROM  
        dbo.spotify
)
SELECT  
    artist,
    track,
    views
FROM  
    RankedTracks
WHERE  
    rank <= 3
ORDER BY  
    artist, rank;
```
# 12. Above-Average Liveness
-	Query: Write a query to find tracks where the liveness score is above the average.
-	Purpose: Discover tracks with higher liveness for live performance or immersive music recommendations.
```sql

SELECT  
   [Track],
   [Artist],
   [Liveness]
FROM  
    [dbo].[spotify]
WHERE  
    [Liveness] > (SELECT AVG([Liveness]) FROM [dbo].[spotify])
ORDER BY  
    [Liveness] DESC;

```
# 13. Energy Value Range Calculation
-	Query: Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.
-	Purpose: Analyze the energy variation within albums to identify dynamic or consistently energetic collections.
``` sql
WITH EnergyStats AS (
    SELECT  
        album,
        MAX(energy) AS max_energy,
        MIN(energy) AS min_energy
    FROM  
        dbo.spotify
    GROUP BY  
        album
)
SELECT  
    album,
    max_energy,
    min_energy,
    (max_energy - min_energy) AS energy_range
FROM  
    EnergyStats
ORDER BY  
    album;
```
# 14. Energy-to-Liveness Ratio
-	Query: Find tracks where the energy-to-liveness ratio is greater than 1.2.
-	Purpose: Pinpoint tracks with a balance favoring energy over liveness for curated playlists.
```sql
SELECT  
    track,
    artist,
    energy,
    liveness,
    (energy / NULLIF(liveness, 0)) AS energy_to_liveness_ratio
FROM  
    dbo.spotify
WHERE  
    (energy / NULLIF(liveness, 0)) > 1.2
ORDER BY  
    energy_to_liveness_ratio DESC;
```
# 15. Cumulative Likes Analysis
-	Query: Calculate the cumulative sum of likes for tracks ordered by the number of views
-	Purpose: Track audience appreciation trends for popular tracks to guide future content strategies.
```sql
SELECT  
    [Track],
    [Artist],
    [Views],
    [Likes],
    SUM(likes) OVER (ORDER BY views DESC) AS cumulative_likes
FROM  
    dbo.spotify
ORDER BY  
     [Views] DESC;
```


