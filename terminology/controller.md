# Controller

The controller of an identity is essentially the one that is most authorized over this object.

isController returns true if:\
\- You are the owner\
or:\
\- There is no owner and you are the server

However, there might also be scenarios where something can be toggled as to whether it is ownerAuthed or not. If it is not owner authed, only the server will ever be the controller. The above logic still applies for owner authed logic.
