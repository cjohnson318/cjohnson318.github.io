---
layout: post
title:  "Simple Deterministic Push Down Automata"
date:   2024-02-09 19:00:00 -0700
categories: kotlin
---

This was another programming exercise to get more comfortable with Kotlin. A deterministic push down automaton (DPDA) is basically a finite state machine with a stack for memory. Since finite state machines (FSM) generally use their states as memory, this allows you to have fewer states in your model. The downside is that you will have more numerous and complex state transitions. Whereas with a finite state machine you consider the current state and a new event to determine the next state, with a push down automaton you need to consider the current state, the incoming event, and the top of the stack before determining the next state and the next action to perform on the stack, either pushing a symbol, popping one off, or doing nothing.

Below, I wrote a DPDA for determining whether a sequence of parentheses were closed properly or not. If the parentheses all have closing pairs, then when the DPDA reaches the semicolon at the end, it will be in the "halt" state, otherwise it will be in the "error" state. The thing to notice here is that I only have three states, "initial", "halt", and "error", but this state machine can handle an arbitrarily long sequence of parentheses, given enough memory.

{% highlight kotlin %}
val pass: Unit = Unit

data class State(val state: String)

data class Event(val event: String)

data class StackSymbol(val symbol: String)

enum class StackActionType {
    PUSH,
    POP,
    PASS,
}

data class StackAction(val type: StackActionType, val symbol: StackSymbol?)

data class CurrentStateEvent(val state: State, val event: Event, val peek: StackSymbol?)

data class NextStateAction(val state: State, val action: StackAction? = null)

class TransitionTable {
    private var stateTable = mutableMapOf<CurrentStateEvent, NextStateAction>()

    fun addTransition(current: State, event: Event, peek: StackSymbol?, next: State, action: StackAction? = null) {
        val currentStateEvent = CurrentStateEvent(current, event, peek)
        stateTable[currentStateEvent] = NextStateAction(next, action)
    }
    fun getNextState(currentStateEvent: CurrentStateEvent): NextStateAction? {
        return when (currentStateEvent) {
            in stateTable -> stateTable[currentStateEvent]
            else -> null
        }
    }
}

class Stack<T> {
    var stack = mutableListOf<T>()

    fun push(item: T) {
        stack.add(item)
    }
    fun pop(): T? {
        return when {
            stack.count() > 0 -> stack.removeAt(stack.count()-1)
            else -> null
        }
    }
    fun peek(): T? {
        return when {
            stack.count() > 0 -> stack[stack.count()-1]
            else -> null
        }
    }
}

class DeterministicPushDownAutomata constructor(initial: State, val transitionTable: TransitionTable) {
    val error = NextStateAction(State("error"))

    var state = initial
    var stack = Stack<StackSymbol>()

    fun send(event: Event) {
        val currentStateEvent = CurrentStateEvent(state, event, stack.peek())
        val next = transitionTable.getNextState(currentStateEvent) ?: error
        println("Received $event. Transitioning from $state to $next")
        state = next.state
        if (next.action != null) {
            when (next.action.type) {
                StackActionType.PUSH -> stack.push(next.action.symbol!!)
                StackActionType.POP -> stack.pop()
                StackActionType.PASS -> pass
            }
        }
    }
}

fun main(args: Array<String>) {
    val initial = State("initial")
    val halt = State("halt")
    val error = State("error")

    val openParen = Event("open-paren")
    val closeParen = Event("close-paren")
    val semicolon = Event("EOL")

    val openParenSymbol = StackSymbol("(")
    val closeParenSymbol = StackSymbol(")")

    val pushOpenParen = StackAction(StackActionType.PUSH, openParenSymbol)
    val pushCloseParen = StackAction(StackActionType.PUSH, closeParenSymbol)
    val popParen = StackAction(StackActionType.POP, null)

    val table = TransitionTable()

    table.addTransition(initial, openParen, null, initial, pushOpenParen)
    table.addTransition(initial, openParen, openParenSymbol, initial, pushOpenParen)
    table.addTransition(initial, openParen, closeParenSymbol, initial, pushOpenParen)

    table.addTransition(initial, closeParen, null, error)
    table.addTransition(initial, closeParen, openParenSymbol, initial, popParen)
    table.addTransition(initial, closeParen, closeParenSymbol, initial, pushCloseParen)

    table.addTransition(initial, semicolon, null, halt)
    table.addTransition(initial, semicolon, openParenSymbol, error)
    table.addTransition(initial, semicolon, closeParenSymbol, error)

    val dpda = DeterministicPushDownAutomata(initial, table)
    dpda.send(openParen)
    dpda.send(openParen)
    dpda.send(closeParen)
    dpda.send(openParen)
    dpda.send(closeParen)
    dpda.send(closeParen)
    dpda.send(semicolon)
    println(dpda.state)
}
{% endhighlight %}

