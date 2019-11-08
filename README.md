# REDIS PATTERNS COOKBOOK
## A collection of most commonly used patterns in [Redis](https://redis.io/)

A collection of most common [Redis](https://redis.io/) patterns, used to resolve the frequently use cases with the most awesome _in memory database_.

You can use this repo like a simple cookbook or use [_Redis Patterns Console_](https://acadevmy.github.io/redis-patterns-console) to try and study these patterns online!

### Contribute 
If you wish to contribute with new or missing patterns, please follow these simple rules or use an existent pattern as a template.
- Describe the pattern in a single [markdown](https://www.markdownguide.org/basic-syntax/) file stored in the _patterns_ folder.
- Use the name of the pattern for the file's name with [snake_case naming style](https://en.wikipedia.org/wiki/Snake_case).
- Use an [H1 heading](https://www.markdownguide.org/basic-syntax/#headings) for the title of the content.
- Use an [H2 heading](https://www.markdownguide.org/basic-syntax/#headings) for the sections of the content.
- Use a [link](https://www.markdownguide.org/basic-syntax/#headings) to define a[Redis](https://redis.io/)command to execute on [_Redis Patterns Console_](https://acadevmy.github.io/redis-patterns-console).  
You must write the string of command to execute inside of square brackets and _#run_ string inside of parentheses.  
For instance, you can use this markdown syntax to create a link used to execute a SET command on [_Redis Patterns Console_](https://acadevmy.github.io/redis-patterns-console): _[SET foo 1]\(#run)_
- Use a [link](https://www.markdownguide.org/basic-syntax/#headings) to show the help of a[Redis](https://redis.io/)command on [_Redis Patterns Console_](https://acadevmy.github.io/redis-patterns-console).  
You must write the string of command to execute inside of square brackets and _#help_ string inside of parentheses.  
For instance, you can use this markdown syntax to create a link used to show a SET command official help on [_Redis Patterns Console_](https://acadevmy.github.io/redis-patterns-console): _[SET foo 1]\(#run)_
- Use [horizontal rules (dashes)](https://www.markdownguide.org/basic-syntax/#horizontal-rules) to break the steps of pattern tutorial on [_Redis Patterns Console_](https://acadevmy.github.io/redis-patterns-console).

Thanks for your future contributions!

&nbsp;

### 🚀 Redis Patterns Console (Angular SPA) 🚀
You can try and go into the deep of[Redis](https://redis.io/)and its patterns with an interactive (and reactive) online console!  
Visit [https://acadevmy.github.io/redis-patterns-console](https://acadevmy.github.io/redis-patterns-console) and enjoy it!

[_Redis Patterns Console_](https://acadevmy.github.io/redis-patterns-console) is an Angular SPA and an open-source project, so you can contribute if you wish.

Visit our repo on GitHub:  
[https://github.com/acadevmy/redis-patterns-console](https://github.com/acadevmy/redis-patterns-console)

### ⚙️ Redis WebSocket Server (NodeJS) ⚙️
[_Redis WebSocket Server_](https://github.com/acadevmy/Redis-websocket-server) is a simple NodeJS script used as a bridge to interact with [Redis Server](https://redis.io/) via WebSocket.
[_Redis Patterns Console_](https://github.com/acadevmy/redis-patterns-console) uses [_Redis WebSocket Server_](https://github.com/acadevmy/Redis-websocket-server) to communicate between [Redis Server](https://redis.io/) and Angular SPA.

Visit our repo on GitHub:  
[https://github.com/acadevmy/Redis-websocket-server](https://github.com/acadevmy/Redis-websocket-server)

&nbsp;

Maintained with ❤️ by [Acadevmy](https://www.acadevmy.it/intro)