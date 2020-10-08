# immer

```js
import produce, { original, current, isDraft } from "immer";

const baseState = [
  {
    todo: "Learn typescript",
    done: true,
  },
  {
    todo: "Try immer",
    done: false,
  },
];

const nextState = produce(baseState, (draftState) => {
  draftState.push({ todo: "Tweet about it" });
  draftState[1].done = true;
  const orig = original(draft);
  const copy = current(draft);
  // if a value is a proxied instance
  isDraft(draftState); // true
});
```

一个基于 [copy-on-write](https://en.wikipedia.org/wiki/Copy-on-write) 机制小而美的 immutable JavaScript library。

The basic idea is that you will apply all your changes to a temporary draftState, which is a proxy of the currentState. Once all your mutations are completed, Immer will produce the nextState based on the mutations to the draft state. This means that you can interact with your data by simply modifying it while keeping all the benefits of immutable data.

![immer](./assets/immer.png)

特性：

- Immutability with normal JavaScript objects, arrays, Sets and Maps. No new APIs to learn!
- Strongly typed, no string based paths selectors etc.
- Structural sharing out of the box
- Object freezing out of the box
- Deep updates are a breeze
- Boilerplate reduction. Less noise, more concise code.
- First class support for patches
- Small: 3KB gzipped

immer 内部新版浏览器基于 Proxy，比如 IE 不支持 proxy 的将降级使用 ES5 的实现。

`create()`方法：

Immer exposes a named export current that create a copy of the current state of the draft. This can be very useful for debugging purposes

`applyPatches()`方法：
