---
layout: post
title: " Using Codex as Background Agent for Front End app "
date: 2026-03-01 21:00:00 +1030
---

Now I have setup a workflow setup with Codex & Streamlit ( on python virtual env ). It was working smoothly.<br>
Whenever I wanted changes to a CV, I would;<br>
- manually alter the latex code
- ask codex to change specific parts of the cv by providing section, subsection information.<br>

The only problem is that, I've to the prompting & validation in two interfaces in above setup.
- manual alteration - entirely carried out in Streamlit app
- custom instructions - first I prompt in Codex cli then rendered output in streamlit app.<br>

I thought of enhancing current streamlit app and use Codex in the background. A crude Application with Agentic backend. The process was indeed very interesting and got to know more about how Codex work under to hood. <br>

