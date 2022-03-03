---
toc: true
layout: post
description: Learn how to utilize Finite State Machines in Part 1
categories: [FTC, Kotlin]
title: Finite State Machines
---

A common problem new programmers face in robotics is running a several complex seriesof actions asynchrounously. Finite state machines (FSMs) are often used to solvethis. FSMs are literally as they sound: machines that have states. At any time, the FSM runs a single state. On completion of this state, this FSM will iterate to the next state. This repeats until the last state in the FSM has been completed.


## Defining our expectations
In each stage of the program, the FSM needs to know the current state and actionto run. Furthermore, the FSM should know when to transition to the next stage, as well as running an action during these transitions. To reiterate:

1. Transition to stage N
2. Stage N-1 exit action runs
3. Stage N enter action runs
4. Stage N loop action runs
5. Check if stage N should transition
6. If true, run stage N exit action, transition, and run stage N+1 enter action

- Implementation wise, we want to instantiate a FSM without too much effort. The easiest way to construct a dynamic object is with the builder pattern.

- Having several actions would be nice as well, so actions for each stage should be held in lists.

- To be as flexible as possible, the FSM should have the ability to stop or reset at any time. 

- Transitions should easily implement a timer based system for timed transitions


## Implementation (Kotlin)
To implement these requirements, we will follow my state machine library writtenin Kotlin. I took inspiration for the library from [@NoahBres's Jotai library](https://github.com/NoahBres/jotai).  

First, lets implement an "action". An action should be a single function that simply runs. In Kotlin, the easiest method of writing such a requirement is to usea Functional SAM (Single Abstract Method) Interface.

```
fun interface Action {
	fun run()
}
```

We can now pass this action interface into functions to invoke the run() method,
similar to how a lambda functions. Functional interfaces in kotlin have special syntax

```
fun runAction(myAction: Action) {
	myAction.run()
}

runAction { println("Hello World!" }
```


Now let us define a similar functional interface for transitioning between states. The state machine will repeatedly check the transition condition to see if the current state is finished, meaning we need a method returning a boolean.

```
fun interface Transition {
	fun shouldTransition(): Boolean
}

```

Similarly to ```Action```, ```Transition``` can also use functional interface syntax.

```
fun printShouldTransition(t: Transition) {
	print(t.shouldTransition())
}
val t = { true }
printShouldTransition(t)
```
>returns true

Now we have enough to define a "State". Referring back to our definitions of a state, we need to have lists of actions for each stage of the state, as well as atransition condition. Each state should have an Enum associated with it, as thiswill simplify writing code. Data classes in Kotlin serve this need well, as theypregenerate some useful functions. Specific actions should be ran when entering the state, when looping before transitioning, and then on transition.

```
data class State<T> {
	var state: T,
	var onEnter: MutableList<Action>,
	var onLoop: MutableList<Action>,
	var onExit: MutableList<Action>,
	var transition: TransitionCondition = { true }
|
```

Notice I defaulted the transition variable to a value of true. This is personal preference, as if I have a state with no transition condition I want the state machine to assume the state is finished. If you don't want this feature, requiring that a state has a transition condition when building the state machine is an easy addition that I will elaborate on later.

Now that we have classes for Actions, Transitions, and States, all that is left is to write the main StateMachine logic and the StateMachineBuilder class utilizng the builder pattern for ease of use.

I will skim over some details of my StateMachine class with psuedocode, as the main class is fairly trivial. If you wish to read my StateMachine code, it is available [here](https://github.com/14607/FF-Private/blob/master/TeamCode/src/main/java/robotuprising/koawalib/statemachine/StateMachine.kt).

The header of the StateMachine class is standard. Each state machine has a generic associated with it, which will be supplied on creation of a StateMachine object as an enum, and a list of states.

```
class StateMachine<T>(private val stateList: List<State<T>>) {
...
}
```

Having the state machine running at all times would be harmful so a ```running``` variable would be useful, as well as a ```currentState``` variable to save the current state we are running in ```stateList```. Current state can be either an indice value (so, an integer) or a "T", but that is up to the reader to decide. There's no real benefit besides whatever is easier to you.

Next, lets define what the state machine does when it starts. On start, the running variable should be set to true, and all enter actions of the current state should run. Note that current state does not necessarily denote the first state to run; a state machine can be stopped and started at any phase while traversing the stateList.

```
fun start() {
	running = true
	currentAction.enterActions(Action::run)
}
```

Our next function, update() is also fairly straightforward. Update is called periodicly while the state machine is running. It first checks if the currentState's transition condition is true, and if so, transitions. If the state machine has stopped running (running == false), it returns and stops the update() function. After these 2 checks, it runs all the loopActions of the currentState. I'll leave the implementation up to the reader for this one.

The last function of the StateMachine class is our transition() function. This is called when the currentState's transition condition is true in the update() method. The function first runs all the exitActions of the currentState, and then sets currentState = to the next state in stateList. As I implemented currentState to be <T>, I used stateList.indexOf(currentState) to find the current state in the stateList, but currentState as an index would simply be currentState++. Then, we run all the enterActions of the newState, which is our currentState.

And that's it! You can now use your StateMachine class to program complicated movements on your robots that might've otherwise taken hours of frustration in a much shorter timeframe. I'll soon write a post on how to implement the builder method to quickly create a stateList effortlessly. In the meanwhile, I'll add a small code snippet displaying how the builder pattern works in my FTC programming ecosystem. Good luck!

```
val IntakeEnum {
	ON,
	CHECKING_FOR_MINERAL,
	OFF
}

val intakeStateMachine = StateMachineBuilder<IntakeEnum>()
	.state(ON)
	.onEnter(intake::turnOn)
	.transition { true }
	
	.state(CHECKING_FOR_MINERAL)
	.transition(sensor::isMineralIn)

	.state(OFF)
	.onEnter(intake::turnOff)
	.transition { true }
	
	.build()
```






