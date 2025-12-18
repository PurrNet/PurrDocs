# SyncBigData

`SyncBigData` is module that can be used by itself but also powers other modules related to sending big amount of data to other players.

The general idea is simple, you give it a big array of bytes, specify if it's owner auth and how many kb/s you want it to use and then it does the rest by itself. This naturally makes it asynchronous, the data your provide will take time to arrive and the receivers need to wait for it to be ready.

Ownership changes and re-connections are also handled gracefully:

* If ownership is lost mid-upload then it is canceled and the previous valid data is kept.
* If owner disconnects and re-connects (using the cookie session) they will start downloading the latest state from the server, if this was an image for example they will retrieve the same image that was last sent.

