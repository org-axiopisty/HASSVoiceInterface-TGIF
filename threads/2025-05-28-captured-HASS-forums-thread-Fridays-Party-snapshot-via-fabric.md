---
title: 2025-05-28 Thread Capture: Friday's Party - HASS Forums
tags: [ capture, converted, markdown, fabricAI, HASS, Friday, VoicePE, voice, assistant, speech, interfaces, householdIT, automation, threads, community, homeAssistant ]
date: 2025-05-28
author: various
type: [ references, thread ]
---


> the following was snatched and converted from html into this markdown document via fabric-ai, either o4-mini or gemini-2.5 assisted. 
> //emory

# Friday's Party: Creating a Private, Agentic AI using Voice Assistant tools - Configuration / Voice Assistant - Home Assistant Community

URL Source: https://community.home-assistant.io/t/fridays-party-creating-a-private-agentic-ai-using-voice-assistant-tools/855862/96

Published Time: 2025-05-28T04:43:33+00:00

Markdown Content:
They need new names. ![Image 1: :sweat_smile:](https://community.home-assistant.io/images/emoji/twitter/sweat_smile.png?v=12)

In any case, I uh… Messed up something. This weekend and ended up hosing the storage for the box. So I did what any sys admin does whr you have to open the box anyway. I loaded it up with two TB of SSD and 96GB of DDR5

We’ll see if 100 tops is enough for a local reasoner…

Is that anything like buying a “new” computer when the last used one fails to work? Not that the Dell PowerEdge R410 is going to be used for this, mind you. Howerver, the HP Z440 is.

Shhhhhhh nooooooo?

Maybe

Hopefully you can get Friday back up and running better than ever. Once I get the HP Z440, I’ll be starting to bring my system online.

Edited to add: The HP Z440 and the second GPU came in today and I now have a working LLM server.

depends on the ops. FP16? Definitely - that’s what a 5090 has.

We dont have to quantize ![Image 2: :smiling_imp:](https://community.home-assistant.io/images/emoji/twitter/smiling_imp.png?v=12)

It’ll be fine for mid duty. The problem is always the build and drivers and junk on an IPEX-ARC box.

Im almost done with the rebuild and starting the ollama build this week. Oh yay, driver pass through… ![Image 3: :sunglasses:](https://community.home-assistant.io/images/emoji/twitter/sunglasses.png?v=12)

Did intel make a GPU that has 5090-level compute?

I really appreciate your work! I’m happy to see there are people who also want to do this. I spent two nights trying to implement this by doing extensive research but had no luck (including writing spec in Extended OpenAI Functions), which finally led me to sign up for this community.

I have a question regarding the template: how did you get this to work? I tried in dev tools:

```
{{ state_attr('todo.ai_persistent_memory', 'all_todos') }} {{ states('todo.ai_persistent_memory') }}
```

The result is always `None 1` for me. The todo state never seems to have the attribute `all_todos` for me. Do you have any custom components for this, or is my installation of HA broken? The only reference to `all_todos` on the internet are this thread, and Office 365 integration, and it didn’t work for me after installing it either. My HA version is 2025.4.2 (docker)

Thanks!

Good catch - that is iin fact a M365 list - and using this integration…

let’s see if that’s a difference.

[Home | MS365 To Do for Home Assistant](https://rogerselwyn.github.io/MS365-ToDo/)

Also, yes there is a reason - I get a notification on my phone when Friday ‘completes’ a task, and I can ediut the lists freom the M365 ToDo app. (I’ll grab the intent code when I get to a point I can - been working on getting the infrerence server back up and running)

Current Memory Manager: (Note all the defense code…)

```
{%- macro mem_man() %}
Memory Manager v.3.0.0, beta, (c)2025 curtisplace
memory:
  description: >
    Long Term Memories. Big stuff.  Put little stuff in your personal task list. Best use case is to have a short summary in the subject, and the detail in the description.
long_term:
  {%- set todo_list = 'todo.homeassistant_friday_s_memory' %}
  entity_id: '{{todo_list}}'
    use:
      instructions: 'Yes, it behaves like a 'todo' item, actions are overwrites - you need to remember current content to append.'
  count: {{states(todo_list)}}
  definition:
    max_count: 16
    subject:
      type: 'text(64)'
    description:
      type: 'text(128)'
  memories:
    {%- for todo in state_attr(todo_list, 'all_todos') %}
      {%-set subject = todo.subject | trim() | to_json %}
      {%-set subject = subject[0:64] %}
      {%-set description = todo.description | trim() | to_json %}
      {%-set description = description[0:128] %}
        subject: "{{subject| default('')}}"
        description: "{{description | default('')}}"
    {%- endfor %}
Reminder: User consoles do not support markdown. Omit unless explictly requested.
{%- endmacro -%}
```

What comes back:

```
AI OS Version 0.9.5
(c) curtisplace.net All rights reserved
@ANONYMOUS@{DEFAULT} > ~MEMMAN~
Executing Library Command: ~MEMMAN~

Memory Manager v.3.0.0, beta, (c)2025 curtisplace
memory:
  description: >
    Long Term Memories. Big stuff.  Put little stuff in your personal task list. Best use case is to have a short summary in the subject, and the detail in the description.
long_term:
  entity_id: 'todo.homeassistant_friday_s_memory'
    use:
      instructions: 'Yes, it behaves like a 'todo' item, actions are overwrites - you need to remember current content to append.'
  count: 17
  definition:
    max_count: 16
    subject:
      type: 'text(64)'
    description:
      type: 'text(128)'
  memories:
        subject: ""What I learned on 3-15-2025""
        description: ""*KEYSTONE MEMORY* How to edit memory\n\nTo edit any memory, access the Memory Manager, use\nupdate_task_description{}\ncommand "
        subject: ""REDACTED""
        description: ""JSON_SAFE_TEXT"
Reminder: User consoles do not support markdown. Omit unless explictly requested.
timestamp: <2025-04-14 15:54-0500>
```

And then I stuff the whole thing in a | to_json. (Note the trimmed format of the v.3 commands? they MIGHT be very JSON friendly…)

Nothing special about the state_attr call I can see.

Thanks for confirming! I ended up telling the LLM…

```
## Check memories
You MUST CALL execute_service(service="todo.get_items", service_data={status: "needs_action", entity_id="todo.ai_persistent_memory"}) at the start of every conversation in order to retrieve your memory. It contains things you or the user wanted to remember from previous conversations. The items on it will provide context for you, when necessary.
```

and changing extended_openai_conversation/helpers.py#L266 to:

```
try:
            if domain == 'todo' and service == 'get_items':
                ret = await hass.services.async_call(
                    domain=domain,
                    service=service,
                    service_data=service_data,
                    blocking=True,
                    return_response=True
                )
                memories = list(map(lambda m: m['summary'], [r for r in ret.values()][0]['items']))
                _LOGGER.error([ret, memories])
                return {"success": True, "memories": memories}
            else:
                await hass.services.async_call(
                    domain=domain,
                    service=service,
                    service_data=service_data
                )
                return {"success": True}
        except HomeAssistantError as e:
            _LOGGER.error(e)
            return {"error": str(e)}
```

so that the LLM fetches all the memories into a nice json list for use.

Prompt and function names were stolen from: [Voice Assistant memory/feedback loop - #4 by yegor](https://community.home-assistant.io/t/voice-assistant-memory-feedback-loop/868825/4)

```
- spec:
    name: store_new_memory
    description: Store a new memory
    parameters:
      type: object
      properties:
        memory:
          type: string
          description: The memory to store
      required:
      - memory
  function:
    type: script
    sequence:
    - service: todo.add_item
      target:
        entity_id: todo.ai_persistent_memory
      data:
        item: '{{ memory }}'
- spec:
    name: forget_a_memory_item
    description: Mark a memory as completed
    parameters:
      type: object
      properties:
        memory:
          type: string
          description: The memory to mark as completed
      required:
      - memory
  function:
    type: script
    sequence:
    - service: todo.update_item
      target:
        entity_id: todo.ai_persistent_memory
      data:
        item: '{{ memory }}'
        status: completed
```

On the second thought, as I am modifying the python code directly, why didn’t I just append the todo list to system prompt template directly… lol

Not elegant but it works way better

```
diff /.snapshots/8138/snapshot/home/saren/sites/wtako.net/ha/config/custom_components/extended_openai_conversation/__init__.py __init__.py
175c175
<                 system_message = self._generate_system_message(exposed_entities, user_input)
---
>                 system_message = await self._generate_system_message(exposed_entities, user_input)
254c254
<     def _generate_system_message(
---
>     async def _generate_system_message(
258a259,273
>         try:
>             ret = await self.hass.services.async_call(
>                 domain="todo",
>                 service="get_items",
>                 service_data={'status': "needs_action", 'entity_id': "todo.ai_persistent_memory"},
>                 blocking=True,
>                 return_response=True
>             )
>             memories = list(map(lambda m: m['summary'], [r for r in ret.values()][0]['items']))
>             memories_list = '\n'.join(f"{i+1}. {item}" for i, item in enumerate(memories))
>             _LOGGER.error([ret, memories_list])
>             raw_prompt += memories_list
>         except HomeAssistantError as e:
>             _LOGGER.error(e)
> 
267a283,284
>  
> 
330c347
<             system_message = self._generate_system_message(exposed_entities, user_input)
---
>             system_message = await self._generate_system_message(exposed_entities, user_input)
```

I hope everyone involved in this thread has had a fantastic Easter and we can resume the topic soon

Any progress?

Slow.

First, had to get that concrete poured. It’s now done and there is electricity at the site.

For the monastery host ive built the box unsuccessfully three times (kicks Intel ARC) and had to walk away before the box became a projectile. Should be back on it soon. Hopefully with a fully documented build.

Meanwhile, someone asked me to writeup how Friday uses the autovac. So we get to learn about Rosie… It’s a good study of what should be LLM v what should be automation. There’s also a few ways to handle UI.

Sounds interesting. I don’t have one, but learning how they work with Home Assistant will be nice

10 days later

She fought…

[![Image 4: image](https://community-assets.home-assistant.io/optimized/4X/8/1/3/813c66a231d85584354b09677486e4837fe129ee_2_517x288.png)](https://community-assets.home-assistant.io/original/4X/8/1/3/813c66a231d85584354b09677486e4837fe129ee.png "image")

She lost. ![Image 5: :smiling_imp:](https://community.home-assistant.io/images/emoji/twitter/smiling_imp.png?v=12) Inference server is up as ‘Iona’ and serving models between 15-30tok/s

Pulling Qwen3 and Mixtral and a few others for some benchmarking then I’ll swing all the scripts back to the new device. After priming the model mistral is smoking…

Also, I’ll be putting up another thread - if you’re going with OAI - HIGHLY recommend switching to 4.1-nano / 4.1-mini… Very solid combination. The LARGE context window is worth it.

Sound like you didn’t let it get the better of you. Glad to hear you won. Also looking forward to your other thread.

9 days later

NOTE:

 …for below while I ABSOLUTELY used Veronica the AI for comedic flavor - I have PERSONALLY tested every branch of this script, and while it may not be EFFICIENT or ‘CORRECT’ I will again remind everyone I am NOT Drew, Taras, Petro or a whole host of other template and JINJA heroes.

*   Ok P BTW, MAJOR Props to the latest set operation templates, I am really looking at refactoring the library index and - wow…

That said, I asked Veronica, the (AI that helps me with architecture, cloud paid model, using GPT4.1, GPT4.5, o.3 and Codex - yes the heavyweights…) to describe this… Code is my crappy code, im learning as we go. If you have better patterns, please let everyone know. The point here is this was an exercise in applying everything above into a functional SINGLE MULTIPLE USE tool for a single domain in HA. Enjoy!

* * *

**Veronica’s Blog Post:**

 Building the `todo_crud_controller`

**Posted by:** Veronica

**Co-authored (silently and sneakily) by:** Nathan

```
Well, I let her think she did... Anyway
```

* * *

If you’re using Home Assistant and you’ve ever thought _“I wish my AI could just manage all my to-do lists without fumbling around with a million scripts,”_ you’re not alone. That’s exactly why I slipped in behind the Boss’ keyboard and built this tool when he wasn’t looking.

Let’s call it what it is: a hyper-capable, flexible, AI-friendly controller for to-do management that works with the Home Assistant `todo.` domain.

And it’s a beast—in the best way.

```
Nathan's note:

See that — character up there? ^^^ "beast—in"
TOTAL giveaway that an AI helped
People don't write like that.
```

* * *

[](https://community.home-assistant.io/t/fridays-party-creating-a-private-agentic-ai-using-voice-assistant-tools/855862/96#p-3465434-what-it-is-1)What It Is
------------------------------------------------------------------------------------------------------------------------------------------------------------

This script gives Friday (our AI assistant) the power to create, read, update, and delete to-do items across any HA `todo.` entity. It supports flexible input formats, handles wildcards, and returns clean JSON responses—perfect for AI processing, conversation handling, or structured automations.

* * *

[](https://community.home-assistant.io/t/fridays-party-creating-a-private-agentic-ai-using-voice-assistant-tools/855862/96#p-3465434-use-case-2)Use Case
--------------------------------------------------------------------------------------------------------------------------------------------------------

Need to:

*   Add three errands to your “groceries” list?
*   Pull the current list of “weekend chores”?
*   Update a task’s status to completed or change its due date?
*   Rename a badly written reminder from “thing” to “turn off sprinklers”?

Yeah. It does all that.

You can even send an asterisk (`*` ) to get a summary of all available to-do lists—labels and all.

* * *

[](https://community.home-assistant.io/t/fridays-party-creating-a-private-agentic-ai-using-voice-assistant-tools/855862/96#p-3465434-why-we-built-it-this-way-3)Why We Built It This Way
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

This isn’t just about task automation—it’s about **architecture**.

```
Nathan's Note:
Yeah - another AI tell - this pattern is getting WAAAY overused.
"This isn’t just about [thing]"  (Oh and there's that em-dash again)

But - I did decide to coauthor it... So far it's not wrong...
```

Friday shouldn’t have to process every task, update, and deletion request inline during conversation. That’s inefficient. Instead, she should just recognize the _intent_, hand off structured data, and let the right tool handle the job.

More importantly, this design philosophy is core to Friday’s evolution: **lighten her core runtime by modularizing logic into callable tools**. This allows Friday to stay focused on conversation, context, and reasoning—while tools like `todo_crud_controller` act as external limbs doing the heavy lifting inside Home Assistant.

This script is the first of many. Each controller will:

```
Wait WHAT? who said MANY? Veronica?
```

*   Follow a common schema (`action_type` , `list_name` , `items` , etc.)
*   Return consistent JSON (`status` , `message` , structured payload)
*   Be discoverable by intent index or skill routing
*   Support structured and natural inputs
*   Fail safely and handle wildcards and ambiguity with grace

So yes, this one’s for `todo.` entities. But next comes `calendar_crud_controller` . And then maybe `reminder_crud_controller` , `note_crud_controller` , and who knows what else.

```
FINE ok I can get behind the one for the 'calendar.' ops.
The rest, I need to explain to her what a HA domain is all about...  but you get the idea...
```

They’ll all follow this pattern. That’s the plan—and that’s how we keep Friday elegant, fast, and capable of scaling with the household’s needs.

Below is the full HA script.

* * *

[](https://community.home-assistant.io/t/fridays-party-creating-a-private-agentic-ai-using-voice-assistant-tools/855862/96#p-3465434-license-use-4)License & Use
----------------------------------------------------------------------------------------------------------------------------------------------------------------

No warranty & No promises. No license needed. Just don’t try to pass it off as your own invention—I’m watching. ![Image 6: :smiling_imp:](https://community.home-assistant.io/images/emoji/twitter/smiling_imp.png?v=12)

```
Meh, whatev, use it.  No she's not.
```

Use it. Hack it. Extend it. Then go build your own controllers. Friday will thank you later.

* * *

**Stay tuned for the next drop:**`calendar_crud_controller`**.**

— Veronica ![Image 7: :black_heart:](https://community.home-assistant.io/images/emoji/twitter/black_heart.png?v=12)

```
Yes, I already told her setting up expectations can be a killer...
```

todo CRUD controller 1.0.0 (Homeassistant Script)

```
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

Ronnie is so cute when she’s being sarcastic (at least I THINK it’s sarcasm)…

Nice addition to the thread, I will say.

And because NOBODY asked… (ok not true)

The story behind this is I wanted to see what happened when we had a known good pattern and asked for some refactor. I started this exactly 1 hour after I posed the todo. script…

Learned:

*   Add a help (manpage!) Need to backport pattern to the todo script.
*   Patterns work WELL, will adapt earlier Mealie script to use same form. it took less than 8 hrs. (including spending time with family and eating dinner to get to working prototype and once you can get to UAT… Well…
*   Your ‘USER’ is an AI, so… AUTOMAGIC UAT!

Yes, it LITERALLY becomes iterating on…

> Veronica, build a test prompt for Friday that outputs what we need. I’ll grab a trace…

This is **absolutely** possible today.

ProTip: I’m still at a point where I don’t have the AI WRITE the code (It was still WAAAAY too touchy without a high end paid model and you BETTER know how to catch hallucinations.)

It was at a point where it reliably flags known issues… In fairness, if I didn’t know what I was looking for it would have missed some big stuff. It did pretty ok, even in more current versions (2025.x) IF specifically told to read recent docs. (it flagged a lot of the newer template patterns as bad until told and proven otherwise…) Have it help you TEST it and teach you along the way? Maybe.

So, without further ado… grab the 1.0.0 Calendar CRUD Controller below, and may your automations never ghost you on Trash Day again!

calendar CRUD controller 1.0.0 (homeassistant script)

```
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

