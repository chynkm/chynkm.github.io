---
author: "Karthik M"
title: "Python Coding Challenge: Filtering Pokémon from a Public API"
date: "2025-04-16"
tags:
- python
- asyncio
- aiohttp
- asynchronous
- api
- coding
- challenge
---

Recently, I encountered an interesting Python coding challenge during an interview. The task involved working with the [PokéAPI](https://pokeapi.co/) — a free and open RESTful API for Pokémon data.

Here's what the challenge asked:

1. Query the endpoint: https://pokeapi.co/api/v2/pokemon
2. From the results, extract and print a list of Pokémon that satisfy both of the following conditions:
    - The Pokémon's `base_experience` is greater than `200`.
    - The Pokémon have a `fire` listed among their `types`.
3. For each matching Pokémon, print the following details:
    - `name`
    - `height`
    - `sprite_url` (which should come from `sprites.front_default`)


## Initial Approach: The Quick and Simple Method

The first solution is straightforward, though not the most efficient. It involves querying the main Pokémon API endpoint to fetch a list of Pokémon. Then, for each Pokémon in that list, we make an additional request to retrieve its detailed data.

From there, we filter the results based on the given conditions. Finally, we return only the Pokémon that meet these criteria.  

While this method works, it's worth noting its a slow and inefficient process as we are working with a large dataset. The programs execution time averages around 170 seconds on my machine.

<script src="https://gist.github.com/chynkm/2447dcb318e40561aad09293b8078547.js"></script>


## Boosting Performance with Coroutines: `asyncio` + `aiohttp`

The next logical step is to parallelize the API calls to improve performance, especially since we need to make individual requests for each Pokémon's detailed data.

By leveraging Python's `asyncio` along with the `aiohttp` library, we can send multiple API requests concurrently instead of waiting for each one to finish before starting the next. This significantly reduces the overall runtime, making the solution much faster and more scalable.

This approach is ideal for I/O-bound tasks like querying external APIs, and it turns a slow, sequential loop into an efficient, coroutine-powered fetch operation. The programs execution time averages around 152 seconds on my machine.

<script src="https://gist.github.com/chynkm/87fd9736daa4f9f83ffda7f7a54e0a03.js"></script>


## Optimized and Scalable: Structured Concurrency with `asyncio.Queue`

While firing off all requests in parallel can work for small datasets, it quickly becomes a bottleneck when working with hundreds of items, especially when dealing with API rate limits or system memory constraints.

To overcome this, we can adopt a producer-consumer pattern using `asyncio.Queue`. This method lets us strike a perfect balance between concurrency and control:

- A producer coroutine fetches the initial list of Pokémon and pushes individual detail URLs into a queue.
- Multiple consumer coroutines then pick items from the queue and process them concurrently, but in a controlled manner, defined by the number of workers(`NUM_CONSUMERS`).

By limiting the number of simultaneous API calls, this approach avoids common pitfalls like connection timeouts, rate limiting, and memory spikes, while still being highly performant due to asynchronous I/O.

Here's why this structure shines:

- Scalable: Easily control concurrency by tweaking the number of consumers.
- Resilient: Gracefully handles large datasets without overwhelming the network or system.
- Organized: The code is modular and easier to extend (e.g., add retry logic, logging, or error handling per task).

This solution combines the best of both worlds, the speed of `aiohttp` with the reliability of structured task management. The programs execution time averages around 29 seconds on my machine(achieves a 6x speed improvement over the initial approach).

<script src="https://gist.github.com/chynkm/25be57285494836b62d8ebf4647148df.js"></script>


## Bonus Thought: Dual Queues for Full Parallelism

An advanced variation of this approach is to introduce a separate queue for page-level tasks (i.e. producers). This would allow you to process paginated API responses and enqueue the `next` pages in parallel while still processing the `results` concurrently.

I'll leave this as an exercise for curious readers who are interested in a setup that handles both pagination and item processing concurrently. It's a great way to push the boundaries of structured concurrency in Python.


## Wrapping up

Using `asyncio` and `aiohttp` with thoughtful concurrency patterns like queues can significantly improve performance and scalability when working with external APIs. Whether you're dealing with a small dataset or a paginated API, understanding when and how to parallelize is key to writing efficient and elegant asynchronous code.

Happy coding and may your Pokémon always be fire-type with a high base experience!


## requirements.txt File

<script src="https://gist.github.com/chynkm/65c992524d3c755715654958fe7accfd.js"></script>

All the required Python dependencies for running the script are listed in the `requirements.txt` file. To set up your environment with these modules, simply run the following command:

```
pip3 install -r requirements.txt
```

This will ensure that all the necessary libraries are installed before you execute the script.