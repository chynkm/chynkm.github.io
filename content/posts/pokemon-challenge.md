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


## Initial Approach: The Quick and Simple Method

The first solution is straightforward — though not the most efficient. It involves querying the main Pokémon API endpoint to fetch a list of Pokémon. Then, for each Pokémon in that list, we make an additional request to retrieve its detailed data.

From there, we filter the results based on the given conditions. Finally, we return only the Pokémon that meet these criteria.  

While this method works, it's worth noting its a slow and inefficient process as we are working with a large dataset. The programs execution time averages around 170 seconds on my machine.

<script src="https://gist.github.com/chynkm/2447dcb318e40561aad09293b8078547.js"></script>


## Boosting Performance with Coroutines: `asyncio` + `aiohttp`

The next logical step is to parallelize the API calls to improve performance, especially since we need to make individual requests for each Pokémon’s detailed data.

By leveraging Python’s `asyncio` along with the `aiohttp` library, we can send multiple API requests concurrently instead of waiting for each one to finish before starting the next. This significantly reduces the overall runtime, making the solution much faster and more scalable.

This approach is ideal for I/O-bound tasks like querying external APIs — and it turns a slow, sequential loop into an efficient, coroutine-powered fetch operation. The programs execution time averages around 152 seconds on my machine.

<script src="https://gist.github.com/chynkm/87fd9736daa4f9f83ffda7f7a54e0a03.js"></script>