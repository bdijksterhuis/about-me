# Technical Writing Sample – HeliFlightSchool Bot Core

Please note: I tend to write technical explanations for mixed audiences (PMs, developers, stakeholders), so my style leans toward clarity and accessibility rather than pure brevity.

The reusable core of the HeliFlightSchool bot is built around Discord's interaction system. It's job is to resolve any incoming interaction and match it up to the correct handler for that interaction.  

On boot, the bot auto-discovers all of its components: commands, handlers, templates, services, and subscribers. 

- Commands: Stored in definition.js files. These contain the Discord.js builders to create commands. 
- Handlers: These contain the actual business logic. 
- Templates: Markdown templates that get parsed by Mustache
- Services: Shared code that needs to be accessed by multiple commands. 
- Subscribers: Listeners for the implementation of the event system. 

As the bot runs through the handlers, templates, and services, it will generate unique identifiers for each. The identifiers for the handlers are called **routes**. These routes serve as keys the system uses to identify which piece of code should handle a given interaction.

This information is all loaded into a central state variable called the **registry**. The bot then registers itself with Discord and starts listening for incoming interactions. 

When a user interacts with the bot, Discord sends an interaction object to the app. The bot uses that data to generate a route key and attempts to match it against a known key in the registry. If no match is found, a user-friendly error is returned. If a match *is* found, the bot immediately decides on how to deal with Discord's strict timeouts. By default, bots have only 3 seconds to respond, however responses can be deferred (up to a minute, usually) if more time is needed. For the most part the bot will defer a response if it is able to due to the inherent unpredictability of the connection speed between the app and the database.

After a response has been deferred, the bot will build a second state object called the **context**. The context includes the original interaction data, plus any other information that might be useful or required to the handler. This context is then passed as a parameter into the handler's main function. 

Once the handler is finished with its logic, it uses a custom **response()** function that is provided by the context. This response function is deliberately built to be very flexible. Based on the arguments passed to it, it will figure out how to format the data and send the correct response to the end-user. A plain string can be sent out as is, a regular JavaScript Object usually gets passed to the bot's templating engine, which will attempt to use the interaction's context to resolve the correct template key, and it then loads the markdown content and uses Mustache to parse the the markdown, using the JavaScript Object as data. A handler can even pass in complex Discord.js Builder objects, which will get properly formatted and sent off to the user. 

Finally, the bot also has extensive logging and error handlers built in, including logging for performance. This ensures issues are easy to trace and that the bot can be monitored over time as new features are added. 

This approach means the bot is capable of taking any incoming Discord interaction, resolving it reliably, and respond in a flexible, user-, *and developer-* friendly way. More importantly, it does all this in a modular, extensible, and maintainable fashion.