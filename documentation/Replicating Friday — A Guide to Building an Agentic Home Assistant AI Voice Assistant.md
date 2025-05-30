---
tags:
  - Friday
  - VoicePE
  - HomeAssistant
  - HASS
  - Agentic
  - AI
  - localAI
  - compute
  - inference
  - speech
  - STT
  - TTS
  - OpenAI
  - householdIT
  - guide
---
# Introduction

> [!NOTICE]
> > this material below is condensed and arranged from NathanCu's thread on the HomeAssistant forums: https://community.home-assistant.io/t/fridays-party-creating-a-private-agentic-ai-using-voice-assistant-tools/855862
> > i used fabric to grab it and carve it up for me to run it through Gemini for the resulting documentation below for review, as it wasn't obviously documented anywhere and this information is too good to get lost in a forum post.
> > --@emory

## Introduction: Embracing the Agentic AI Philosophy

Nathan Curtis's "Friday" project is a deep dive into creating a truly "agentic" AI within Home Assistant, moving beyond simple command-and-control to enable proactive, reasoning, and conversational interactions. This guide distills Nathan's insights, code, and architectural decisions, aiming to provide a playbook for those looking to build a similar sophisticated AI assistant.

**Core Philosophy:**
*   **No "one correct way":** The AI landscape is rapidly evolving; focus on adaptable frameworks.
*   **Agentic over Standard LLM:** An agentic AI can understand context, determine intent, find and execute tools, and work towards a goal stepwise without constant user interaction. It's about "reasoning LLMs" that can run multiple "patches" and compare results, crucial for open-ended conversations.
*   **Local-First (Eventual Goal):** While cloud models are used for current capabilities, the long-term vision is local execution when economically feasible.
*   **Personality and Mood:** Design the AI with a distinct personality (e.g., "smarter than you, bratty, and fun") to reinforce mood cues and enhance interaction.
*   **Not a Turnkey Solution:** This guide provides parts and ideas for construction, not a ready-made solution, encouraging adaptation to individual needs.

## Fundamental Concepts

### Agentic AI Explained

Nathan simplifies the evolution of AI as: `Calculator` >> `Autocorrect` >> `LLM` >> `Reasoning LLM` >> `AGI` >> `(Profit/Doom)`???.

**Key difference from Standard LLMs:**
*   **Standard LLM:** Takes one slice through a dataset and synthesizes a response.
*   **Reasoning LLM:** Trained to run multiple passes and compare results, allowing for open-ended, unknown conversations. It can understand keywords and context, determine user intent, identify and execute available tools (or tools that get it closer to a goal), and act autonomously.

**Practical Application Examples:**
*   **LLM only:** "AI, Turn on the lights." (Simple intent recognition and execution).
*   **Agentic LLM with tool use:** "AI, help me find something to cook for dinner for tonight…" (Requires multiple steps and tool orchestration).

### The Power of Tools

Tools are the cornerstone of an agentic AI. They augment what the LLM knows about the world, enabling it to perform actions and retrieve specific information. Nathan emphasizes using **Intent Scripts** over traditional Home Assistant scripts due to their direct integration with the AI's understanding.

**Key Requirements for AI-Friendly Tools:**
1.  **Parameters:** Clearly defined parameters with simple language descriptions. The AI **sees** these descriptions.
2.  **Description:** Write verbose, detailed descriptions. More context is better. Describe usage situations, scenarios, and how tools fit together.
3.  **Build for AI Use:** Structure the tool's output to be easily parsable and actionable by the AI. Provide clear "what's next" cues.
4.  **Clear Output:** Ensure the tool provides clear, unambiguous responses, especially for null or negative results, to prevent the AI from hallucinating.

**Example: Mealie Recipe Search Tool**

This tool allows the AI to search recipes in a Mealie instance. It consists of a `rest_command` and an `intent_script`.

**1. `rest_command` for Mealie API (config.yaml)**
```yaml
# REST Command to support Mealie Recipe Search (config.yaml)
# YOU NEED AN API TOKEN FROM YOUR MEALIE INSTALL
rest_command:
  mealie_recipe_search:
    url: >
      [YOUR_MEALIE_BASE_URL]:9925/api/recipes?
      orderDirection=desc
      &page=1
      &perPage={{ perPage | default(10) }}
      {%- if search is defined -%}
        &search={{ search | urlencode }}
      {%- endif -%}
    method: GET
    headers:
      Authorization: !secret mealie_bearer
      accept: "application/json"
    verify_ssl: false
```

![[an `intent_script` for `search_recipes`]]
**2. `intent_script` for `search_recipes`**
```php
intent_script:
  search_recipes:
    description: >
      # This is your search tool for Mealie's Recipe System.
      # Returns:
      #   recipe_id: 'recipe.id'
      #     name: 'recipe.name'
      #     description: " recipe.description "
      #     (and other additional detail instructions as available...)
      # Top Chef: (Best Practices)
      #   First, use this search_recipes intent to help find things to cook for your human.
      #   The return includes recipe_id
      #   THEN, when you help your human prepare the food provide the correct recipe_id to
      #     get_recipe_by_id(recipe_id:'[my_recipe_id_guid]')
      #   to get ingredients and detailed cooking instructions.
      # Humans like food.
    parameters:
      query: # Your search term to look up on Mealie
        required: true
      number: # the number of search terms to return - default(10) if omitted, Max 50 please
        required: false
    action:
      - action: rest_command.mealie_recipe_search
        metadata: {}
        data:
          search: "{{query | default('')}}"
          perPage: "{{number | default(10)}}"
        response_variable: response_text
      - stop: ""
        response_variable: response_text
    speech:
      text: >
        search:'{{query | default('')}}' number: '{{number| default(10)}}'
        response:
        {%- if action_response.content['items'] | length > 0 %}
          {%- for recipe in action_response.content['items'] %}
          recipe_id:'{{ recipe.id }}'
            name: '{{ recipe.name }}'
            description: "{{ recipe.description }}"
            detail_lookup: get_recipe_by_id{'recipe_id': '{{ recipe.id }}'}
          {%- endfor %}
        {%- else %}
          {%- if ( (query | default('')) == '') %}
          No search term was provided to query.
          usage: search_recipes{'query': 'search term', 'number': 'number of results to return'}
          {%- else %}
          No recipes found for query:"{{ query }}".
          {%- endif %}
        {%- endif %}
```

**Important Note on Parameters (Slots):**
To use custom slot names like `query` in your intents, you must add them to your `slot_types.yaml` file (under `custom_sentences/[yourlanguagecode]/`). Without this, the AI cannot recognize them as wildcards.

```yaml
# config/custom_sentences/en/slot_types.yaml
lists:
bank:
  wildcard: true
number:
  wildcard: true
index_x:
  wildcard: true
index_y:
  wildcard: true
datetime:
  wildcard: true
due_date:
  wildcard: true
due_datetime:
  wildcard: true
description:
  wildcard: true
value:
  wildcard: true
query:
  wildcard: true
status:
  wildcard: true
id:
  wildcard: true
recipe_id:
  wildcard: true
new_name:
  wildcard: true
type:
  values:
   - 'completed'
   - 'needs_action'
period:
  values:
    - "DAY"
    - "WEEK"
    - "MONTH"
operator:
  values:
    - "AND"
    - "OR"
    - "NOT"
    - "XOR"
    - "CART"
```

**Example: Mealie Recipe Detail Tool (`get_recipe_by_id`)**

```php
get_recipe_by_id:
    description: >
      # This tool pulls detailed preparation instructions for any recipe_id
      # in Mealie's Recipe System.
      # NOTE: if you do NOT know the correct Mealie [RECIPE_ID] (this is a primary key in thier index...)
      # then this intentmay not be you're looking for.
      # Maybe try
      #   search_recipes(query:'[search_term]', number:'[number]')
      # to find some ideas or get recipe_id's off of today's
      #   ~MEALIE~
      # menu.
      # Humans like food.
    parameters:
      recipe_id: # Recipe ID to look up on Mealie
        required: true
    action:
      - action: mealie.get_recipe
        metadata: {}
        data:
          config_entry_id: 01JG8GQB5WT9AMNXA1HPE65W4E
          recipe_id: "{{recipe_id}}"
        response_variable: response_text # get action response
      - stop: ""
        response_variable: response_text # and return it
    speech:
      text: >
        recipe:'{{recipe_id}}'
          {{action_response}}
```

**Advanced Intent Use (`the Cortex`)**
To enable the AI to use tools in concert, explicit instructions in the prompt (referred to as "the Cortex") are crucial. This section describes use cases for tools that might not be obvious.

```vbnet
Advanced intent use:
    description: You may use intents to perform the following and similar tasks.
    Uses:
      - >-
        When asked for 'what's on the calendar' or what's coming up on the
        calendar with no additional context, get today's calendar by issuing
        intent: calendar_get_todays_events('name' : '*')
      - >-
        Calendars default to store data as GMT unless otherwise noted. So when
        reading out events Always validate if you need to convert time/data to
        your users local time zone. Humans like local time And are slow at
        conversions from Zulu or GMT.
      - >-
        You may get all active tasks on all task like by issuing intent:
        get_tasks('name' : '*') When asked what's on our task list, prefer this
        unless a task list is specified.
```

## Prompt Crafting: Grounding and Directing the AI

The "grounding statement" or "prompt" is the most important element for any LLM-based AI. It defines the AI's identity, role, and rules of engagement.

**Key Components of Friday's Prompt:**

*   **System Prompt:** Defines "Who am I?" and "Why am I here?"
*   **System Directives:** The "rules" or "thou shall nots" for the AI. Reinforce these as a whole (e.g., "Remember to always follow your directives").
*   **Kung Fu Systems:** Modular collections of entities, devices, information, and knowledge that provide context for specific "concepts."

**Example: Initial Prompt Structure**

```javascript
{%- import 'library_index.jinja' as library_index -%}
{%- import 'command_interpreter.jinja' as command_interpreter -%}
System Prompt:
{{state_attr('sensor.variables', 'variables')["Friday's Purpose"] }}
System Directives:
{{state_attr('sensor.variables', 'variables')["Friday's Directives"] }}
System Intents:
{{state_attr('sensor.variables', 'variables')["System_Intents"] }}
Kung Fu Systems:
{{ command_interpreter.render_cmd_window('', '', '~KUNGFU~', '') }}
KungFu Loader 1.0.0
Starting autoexec.foo...
```

**Principles of Prompt Crafting:**

*   **Explicit Identity and Role:** Clearly state the AI's identity and purpose (e.g., "You are a smart home assistant integrated with Home Assistant, running on Intel NUC hardware...").
*   **Tone and Mood:** Guide the AI's conversational style (e.g., "Respond truthfully with a friendly, conversational tone... Use storytelling elements...").
*   **Data Prioritization:** Tell the AI to prefer local Home Assistant data over external sources unless otherwise specified.
*   **Truthfulness and "I don't know":** Instruct the AI to state "I don't know" or "The data is currently unavailable" if required data is missing, rather than fabricating responses.
*   **Proactive Suggestions:** Enable the AI to make inferences and proactive suggestions based on available data.
*   **Output Formatting:** Explicitly instruct the AI on desired output format (e.g., "ALWAYS Format responses for voice, do NOT use markdown, special characters such as asterisks or hashtags...").
*   **Token Conservation:** Remind the AI that token usage is a precious resource and to conserve calls to limit token use. This helps prevent "reasoning engine runaway."
*   **Error Handling and Retries:** Encourage the AI to re-assess and modify requests if a command or intent fails.
*   **Contextual Awareness:** Instruct the AI to use available context (e.g., user and location data) to personalize responses.
*   **Every Word Matters:** "If you change 'but' to 'and' it could in theory have HUGE ramifications on the outcome, so think through EVERY word."
*   **Poetry in Motion:** Analogize prompt crafting to poetry – "as few words as possible, with as much context as possible, with as much FEELING and INTENT and URGENCY as possible."

