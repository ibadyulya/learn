# React — tasks

Answer in your own words, then check against [01-react.answers.md](./01-react.answers.md).

> 🌐 Russian version: [01-react.tasks.md](../ru/01-react.tasks.md)

---

## Question 1
What does "UI = f(state)" mean and how does React's declarative approach differ from
imperative DOM work?

## Question 2
What are the virtual DOM and reconciliation? Why do lists need a `key` and why is the
array index a bad key?

## Question 3
What does `useEffect` do? What are the dependency array and the cleanup function for?
Give an example of a leak without cleanup.

## Question 4
What's the difference between `useMemo`, `useCallback` and `useRef`? When do you use each?

## Question 5
Why is `setCount(c => c + 1)` often preferable to `setCount(count + 1)`? How does this
relate to closures?

## Question 6
How do you reduce excess re-renders? Why does `React.memo` need data immutability?

## Question 7
What is a "stale closure" in hooks and how do you avoid it?
