# Predicted Projectile (3D)

* Purpose: Emulate simple physics like a straight velocity or simple bouncing bullets or other spherical objects. Utilizes casting during flight to avoid weird collision issues as seen with the Unity Rigidbody.
* Notes:
  * Similar to the PredictedRigidbody, it has it's own physics events to handle collision detection. These are similar to the known Unity ones and can be subscribed to by referencing the component.
  * Can utilize a physics material
  * Doesn't require collider setup, as it uses it's own build in radius value for the casting.
  * Can act as a trigger

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
