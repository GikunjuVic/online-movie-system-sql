## Problem 1 : Most watched Genre Sections.

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

## Problem 2: Most watched movies 

**Insight:** This query lists the top-performing movies by total view count. It helps identify the titles are driving the most engagement and content that might deserve featured placement helping the organization in promotion and marketing. 

## SQL Query

``sql

SELECT title AS title,
       sum(views) AS `SUM(views)`
FROM
  (select *
   from bunny_videos) AS virtual_table
GROUP BY title
ORDER BY `SUM(views)` DESC
LIMIT 5000;

```
