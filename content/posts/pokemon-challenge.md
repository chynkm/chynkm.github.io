---
author: "Karthik M"
title: "Python Coding Challenge: Filtering Pokémon from a Public API"
date: "2025-04-16"
tags:
- python
- asyncio
- api
- coding
- challenge
---

Recently, I encountered an interesting Python coding challenge during an interview. The task involved working with the [PokéAPI](https://pokeapi.co/) — a free and open RESTful API for Pokémon data.

Here’s what the challenge asked:
1. Query the endpoint: https://pokeapi.co/api/v2/pokemon
2. From the results, extract and print a list of Pokémon that satisfy both of the following conditions:
    - Their `base_experience` is greater than `200`.
    - They have `fire` listed among their `types`.
3. For each matching Pokémon, print the following details:
    - `name`
    - `height`
    - `sprite_url` (which should come from `sprites.front_default`)
