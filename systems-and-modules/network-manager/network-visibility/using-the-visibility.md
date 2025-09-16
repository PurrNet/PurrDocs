# Using the visibility

You have to manually tick the visibility check, and this is for good reason!

Working with custom conditions could potentially be taxing in terms of processing, and a lot of unnecessary evaluations could easily happen. Working with visibility is generally depending on the use case, for example:

If you work with the [Distance condition](distance-condition.md), you might only want to re-evaluate the visibility once the identity has moved x-amount of units. So if we just evaluated every 1 second, and you have 1.000 identities all standing still, that's 1.000 evaluations every second that could have been easily avoidable.

Actually evaluating and identities visibility is luckily very easy! All you have to do is call the `EvaluateVisibility` method on any given identity. So this way you can easily custom make your own logic for when you check visibility.
