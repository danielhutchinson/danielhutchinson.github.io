---
layout: post
title:  "Encapsulating Logic Using the Command Pattern"
date:   2019-04-19
categories: javascript design-patterns
---

Separation of Concerns (SoC), you have probably heard of this term already. In short, it is an important design principle in which you separate your code into individual distinct sections. A piece of software that has a good SoC is normally referred to as being modular. Modular code has the benefits of being reusable, easier collaboration, better testability, easier controlling of application scope, and so on. A great way of ensuring modularity in your code is by implementing the Command Pattern.

> This is the first in a series of posts on Design Patterns, implemented in Typescript. This series assumes that you have a basic knowledge of Object Oriented (OO) principles, and know your way around classes and interfaces. The examples given are all written in Typescript, but the concepts outlined here apply to most any programming language that supports OO concepts.

## What is the problem we are trying to solve?
Design patterns are tools, and like with any other tool we should use it to solve a problem… say for example we are building a remote control for a television, with a bunch of buttons for adjusting volume, or setting the power state on whatever active television the remote control is linked to.

You might initially go in and add a bunch of if else statements...

```javascript
export default class RemoteControl {
    private tv: Television;
    
    constructor(activeTelevision: Television) {
        this.tv = activeTelevision;
    }

    public pressButton(button: string): void {
        if(button === 'VolumeUp') {
            this.tv.adjustVolumeUp();
        }
        else if(button === 'VolumeDown') {
            this.tv.adjustVolumeDown();
        }
        else if(button === 'PowerOff') {
            this.tv.powerOff();
        }

        //... and so on
    }
}
```
Great! It works!

But… it’s a little bit of a mess. I can already see that this solution isn’t going to scale very well.

We have no encapsulation of commands, not to mention that every time we want to add a new button command in the future we need to go in and add a new else if to the RemoteControl class. Not so great…

> Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification <small>- The O in SOLID.</small>

It’d be great if we could extend the behaviour of the remote without having to modify its code in any way.

At this point, we should take a step back and look at our design.

## Enter the Command Pattern
The Command Pattern is a behavioural design pattern that allows us to decouple the requester of an action, from the receiver.

In our case, the requester (or client) would be the `RemoteControl` class and the receiver would be the `Television` class.

We do this by introducing command objects into our design:

- We create a new command for each piece of logic that our remote control needs to execute
- We create an invoker, that has these commands set at run time
- When a button is pressed our `RemoteControl` tells the invoker to invoke the command assigned to that action

In this scenario, we have decoupled our `RemoteControl` object from our `Television` object.

Let's take a look at this in practice.

## Implementation

First things first, we need to create an interface for the command object

```javascript
export default interface Command {
    execute(): void;
}
```

A simple interface called `Command`, with a single method call on it that returns void is what we need to implement on each of our command objects.

Let’s go ahead and create a concrete command to turn the TV volume up.

```javascript
export default class VolumeUpCommand implements Command {
    private tv: Television;

    constructor(tv: Television) {
        this.tv = tv;
    }
    
    public execute(): void {
        this.tv.adjustVolumeUp();
    }
}
```
So we have a class that implements our `Command` interface, when constructing this command we will be accepting an instance of the receiver, in this case, the Television class. When execute is invoked on our new command it will simply call `adjustVolumeUp` on the tv object.

Alright, let’s create an invoker to call this command.

```javascript
export default class RemoteControlButton {
    private command: Command;

    public setCommand(command: Command): void {
        this.command = command;
    }

    public onButtonPressed(): void {
        this.command.execute();
    }
}
```

We have created a `RemoteControlButton` class with a field called command, that not surprisingly holds any class that conforms to the `Command` interface. We have also added a `setCommand` method, that we can use to set commands!

The `onButtonPressed` method is used to call `execute` on whatever command the class has assigned. With this change, the `RemoteControlButton` no longer needs to know about the `Television`. We have successfully decoupled these constructs from one another.

## What have we done!?
We have encapsulated a request as an object. With these requests, we are able to create other objects that accept these requests as parameters.

This means that our concrete commands are totally reusable and sharable across any number of objects.

But there is still a bit missing for a full solution… we need to create the client.

## Assigning Commands
We need a solution for creating and assigning multiple concrete commands. Let’s finally revisit our `RemoteControl` class, and modify it to create concrete implementations of the necessary commands and assign them to its buttons. For the sake of brevity, I will just add three buttons, in the real world, your solution would probably be a lot more sophisticated.

First, we will implement our previously created `VolumeUpCommand` into the client.

```javascript
export default class RemoteControl {
    public volumeUpButton: RemoteControlButton;

    constructor(tv: Television) {
        this.volumeUpButton = new RemoteControlButton();
        this.volumeUpButton.setCommand(new VolumeUpCommand(tv));
    }
}
```

When the `volumeUpButton` has its `onButtonPressed` method called (from a GUI or somesuch), it will `execute` the `VolumeUpCommand`.

Now let's create two more commands, and assign them to the `RemoteControl` client class.

```javascript
export default class VolumeDownCommand implements Command {
    private tv: Television;

    constructor(tv: Television) {
        this.tv = tv;
    }
    
    public execute(): void {
        this.tv.adjustVolumeDown();
    }
}
```

```javascript
export default class PowerOffCommand implements Command {
    private tv: Television;

    constructor(tv: Television) {
        this.tv = tv;
    }
    
    public execute(): void {
        this.tv.powerOff();
    }
}
```

```javascript
export default class RemoteControl {
  public volumeUpButton: RemoteControlButton;
  public volumeDownButton: RemoteControlButton;
  public powerOffButton: RemoteControlButton;

  constructor(tv: Television) {
    this.volumeUpButton = new RemoteControlButton();
    this.volumeDownButton = new RemoteControlButton();
    this.powerOffButton = new RemoteControlButton();

    this.volumeUpButton.setCommand(new VolumeUpCommand(tv));
    this.volumeDownButton.setCommand(new VolumeDownCommand(tv));
    this.powerOffButton.setCommand(new PowerOffCommand(tv));
  }
}
```

Great! Now it’s pretty trivial to add as many commands as we like, in nicely encapsulated chunks of logic that can be maintained and tested separately from the implementation of the remote control itself!

While we may have ended up with more code than our first solution of using if else statements, our new design is much more maintainable, scalable and contains more reusable components.

## Live example

I have embedded a full working example of what we created above here:
<iframe src="https://codesandbox.io/embed/n97ynm82pl?previewwindow=console&runonclick=1&codemirror=1&editorsize=60" style="width:100%; height:500px; border:0; border-radius: 1em; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>

> This is the first in a series of posts on Design Patterns, implemented in Typescript. See you soon!
