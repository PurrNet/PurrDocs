# SyncTimer

The SyncTimer allows for easy synchronizing of a timer automatically counting down.

You can easily do actions such as **starting**, **stopping, pausing and resuming** the timer. Most methods accessible should be quite self-explanatory.

The SyncTimer also automatically handles reconciliation of the timer, meaning that it will force align all clients to ensure de-syncing doesn't happen. The more frequent, the more precise it will be, but the more data is used. Generally it is very data light, so don't fear for making the number lower if necessary.

Below is an example usage of the SyncTimer being server authorized (default) and having a reconcile interval of 3 (default)

```csharp
public TMP_Text timerText;

//false = OwnerAuth
//3 = Reoncile interval
//The reconcile interval deciphers how often it will force align all clients
private SyncTimer timer = new();

private void Awake() 
{
    //onTimerSecondTick is called every 1 second
    timer.onTimerSecondTick += OnTimerSecondTick;
}

protected override void OnSpawned(bool asServer)
{
    //This starts the timer with 30 seconds countdown
    if(isOwner)
        timer.StartTimer(30f);
}

private void OnTimerSecondTick()
{
    //You can also get .remaining to get the precise float value
    //For displaying timers, the remainingInt makes it easy
    timerText.text = timer.remainingInt.ToString();
}

private void PauseGameTimer() 
{
    //Pauses the timer and will sync the remaining time because it's set to true
    timer.PauseTimer(true);
}

private void ResumeGameTimer()
{
    //Will resume the timer from where it was paused
    timer.ResumeTimer();
}
```

Beyond using the SyncTimers automatic countdown, you can also advance the timer yourself with whatever delta you please giving you more freedom to count up, down, faster or slower.&#x20;

The bare minimum setup of this would love like this:

```csharp
private SyncTimer _timer = new(manualUpdate:true);

private void Update()
{
    _timer.Advance(Time.deltaTime);
}
```
