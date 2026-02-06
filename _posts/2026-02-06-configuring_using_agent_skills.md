---
layout: post
title: " Configuring and using Agentic Skills "
date: 2026-02-06 05:00:00 +1030
---
Simon Wilson [predicted that skills will be a bigger deal than MCP](https://simonwillison.net/2025/Oct/16/claude-skills/) s five months ago ! With agent skills, the coding CLI s had been given the ability to become more generic agents. <br>

Meantime, I was in the midst of changing jobs and migration and in the lookout for new opportunities in a different country. <br>

I've been using Chatgpt projects, Gemini Gems to customize my resume to suite different roles. A project or gem has a context determined by the files, instructions you give and you can call that specialized project to continue work , such as tailoring a CV for a new job description based on your past experience. The approach has worked well and I assume, many people use chatgpt or gemini like that. <br>

Also, in the meantime, I came across a project that can generate cvs in bulk, given a URL of a job posting. I tried to use it, but it require following a certain framework, which needed some time to setup. <br>

I was also facinated by the power of latex and how latex compiled pdf s (including CV s) looks neat and professional. It would be really hard to match the styles and typesetting with a word processing software. So I decided to ask ChatGPT & Gemini to customize cv and generate a latex code, which should be compiled to generate a CV. <br>

Thus I used codex to build a streamlit app that can intake a latex code and generate a PDF. <br>
Actually, my first option was to build this as a browser based app. I tried ChatGPT canvas ( with 5.1), Gemini Canvas ( 3 pro), Claude artefacts & Google AI Studio's app builder ( 3 pro) at the same time. To my great surprise none of them worked. It appears that certain latex to pdf conversion libraries cannot be downloaded to browser. After spending some time on this, I realised it'd be better off to spend time on creating this as a streamlit app. To make the app available on the go, I used github codespaces as a runtime, which worked pretty well. [Credit to Simon agan !](https://simonwillison.net/tags/github-codespaces/) <br>
The app interface looks like below ; <br>