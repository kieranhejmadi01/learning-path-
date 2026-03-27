---
agent: agent
---
This repository is described in the `copilot-instructions.md` file in the `.github` directory. This repository contains a collection of tutorials, install guides written in the form of markdown for different platforms (i.e. servers-and-cloud-computing). The webpage is generated using `hugo`. Currently the limitation is that the site does not have the ability to display challenges to users. 

## Task

The task is to generate a new challenges section that broadly follows the archetype in the `multi-tool-install-guide` directory. It should be a separate section on the home page for challenges that is placed next to the install guide link ![homepage](./homepage.jpg). Users will be able to navigate from the homepage to a separate challenges section whereby challenges will be rendered in a tile grid format similar to that of other content sections.  

The new challenges section should accept raw markdown and generate the html is an identical way to the install guides. An example challenge has been given in `.github/prompts/Architecture-Insight-Dashboard.md`. 

You are working on a separate branch so you must NOT switch branches or attempt to push anything upstream. 

## Helpful files to reference

I recommend completely ignoring all files within the `content/learning-paths` directory, you will need to look into the `tools`, `themes`, `assets`, `archetypes` directories to understand the structure. 