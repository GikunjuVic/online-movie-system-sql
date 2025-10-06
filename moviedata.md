## 📑 Table of Contents

1. [Problem 1: Most watched movies](#problem-1-most-watched-movies)  
2. [Problem 2: Most watched Genre Sections](#problem-2-most-watched-genre-sections)  
3. [Problem 3: Total number of movies watched per day](#problem-3-total-number-of-movies-watched-per-day)  
4. [Problem 4: List of distinct movies watched per day and number of times](#problem-4-list-of-distinct-movies-watched-per-day-and-number-of-times)  
5. [Problem 5: Distinct movies watched in a month](#problem-5-distinct-movies-watched-in-a-month)  
6. [Problem 6: Total number of movies watched in a month and the titles](#problem-6-total-number-of-movies-watched-in-a-month-and-the-titles)

## Problem 1: Most watched movies 

**Insight:** This query lists the top-performing movies by total view count. It helps identify the titles are driving the most engagement and content that might deserve featured placement helping the organization in promotion and marketing. 

## SQL Query

```sql

SELECT title AS title,
       sum(views) AS `SUM(views)`
FROM
  (select *
   from bunny_videos) AS virtual_table
GROUP BY title
ORDER BY `SUM(views)` DESC
LIMIT 5000;

```
## Problem 2: Most watched Genre Sections

**Insight:** This query surfaces the most engaging sections on the PesaFlix platform based on actual user watch activity.
Sections at the top of this list are high-performing placements that attract more viewer attention. This can influence homepage layout, recommendation strategies, and promotional content placement.

## SQL Query

```sql

SELECT section_name AS section_name,
       total_watched AS total_watched
FROM
  (SELECT ts.description AS section_name,
          COUNT(s.video_guid) AS total_watched
   FROM main.stats_video_charts s
   JOIN content c ON c.video_id = s.video_guid
   JOIN main.template_section_contents tsc ON tsc.content_id = c.id
   JOIN main.template_sections ts ON ts.id = tsc.template_content_id
   JOIN main.templates t ON t.id = ts.template_id
   WHERE s.watchtimechart > 0
   GROUP BY ts.description
   ORDER BY total_watched DESC
   LIMIT 100) AS virtual_table
LIMIT 1000;

```

## Problem 3: Total number of movies watched per day 

**Insight:** This query provides a daily watch activity timeline, revealing how engagement fluctuates across days. This is useful to the organization to show the watching trend, identify high traffic days and engagement stability. 

## SQL Query

```sql

SELECT 
    DATE(watchdatetime) AS watch_date, 
    COUNT( video_guid) AS total_movies_watched
FROM 
    main.stats_video_charts
WHERE 
    watchtimechart > 0
GROUP BY 
    watch_date
ORDER BY 
    watch_date;

```

## Problem 5: List of distinct movies watched per day and the no of times 

**Insight:** This query provides a day-by-day breakdown of viewing diversity and engagement intensity on the PesaFlix platform. It helps identify how many unique movies users are watching daily, and which titles attract repeat views.

## SQL Query

```sql

SELECT 
    watch_date, 
    COUNT(DISTINCT video_guid) AS total_movies_watched,
    GROUP_CONCAT(
        CASE 
            WHEN times_watched > 1 THEN CONCAT(c.name, ' (', times_watched, ' times)')
            ELSE c.name
        END 
        ORDER BY c.name ASC SEPARATOR ', ') AS movies_list
FROM (
    SELECT 
        DATE(watchdatetime) AS watch_date, 
        video_guid, 
        COUNT(video_guid) AS times_watched
    FROM 
        main.stats_video_charts
    WHERE 
        watchtimechart > 0
    GROUP BY 
        watch_date, video_guid
) AS movie_counts
JOIN 
    content c ON c.video_id = movie_counts.video_guid
GROUP BY 
    watch_date
ORDER BY 
    watch_date DESC;

```

## Problem 5: Distinct movies watched in a month 

**Insight:** This query shows the distinct number of movies watched in a month and lists their titles. This helps in analyzing whether users are repeating movies or exploring new movies. This helps in content planning and delivery. 

## SQL Query

```sql

SELECT 
    DATE_FORMAT(s.watchdatetime, '%Y-%m') AS month, 
    COUNT(DISTINCT s.video_guid) AS distinct_videos_watched,  
    GROUP_CONCAT(DISTINCT c.name ORDER BY c.name) AS movies_watched 
FROM 
    main.stats_video_charts s
JOIN 
    main.content c ON s.video_guid = c.video_id 
WHERE 
    s.watchtimechart > 0  
GROUP BY 
    month  
ORDER BY 
    month;

```

## Problem 6: Total number of movies watched in a month and the titles. 

**Insight:** This query shows the total number of movies watched in a month while also showing the titles. However, due to the bulkiness of the data, it is visualized in a bar chart showing the total number of movies. It reveals how active the platform is over time and helps detect content performance trends at the monthly level.

## SQL Query

```sql

SELECT 
    DATE_FORMAT(s.watchdatetime, '%Y-%m') AS month, 
    COUNT(s.video_guid) AS total_movies_watched,
    GROUP_CONCAT(c.name ORDER BY c.name) AS movies_watched
FROM 
    main.stats_video_charts s
JOIN 
    main.content c ON s.video_guid = c.video_id 
WHERE 
    s.watchtimechart > 0  
GROUP BY 
    month  
ORDER BY 
    month;

```
