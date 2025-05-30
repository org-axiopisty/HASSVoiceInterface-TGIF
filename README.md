# HASSVoiceInterface-TGIF
(or: **<mark style="background: #FFB8EBA6;">Home Assistant Voice Interface - Thank G-d it's Friday</mark>)**

> [!info] Names and Local Customs
> I just picked this name for the repo because it's descriptive, easy to spot in a list of directories, and since Friday is arguably one of the first voice assistants for HASS that is evolving, Friday may become part of our collective folklore and I do think she deserves a nod! But this is still bootstrapping this repository and KB, so I'm making decisions so they don't block me from doing what I can do to add value to the work being done.

This is becoming a collection of code snippets, samples, starters, and lessons learned from the (in this contributor's opinion) [legendary thread called Friday's Party, started by NathanCu](https://community.home-assistant.io/t/fridays-party-creating-a-private-agentic-ai-using-voice-assistant-tools/855862), the intrepid explorer that has been generous enough to share with everyone.

This repository of resources and references and experimentation is also an [ObsidianMD Vault](https://obsidian.md) and if you are not familiar with #ObsidianMD I will give a brief explainer: Obsidian is a knowledge management application that allows you to take notes, write, record thoughts or ideas in freeform datasets called a vaults. They provide easy inter-document linking, comprehensive metadata, embedding, and is generally excellent at arranging knowledge.

The added bonus of opening this repository in Obsidian (you don't have to, everything here will be text, markdown formatted text, or media files, all things easily understood by humans and machines) the metadata in yaml frontmatter and incoming and outgoing links are something that any LLM immediately benefits from, so this `git`repository is also a knowledge base, and, soon, a valuable RAG environment that you can leverage with your own AI tools! Ask Codex or Cursor with access to this vault and hopefully the friction will evaporate and allow people to contribute to this effort and it might be the first time for some of us. Feel free to assist wherever you can, we're still figuring out what this looks like but for a Rev A iteration 1, this contributor (@emory) thinks this is a great starting point, but not necessarily the best for everyone. We'll iterate and adapt as we go, it'll be a hoot.

# Contents:

- [[Replicating Friday — A Guide to Building an Agentic Home Assistant AI Voice Assistant]] (currently embedded in this README)
- Initial summary of the thread in the Community forum, tablesetting
- code snippets and examples will be refactored out into their own individual atomic blocks as markdown files containing code blocks for !inclusion in other documents or notes
- reference material useful in the pursuit of advancing speech interfaces into HomeAssistant and beyond
- exported chats from troubleshooting with an AI co-pilot
- documentation that supplements everything else here


## Assumptions

Most of what is here will be very specific for implementing speech and voice interfaces into HomeAssistant, and typically using HomeAssistant's button in their App, a [[VoicePE Puck]], or other device.

### Compute and Recommendations

> [!TIP] Oh, here's an example of an Embedded Note
> The section on Compute Recommendations is actually in a dedicated note [[Generalized Compute Recommendations]], which renders seamlessly when viewing this README. But we'll get to separate out all code snippets and examples and discoveries and then arrange them and refer to them any number of ways, and as this repository continues to be a valuable resource for people it will become even more useful to the machines we build.

![[Generalized Compute Recommendations]]

![[Auxiliary Compute for AI]]


## Shout-outs and Appreciation

[Home Assistant](https://www.home-assistant.io) for creating #HASS — a household automation platform, and double plus appreciation for them coming up with a sustainability model that has been able to ensure we have a platform like this. Their promotion of technology and new features has driven Voice and Speech AI interfaces into focus in a way others have not. Their VoicePE devices are affordable and for a Rev A of a product, quite good! 

HomeAssistant is the backbone of household automation for many households, and it also allows users to present their system into other home automation software and frameworks too; e.g. there is a bridge that allows you to ferry in your HomeAssistant devices and sensors into an Apple HomeKit home for example, and selectively presenting a simplified interface for less-technical users that is also accessible via HomePods or any device running Siri is a definite quality of life improvement for many of our household's end-users.

Obviously this is entirely inspired by the work of @NathanCu (gitHub uid?) for leading this roundtable and hacking his way to greatness so we can build better human-oriented technology that can augment and enable our families in ways we all enjoy discovering.

***
## Administrivia

A few notes suitable for the bottom of a readme.

![[License]]


> [!TIP] Security and Privacy Harms
> ![[hygiene-for-code]]
