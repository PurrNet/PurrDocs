# Network Animator

This allows you to automatically network any parameter value within the Animator. This component needs to be on the same object as an Animator, otherwise it won't work.

{% embed url="https://youtu.be/rNtxwYkAVT4" %}

It will expose the values within the animator, of which you can toggle on and off in order to choose which to sync and which not to sync.

**Beware:** This does not automatically sync "triggers". A trigger would have to be called through the NetworkAnimator component, similar to how it is called on a regular animator.&#x20;

This is how you'd call a trigger:

```csharp
private NetworkAnimator _netAnimator;

private void DoPunch() {
    _netAnimator.SetTrigger("Punch");
}
```

The Network Animator also allows for you to control the animations directly through the Network Animator, so you don't need a reference to both the normal Animator as well as the Network Animator. So you can easily handle parameters as well:

```csharp
private NetworkAnimator _netAnimator;
private RigidBody _playerRigidbody;

private void DoPunch() {
    _netAnimator.SetFloat("Speed", _playerRigidbody.Velocity.magnitude);
    _netAnimator.SetBool("IsMoving", _playerRigidbody.Velocity.magnitude > 0.1f);
    //This can be done for all actions of the Animator. Above is just two examples.
}
```
