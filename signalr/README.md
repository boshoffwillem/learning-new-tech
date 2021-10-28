# SignalR Documentation

SignalR is used when real-time commucation is necessary.
In SignalR there is what's called a "Hub" that's on the server.
All clients connect to the Hub.
The Hub can push out data to all the clients at once or only to
a specific client. Obviously clients can push data to the Hub as well.

SignalR's Hub can only be written in C#,
but clients can be any language.
