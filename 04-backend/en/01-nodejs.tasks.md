# Node.js — tasks

Answer in your own words, then check against [01-nodejs.answers.md](./01-nodejs.answers.md).

> 🌐 Russian version: [01-nodejs.tasks.md](../ru/01-nodejs.tasks.md)

---

## Question 1
What three main parts is Node made of? What is each responsible for?

## Question 2
Why is Node good for I/O load and bad for CPU-bound? What happens to the server while
a heavy synchronous computation runs?

## Question 3
Node is "single-threaded" — but libuv has a thread pool. How do these coexist? What
exactly is single-threaded?

## Question 4
What is `EventEmitter`? What's special about the `'error'` event?

## Question 5
Why do you need streams if you can read a file whole? What is backpressure?

## Question 6
List the ways to handle errors in Node for different models (sync, promise,
error-first callback, stream). What is `process.on('uncaughtException')` good for, and
what is it not?

## Question 7
You need to hash passwords for 10,000 users in one process (heavy CPU). How do you
avoid blocking the handling of HTTP requests?
