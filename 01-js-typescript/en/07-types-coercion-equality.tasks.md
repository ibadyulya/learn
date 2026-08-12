# Types, coercion and equality — tasks

Answer in your own words, then check against [07-types-coercion-equality.answers.md](./07-types-coercion-equality.answers.md).

> 🌐 Russian version: [07-types-coercion-equality.tasks.md](../ru/07-types-coercion-equality.tasks.md)

---

## Question 1
Name the 7 primitives. What's the fundamental difference between copying a primitive
and an object?

## Question 2
Why is `{a:1} === {a:1}` `false`? How then do you compare two objects by content?

## Question 3
What's the difference between `==` and `===`? Give two examples where `==` gives an
unexpected result due to coercion.

## Question 4
List all falsy values. Which of these are truthy: `'0'`, `''`, `[]`, `{}`, `0`,
`'false'`?

## Question 5
Explain the result of each:
```ts
1 + '2'      // ?
'5' - 2      // ?
1 + 2 + '3'  // ?
'3' + 2 + 1  // ?
```

## Question 6
Why is `NaN === NaN` `false`? How do you correctly check that a value is `NaN`?

## Question 7
The difference between `null` and `undefined`: when does each appear? What do
`null == undefined` and `null === undefined` return?

## Question 8
How does `Object.is` differ from `===`? Name two cases where they diverge.
