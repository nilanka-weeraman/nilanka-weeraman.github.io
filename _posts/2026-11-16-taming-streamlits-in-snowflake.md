---
layout: post
title: "Streamlit in Snowflake is Great , Until.."
date: 2026-11-16 10:00:00 +0930
tags: [dataengineering, dataproducts, optimisations, streamlit]
---

_Work In Progress Article_<br>

If you had tried Streamlit Apps, you'd agree with me that it is a very promising framework to build data products, which was a bottleneck for any data analytics, science team. Frameworks like [Streamlit](https://github.com/streamlit/streamlit) and [Gradio](https://github.com/gradio-app/gradio) fit nicely into building the UI layer or the tip of the iceberg of a complex data system. <br>
Seeing the promise, Snowflake acquired Stremlit in 2022 and still maintains it as an open source project. And since then Snowflake provide hosted Streamlit service and seems to be [adopting it heavily for internal tooling](https://www.snowflake.com/en/blog/engineering/scaling-enterprise-analytics-streamlit-apps/). <br>

Streamlit in Snowflake is so seamless and addictive in some sense. You can quickly create an app and then do a one-click deploy. No more complex configuration, instance sizing selection. This of course proves to be a way of accumulating debt and hedeache for platform teams. Most successful POC s end up being in production and its hard to take time off for re-architecting after adoption. <br>

At my current orgnisation we are going through the same experience. <br>

The data platform on Snowflake is just started to reap its benefits to wider organisation. The data is consumed by very data literate users who have a knack of building tools on whichever platform they are provided with. So we introduced Streamlit to these users and it triggered a WOW momemnt for most users. Fast forward three months, we have close to 50 streamlit apps, mostly in personal schemas, but the team is not slowing down. <br>
On other hand, Snowflake made changes to and now prefers Snowpark Container Service (SCS) to host Streamlits over their legacy Warehouse based hosting. With this our Snowpark container costs started to climb. Why ? <br>

Currently Streamlit in Snowflake (SiS) doesn't have auto suspend type feature, which is available for its Warehouses. So an idle SiS App would go on for 3 days , making the hosting compute pool active before triggering a suspension. Unfortunately there is no auto spend timer for SiS on SCS. <br>

While thinking any workarounds, I stumbled upon that Snowflake's telemetry tables gathering metrics about running services and Jobs. This is pretty much AWS's Cloudwatch equivalent, but in a ... Table of course. The event table is - `snowflake.events.telemetry`. 
