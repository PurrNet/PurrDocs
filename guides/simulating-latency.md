# Simulating latency

In order to test locally and still have a realistic scenario, we need to be able to simulate latency. Some tools have built in latency simulators, but we've found these to not be good representations of reality (as some things might get simulated different to others)

The best/only way to get a true to world latency experience when testing locally, would be to throttle the whole process (Unity or builds) or even the whole system.

We do intend to implement a system in the future that can throttle on a process specific basis for you, but this is not something we have for now.

For this there are plenty of tools that can work, we personally use [Clumsy](https://github.com/jagt/clumsy)
