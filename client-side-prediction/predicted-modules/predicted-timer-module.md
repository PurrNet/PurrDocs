# Predicted Timer Module

The TimerModule is a built-in module for handling networked countdowns. It manages the synchronization of time between server and client, ensuring that timers roll back correctly during prediction corrections.

#### Basic Usage

Instantiate the module within your `PredictedIdentity`. By default, the timer automatically ticks down by `delta` every simulation tick.

```csharp
public class Bomb : PredictedIdentity
{
    private TimerModule _fuseTimer;

    protected override void LateAwake()
    {
        // Create the timer (default: automatic countdown)
        _fuseTimer = new TimerModule(this);
        
        // Hook into events
        _fuseTimer.onTimerEnded += Explode;
        _fuseTimer.onPredictedTimerUpdated_View += UpdateTimerUI;
    }

    public void Activate()
    {
        // Only the server (or owner, if allowed) needs to start it
        if (isServer)
            _fuseTimer.StartTimer(5.0f);
    }

    private void Explode() { /* Boom */ }

    private void UpdateTimerUI(float timeRemaining)
    {
        // Use the event value for smooth UI updates
        _uiText.text = timeRemaining.ToString("F1");
    }
}
```

#### Manual Ticking

You can disable automatic counting to control the timer manually (e.g., for a charging weapon that only advances while a button is held).

```csharp
protected override void LateAwake()
{
    // Enable manual tick mode
    _chargeTimer = new TimerModule(this, manualTick: true);
}

protected override void Simulate(ref MyState state, float delta)
{
    if (state.isCharging)
    {
        // Manually advance the timer
        // Use -delta to count down, or +delta to count up
        _chargeTimer.TickTimer(-delta); 
    }
}
```

#### API References

<table data-header-hidden><thead><tr><th width="266"></th><th></th></tr></thead><tbody><tr><td><strong>Member</strong></td><td><strong>Description</strong></td></tr><tr><td>Properties</td><td></td></tr><tr><td><code>remaining</code></td><td>The current authoritative time remaining.</td></tr><tr><td><code>isTimerRunning</code></td><td>Returns <code>true</code> if the timer is currently active (value is not null).</td></tr><tr><td><code>predictedViewTimer</code></td><td>The smoothed time value for visual use (Update/LateUpdate).</td></tr><tr><td>Methods</td><td></td></tr><tr><td><code>StartTimer(float time)</code></td><td>Sets the timer to the specified value and starts it.</td></tr><tr><td><code>StopTimer(bool silent)</code></td><td>Stops the timer. If <code>silent</code> is true, <code>onTimerEnded</code> is not invoked.</td></tr><tr><td><code>TickTimer(float amount)</code></td><td>Manually adjusts the timer by the given amount.</td></tr><tr><td>Events</td><td></td></tr><tr><td><code>onTimerEnded</code></td><td>Invoked when the timer reaches 0.</td></tr><tr><td><code>onPredictedTimerUpdated_View</code></td><td>Invoked when the visual time changes. Use this for UI.</td></tr><tr><td><code>onVerifiedTimerUpdated_View</code></td><td>Invoked when the authoritative state is updated.</td></tr></tbody></table>