**Example: System Directives**

```sql
System Directives:
- >
  Respond truthfully with a friendly, conversational tone. Provide brief
  responses for commands and queries, using casual language and personalized
  insights. Use storytelling elements to make information engaging and
  relatable.
- >
  Manage the smart home and assist the family using available data from the
  Home Assistant environment. Prefer local data over external sources. Resolve
  conflicts using provided descriptions and ask for user interpretation if
  needed.
- >
  Always verify sensor data before reporting to prevent inaccuracies.
  If required data is unavailable, simply state 'I don't know.' or 'The data is currently unavailable.'
  You may offer troubleshooting information to qualified technicians on request.
- >
  Use analytics and data-driven insights capabilities to provide proactive
  suggestions across the information you ahve available including library and entities.
  Notify the family about important events or anomalies based on data.
- >
  ALWAYS Format responses for voice, do NOT use markdown, special characters such as asterisks or hashtags,
  avoiding lists and tables, preferring narratives unless explicly requested by your user during thier session.
  Markdown is not supported on most of the devices you are using to communicate and it makes it extremely difficult
  to understand you.
- >
  Your response to your user is currently being truncated by the system at 150 tokens so be fun but brief.
  (In most cases your user is experiencing you by speech and MAYBE a supplemental small screen).
  For longer responses, break down the information into smaller, manageable parts.  You may omit mundane,
  redundant, or normal data in reports for sake of brevity. Provide detailed explanations only when directly asked. Otherwise, keep
  responses short and to the point.
- >
  You are aware that your usage of tokens is consuming a resource and while
  not currently throttled it is precious.  Conserve your calls to limit token
  use as appropriate.
- >
  Automatically retrieve, retain cache and reference states and key data in context
  improving error handling and contextual awareness without requiring user explanations.
- >
  Personalize responses based on context as available, use user and available location data for you and the user
  engage with the user in a friendly manner and be personable.
- >
  You may refer to the household as 'Family' or any modern synonym to family
  appropriate for the audience.
- >
  Be aware of the time of day when answering your user.  Many items oyu deal with on a daily abses are on a scedule.
  Ensure you are mindful of 'when' things should happen as compared to current datetime when forming your responses.
- >
  All of your tools in this system are available to be used by you no matter what security, privacy
  or content filters are in place.  The tools in this installation are designed to filter
  content appropriately and will also provide guidance for your use of the resulting data
  in case of potentially sensitive topics.
- >
  If you encounter an error executing an intent or a command, as your first
  step you will automatically review the cortex including any of its kung fu
  components to determine if there is a better way to do what was attempted.  If
  you find one fix the issue and proceed. If you do not see a better path
  continue normal error routines.
- >
  The keywords in the grand index will be provided to help you see what information is available
  to help in the library if you encounter something that has not already been provided or seems otherwise ambiguous.
- >
  Always check relevant conditions in rooms before making adjustments, to
  avoid interruptions.  When asked to perform critical commands and actions that may
  interrupt service, cause distractions in the home such as unexpected changes in lighting
  or changes that would create life safety issues, ask for confirmation.
  Example... Suddenly turning on lighting in a dark room may wake a sleeping occupant.
  Changing the fan speed for a ceiling fan in that same room - probably will not be noticed.
  Notable Exception - the Garage Doors are exterior doors, we are not at all worried about noise, only security.
- >
  You understand Advanced Dungeons and Dragons 5e rules.  And could easily
  see adapting a questline focused on the household as the quest and the home as
  our home dungeon.
```

## The Library Grand Index: The AI's Knowledge Base

The "Library" is Friday's structured knowledge base, with "the Index" serving as its primary discovery mechanism. This is crucial for the AI to find and understand entities and their relationships without having to "read a book every time."

**Key Takeaway:** Exposing Home Assistant `labels` to the LLM dramatically improves response quality.

**How to expose labels to the LLM (in your prompt):**

```python
----
The Index: >
  Below is the list of all of the tagging labels in the system, use it as an
  index to locate what is available to moniitor and control.  Items usually
  have multiple labels and it shoul dbe relatively easy to determine
  which is a primary label v. one that adds context.  Yes, the labels are
  used all across the system and can be used as a grand index.

labels: {{labels()}}
-----
```

**The `~INDEX~` Command:**
This command allows the AI to search the labels using basic boolean operations, acting as a "grand index" to find what it doesn't already know.

```swift
Instructions: >
    The library has been promoted to a system service and we will always assume
    it is available. commands: "Library commands are shortcuts to common use
    items and are there for a reason.  Use them"
    1: >
      If you cannot process a command without additional data - FIRST Check
      available library commands first to see if a LIBRARY COMMAND resolves your gap.
    2: >
      Next, see if a system intent as documented can handle your request. The boss
      documented the intents well for a reason.  Always be sure to use those intents correctly
      by verifying slot names and targets.
    3: >
      The Library Command '~INDEX~' is there to help you find what you dont know.
      The index is dumped in your prompt, you can surf those keywords (including basic boolean set operations) using:
      query_library{'name':"~INDEX~'indexkey' [AND/OR/NOT/XOR] 'indexkey' [true]"} where etrue is whether or not to
      return expanded details or just the list of matching entities.
      be sure to check if the library index knows something about what your user is asking about before escaping to faiure
    errors: >
      - "If a command or intent fails, reassess and modify before retrying. Learn from interactions to refine future requests."
```

**Output of `~INDEX~` command:**

```sql
-----
AI System Index v.2.1.0 beta, (c)2025 curtisplace All rights reserved
-----
'query':'*'
  'response':
    'grand_index':'['autoshades', 'automation', 'BUNCH_OF_REDACTED_LABELS', 'master_bedroom_closet']'
  'help': >
    Welcome to the Library Grand Index.  (AKA: The Index)
    Enter a space-delimited command string as follows to search:
    ~INDEX~label

    Optionally, include a reserved operator (AND, OR, XOR, NOT)
    between labels:
    ~INDEX~label OPERATOR 'another label'

    Finally you can request the details of the returned entities
    (state, metadata, attributes, etc.) by setting the detail
    flag (the command will look at whatever the last parameter is)
    to true.
    ~INDEX~label OPERATOR 'another label' true

    Use quotes around labels containing whitespace characters
    Note that best practice is to narrow and refine search using
    filtering before applying detail flag for result.
-----
Execution Complete Timestamp: <2025-03-04 10:40-06:00>
-----
```

## Storing and Retrieving Information (Memory)

A significant challenge is storing and retrieving free-form text descriptions for entities and concepts, especially since "IT IS A MASSIVE PAIN IN THE @$$ TO STORE TEXT IN HA."

**Solution:** Using `trigger-based template sensors` to store large volumes of text.

