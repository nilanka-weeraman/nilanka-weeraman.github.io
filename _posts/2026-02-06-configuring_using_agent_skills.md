---
layout: post
title: " Configuring and using Agentic Skills "
date: 2026-02-06 05:00:00 +1030
---
Simon Wilson [predicted that skills will be a bigger deal than MCP](https://simonwillison.net/2025/Oct/16/claude-skills/) s five months ago ! With agent skills, the coding CLI s had been given the ability to become more generic agents. <br>

Meantime, I was in the midst of changing jobs and migration and in the lookout for new opportunities in a different country. <br>

I've been using Chatgpt projects, Gemini Gems to customize my resume to suite different roles. A project or gem has a context determined by the files, instructions you give and you can call that specialized project to continue work , such as tailoring a CV for a new job description based on your past experience. The approach has worked well and I assume, many people use chatgpt or gemini like that. <br>

Also, in the meantime, I came across a great comprehensive open source project called [AI-Hawlk _29k github stars_](https://github.com/feder-cr/Jobs_Applier_AI_Agent_AIHawk) that can generate CVs in bulk, given a URL of a job posting. I tried to use it, but it require following a certain framework, which needed some time to setup. <br>

I was also facinated by the power of latex and how latex compiled pdf s (including CV s) look professional. It would be really hard to match the styles and typesetting with a word processing software. So I decided to ask ChatGPT & Gemini to customize CV and generate a latex code, which should be compiled to generate a CV. <br>

My simple planned workflow was ; Upload documents related to past experience, projects to ChatGPT project / Gemini Gem scope -> Customize instructions for ChatGPT / Gems on how to tailor CV -> _For each application_ Upload Job Description document / copy paste content -> Ask ChatGPT / Gemini to generate the latex code for a CV -> Compile latex code and proof read -> Adjustments -> Final CV & Cover letter. <br>

Thus I used codex to build a streamlit app that can intake a latex code and generate a PDF. <br>

Actually, my first option was to build this as a browser based app. I tried ChatGPT canvas ( with 5.1), Gemini Canvas ( 3 pro), Claude artefacts & Google AI Studio's app builder ( 3 pro) at the same time. To my great surprise none of them worked. It appears that setting up the latex to pdf conversion on browser has complicated dependencies on a Canvas like web app ( from downloading and using them browser runtime ). After spending some time on this, I realised it'd be better off to spend time on creating the app as a streamlit than taking a web development 101 course. To make the app available on the go, I used github codespaces as a runtime, which worked pretty well. [Credit to Simon agan !](https://simonwillison.net/tags/github-codespaces/) <br>
The app interface looks like below ; <br>

### How to get started
In basic, skills are simply instructions that you give to carryout a specific task to AI Agent.<br>
It is similar to a Custom GPT, Gemini GEM where you set the context by uploading files, giving specific instructions for a curated output. Compared to MCPs or tool calls skills are supposed to use less tokens as agent can retrieve skill info on the fly by reading skills.md. Only the skill metadata is known to agent upfront.

Good news is that, there are skill creators ! Very similar to how you meta prompt to create a desired prompt, there is a meta skill to create a specific skill. <br>

In Codex <br>
go to /skills

```/skills  use skills to improve how Codex performs specific tasks

enter

```1. List skills            Tip: press $ to open this list directly.
2. Enable/Disable Skills  Enable or disable skills.```

For the first time when you go to 2. Enable/Disable Skills, you'd see <br>

``` [x] Skill Creator    Create or update a skill
 [ ] Skill Installer  Install curated skills from openai/skills or other repos```

If you want to create a skill ( like tailoring a cv, choose 'Skill Creator' ). It is pretty intuitive and easy to create when you articulate it clearly ( pretty much prompt engineering ) <br>

OpenAI provides a skill library that you can choose and install from. 

in codex prompt type $skill-installer which is another _skill_ <br>

```I’m using the skill-installer skill. Its list-skills.py script needs network access, and this environment won’t allow me to run it 
  with escalation.

  Please run this command locally and paste the output here:

  python "C:\Users\DELL\.codex\skills\.system\skill-installer\scripts\list-skills.py"```

  Then tell me which skills you want installed.

Inside 'Skill Installer' you can choose what to install. For example, if Agent wants to read PDFs, create PDFs , .docx etc.. then openAI has 


### Anatomy of skill specification
