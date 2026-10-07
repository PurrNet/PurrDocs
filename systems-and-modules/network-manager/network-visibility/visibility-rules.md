# Visibility rules

A visibility rule is a small asset that answers one question: can this player see this identity? Rules are grouped in a **rule-set**, and the rule-set is what you assign to the [Network Manager](../) or to an identity.

## Creating a rule-set

1. Create the rules you need through `Create > PurrNet > NetworkVisibility`.
2. Create a rule-set through `Create > PurrNet > NetworkVisibility > Rule Set` and add your rules to its list.
3. Assign the rule-set to **Visibility Rules** on the Network Manager.

That rule-set is now the default for every identity.

A player sees an identity as soon as **one** rule in the set says yes. Rules don't have to agree. This also means a rule-set without any rules hides everything it applies to.

Regardless of what the rules say, a player sees the identities they own, and [whitelists and blacklists](whitelist-and-blacklist.md) are checked before anything else. The full order is listed in [How the server decides](./#how-the-server-decides).

## Built-in rules

| Rule               | What it does                                                                                              |
| ------------------ | --------------------------------------------------------------------------------------------------------- |
| **Always Visible** | Everyone sees the identity. Handy as an override for objects that should ignore a stricter default.       |
| **No Visibility**  | Nobody sees the identity, apart from its owner and whitelisted players.                                   |
| **Distance Rule**  | Players see the identity while something they own is within range. See [Distance condition](distance-condition.md). |

## Overriding the rule-set per identity

The rule-set on the Network Manager is only the default. On any Network Identity you can open **Override Defaults** in the inspector and assign a different rule-set to **Visibility Override**. That identity then ignores the default and uses its own.

An override is meant to be set once per prefab. A rule-set has a **Children Inherit** toggle, on by default, that makes it spread: the override is also used by the network components below it on the same GameObject and by the networked children, until it reaches a component that has an override of its own. With the toggle off, it applies to the component it is assigned on and nothing else.

{% hint style="warning" %}
The override does not reach the network components that sit **above** it on the same GameObject. Assign it on the first network component of the object, the top one in the inspector, or the object stays visible through the components above.
{% endhint %}

Two more things to know about how it spreads:

* **It isn't passed on for players on that component's lists.** For a player who is on the whitelist or blacklist of the component carrying the override, the components below it and the children are checked as if the override wasn't there.
* **A child evaluated on its own doesn't see it.** When `EvaluateVisibility` is called on a child, or one of the child's lists changes, that child is checked against the Network Manager's rule-set, not the one it inherits. Evaluate from the root, or give such a child its own override.

The same can be done from code with `SetVisibilityRules(ruleSet)`, before or after the identity spawns. Like any change of rules, it shows once the identity is [evaluated](using-the-visibility.md) again.

## Writing your own rule

Inherit from `NetworkVisibilityRule`, give it a menu entry so you can create the asset, and implement `complexity` and `CanSee`:

```csharp
using PurrNet;
using UnityEngine;

[CreateAssetMenu(menuName = "PurrNet/NetworkVisibility/Same Team")]
public class SameTeamRule : NetworkVisibilityRule
{
    public override int complexity => 10;

    public override bool CanSee(PlayerID player, NetworkIdentity target)
    {
        // TeamMember and Teams stand in for your own game code
        return target.TryGetComponent(out TeamMember member) && Teams.GetTeam(player) == member.team;
    }
}
```

Create the asset from the menu you gave it and add it to a rule-set like any built-in rule.

A few things to keep in mind:

* `CanSee` only runs on the server.
* It isn't called for the owner of the identity, nor for whitelisted or blacklisted players. Those are settled before the rules.
* Rules are asked from the lowest `complexity` to the highest, and the first yes ends the check. Give cheap rules a low number so the expensive ones often don't have to run.
* The `manager` field gives you the Network Manager the rule belongs to.
* One rule asset is shared by every identity that uses it, so don't store per-object state on it.
* Your rule is only asked when visibility is evaluated. If the answer depends on something that changes during play, you have to trigger a new evaluation yourself. See [Evaluating visibility](using-the-visibility.md).

## Changing the default rules at runtime

Rules can be added to and removed from the Network Manager's rule-set while the game runs. This needs a rule-set to be assigned on the Network Manager to begin with, otherwise adding a rule logs an error and does nothing.

```csharp
networkManager.AddVisibilityRule(networkManager, rule);
networkManager.RemoveVisibilityRule(rule);
```

Rules added this way don't have to be assets. Any class implementing `INetworkVisibilityRule` works. Adding or removing a rule doesn't change what anyone sees until the affected identities are [evaluated](using-the-visibility.md) again.
