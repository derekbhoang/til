# Asynchronous Programming

A way of writing software so a program can start a task, continue doing other work, and handle the result later instead of waiting for the task to finish.

The core idea is:
> Don’t block the whole program while waiting for *slow operations*.

Slow operations include:
- Network requests
- Reading/writing files
- Database queries
- Timers
- User input

## Real-world analogy

### 1. Synchronous (blocking) way

You walk up to the counter, order your food, and then stand there, *wating*, doing nothing until your meal is ready.

- You can’t do anything else.
- Everyone behind you waits too.
- One task blocks progress.

### 2. Asynchronous way

You order your food, get a buzzer, and sit down.

While the kitchen prepares your meal:

- you chat with friends,
- check your phone,
- read a book,
- or do other tasks.

When the food is ready, the buzzer notifies you.

That’s asynchronous programming:

- Start a task that takes time (network request, file read, database query).
- Don’t wait idly.
- Continue doing other work.
- Handle the result later when it finishes.