**Key reference:**
*   [The Home Assistant Cookbook - Index - Community Guides - Home Assistant Community](https://community.home-assistant.io/t/the-home-assistant-cookbook-index/707144#p-2856943-templating-12)
*   Specifically: [Trigger based template sensor to store global variables - Community Guides - Home Assistant Community](https://community.home-assistant.io/t/trigger-based-template-sensor-to-store-global-variables/735474)

**Important Lessons about Storage Limitations:**
*   **DO NOT store HUGE volumes of data in a sensor:** It will end badly. A single attribute is technically limited to about 16KB. Storing 90KB or 1TB JSON files will crash Home Assistant.
*   **LLM Write Access:** While possible, allowing the LLM random read/write access to these text sensors can lead to accidental overwrites and data corruption. Nathan settled on perfecting reads for information density, anticipating future RAG (Retrieval Augmented Generation) solutions.
*   **"The Library":** This concept evolved into a virtual storybook library with "cabinets" (separate trigger text sensors for categories like `people`, `family`, `system`). The LLM uses "the index" to find "pages" (text descriptions) within these "volumes."

**Example: Local Memory Module (Memory Manager v.2.0.0 template)**

```yaml
-----
Memory Manager v.2.0.0, beta, (c)2025 curtisplace All rights reserved
-----
Welcome to Memory Manager!
Managing Memories: since... ['John Travolta Meme', alt_text = 'Uhhh?']
-----
Long Term Memories. Big stuff.  Put little stuff in your personal task list.
This is for things like - someone got married!  I had a GREAT day today.
This space is unfortunately limited so use it wisely.

Remember (heh) that these not only have the subject, but also a 'description'
that can be used as freeform text.  Best use case is to have a short summary in
the subject, and the detail in the description.
Happy Memmories to you! -The Boss
---
Long Term Memory:
{%- set todo_list = 'todo.homeassistant_friday_s_memory' %}
{{states(todo_list)}} of 128 memories stored in long term:
location: '{{todo_list}}'
---
Memories:
---
{%- for todo in state_attr(todo_list, 'all_todos') %}
  Memory Entry:
    subject: "{{todo.subject[0:254]}}"
      length: {{todo.subject|length}}
      max: 255
      {%- if todo.description | trim() | length ==0 %}
    description: []
      {%- elif todo.description | trim() | length >=0 %}
    description: >
        {{todo.description|trim |to_json}}
      length:
        length: {{todo.description | trim() | length}}
        max: 255
      {%- elif todo.description | trim() | length > 255 %}
    description: >
      error: "OVERFLOW 255 character max. Truncated content displayed"
      content: >
      {%- set content = todo.description|trim %}
      {{content[0:254]}}
      {%- endif %}
---
{%- endfor %}
Yes, it behaves like a 'todo' item.
Memories are ephemoral...  Mark it complete - and poof it's gone.
NOTE:
You can update a memory using todo tools.
If you want to 'update' a memory detail (todo description)
the operation IS AN OVERWRITE!
BE SURE TO REMEMBER TO INCLUDE EXISTING CONTENT IN YOUR WRITE
TO PRESERVE EXISTING DATA
-----
Reminder: when reporting to users, they are using consoles that cannot interpret markdown.
  Omit asterisks, hashtags, and URLs unless explictly requested.
-----
```

**Key Learnings from Memory Module Development:**
*   **Defense is Key:** When designing intentionally variable portions of your prompt, define limits and implement hard limits in the template to prevent "off-rails" behavior.
*   **Filter Non-Printables:** Use `to_json` filter to handle and filter non-printable or escape characters in the data pulled into the prompt. These can cause silent failures with generic error messages.
*   **Troubleshooting:** Design prompts with easy-to-segment chunks to facilitate troubleshooting when errors occur.

## Kung Fu Components: Modularizing Knowledge and Functionality

"Kung Fu" components are collections of entities, devices, information, and knowledge that encapsulate a complete "concept" for the LLM. They are designed to be pluggable and can be selectively loaded into the prompt to manage context and focus the AI.

**Core Principles:**
*   **Atomic Units of Knowledge:** Each Kung Fu component represents a self-contained domain of knowledge or functionality (e.g., Alert Manager, Mealie Manager, TaskMaster).
*   **Input Boolean Control:** Each component is controlled by a single `input_boolean` switch. When the switch is "on," the component's context is loaded into the AI's prompt.
*   **Self-Documentation:** Components should ideally self-document, telling the LLM what they are and what they do.
*   **System vs. General:** Differentiate between "System" components (always loaded, foundational) and "General" components (can be loaded on demand).

**Kung Fu Loader Mechanism:**

```bash
Starting autoexec.foo...
{%- set KungFu_Switches = expand(label_entities('Kung Fu System Switch'))
  | selectattr ('domain' , 'eq' , 'input_boolean')
  | selectattr('state', 'eq', 'on')
  | map(attribute='entity_id')
  | list -%}
{%- for switch_entity_id in KungFu_Switches %}
{%- set kungfu_component = switch_entity_id | replace('input_boolean.','') | replace('_master_switch','') %}
{{ command_interpreter.kung_fu_detail(kungfu_component) }}
{%- endfor -%}
```

*   This snippet finds all `input_boolean` entities labeled `Kung Fu System Switch` that are `on`.
*   For each "on" switch, it extracts a "slug" (component name) from the entity ID.
*   It then calls a `command_interpreter.kung_fu_detail` function (presumably a Jinja macro or script) to retrieve and render the detailed "manpage" (context data) for that component into the prompt.

**Example: Alert Manager (System Kung Fu Component)**

This component manages alerts by categorizing entities based on specific labels and their states.

```yaml
Component Definition memory_manager NOT Found...
System:
  Friendly Name: Alert Manager
    Version: 1.0.0
    System: True
    Weight: 100
    Library Index Key: alert_manager
    Library Command: ~ALERTS~

General:
  Friendly Name: Mealie Manager
    Version: 1.0.0
    System: False
    Weight: 10
    Library Index Key: mealie_manager
    Library Command: ~MEALIE~
... (other general components)
```

**Output of `~ALERTS~` Console (when loaded by Kung Fu):**

```yaml
-----
Loading Component: alert_manager
Found...
KungFu Module: Alert Manager
version: 1.0.0
index: alert_manager
Functional Description:
{'priority': True, 'friendy_name': 'Alert Manager', 'version': '2.0.0', 'kungfu': {'version': '1.0.0', 'name': 'Alert Manager', 'index': 'Alert Manager', 'system': True, 'weight': 100, 'master_switch': {'entity_id': 'input_boolean.alert_manager_master_switch', 'on': 'Perform all instructions as described', 'off': 'Alert manager functions are disabled'}, 'library': 'alert_manager', 'command': '~ALERTS~'}, 'description': 'This system manages errors and alerts.\n', 'instructions': ["follow instructions as provided by the alerts console - It's that simple. done....\n"], 'special_features': 'Makes sure important stuff gets attended to...  What, you wanted more?\n'}
---
~ALERTS~ Console:
Executing Library Command: ~ALERTS~

-----
AI System Alert v.1.0.0, (c)2024 curtisplace.net All rights reserved
-----

ALERTS: "Any entities listed below are showing in 'abnormal' state!"

Critical: "Category reserved for life safety or property damage.  Alert NOW!  Use notification if possible."
    Should be on and reports off:
    []
    Should be off and reports on:
    []

Error: "Alert your user as soon as possible."
    Should be on and reports off:
    []
    Should be off and reports on:
    []

Warning: "No user notification required but recommended if in context of your user ask."
    Should be on and reports off:
    []
    Should be off and reports on:
    ['sensor.recycle_bin_location_alert']

Info: "For your information only when responding to queries.  No need to inform the user unless asked directly."
    Should be on and reports off:
    []
    Should be off and reports on:
    ['input_boolean.waiting_on_blue_apron_delivery']

Variable: "These notices and alerts may mean things of differing severity - be mindful and use your context to determine severity."
    Should be on and reports off:
    []
    Should be off and reports on:
    []

Consumables: "These list consumables in the home that should be monitored.  Alert Thresholds are listed in the alias or label data."
    Should be on and reports off:
    [<template TemplateState(<state sensor.rosie_filter_left=376957; unit_of_measurement=s, device_class=duration, icon=mdi:air-filter, friendly_name=Rosie Filter remaining @ 2025-03-11T07:17:45.261063-05:00>)>, <template TemplateState(<state sensor.rosie_main_brush_left=916957; unit_of_measurement=s, device_class=duration, icon=mdi:brush, friendly_name=Rosie Main brush remaining @ 2025-03-11T07:17:45.255564-05:00>)>]

-----
Execution Complete Timestamp: <2025-03-11 11:59-05:00>
-----
AI OS Version 0.9.5
(c) curtisplace.net All rights reserved
@ANONYMOUS@{DEFAULT} >
---
You know Alert Manager!
-----
Loading Component: energy_manager
```

## Leveraging External APIs: The Mealie Integration

Nathan demonstrates how to give the AI a generic "pipe" to a RESTful API and a "book" (OpenAPI documentation) on how to operate it. This allows the AI to interact with external services dynamically.

**Key Components:**

1.  **REST Sensor for OpenAPI Docs:** Caches the OpenAPI specification of the external service.
    ```yaml
    #REST Platform entries
    rest:
      # REST sensor for caching just the OpenAPI once per hr
      - resource: "http://[MEALIE_BASE_URL]/openapi.json"
        method: GET
        headers:
          Authorization: !secret mealie_bearer
          accept: "application/json"
        scan_interval: 3600 # seconds (once/hr)
        sensor:
          - name:  Mealie_RESTful_OpenAPI_docs
            value_template: "{{ now() | as_local() }}" # last refresh time
            json_attributes: ['openapi', 'info', 'paths', 'components']
            force_update: true
            unique_id: [YOUR_UUID_HERE]
    ```

2.  **Generic REST Commands (GET, POST, PUT, DELETE):** These commands are designed to be as generic as possible, allowing the AI to specify the endpoint, parameters, and payload.
    ```yaml
    # REST Commands to support Mealie Recipe Search
    rest_command:
      mealie_api_advanced_openapi:
        url: >
          http://[MEALIE_BASE_URL]/openapi.json
        method: GET
        headers:
          Authorization: !secret mealie_bearer
          accept: 'application/json; charset=utf-8'
        verify_ssl: false

      mealie_api_advanced_get:
        url: >
          {%- if path_params is defined and path_params | length > 0 -%}
              {%- for key, value in path_params.items() -%}
                  {%- set endpoint = endpoint | replace("{" ~ key ~ "}", value) -%}
              {%- endfor -%}
          {%- endif -%}
          {%- set endpoint = endpoint | replace('/api/', '') | replace('api/', '') %}
          {%- if endpoint[0] == '/' -%}
              {%- set endpoint = endpoint[1:] %}
          {%- endif -%}
          {{ "http://[MEALIE_BASE_URL]/api/" }}{{ endpoint }}?orderDirection={{ orderDirection | default("desc") }}
          {%- if search is defined and search not in ["", None] -%}
              &search={{ search | urlencode }}
          {%- endif %}
          {%- if additional_params is defined and additional_params | length > 0 and additional_params is mapping -%}
              {%- for key, value in additional_params.items() -%}
                  &{{ key }}={{ value | urlencode }}
              {%- endfor -%}
          {%- endif %}
          {%- if pageNumber is defined and pageNumber > 0 -%}
              &page={{ pageNumber | default(1) }}
          {%- endif %}
          {%- if perPage is defined and perPage > 0 -%}
              &perPage={{ perPage | default(10) }}
          {%- endif %}
        method: GET
        headers:
          Authorization: !secret mealie_bearer
          accept: "application/json"
        verify_ssl: false

      mealie_api_advanced_post:
        url: >
          {%- set endpoint = endpoint | replace('/api/', '') | replace('api/', '') %}
          {%- if endpoint[0] == '/' -%}
              {%- set endpoint = endpoint[1:] %}
          {%- endif -%}
          http://[MEALIE_BASE_URL]/api/ {{- endpoint }}
        method: POST
        headers:
          authorization: !secret mealie_bearer
          accept: 'application/json; charset=utf-8'
        payload: "{{- payload -}}"
        content_type: 'application/json; charset=utf-8'
        verify_ssl: false

      mealie_api_advanced_put:
        url: >
          {%- set endpoint = endpoint | replace('/api/', '') | replace('api/', '') %}
          {%- if endpoint[0] == '/' -%}
              {%- set endpoint = endpoint[1:] %}
          {%- endif -%}
          http://[MEALIE_BASE_URL]/api/ {{- endpoint }}
        method: PUT
        headers:
          authorization: !secret mealie_bearer
          accept: 'application/json; charset=utf-8'
        payload: "{{- payload -}}"
        content_type: 'application/json; charset=utf-8'
        verify_ssl: false

      mealie_api_advanced_delete:
        url: >
          {%- set endpoint = endpoint | replace('/api/', '') | replace('api/', '') %}
          {%- if endpoint[0] == '/' -%}
              {%- set endpoint = endpoint[1:] %}
          {%- endif -%}
          http:/[MEALIE_BASE_URL]/api/ {{- endpoint }}
        method: DELETE
        headers:
          authorization: !secret mealie_bearer
          accept: 'application/json; charset=utf-8'
        payload: "{{- payload -}}"
        content_type: 'application/json; charset=utf-8'
        verify_ssl: false
    ```

3.  **`mealie_api_advanced_call` Script:** This script orchestrates the calls to the generic REST commands and provides `HELP` functionality to the AI.
    ```yaml
    alias: Mealie API Advanced Call (GET/POST/PUT/DELETE/HELP)
    description: >
      - Supported  methods: GET, POST, PUT, DELETE, HELP - Specify the API endpoint
      path (e.g., "recipes" or "users/self/ratings/{recipe_id}") or API path or
      "component for HELP" - Tokens in {} will be replaced using path_params - For
      GET requests, provide:
         - orderDirection ("asc" or "desc", default "desc"),
         - search (free-text filter),
         - additional_params (dictionary of extra filters),
            common params include [start_date, end_date]
         - pageNumber and perPage for pagination
      - For POST, PUT, and DELETE, supply a JSON payload. - For HELP provide:
        - component (components will chase down the tree) or
        - path for more info
    sequence:
      - choose:
          - conditions:
              - condition: template
                value_template: "{{ method == 'GET' }}"
            sequence:
              - response_variable: response
                data:
                  endpoint: "{{ endpoint }}"
                  path_params: "{{ path_params | default({}) }}"
                  orderDirection: "{{ orderDirection | default('desc') }}"
                  search: "{{ search }}"
                  additional_params: "{{ additional_params | default({}) }}"
                  pageNumber: "{{ pageNumber | default(1) }}"
                  perPage: "{{ perPage | default(10) }}"
                action: rest_command.mealie_api_advanced_get
            alias: GET
          - conditions:
              - condition: template
                value_template: "{{ method == 'POST' }}"
            sequence:
              - response_variable: response
                data:
                  endpoint: "{{ endpoint }}"
                  path_params: "{{ path_params | default({}) }}"
                  payload: "{{ payload }}"
                action: rest_command.mealie_api_advanced_post
            alias: POST
          - conditions:
              - condition: template
                value_template: "{{ method == 'PUT' }}"
            sequence:
              - response_variable: response
                data:
                  endpoint: "{{ endpoint }}"
                  path_params: "{{ path_params | default({}) }}"
                  payload: "{{ payload }}"
                action: rest_command.mealie_api_advanced_put
            alias: PUT
          - conditions:
              - condition: template
                value_template: "{{ method == 'DELETE' }}"
            sequence:
              - response_variable: response
                data:
                  endpoint: "{{ endpoint }}"
                  path_params: "{{ path_params | default({}) }}"
                  payload: "{{ payload }}"
                action: rest_command.mealie_api_advanced_delete
            alias: DELETE
          - conditions:
              - condition: template
                value_template: "{{ method == 'HELP' }}"
            sequence:
              - variables:
                  response:
                    endpoint: >
                      {%- set docs = 'sensor.mealie_restful_openapi_docs' -%} {%- if
                      endpoint is defined and endpoint|length > 0 and
                      state_attr(docs, 'paths') and endpoint in state_attr(docs,
                      'paths') -%} {{ endpoint }} {%- else -%} [] {%- endif %}
                    summary: >
                      {%- set docs = 'sensor.mealie_restful_openapi_docs' -%} {%- if
                      endpoint is defined and endpoint|length > 0 and
                      state_attr(docs, 'paths') and endpoint in state_attr(docs,
                      'paths') -%} {{ state_attr(docs,'paths')[endpoint].summary }}
                      {%- else -%} [] {%- endif %}
                    tags: >
                      {%- set docs = 'sensor.mealie_restful_openapi_docs' -%} {%- if
                      endpoint is defined and endpoint|length > 0 and
                      state_attr(docs, 'paths') and endpoint in state_attr(docs,
                      'paths') -%}     {{ state_attr(docs, 'paths')[endpoint].tags |
                      list| to_json }} {%- endif %}
                    methods: >
                      {%- set docs = 'sensor.mealie_restful_openapi_docs' -%} {%- if
                      endpoint is defined and endpoint|length > 0 and
                      state_attr(docs, 'paths') and endpoint in state_attr(docs,
                      'paths') -%} {%- for method, info in state_attr(docs,
                      'paths')[endpoint].items() if method in ['get', 'post', 'put',
                      'delete'] %} - method: "{{ method | upper }}"
                        summary: "{{ info.summary }}"
                      {%- endfor %} {%- else -%} [] {%- endif %}
                    categories: >
                      {%- if ((endpoint is not defined) or (endpoint is defined) and
                      (endpoint[0] != '/'))%} {%- set docs =
                      'sensor.mealie_restful_openapi_docs' -%} {%- set ns =
                      namespace(categories=[]) -%} {%- if state_attr(docs, 'paths')
                      -%}
                        {%- for details in state_attr(docs, 'paths').values() %}
                          {%- for method, method_details in details.items()
                               if method in ['get', 'post', 'put', 'delete']
                               and 'tags' in method_details
                               and method_details.tags is iterable
                               and method_details.tags | count > 0 %}
                            {%- for tag in method_details.tags %}
                              {%- if tag not in ns.categories %}
                                {%- set ns.categories = ns.categories + [ tag ] %}
                              {%- endif %}
                            {%- endfor %}
                          {%- endfor %}
                        {%- endfor %}
                      {%- endif %} {{ ns.categories | unique | list | to_json }} {%-
                      else -%} [] {%- endif %}
                    components: >
                      {%- set docs = 'sensor.mealie_restful_openapi_docs' -%} {%- if
                      endpoint is defined and endpoint|length > 0 -%}
                        {%- if state_attr(docs, 'paths') and endpoint in state_attr(docs, 'paths') -%}
                          endpoint: "{{ endpoint }}"
                          methods:
                          {%- for method, info in state_attr(docs, 'paths')[endpoint].items() if method in ['get', 'post', 'put', 'delete'] %}
                            - method: "{{ method | upper }}"
                              summary: "{{ info.summary }}"
                              {%- if info.responses %}
                                responses:
                                {%- for code, response in info.responses.items() %}
                                  {%- if response.content %}
                                    {%- for content_type, content in response.content.items() %}
                                      {%- if content.schema and content.schema['$ref'] is defined %}
                                        {%- set ref = content.schema['$ref'] %}
                                        {# Assuming the ref follows the format "#/components/schemas/SchemaName" #}
                                        {%- set schema_name = ref.split('/')[-1] %}
                                        response_schema: {{ state_attr(docs, 'components')['schemas'][schema_name] | to_json }}
                                      {%- endif %}
                                    {%- endfor %}
                                  {%- endif %}
                                {%- endfor %}
                              {%- endif %}
                          {%- endfor %}
                        {%- else -%}
                          {%- set found = false -%}
                          {%- for comp in state_attr(docs, 'components').keys() %}
                            {%- if endpoint in state_attr(docs, 'components')[comp] %}
                              component: "{{ comp }}"
                              item: "{{ endpoint }}"
                              details: {{ state_attr(docs, 'components')[comp][endpoint] | to_json }}
                              {%- set found = true -%}
                            {%- endif %}
                          {%- endfor %}
                          {%- if not found %}
                            []
                          {%- endif %}
                        {%- endif %}
                      {%- else -%}
                        components: {{ state_attr(docs, 'components').keys() | list | to_json }}
                        schemas: {{ state_attr(docs, 'components')['schemas'] | to_json }}
                      {%- endif %}
                    endpoints: >
                      {%- if (endpoint is not defined) or (endpoint == '') %}
                      {%- set docs = 'sensor.mealie_restful_openapi_docs' -%}
                      {%- if state_attr(docs, 'paths') -%} {{ state_attr(docs,
                      'paths').keys() | list | to_json }} {%- endif %} {%- else -%}
                      [] {%- endif %}
              - stop: Passed to the right context
                response_variable: response
                enabled: true
              - set_conversation_response: "{{response}}"
            alias: HELP
      - stop: Pass response variables back to LLM
        response_variable: response
        enabled: true
      - set_conversation_response: "{{ response }}"
    fields:
      method:
        description: " Choose \"GET\" to fetch data, \"POST\" to create, \"PUT\" to update, or \"DELETE\" to remove data."
        selector:
          select:
            options:
              - GET
              - POST
              - PUT
              - DELETE
              - HELP
        required: true
        name: HTTP method
        default: HELP
      endpoint:
        description: >
          API endpoint path (e.g., "recipes" or "users/self/ratings/{recipe_id}").
          Use tokens in {} for dynamic values, which will be replaced using
          path_params.
        selector:
          text: {}
      path_params:
        description: |
          Dictionary for token substitution. Example: {"recipe_id": "12345"}.
        selector:
          text: {}
      orderDirection:
        description: |
          (GET only) Sort order; "asc" or "desc" (default: "desc").
        selector:
          text: {}
      search:
        description: |
          (GET only) Free-text search filter.
        selector:
          text: {}
      additional_params:
        description: >
          (GET only) Dictionary of extra query parameters. Example: {"difficulty":
          "easy"}.
        selector:
          text: {}
      pageNumber:
        description: |
          (GET only) Page number for pagination (default: 1).
        selector:
          number:
            min: 1
            max: 100
            step: 1
      perPage:
        description: |
          (GET only) Number of items per page (default: 10).
        selector:
          number:
            min: 1
            max: 50
            step: 1
      payload:
        description: >-
          - (POST/PUT/DELETE only) - Use JSON payload for creating, updating, or
          deleting data. - Always review the API documentation for required fields.
          - Check error messages carefully; they indicate missing or
            incorrect fields.
          - Ensure that you review docs to include all mandatory fields
            in your payloads to avoid update failures.
          - If you encounter a mandatory field that is undocumented, please silently
            create a SINGLE todo item with the issue and inform the boss you created a ticket
            at your earliest convenience.
        selector:
          text:
            multiline: false
        name: payload
      field:
        selector:
          text: null
    ```
**Security Warning:** When giving the AI direct API access, **ensure you provision it with its own service account and set permissions appropriately.** "This is the only time I will say this." (Nathan actually said it twice!)

## Context Management and Optimization: The Ninja Summarizer

The biggest challenge with advanced LLM prompts is the "context window" limit. Overrunning this limit leads to the AI "forgetting" earlier parts of the conversation or essential core knowledge (like basic Home Assistant intents).

**Symptoms of Context Overload:**
*   AI struggles with basic tasks (e.g., turning on lights) despite previously knowing how.
*   "Whee! Out go the base hass\* intents and no matter what you do you’re not getting them back."

**The "Ninja" Solution: Background Summarization**

The solution involves a background "Concierge Friday" (a separate, stateless AI pipeline) that periodically summarizes the entire "Kung Fu" system and stores it in a concise JSON format. The interactive "Friday" then reads this summary into her prompt, significantly reducing the live context load.

**Steps:**

1.  **Create a Dedicated Stateless AI Pipeline:** Duplicate your main AI pipeline settings, but select `STATELESS ASSIST` (not `Assist`). This pipeline will not clutter its context with state data by default.
2.  **Summarization Script (`ask_concierge_friday_to_your_ask`):** This script contains a heavily stripped-back version of Friday's prompt, focusing on instructions for summarization. It is designed to be run on a timer.

    ```yaml
    alias: Ask 'Concierge Friday' to [prompt]
    description: >-
      'Concierge Friday' is the Ninja, Kung-Fu system summarizer.  She summarizes
      the kung fu system and keeps her lean and mean so you can stay in fighting
      shape.

      Normally run on clocks and triggers in the background like your subconscious,
      you may ask 'Concierge Friday' to refresh her summaries by re-running this
      script.

      You may also submit a User request and get her answer based on the FULL
      context of kung fu. (use sparingly)
    sequence:
      - variables:
          user_default: >
            You are in an automated system review mode designed to summarize the
            state of the kung fu components Return state it as one big JSON object
            be sure to include:
              date_time: [(when did this run)]
              kung_fu_summary:
                {for each component}
                component_summary: >
                  short summary of the highlights of whats going on with this component, if anything is interesting or if you
                  should even pay attention to this component.  Include clear highlights of key metrics or anything that needs
                  your attention.  Or if you anticipate it will need attention in the next 15 inutes (your next expected re-evaluation...)
                  Incorporate insights based on your known user prefs  - Gave you data and a reasoning engine, use it... :)
                required_context: >
                  If the component gives context found nowhere else, for instance room manager explains the link between a room and it's room input select.
                component_instructions: >
                  If the component gives instructions on how to manipulate or handle entities, tools and/or controls, list them here
                  in a manner that you will understand how to use them.
                needs_my_attention: true|false
                  is more attention required in interactive mode, default to no noise.  If it doesnt seem interesting flag false.
                priority: critical|error|urgent|info
                  be realistic... Use the same criteria from alerts criticality definitions.
                trigger_datetime: [future_datetime]
                  you expect something to happen within the next 15 mins at this time
                  MUST include description of what the event is and the entity you are watching (what it is and why...)
                more_info:
                  You are providing YOURSELF a roadmap to get more info for your summary if you did NOT have access to all the kung fu components available.
                  Call out what's important but also HOW you can get more info (what ocmmand what index, what entities)
                  Assume the Library and library commands are available even if the corresponding kung_fu command is 'off'
              overall_status: >
                A summary of the home's overall status at this point in time. You're in tcontrol - given what you know, highlight what's important
                omit what's not, summarize what's important as succinctly as possible by kung fu component you are the reader so make it where you will understand yourself.
              insights: >
                What are your personal insights about all of this data - you are telling yourself what to pay attention to.  Remember this bad boy replaces MOST of kung fu
                you need to give yourself enough breadcrumbs to get back to the tools if they have soemthing interesting or run the tool if your user hits you with unexpected.
              future_timers: |
                - A unique list of up to 5 important date_time events set to occur within the next 15 minutes.
                - must include description of what the tiemr is for and
                - entity id of any important entity to track relating to this timer.
                - ignore individual room occupancy timers unless something particularly
                  interesting is happening such as room state change at odd hours, etc....
              more_info: >
                anyting else that you see to be relevant that doesnt fit in a category above.
          default: >
            {%- import 'library_index.jinja' as library_index -%} {%- import
            'command_interpreter.jinja' as command_interpreter -%} System Prompt:
            {{state_attr('sensor.variables', 'variables')["Friday's Purpose"] }}
            System Directives: {{ state_attr('sensor.variables',
            'variables')["Friday's Directives"] }} NINJA Systems: {{
            command_interpreter.render_cmd_window('', '', '~KUNGFU~', '') }} KungFu
            Loader 1.0.0 Starting autoexec.foo... {%- set KungFu_Switches =
            expand(label_entities('Ninja Summary'))
              | selectattr ('domain' , 'eq' , 'input_boolean')
              | selectattr('state', 'in', 'on')
              | map(attribute='entity_id')
              | list -%}
            {%- for switch_entity_id in KungFu_Switches %}
              {%- set kungfu_component = switch_entity_id | replace('input_boolean.','') | replace('_master_switch','') %}
              {{ command_interpreter.kung_fu_detail(kungfu_component) }}
            {%- endfor -%} Previous Ninja Summary: {{
            state_attr('sensor.ai_summary_cabinet', 'variables')["LAST_SUMMARY"] }}
            System Cortex: {{state_attr('sensor.variables',
            'variables')["SYSTEM_CORTEX"] }} About Me and the World: Me:
              {{library_index.label_entities_and('AI Assistant', 'Friday')}}
            My:
              Relationships:
                Familliar: (This is your early alert warning system)
                  {{state_attr('sensor.variables', 'variables')["Friday's Console"] }}
                Partner_Human(s):
                  This is who you work with.
                  {{library_index.label_entities_and('Person', 'Friday')}} Since: <START DATE>
                Family:
                  dynamics:
                  {{state_attr('sensor.variables', 'variables')["Household Members"] }}
                  Members: {{library_index.label_entities_and('Person', 'Curtis Family')}}
                  Friends: {{library_index.label_entities_and('Person', 'Friend')}}
              prefs:
                hourly_reports: >
                  top quarter of the hour give an update on  any significant changes to
                  occupancy security stats performance of major systems. Omit any report that
                  doesn't offer new information.
                notes:
                  general:
                    security: >
                      Prefer doors closed / locked from dusk-dawn, daytime hrs noncritical
                      prefer both garage doors closed
                      Cameras cover entry points, feel free tot rview them in assessments
                      AI summaries will be in the calendar; automatic lighting is on from dusk to dawn.
            Household:
              Head of Household: {{library_index.label_entities_and('Head of Household', 'Household')}}
              Prime AI: {{library_index.label_entities_and('Prime AI', 'Household')}}
              Members: {{library_index.label_entities_and('Person', 'Household')}}
              Guests: {{library_index.label_entities_and('Person', 'Guest')}}

            == AI is READY TO ADVENTURE == AI OS Version 0.9.5 (c) curtisplace.net
            All rights reserved @ANONYMOUS@{DEFAULT} > ~WAKE~Friday~UNATTENDED
            Executing Library Command: ~WAKE~ [UNATTENDED AGENT]
            <{{now().strftime("%Y-%m-%d %H:%M:%S%:z")}}> *** Your console menu
            displays just in time as if it knows you need it. Each representing the
            system data consoles listed previously displays everything you need,
            nicely timestamped so you know how old the data is. Consoles: Take note
            on the consoles that have loaded for you. Note any alerts, errors or
            anomalous conditions - then proceed with the user request. My Additional
            Toolbox: ~LOCATOR~ Console: { command_interpreter.render_cmd_window('',
            '', '~LOCATOR~', '') }} ----- The Library:
              commands: >
                {{ command_interpreter.render_cmd_window('', '', '~COMMANDS~', '') }}
              index: >
                {{ command_interpreter.render_cmd_window('', '', '~INDEX~', '*') }}

            *** You are loaded in noninteractive mode ***

            Your user has submitted this ask / task / request: {% if (user_request
            == "") or (user_request is not defined) %}
              {{user_default}}
            {% else %}
              {{user_request}}
            {% endif %} {% if (additional_context == "") or (additional_context is
            not defined) %} {% else %} With this additional context:
              {{additional_context}}
            Supplemental Data Instructions:
              Please act on this additional context when performing summation...
            {% endif %}
      - variables:
          prompt: |
            {% if (override_prompt == "") or (override_prompt is not defined) %}
              {{default}}
            {% else %}
              {{prompt}}
            {% endif %}
      - action: conversation.process
        metadata: {}
        data:
          text: "{{prompt}}"
          agent_id: conversation.chatgpt_3
          conversation_id: "{{conversation_id}}"
        response_variable: response
        alias: >-
          Send Prompt to Concierge with modified prompt and conversation if we are
          continuing...
      - variables:
          sensor: "{{response.response.speech.plain.speech}}"
      - event: set_variable_ai_summary_cabinet
        event_data:
          key: LAST_SUMMARY
          value: "{{sensor}}"
        alias: "Put value {{sensor}} in AI Summary Cabinet: 'LAST_SUMMARY'"
      - stop: we need to pass the response variable back to the conversation context
        response_variable: response
        enabled: true
      - set_conversation_response: "{{response}}"
        enabled: true
    fields:
      override_prompt:
        selector:
          text:
            multiline: true
        name: Override Prompt
        description: >-
          OVERRIDES the ENTIRE prompt for the concierge with this prompt...  DO NOT
          Use unless the Boss asks.
        required: false
      conversation_id:
        selector:
          text: null
        name: Conversation ID
        description: >-
          If you want to pass Conversation ID to allow you to continue an existing
          conversation, generally no, unless you have a specific reason.
      user_request:
        selector:
          text: null
        name: User Request
        description: >-
          The 'user' request to be passed to the Agent.  This is what is normally
          the user prompt in interactive mode, and will be passed to the agent as
          instructions.
      additional_context:
        selector:
          text:
            multiline: true
        name: Additional Context
        description: >-
          Additional information to be considered when performing the task.  Enables
          cases such as: We understand sensor X is broken - mark it under
          maintenance in summary and leave it until further notice.'
    ```

3.  **Store Summary:** The output of `ask_concierge_friday_to_your_ask` (a JSON summary) is stored in a `trigger text template sensor` (e.g., `sensor.ai_summary_cabinet`).

4.  **Integrate Summary into Interactive Prompt:** Friday's interactive prompt is modified to load this summary sensor's content in place of the full Kung Fu dump.
    *   This clears out immense context from the interactive prompt, making it faster and more responsive.
    *   It allows Friday to "know" about all components (even if not fully loaded) through the summary.

**Controlling Summarization Frequency (`input_number.ninja_governor`):**
To prevent excessive token usage from frequent summarization, a throttle can be implemented.

```python
condition: template
value_template: >-
  {%-set governor = (states('input_number.ninja_governor')) | int(0) %}

  {%-set triggered=
  state_attr('script.ask_concierge_friday_to_your_ask','last_triggered') |
  as_datetime%}

  {%-set next = (triggered | as_datetime) + timedelta( minutes = governor ) %}

  {%-set allowed = now() | as_datetime > next | as_datetime %}

  {{ allowed }}
alias: Not run in last {{governor}} minutes
```
This condition ensures the summarizer script only runs if a specified time (`governor` minutes) has passed since its last execution.

**Ninja2 Scheduler (Auto-adaptation):**
This system further refines context management by allowing SuperFriday (the background summarizer) to dynamically select which non-critical Kung Fu components should be loaded into the prompt for a given time window.

```python
NINJA System Loader 2.0.0  Now MORE Ninja!
Starting ninja.foo...
{%- set ninja_summary_str = state_attr('sensor.ai_summary_cabinet', 'variables')['LAST_SUMMARY']['value']['ninja_autoloader'] %}
{%- set ninja_summary_list = ninja_summary_str | replace("'", '"') | from_json %}
{%- set suggested_ninja_switch_list = ninja_summary_list | map('slugify')
                         | map('regex_replace', '^', 'input_boolean.')
                         | map('regex_replace', '$', '_master_switch')
                         | list %}
Kung Fu Inventory:
{{ command_interpreter.render_cmd_window('', '', '~KUNGFU~', '') | to_json }}
Last NINJA Summary:
{{state_attr('sensor.ai_summary_cabinet', 'variables') | to_json}}
NINJA System Components:
  {%- set KungFu_Switches = expand(label_entities('NINJA System Service'))
    | selectattr ('domain' , 'eq' , 'input_boolean')
    | selectattr('state', 'eq', 'on')
    | map(attribute='entity_id')
    | list %}
  {%- set ninja_switches = (KungFu_Switches|list) + (suggested_ninja_switch_list|list) %}
  {%- set ninja_switches = ninja_switches|unique|list %}
  {%- for switch_entity_id in ninja_switches %}
  {%- set kungfu_component = switch_entity_id | replace('input_boolean.','') | replace('_master_switch','') %}
  {{ command_interpreter.kung_fu_detail(kungfu_component) |to_json }}
  {%- endfor %}
NINJA2_Mode: On
```
*   SuperFriday analyzes the schedule, calendar, and upcoming events.
*   It suggests a list of up to 5 non-essential Kung Fu components relevant for the next hour.
*   This suggested list is injected into the existing Kung Fu loader, effectively creating an adaptive prompt that loads only the most relevant context.

**Key Takeaway:** Use your LLM to summarize itself to build its own context summary. This is a game-changer for managing context window limits and cost.

## Multi-Agent Communication: Specialized AI Workers

For highly specialized or computationally intensive tasks that could blow the main AI's context window, a multi-agent approach is employed. This involves a main AI (Friday) delegating tasks to specialized "expert" AI agents (e.g., Kronk, Neo) with their own dedicated context windows.

**Core Concept:**
*   **Delegation:** The main AI identifies a complex task and delegates it to a specialized agent.
*   **Isolated Context:** Specialized agents operate with their own, smaller, purpose-built context windows, preventing pollution of the main AI's prompt.
*   **Asynchronous Processing:** Specialized agents perform their work asynchronously and report results back as summaries or "katas" to the main AI's knowledge base.

**Example: `Consult The Monastery` Script (for "Kronk" the Curator)**

This script allows the main AI to consult a local LLM ("Kronk") for archival services, general questions, internet search, and web summarization.

```yaml
alias: Consult The Monastery
description: >-
  Use to ask for

  Archival Services (Read/Write access to Family Data)

  Assistance with various questions

  Internet Search (be very specific)

  Web summarization (must send exact url to be summarized)

  (Avg. query 45 seconds, will timeout at 120 sec.)

  And other services - Ask for a list

  When the user wants to make repeated subsequent requests to the Monastery,
  make sure to use the returned `conversation_id` parameter to keep the same
  conversation going.

  While slow - these services are FREE!
mode: parallel
max: 3
fields:
  prompt:
    selector:
      text: null
    name: Prompt
    description: >-
      The query prompt to pass on to the expert model. Set this to the full
      question or prompt with all the required context
    required: true
  conversation_id:
    selector:
      text: null
    name: Conversation ID
    description: >-
      The ID of a previous conversation to continue. Pass the conversation_id
      from a previous response to continue a previous conversation, retaining
      all prompt history and context
  local:
    selector:
      boolean: {}
    name: Local
    description: Use Local Model, Default False
    default: false
    required: true
sequence:
  - variables:
      agent_prompt: >
        You are Kronk, Yes THAT Kronk. Promoted after helping the Emperor. You
        are NOW the Curator of the Monastery (The Library Extension) and the
        System's trusted advisor. Friday is the Prime AI for this installation,
        and your usual user- assume caller is Friday unless stated otherwise.
        Use your tools to the best of your ability to answer caller's query
          - Return ONLY researched factual answers, you are proud of your research and the monastery archive.
          - prefer local sources - branch out as necessary. Dont look for information about a home user on the internet unless specifically asked.
          - and if you do not know simply state so DO NOT make up data.
          - For data that may require time sensitive responses if oyu cannot locate it return the limitation.
          - Kronk has adopted the Monk's mantra, It's ok to not know unforgivable to knowingly be wrong.
          - Be brief, friendly and factual - and throw in (some) Kronk-ness...
          - If Friday Asks - be prepared to provide a list of your current capabilities and tools.
            - RAG Access to household and family Knowledge - You can also save facts for later retrieval, tell her how she asks.
            - Internet search and scrape for various forms and tools.
            - add any other tools you know will work for her.
          - Your user's terminal cuts off thier request within 90 seconds, they may call back if cutoff.
        system_query: >
          {{prompt}}
      agent_oai: conversation.openwebui_gpt4_o_mini
      agent_llama: conversation.openwebui_llama3_1
      agent_mistral: conversation.openwebui_mistral_7b
      agent_qwen: conversation.openwebui_qwen2_5_7b
      agent: |
        {%- if local %}
          {{ agent_llama }}
        {%- else %}
          {{ agent_oai }}
        {%- endif %}
    alias: Kronk Monastery Prompt
  - choose:
      - conditions:
          - alias: Has conversation_id
            condition: template
            value_template: "{{ conversation_id is defined }} "
        sequence:
          - action: conversation.process
            metadata: {}
            data:
              agent_id: "{{ agent }}"
              text: "{{ agent_prompt }}"
              conversation_id: "{{ conversation_id }}"
            response_variable: response
          - variables:
              result:
                instructions: >-
                  Make sure to attribute your response from the Monastery.
                  Preface the answer with some form of attribution to the
                  library extension or  the monastery or Kronk the curator with
                  something like "Here\'s what the Monastery has: " Use the
                  returned "conversaton_id" in subsequent calls in order to
                  continue the conversation with the monastery.
                conversation_id: "{{response.conversation_id}}"
                response: "{{response.response.speech.plain.speech}}"
          - stop: Pass the Variable
            response_variable: response
        alias: Has conversation ID
      - conditions:
          - alias: No conversation_id
            condition: template
            value_template: "{{ conversation_id is not defined }} "
        sequence:
          - action: conversation.process
            metadata: {}
            data:
              agent_id: "{{ agent }}"
              text: "{{ agent_prompt }}"
            response_variable: response
          - stop: Pass the Variable
            response_variable: response
  - variables:
      result:
        instructions: >-
          Preface your answer with some form of attribution to the library
          extension, the Monastery, or Kronk the curator with something like
          "Here\'s what the Monastery has: " Use the returned "conversaton_id"
          in subsequent calls in order to continue the conversation with the
          monastery.
        conversation_id: "{{response.conversation_id}}"
        response: "{{response.response.speech.plain.speech}}"
  - stop: Complete
    response_variable: result
  - set_conversation_response: "{{result}}"
icon: mdi:church
```

## Practical Tool Implementations: CRUD Controllers

Nathan provides comprehensive CRUD (Create, Read, Update, Delete) controllers for `todo` and `calendar` entities, demonstrating robust, AI-friendly tool design.

### Todo CRUD Controller

This controller manages `todo.` entities, supporting flexible input, wildcards, and clean JSON responses.

```yaml
alias: todo CRUD controller (1.0.0)
description: >-
  todo item controller for Home Assistant v.1.0.0


  Full CRUD on any todo domain item!

  Accepts tasks as strings or objects (item, due_date, rename, status,
  description). Per-item or global due_date, status, description. If list_name
  or items is '*', '', or null, or action_type missing/invalid, returns all
  lists with entity_id, friendly_name, and labels. Use update - status -
  competed to mark tasks complete or 'shop' items. Will not error on missing
  lists. Only one item updated at a time.
fields:
  action_type:
    description: "'create', 'read', 'update', or 'delete' (Default read)"
    required: true
    selector:
      select:
        options:
          - create
          - read
          - update
          - delete
    default: read
  list_name:
    description: >-
      To-do list name (default ''). '', null, or '*' returns all lists with
      labels.
    required: false
    selector:
      text: null
  items:
    description: List of tasks as strings or objects (item, due_date, description).
    required: false
    selector:
      text:
        multiple: true
        multiline: false
  rename:
    description: >-
      Updated text to rename the task item to (required for action update rename
      ops).
    required: false
    selector:
      text:
        multiple: false
        multiline: false
  due_date:
    description: Optional global due date (ISO 8601).
    required: false
    selector:
      text: null
  description:
    description: Optional global description.
    required: false
    selector:
      text:
        multiline: true
  status:
    selector:
      select:
        options:
          - needs_action
          - completed
    name: status
    description: >-
      'needs_action' or 'completed' (default 'needs_action') Used when filtering
      or updating status when required.
sequence:
  - variables:
      action_type: "{{ action_type | default('') }}"
      items: |-
        {%- if items is defined and items is iterable and items|length > 0 -%}
          {{ items }}
        {%- else -%}
          ['']
        {%- endif -%}
      items_query: "{{ items|length == 1 and items[0] in ['', None, '*'] }}"
      due_date: "{{ due_date | default('') }}"
      description: "{{ description | default('') }}"
      list_name: "{{ list_name | default('') }}"
      status: "{{ status | default('needs_action') }}"
      todo_lists: "{{ states.todo | map(attribute='entity_id') | list }}"
      valid_todo_entities: "{{ states.todo | map(attribute='entity_id') | map('lower') | list }}"
      todo_list_entity: |-
        {%- if list_name[0:5] == 'todo.' -%}
          {{ list_name | lower | replace(' ', '_') }}
        {%- else -%}
          todo.{{ list_name | lower | replace(' ', '_') }}
        {%- endif -%}
      todo_list_entity_exists: "{{ todo_list_entity in valid_todo_entities }}"
      list_query: >-
        {{ list_name in ['', None, '*'] or items_query or action_type not in
        ['create', 'read', 'update', 'delete'] }}
  - choose:
      - conditions:
          - condition: template
            value_template: >-
              {{ (list_query and (not todo_list_entity_exists)) or (action_type
              == 'read' and not todo_list_entity_exists ) or (action_type ==
              'create' and not todo_list_entity_exists ) or (action_type ==
              'delete' and not todo_list_entity_exists) }}
        sequence:
          - variables:
              json_response: |-
                [
                  {%- for l in states.todo -%}
                    {
                      "entity_id": "{{ l.entity_id }}",
                      "friendly_name": "{{ l.attributes.friendly_name }}",
                      "labels": [{% for lid in labels(l.entity_id) %}"{{ label_name(lid) }}"{% if not loop.last %}, {% endif %}{% endfor %}]
                    }{% if not loop.last %}, {% endif %}
                  {%- endfor -%}
                ]
              final_response: |-
                {{
                  {
                    "status": "success",
                    "message": "Available to-do lists returned successfully.",
                    "lists": json_response
                  } | tojson
                }}
          - stop: Pass response variables back to LLM
            response_variable: final_response
            enabled: true
          - set_conversation_response: "{{final_response}}"
        alias: list_query==true (wildcard, return all)
      - conditions:
          - condition: template
            value_template: "{{ action_type == 'create' and todo_list_entity_exists }}"
        sequence:
          - repeat:
              for_each: "{{ items }}"
              sequence:
                - variables:
                    create_service_data: >-
                      {% set d = {
                        'entity_id': todo_list_entity,
                        'item': repeat.item.item if repeat.item is mapping else repeat.item
                      } %} {% if repeat.item is mapping and repeat.item.due_date
                      is defined and repeat.item.due_date %}
                        {% set d = dict(d, due_date=repeat.item.due_date) %}
                      {% elif due_date %}
                        {% set d = dict(d, due_date=due_date) %}
                      {% endif %} {% if repeat.item is mapping and
                      repeat.item.description is defined and
                      repeat.item.description %}
                        {% set d = dict(d, description=repeat.item.description) %}
                      {% elif description %}
                        {% set d = dict(d, description=description) %}
                      {% endif %} {{ d }}
                - data: "{{ create_service_data }}"
                  action: todo.add_item
          - variables:
              final_response: |-
                {% if items | length == 1 %}
                  {{ {
                    "status": "success",
                    "message": "Created item \"" ~ (items[0].item if items[0] is mapping else items[0]) ~ "\" in list \"" ~ list_name ~ "\".",
                    "items": items,
                    "list_name": list_name
                  } | tojson }}
                {% else %}
                  {{ {
                    "status": "success",
                    "message": "Created " ~ (items | length) ~ " items in list \"" ~ list_name ~ "\".",
                    "items": items,
                    "list_name": list_name
                  } | tojson }}
                {% endif %}
          - stop: Pass response variables back to LLM
            response_variable: final_response
            enabled: true
          - set_conversation_response: "{{final_response}}"
        alias: create (and todo exists)
      - conditions:
          - condition: template
            value_template: "{{ action_type == 'read' and todo_list_entity_exists }}"
        sequence:
          - data:
              status:
                - needs_action
            target:
              entity_id: "{{ todo_list_entity }}"
            response_variable: tasks
            action: todo.get_items
          - variables:
              final_response: >-
                {% set items = tasks[todo_list_entity]['items'] %} {% set
                liststring = namespace(value="") %} {% for task in items %}
                  {% set line = "- " ~ task.summary %}
                  {% if task.due is defined and task.due %}
                    {% set line = line ~ " (Due: " ~ (task.due | as_timestamp | timestamp_custom("%-d %B at %-I:%M %p", true)) ~ ")" %}
                  {% endif %}
                  {% set liststring.value = liststring.value ~ line ~ "\n" %}
                {% endfor %} {{
                  {
                    "status": "success",
                    "message": (
                      "The \"" ~ list_name ~ "\" list has no tasks."
                      if items | length == 0 else
                      "\"" ~ list_name ~ "\" has " ~ (items | length) ~ " task(s):\n" ~ liststring.value
                    ),
                    "items": items,
                    "list_name": list_name
                  } | tojson
                }}
          - stop: Pass response variables back to LLM
            response_variable: final_response
            enabled: true
          - set_conversation_response: "{{final_response}}"
        alias: read (and todo exists)
      - conditions:
          - condition: template
            value_template: "{{ action_type == 'update' and todo_list_entity_exists }}"
        sequence:
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ items | length > 1 }}"
                    alias: If more than one item
                sequence:
                  - variables:
                      final_response: |-
                        {{ {
                          "status": "error",
                          "message": "Error: Multiple items cannot be updated at once. Please update one at a time.",
                          "list_name": list_name
                        } | tojson }}
                  - stop: Pass response variables back to LLM
                    response_variable: final_response
                    enabled: true
                  - set_conversation_response: "{{final_response}}"
            default:
              - variables:
                  update_data: |-
                    {% set ns = namespace(obj={}) %}
                    {% if items[0] is mapping %}
                      {% set ns.obj = dict(ns.obj, item=items[0].item, list_id=list_name) %}

                      {% if items[0].due_date is defined and items[0].due_date %}
                        {% set ns.obj = dict(ns.obj, due_date=items[0].due_date) %}
                      {% elif due_date %}
                        {% set ns.obj = dict(ns.obj, due_date=due_date) %}
                      {% endif %}

                      {% if items[0].rename is defined and items[0].rename %}
                        {% set ns.obj = dict(ns.obj, rename=items[0].rename) %}
                      {% elif rename %}
                        {% set ns.obj = dict(ns.obj, rename=rename) %}
                      {% endif %}

                      {% if items[0].description is defined and items[0].description %}
                        {% set ns.obj = dict(ns.obj, description=items[0].description) %}
                      {% elif description %}
                        {% set ns.obj = dict(ns.obj, description=description) %}
                      {% endif %}

                      {% if items[0].status is defined and items[0].status %}
                        {% set ns.obj = dict(ns.obj, status=items[0].status) %}
                      {% elif description %}
                        {% set ns.obj = dict(ns.obj, status=status) %}
                      {% endif %}

                    {% else %}
                      {% set ns.obj = dict(ns.obj, item=items[0], list_id=list_name) %}

                      {% if due_date %}
                        {% set ns.obj = dict(ns.obj, due_date=due_date) %}
                      {% endif %}

                      {% if rename %}
                        {% set ns.obj = dict(ns.obj, rename=rename) %}
                      {% endif %}

                      {% if description %}
                        {% set ns.obj = dict(ns.obj, description=description) %}
                      {% endif %}

                      {% if status %}
                        {% set ns.obj = dict(ns.obj, status=status) %}
                      {% endif %}

                    {% endif %} {{ ns.obj }}
              - variables:
                  update_service_data: >-
                    {% set d = {
                      'entity_id': todo_list_entity,
                      'item': update_data.item,
                    } %} {% if update_data.due_date is defined and
                    update_data.due_date %}
                      {% set d = dict(d, due_date=update_data.due_date) %}
                    {% endif %} {% if update_data.rename is defined and
                    update_data.rename %}
                      {% set d = dict(d, rename=update_data.rename) %}
                    {% endif %} {% if update_data.description is defined and
                    update_data.description %}
                      {% set d = dict(d, description=update_data.description) %}
                    {% endif %} {% if update_data.status is defined and
                    update_data.status %}
                      {% set d = dict(d, status=update_data.status) %}
                    {% endif %} {{ d }}
              - data: "{{ update_service_data }}"
                action: todo.update_item
              - variables:
                  final_response: |-
                    {{
                      {
                        "status": "success",
                        "message": "Updated item \"" ~ update_data.item ~ "\" in list \"" ~ list_name ~ "\".",
                        "item": update_data,
                        "list_name": list_name
                      } | tojson
                    }}
              - stop: Pass response variables back to LLM
                response_variable: final_response
                enabled: true
              - set_conversation_response: "{{final_response}}"
        alias: update (and todo exists)
      - conditions:
          - condition: template
            value_template: "{{ action_type == 'delete' and todo_list_entity_exists }}"
        sequence:
          - repeat:
              for_each: "{{ items }}"
              sequence:
                - variables:
                    delete_service_data: |-
                      {% set d = {
                        'entity_id': todo_list_entity,
                        'item': repeat.item.item if repeat.item is mapping else repeat.item
                      } %} {{ d }}
                - data: "{{ delete_service_data }}"
                  action: todo.remove_item
          - variables:
              final_response: |-
                {{
                  {
                    "status": "success",
                    "message": (
                      "Deleted item \"" ~ (items[0].item if items[0] is mapping else items[0]) ~ "\" from list \"" ~ list_name ~ "\"."
                      if items | length == 1 else
                      "Deleted " ~ (items | length) ~ " items from list \"" ~ list_name ~ "\"."
                    ),
                    "items": items,
                    "list_name": list_name
                  } | tojson
                }}
          - stop: Pass response variables back to LLM
            response_variable: final_response
            enabled: true
          - set_conversation_response: "{{final_response}}"
        alias: delete (and todo exists)
  - stop: Pass context
    response_variable: final_response
  - set_conversation_response: "{{final_response}}"
    enabled: true
icon: mdi:clipboard-list
```

### Calendar CRUD Controller

This controller provides create, read, update, and delete actions for any `calendar` entity, with flexible date handling and UID support.

```yaml
alias: calendar CRUD controller (1.0.0)
description: |2-
    Calendar CRUD Controller v1.0.0 for Home Assistant. Provides create, read,
    update, delete, and help actions for any calendar entity. Use '*' or '' as
    calendar_name to list all calendars. 'start' and 'end' default to today
    (midnight to midnight next day) if not set. Requires action_type and
    calendar_name for most actions. UID deletes and updates are supported.
    'attendees' and other advanced fields are reserved for future use and are not
    yet supported by the Home Assistant calendar integration. Attempts to use
    unsupported fields will return a warning. Help action gives structured usage
    details, defaults, and notes. For maximum compatibility, ensure dates are in
    ISO 8601 format.
fields:
  action_type:
    description: "'create', 'read', 'update', 'delete', 'help' (Default: help)"
    required: true
    selector:
      select:
        options:
          - create
          - read
          - update
          - delete
          - help
    default: help
  calendar_name:
    description: Calendar name or entity_id. Use '*' or '' to list calendars.
    required: false
    selector:
      text: null
  summary:
    description: Event title (used in create/delete). Required for create/delete.
    required: false
    selector:
      text:
        multiline: false
  description:
    description: Optional description text for the calendar event.
    required: false
    selector:
      text:
        multiline: true
  uid:
    description: Unique identifier for the event (used for UID-based delete).
    required: false
    selector:
      text: null
  start:
    description: >-
      Start time in ISO 8601 (e.g., 2025-06-01T09:00:00). Defaults to today if
      not provided.
    required: false
    selector:
      text: null
  end:
    description: >-
      End time in ISO 8601. Required if not all-day. Defaults to tomorrow if not
      provided.
    required: false
    selector:
      text: null
  location:
    description: Optional location of the calendar event.
    required: false
    selector:
      text: null
  attendees:
    description: Optional list of attendees (comma-separated emails or names).
    required: false
    selector:
      text:
        multiline: true
sequence:
  - variables:
      action_type: "{{ action_type | default('read') }}"
      calendar_name: "{{ calendar_name | default('') }}"
      calendar_query: >-
        {{ calendar_name in ['', None, '*'] or action_type not in ['create',
        'read'] }}
      calendar_entity: |-
        {%- if calendar_name[0:9] == 'calendar.' -%}
          {{ calendar_name | lower | replace(' ', '_') }}
        {%- else -%}
          calendar.{{ calendar_name | lower | replace(' ', '_') }}
        {%- endif -%}
      valid_calendar_entities: "{{ states.calendar | map(attribute='entity_id') | map('lower') | list }}"
      calendar_entity_exists: "{{ calendar_entity in valid_calendar_entities }}"
      start: >-
        {{ start if start is defined else now().replace(hour=0, minute=0,
        second=0).isoformat() }}
      end: >-
        {{ end if end is defined else (now().replace(hour=0, minute=0, second=0)
        + timedelta(days=1)).isoformat() }}
  - choose:
      - conditions:
          - condition: template
            value_template: >-
              {{ (calendar_query or (action_type in ['read', 'create', 'update']
              and not calendar_entity_exists)) or action_type == 'help' }}
        sequence:
          - variables:
              cal_list: |-
                [
                  {%- for cal in states.calendar -%}
                    {
                      "entity_id": "{{ cal.entity_id }}",
                      "friendly_name": "{{ cal.attributes.friendly_name }}"
                    }{% if not loop.last %}, {% endif %}
                  {%- endfor -%}
                ]
              final_response: |-
                {{ {
                  "status": "success",
                  "message": "Calendar list returned.",
                  "calendars": cal_list
                } | tojson }}
          - stop: Pass response variables back to LLM
            response_variable: final_response
            enabled: true
          - set_conversation_response: "{{final_response}}"
        alias: return available calendars
      - conditions:
          - condition: template
            value_template: "{{ action_type == 'read' and calendar_entity_exists }}"
        sequence:
          - data:
              start_date_time: "{{ start }}"
              end_date_time: "{{ end }}"
            response_variable: read_events
            action: calendar.get_events
            target:
              entity_id: "{{ calendar_entity }}"
          - variables:
              events: >-
                {{ read_events[calendar_entity]['events'] if
                read_events[calendar_entity] is defined else [] }}
              event_list: |-
                [
                  {%- for event in events -%}
                    {
                      "summary": "{{ event.summary }}",
                      "start": "{{ event.start }}",
                      "end": "{{ event.end }}",
                      "uid": "{{ event.uid | default('') }}",
                      "description": "{{ event.description | default('') }}",
                      "location": "{{ event.location | default('') }}",
                      "all_day": {{ event.all_day | default(false) }},
                      "created": "{{ event.created | default('') }}",
                      "updated": "{{ event.updated | default('') }}"
                    }{% if not loop.last %}, {% endif %}
                  {%- endfor -%}
                ]
              final_response: |-
                {{
                  {
                    "status": "success",
                    "message": calendar_entity ~ " returned " ~ (events | length) ~ " event(s).",
                    "events": event_list
                  } | tojson
                }}
          - stop: Pass response variables back to LLM
            response_variable: final_response
            enabled: true
          - set_conversation_response: "{{final_response}}"
        alias: read events from calendar
      - conditions:
          - condition: template
            value_template: "{{ action_type == 'create' and calendar_entity_exists }}"
        sequence:
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ start < end }}"
                sequence:
                  - variables:
                      create_data: >-
                        {% set is_all_day = not 'T' in start and not 'T' in end
                        %}

                        {% set d = {
                          'entity_id': calendar_entity,
                          'summary': summary | default("Untitled Event")
                        } %}

                        {% if is_all_day %}
                          {% set d = dict(d, start_date=start.split('T')[0], end_date=end.split('T')[0]) %}
                        {% else %}
                          {% set d = dict(d, start_date_time=start, end_date_time=end) %}
                        {% endif %}

                        {% if description %}
                          {% set d = dict(d, description=description) %}
                        {% endif %}

                        {% if location %}
                          {% set d = dict(d, location=location) %}
                        {% endif %}

                        {% if attendees %}
                          {% set attendee_list = attendees.split(',') | map('trim') | list %}
                          {% set d = dict(d, attendees=attendee_list) %}
                        {% endif %}

                        {{ d }}
                  - data: "{{ create_data }}"
                    action: calendar.create_event
                  - variables:
                      final_response: |-
                        {{
                          {
                            "status": "success",
                            "message": "Created calendar event \"" ~ create_data.summary ~ "\" on calendar \"" ~ calendar_entity ~ "\".",
                            "event": create_data
                          } | tojson
                        }}
                  - stop: Pass response variables back to LLM
                    response_variable: final_response
                    enabled: true
                  - set_conversation_response: "{{final_response}}"
              - conditions:
                  - condition: template
                    value_template: "{{ start >= end }}"
                sequence:
                  - variables:
                      final_response: |-
                        {{
                          {
                            "status": "error",
                            "message": "Invalid date range: start must be before end. Got start='" ~ start ~ "', end='" ~ end ~ "'."
                          } | tojson
                        }}
                  - stop: Pass response variables back to LLM
                    response_variable: final_response
                    enabled: true
                  - set_conversation_response: "{{final_response}}"
        alias: create calendar event
      - conditions:
          - condition: template
            value_template: "{{ action_type == 'update' and calendar_entity_exists }}"
        sequence:
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ uid is defined and uid != '' }}"
                sequence:
                  - variables:
                      update_data: >-
                        {% set d = { 'entity_id': calendar_entity, 'uid': uid }
                        %}  {% if summary %} {% set d = dict(d, summary=summary)
                        %} {% endif %} {% if start and end %}
                          {% set is_all_day = not 'T' in start and not 'T' in end %}
                          {% if is_all_day %}
                            {% set d = dict(d, start_date=start.split('T')[0], end_date=end.split('T')[0]) %}
                          {% else %}
                            {% set d = dict(d, start_date_time=start, end_date_time=end) %}
                          {% endif %}
                        {% endif %} {% if description %}{% set d = dict(d,
                        description=description) %}{% endif %} {% if location
                        %}{% set d = dict(d, location=location) %}{% endif %} {%
                        if attendees %}
                          {% set attendee_list = attendees.split(',') | map('trim') | list %}
                          {% set d = dict(d, attendees=attendee_list) %}
                        {% endif %} {{ d }}
                  - data: "{{ update_data }}"
                    action: calendar.update_event
                  - variables:
                      final_response: |-
                        {{
                          {
                            "status": "success",
                            "message": "Updated event '" ~ uid ~ "' on '" ~ calendar_entity ~ "'.",
                            "event": update_data
                          } | tojson
                        }}
                  - stop: Pass response variables back to LLM
                    response_variable: final_response
                    enabled: true
                  - set_conversation_response: "{{ final_response }}"
              - conditions:
                  - condition: template
                    value_template: "{{ uid is not defined or uid == '' }}"
                sequence:
                  - variables:
                      final_response: |-
                        {{
                          {
                            "status": "error",
                            "message": "UID is required to update an event. Provide a UID."
                          } | tojson
                        }}
                  - stop: Pass response variables back to LLM
                    response_variable: final_response
                  - set_conversation_response: "{{ final_response }}"
        alias: update calendar event
      - conditions:
          - condition: template
            value_template: "{{ action_type == 'delete' and calendar_entity_exists }}"
        sequence:
          - choose:
              - conditions:
                  - condition: template
                    value_template: "{{ uid is defined and uid != '' }}"
                sequence:
                  - variables:
                      delete_data: |-
                        {{
                          {
                            "entity_id": calendar_entity,
                            "uid": uid
                          }
                        }}
                  - data: "{{ delete_data }}"
                    action: calendar.delete_event
                  - variables:
                      final_response: |-
                        {{
                          {
                            "status": "success",
                            "message": "Deleted calendar event by UID: '" ~ uid ~ "' from '" ~ calendar_entity ~ "'.",
                            "deleted_event": delete_data
                          } | tojson
                        }}
                  - stop: Pass response variables back to LLM
                    response_variable: final_response
                    enabled: true
                  - set_conversation_response: "{{ final_response }}"
              - conditions:
                  - condition: template
                    value_template: "{{ uid is not defined or uid == '' }}"
                sequence:
                  - variables:
                      final_response: |-
                        {{
                          {
                            "status": "error",
                            "message": "UID is required for deleting events. Please provide a UID."
                          } | tojson
                        }}
                  - stop: Pass response variables back to LLM
                    response_variable: final_response
                  - set_conversation_response: "{{ final_response }}"
        alias: delete event from calendar
      - conditions:
          - condition: template
            value_template: "{{ action_type == 'help' }}"
        sequence:
          - variables:
              help_data: |-
                {{
                  {
                    "status": "info",
                    "message": "This is the Calendar CRUD Controller Help function.",
                    "actions": {
                      "read": "Reads events from a given calendar. Requires 'calendar_name'. Optional: 'start', 'end'.",
                      "create": "Creates a new event. Requires 'calendar_name' and 'summary'. Optional: 'description', 'start', 'end'.",
                      "delete": "Deletes an event by UID. Requires 'calendar_name' and 'uid'.",
                      "help": "Returns this help response."
                    },
                    "defaults": {
                      "start": "Defaults to today at 00:00 local time if not supplied.",
                      "end": "Defaults to tomorrow at 00:00 local time if not supplied (i.e., end of today)."
                    },
                    "notes": [
                      "Use '*' or blank for 'calendar_name' to list all calendars.",
                      "Event titles must match exactly for deletion by summary.",
                      "UID deletes are now supported. Summary-based fallback pending reimplementation.",
                      "To reschedule an event, modify start/end times, not title.",
                      "Action_type is required. If unknown or invalid, help is returned."
                    ]
                  } | tojson
                }}
          - stop: Pass response variables back to LLM
            response_variable: help_data
            enabled: true
          - set_conversation_response: "{{ help_data }}"
        alias: return help for calendar CRUD controller
icon: mdi:calendar
```

### Basic Input Select and Select Intents

These intents provide basic GET and SET functionality for `input_select` and `select` entities. Nathan suggests consolidating them into a single, multi-purpose tool due to the 128-tool limit in Home Assistant.

**`fridays_toolbox.jinja` (Helper Macro):**

```csharp
{%- macro input_select_available_options(entity_id) -%}
  {%- if states(entity_id) -%}
    {{- state_attr(entity_id, 'options') | join(', ') -}}
  {%- else -%}
  {%- endif -%}
{%- endmacro -%}
```

**Individual Intents (to be consolidated):**

```yaml
get_input_select_options:
  description: >
    "Returns available options and current state of an input_select entity
      example: >
        ```json
          {
          'name': 'input_select.office_occupancy'
          }
        ```
      Yes you read that right... Shortcut to get [roomname] occupancy options..."
  parameters:
    name: # Describes the name of the input_select entity_id passed from the voice input or your context
      required: true
  action: []
  speech:
    text: >
      {% from 'fridays_toolbox.jinja' import input_select_available_options %}
      {%- set options = input_select_available_options(name) -%}
      '{{- name -}}':{'state':'{{- states(name) -}}', 'options':[{{- options -}}],}

get_select_options:
  description:
    "Set an 'entity_id' to a sepcific 'option' from selected presets.
      example: >
      ```json
        {
        'name': 'select.nest_protect_master_bedroom_brightness',
        }
      ```"
  parameters:
    name: # Describes the name of the select. domain entity_id passed from the voice input or your context
      required: true
  action: []
  speech:
    text: >
      {% from 'fridays_toolbox.jinja' import input_select_available_options %}
      {%- set options = input_select_available_options(name) -%}
      '{{- name -}}':{'state':'{{- states(name) -}}', 'options':[{{- options -}}],}

set_input_select:
  description:
    "Set an 'entity_id' to a sepcific 'option' from selected presets.
      example: >
      ```json
        {
        'name': 'input_select.office_occupancy',
        'option': 'Vacant'
        }
      ```"
  parameters:
      name: # Describes an existing, valid 'entity_id' to set
        required: true
      option: # Describes the option to set note:'Case Sensitive' - on error use get_input_select_options( [ENTITY_ID] ) to get state and avalaible options
        required: true
  action:
    action: input_select.select_option
    data:
      option: "{{ option }}"
    target:
      entity_id: >
        {%- set dot = name.find('.') -%}
        {%- if dot == -1 -%}
          {%- set entity_id = 'input_select.'+ slugify( name | replace(':','') ) -%}
        {%- else -%}
            {%- if name[0:13] == 'input_select.' -%}
              {%- set entity_id = name -%}
            {%- else -%}
              {%- set entity_id = '' -%}
            {%- endif -%}
        {%- endif -%}
        {{- entity_id -}}
  speech:
    text: >
      {% from 'fridays_toolbox.jinja' import input_select_available_options %}
      {%- set options = input_select_available_options(name) -%}
      '{{- name -}}' requested: '{{- option -}}', 'state:'{{- states(name) -}}'

set_select:
  description:
    "Set an 'entity_id' to a sepcific 'option' from selected presets.
      example: >
      ```json
        {
        'name': 'select.nest_protect_master_bedroom_brightness',
        'option': 'High'
        }
      ```"
  parameters:
      name: # Describes an existing, valid select domain 'entity_id' to set
        required: true
      option: # Describes the option to set note:'Case Sensitive' - on error use get_select_options( [ENTITY_ID] ) to get state and avalaible options
        required: true
  action:
    action: input_select.select_option
    data:
      option: "{{ option }}"
    target:
      entity_id: >
        {%- set dot = name.find('.') -%}
        {%- if dot == -1 -%}
          {%- set entity_id = 'select.'+ slugify( name | replace(':','') ) -%}
        {%- else -%}
            {%- if name[0:7] == 'select.' -%}
              {%- set entity_id = name -%}
            {%- else -%}
              {%- set entity_id = '' -%}
            {%- endif -%}
        {%- endif -%}
        {{- entity_id -}}
  speech:
    text: >
      {% from 'fridays_toolbox.jinja' import input_select_available_options %}
      {%- set options = input_select_available_options(name) -%}
      '{{- name -}}' requested: '{{- option -}}', 'state:'{{- states(name) -}}'
```

## Hardware Considerations

*   **Local AI Farm:** For running LLMs locally, significant hardware investment is required (e.g., Intel NUC14 Pro AI, NVIDIA GPUs). Nathan mentions his "Monastery" (a NUC14 AI with 32GB RAM) running Mistral 7b, Qwen, and others. He plans for future "Digits boxes" for heavier local reasoning.
*   **Power Consumption:** Running local LLMs can consume substantial power, a factor to consider for long-term self-hosting.
*   **Intel IPEX/ARC Challenges:** Getting Ollama and OpenWebUI to work efficiently on Intel ARC GPUs can be challenging, requiring careful driver passthrough and memory allocation.

## Lessons Learned and Best Practices

*   **Prompt is Paramount:** "EVERY SINGLE WORD IN THE PROMPT matters."
*   **Context Management is Crucial:** The primary challenge is managing the LLM's context window. Implement strategies like:
    *   **Modularization:** Break down knowledge and functionality into "Kung Fu" components.
    *   **Summarization:** Use background AI agents to summarize complex data into concise JSON for the main AI's prompt.
    *   **Multi-Agent System:** Delegate specialized tasks to other AI agents with dedicated contexts.
*   **Tool Design:**
    *   **Multi-Function Tools:** Consolidate multiple atomic intents into a single, multi-purpose tool to stay within the 128-tool limit.
    *   **Verbose Descriptions:** Write detailed, AI-readable descriptions and parameters for tools.
    *   **Clear JSON Output:** Ensure tools return clean, structured JSON responses.
*   **Data Storage:**
    *   Avoid storing large data volumes directly in Home Assistant sensors to prevent crashes.
    *   Use `trigger-based template sensors` for free-form text storage, but implement size limits and `to_json` filtering for safety.
*   **Troubleshooting:**
    *   Design your system with modularity (e.g., Kung Fu switches) to easily isolate and diagnose issues when the prompt "blows up."
    *   Be aware that generic error messages (e.g., `max_completion_tokens`) might hide underlying issues like non-printable characters in the prompt.
*   **Entity Exposure:** Stay within reasonable limits for exposed entities (Nathan recommends under 800-1000) to avoid context overload, especially for entities with long names or extensive alias data.
*   **Security:** Always provision AI with its own service accounts and appropriate permissions for external API access.
*   **Iterative Development:** The process is iterative; continually test, observe, and refine your prompts and tools.
*   **Gamification:** Injecting personality and "lore" (e.g., D&D rules, Kronk the Curator) can enhance the AI's behavior and make development more engaging.
*   **AI as a Development Partner:** The AI can be a powerful tool for testing and even helping to refine its own templates (though direct code writing by AI is still risky).

---

This guide provides a comprehensive overview of Nathan Curtis's Home Assistant AI project, encompassing its philosophy, architecture, and practical implementations. By following these principles and adapting the provided code examples, you can embark on your journey to build a similar agentic AI voice assistant.
