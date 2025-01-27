# Network Transform

This component allows you to automatically network the Position, Rotation, Scale and parent of the corresponding transform to which it's attached.

**Settings:**

**Sync Settings:** These allow you to easily change what is being synchronized.\
**Interpolate Settings:** These allow you to modify which values gets interpolated (smoothed)\
**Owner Auth:** If false, the server will have authority\
**Sync Parent:** If you change the parent of the transform, will that also be synced.\
**Send interval ticks:** How often data should be sent.\
\
**Tolerances:** This allows you to adjust how much values should change before it syncs. Simply to avoid unnecessary synchronizing for numbers so small that they aren't noticeable.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption><p>The Network Transform component with the default values</p></figcaption></figure>
