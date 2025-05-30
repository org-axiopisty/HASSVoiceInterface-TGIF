---
aliases:
  - 2. an `intent_script` for `search recipes`
tags:
  - intent
  - script
  - search
  - recipes
  - code
  - codeblock
  - Friday
  - Mealie
  - yaml
date: 2025-05-30
---
```yaml
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