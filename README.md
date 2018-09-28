A very simple actor implementation.
The Actor model has two basic characteristics:
1) each actor instance sequentially processes the messages it received, one at a time, single threaded
2) different actor instances do the above concurrently

Here is a very simple version of how to do the Actor model (both C# and java).

First we define that an actor will have three status: available, executing and ended. When it is available, it means it can process a new message. When it is executing, it means it is currently processing a message and cannot take on a new message at this moment. And when it is ended, it will not process new messages.

The Actor class is in charge of the internal queue-like data structure to keep the messages received in order, while the GateKeeper class should have no idea of that. The GateKeeper is only responsible for checking the status of the actor. If the actor is available, then just call the actor's execute() method to let it work on its own thing. It uses CAS operation to make sure the status of the actor is correct and isolated from each message.

The main idea is, client -> Actor.addMessages() -> add message to queue / GateKeeper.readyToExecute() -> CAS actor status / execute in thread pool -> call real actor execute() method / CAS actor status / check if need to call GateKeeper.readyToExecute().

![Drag Racing](http://cloudjun.azurewebsites.net/posts/files/4d4bde10-471e-4369-b9ea-d394529cdf98.png)

Two key points are how we maintain the received message queue and how accurate we use CAS operations to keep track of the actor status. The message queue can actually be shared between different actors, not necessarily just one actor or multiple instances of the same actor.

The C# actor is here https://github.com/cloudjun/SimpleActor, and the java actor is here https://github.com/cloudjun/SimpleActorJava.
