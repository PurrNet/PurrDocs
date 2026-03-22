# Network Rules

In most networking solutions, you're locked into a single authority model. Either the server controls everything, or you're on your own. This means that during early development, you're often fighting the networking just to test basic gameplay, and when you're ready to ship, switching to a stricter model can require rewriting a lot of code.

Network Rules solve this by letting you configure your authority model in one place, without changing any of your game code. These allow for you to fully customize your networking experience for the easiest possible workflow or full server auth to ensure a cheat free experience.

{% embed url="https://youtu.be/oa5Y46ruuGk" %}

You can make your own network rule-set, but PurrNet also comes pre-packaged with some. In most cases, you won't need any more than these. If you want the best/easiest possible development loop, you should go for the "Unsafe" rule-set. This allows for the most freedom as a developer. If you want it to act as most networking systems with full server authority, you should select the "ServerStrict". However, the "ServerOwner" will also allow a great deal more flexibility, without great cost of cheating.

The rules are pretty self-explanatory if you take a look at the scriptable. It will allow you to adjust things such as (but not exclusive to):

* Authorization to spawn and despawn
* The default owner
* How ownership acts when given
* Who can assign ownership
* Who is responsible of synchronizing values
* And much more.
