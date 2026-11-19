---
layout: post
title: "Make Streamlit in Snowflake Great Again"
date: 2026-11-16 10:00:00 +0930
tags: [dataengineering, dataproducts, optimisations, streamlit]
---

_Work In Progress Article_<br>

If you had tried Streamlit Apps, you'd agree with me that it is a very promising framework to build data products, which was a bottleneck for any data analytics, science team.<br> Frameworks like [Streamlit](https://github.com/streamlit/streamlit) and [Gradio](https://github.com/gradio-app/gradio) fit nicely into building the UI layer or the tip of the iceberg of a complex data system. <br>
Seeing the promise, Snowflake acquired Stremlit in 2022 and still maintains it as an open source project. And since then Snowflake provide hosted Streamlit service and seems to be [adopting it heavily for internal tooling](https://www.snowflake.com/en/blog/engineering/scaling-enterprise-analytics-streamlit-apps/). <br>

Streamlit in Snowflake is so seamless and addictive in some sense. <br>
Why - You can quickly create an app and then do a one-click deploy. No more complex configuration, instance sizing selection. This of course proves to be a way of accumulating debt and hedeache for platform teams. Most successful POC s end up being in production and its hard to take time off for re-architecting after adoption. <br>

At my current orgnisation we are going through the same experience. <br>

The data platform on Snowflake is just started to reap its benefits to wider organisation. At my orgnisation, the data is consumed by very data literate users who have a knack of building tools on whichever platform they have. The data team introduced Streamlit to these users and it triggered a WOW momemnt for most users. Fast forward three months, we have close to 50 streamlit apps, mostly in personal schemas, but the team is not slowing down. <br>
On the other hand, Snowflake made changes to Streamlit deployment by preferring Compute Pools over Warehouse hosting. <br> This comes with a caveat though. Streamlit in Warehouse has a 15 min idle timeout, but Streamlit on Snowpark Container Service (SCS) has a 3-day idle suspension. According to documentation, SCS objective is to host long running services, thus SiS is also considered such long running service. <br> But most users built Streamlits for personal use which often used during working hours and idle thereafter. We can't switch between Warehouse hosted Streamlit & Container hosted as there are library compabitlity issues. <br> This result short lived Streamlits running in containers for longer times - driving SCS costs<br>N
Now we knew the problem, but what are the solutions. <br>

Snowflake documentation is very silent on this and all they mention as of Nov-2026 is Streamlit apps should include codes to emit events to Snowflake telemetry tables. Thus making it possible path to detect user activity. But this require every developer including logging to telemetry, which would take a little bit of time for adoption. <br>

While thinking about workarounds, I stumbled upon that Snowflake's telemetry keep track of metrics about the services it runs. This is pretty much AWS's Cloudwatch equivalent, but in a ... Table of course. The event table is - `snowflake.telemetry.events`. And one of those metrics were `streamlit_app_active_sessions` !! <br>
The stragest thing is that in any Snowflake documentation I couldn't find any details about this metric. Snowflake documentation says, that you can configure logging to collect event data to telemetry tables, that's all <br>
Anyways after running a few experiments, I realised `streamlit_app_active_sessions` value is 0 when there are no active browser opend with the streamlit app. As long a Streamlit is open in a browser tab the value would be 1. <br>

If your Streamlit uses SCS, the container will shut down after 3 days of no activity. This is strage considering Snowflake providing more granular Warehouse suspend controls. The 3 days is not configurable and we are at mercy of Snowflake. This is of course as of Nov-2026. <br>
Thus, untill Snowflake comes up with a solution, I implemented the following mechanism to detect inactive sessions and suspend Compute Pool. <br>

- Query event table as <br>
  ```sql
  select      
    timestamp,
    resource_attributes['snow.service.name']::VARCHAR as service_name,
    record['metric']['name']::VARCHAR metric,
    value
  from
  snowflake.telemetry.events where timestamp  >= timestampadd('minute',-1*$idle_mins, current_timestamp())
      and scope['name']::VARCHAR like 'sis-streamlit-app'
      and resource_attributes['snow.compute_pool.name']::VARCHAR like $cp 
      and record['metric']['name']::VARCHAR like 'streamlit_app_active_sessions'
  order by timestamp desc
  ```
The query result in very granular values for active session count for each streamlit app in your account.
- Compute a summary for any active sessions and suspend the Pool is value is zero.
  
  Simple as that. <br>
  A snowflake procedure and a task is all you need. <br>

  Also this require your Apps to be spread across different compute pools. Otherwise a opened browser page will keep the entire Pool up. As a pre-requsite , it is required to comeup with several compute pools based on Streamlit traffic. <br>