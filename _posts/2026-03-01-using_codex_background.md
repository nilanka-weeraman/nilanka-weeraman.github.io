---
layout: post
title: " Using Codex as Background Agent for Front End app "
date: 2026-03-01 21:00:00 +1030
---
__This is a continuation of my previous writeup of using [Agentic Skills](https://nilanka-weeraman.github.io/configuring_using_agent_skills/) ___<br>

I had setup a workflow to tailor CV s with Codex & Streamlit ( on python virtual env ). It was working smoothly.<br>
Whenever I wanted changes to a CV, I would;<br>
- manually alter the latex code
- ask codex to change specific parts of the cv by providing section, subsection information.<br>

The only problem is that, I've to the prompting & validation in two interfaces in above setup.
- manual alteration - entirely carried out in Streamlit app
- custom instructions - first I prompt in Codex cli then rendered output in streamlit app.<br>

I thought of enhancing current streamlit app and use Codex in the background. A crude Application with Agentic backend, without an orchestrator like Langchain.<br>
While developing this feature , I got the chance to explore what is happening under-the-hood in Codex. <br> 
Efficient wise this is not good as it consumes lot of tokens , as Codex is a general agent. Other complications such as permission / sandboxing needs to be carefully thought of. The advantage of using langchain would be to be efficient and fast. However I still think that using Codex ( with skills & compute env ) will get the MVP version of the app much faster. Once you settle with app features, the agent workflow could be converted to a langchain like workflow implmentation. That is something I'll be trying next. <br>

Now my interactin sequence looks like below <br>

```mermaid
sequenceDiagram
    JD->>+Codex: user ask to customize cv
    Note left of Codex : Input - jd.md/jd.pdf
    Codex->>+Latex : load skill & generate cv code
    Note right of Latex : output - cv_v1.tex
    Note left of Latex : create - codex_session_ID

    Latex->>+Rendered_cv: user render PDF & validate
    Note right of Rendered_cv : output - cv_v1.pdf

    Rendered_cv->>+Latex: user manually edit Latex
    Note right of Latex : modified - cv_v1.tex

    Latex->>+Rendered_cv: user render PDF & validate
    Note right of Rendered_cv : modified - cv_v1.pdf


    Rendered_cv->>+Codex: user provide instruction to edit
    Note left of Latex : resume codex_session_ID
    

    Codex->>+Latex : resume codex session & modify latex
    Note right of Latex : modifid : cv_v1.tex
    
    
    Latex->>+Rendered_cv: user render PDF & validate
    Note right of Rendered_cv : modified - cv_v1.pdf

```