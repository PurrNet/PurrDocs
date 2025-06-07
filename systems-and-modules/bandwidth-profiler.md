---
icon: chart-column
---

# Bandwidth Profiler

The Bandwidth profiler allows you to do:

* Real-time Analysis: Visualize sent and received data with a live traffic graph.
* Detailed Breakdown: Inspect every RPC, see its parameters, and instantly highlight the source `GameObject` in the hierarchy.
* Save & Load Sessions: Debug builds by saving profiler data to a file and loading it back in the editor for analysis.

Accessing the profiler through `Tools/PurrNet/Analysis/Bandwidth Profiler`

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Bandwidth Profiler Window</p></figcaption></figure>

The profile also tries when possible to keep references to the sender/receiver so that you know exactly which components are sending the data.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

It also allows you to save the data at runtime to a file to allow inspecting and analyzing it post play sessions.

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>
