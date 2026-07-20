---
icon: chart-column
---

# Bandwidth Profiler

Networking bugs are some of the hardest to track down. When something feels laggy or your game starts stuttering with more players, you need to know exactly what data is being sent, how much of it, and which objects are responsible. Without visibility into your network traffic, you're debugging blind.

The Bandwidth profiler gives you that visibility. It allows you to do:

* Real-time Analysis: Visualize sent and received data with a live traffic graph.
* Detailed Breakdown: Inspect every RPC, see its parameters, and instantly highlight the source `GameObject` in the hierarchy.
* Dropped Messages: See which fragmented unreliable messages never completed, including the payload type and why they were dropped.
* Save & Load Sessions: Debug builds by saving profiler data to a file and loading it back in the editor for analysis.

Accessing the profiler through `Tools/PurrNet/Analysis/Bandwidth Profiler`

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Bandwidth Profiler Window</p></figcaption></figure>

The profile also tries when possible to keep references to the sender/receiver so that you know exactly which components are sending the data.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

When a [fragmented unreliable message](network-manager/README.md#unreliable-messages-and-the-mtu) fails to assemble, the sample shows it under Dropped Messages grouped by payload type and drop reason. Expired means a fragment never arrived within the timeout, Evicted and BudgetExceeded mean the receiver was under reassembly pressure, and SequencedStale means a newer sequenced message superseded it. If the drops are unexpected, the type tells you exactly which message to shrink or move to a reliable channel.

It also allows you to save the data at runtime to a file to allow inspecting and analyzing it post play sessions.

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>
