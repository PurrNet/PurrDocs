# Distance condition

The Distance Rule is a built-in [visibility rule](visibility-rules.md). A player sees an identity while one of the objects that player owns is within range of it, and stops receiving it once they are too far away.

## Setting it up

1. Create the rule through `Create > PurrNet > NetworkVisibility > DistanceRule`.
2. Add it to a rule-set, and assign that rule-set to the [Network Manager](../) or as an override on the identities that should use it. See [Visibility rules](visibility-rules.md).
3. Evaluate visibility as things move. See [Evaluating visibility](using-the-visibility.md).

{% hint style="warning" %}
The rule is not re-checked on its own when objects or players move. Without step 3, what a player sees is decided once when the object spawns or when the player joins, and then never changes.
{% endhint %}

## Settings

| Setting        | What it does                                                                                                                                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Layer Mask** | Which of the player's owned objects count as "where the player is". Only owned identities on these layers are measured from.                                                                                 |
| **Distance**   | The range, in units. Defaults to 30.                                                                                                                                                                         |
| **Dead Zone**  | Extra distance an identity that is already visible has to move away before it is hidden again. Defaults to 5. With the defaults an object appears within 30 units and only disappears beyond 35, which keeps it from flickering when a player hovers around the edge. |

The distance is measured from the identities the player **owns**, typically their character. A player who doesn't own anything on the layer mask can't see any object that relies on this rule. Owned objects that are inactive or disabled are skipped.

## How it works

This is the logic of the rule. It can also serve as a starting point if you want to write your own variation, for example one that only measures on the horizontal plane.

```csharp
using PurrNet;
using UnityEngine;

namespace YourNamespace
{
    [CreateAssetMenu(menuName = "PurrNet/NetworkVisibility/My Distance Rule")]
    public class MyDistanceRule : NetworkVisibilityRule
    {
        [SerializeField] private LayerMask _layerMask = ~0;
        [SerializeField, Min(0)] private float _distance = 30f;
        [SerializeField, Min(0)] private float _deadZone = 5f;

        public override int complexity => 100;

        public override bool CanSee(PlayerID player, NetworkIdentity networkIdentity)
        {
            var myPos = networkIdentity.transform.position;
            bool wasPreviouslyVisible = networkIdentity.IsObserver(player);

            foreach (var playerIdentity in manager.EnumerateAllPlayerOwnedIds(player, true))
            {
                var layer = playerIdentity.layer;

                if ((_layerMask & (1 << layer)) == 0)
                    continue;

                if (!playerIdentity.isActiveAndEnabled)
                    continue;

                var playerPos = playerIdentity.transform.position;
                var distance = Vector3.Distance(myPos, playerPos);

                if (wasPreviouslyVisible)
                {
                    if (!(distance <= _distance + _deadZone))
                        continue;
                }
                else if (!(distance <= _distance))
                    continue;

                return true;
            }

            return false;
        }
    }
}
```
