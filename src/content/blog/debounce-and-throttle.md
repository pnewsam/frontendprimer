---
title: Debounce and Throttle
description: Sometimes you want to trigger an action based on a user event, but need ensure the action isn't triggered too often or too many times. This is usually implemented either as a debounce or a throttle.
pubDate: 2024-11-29
---

# Debounce and Throttle

Sometimes you want to trigger an action based on a user event, but need ensure the action isn't triggered too often or too many times. This is usually implemented either as a **debounce** or a **throttle**.

## Debounce

A debounce is used for **"wait until done"** scenarios. For example, you might have a **typeahead** component that automatically populates a list of suggestions based on the user's input. You don't want to update the results on _every keypress_. Instead, you want to wait until the user is finished typing.

In order to do this, you implement a **debounce**.

```js
const debounce = (fn, delay) => {
  let timoutId;

  return (...args) => {
    clearTimeout(timeoutId);
    setTimeout(() => {
      fn(...args);
    }, delay);
  };
};

const handleInput = throttle(updateSuggestions, 1000);
```

Every invocation is delayed by the specified interval, which is reset at each invocation. As a result, only the most recent invocation executes.

**Use Cases**

- Search input fields
- Form validation
- Window resize handlers
- Saving drafts
- API calls on input change

## Throttle

A **throttle** is very similar, but used instead for **"regular interval"** scenarios. For example: infinite scrolling. You have a list of items, and you want it to expand automatically whenever the user reaches the bottom. But you don't want to fetch data on _every_ scroll event, because then you would end up with a lot of fetches. So you add a throttle.

```js
const throttle = (fn, limit) => {
  let isThrottled = false;

  return (...args) => {
    if (!isThrottled) {
      isThrottled = true;
      fn(...args);
      setTimeout(() => {
        isThrottled = false;
      }, limit);
    }
  };
};

const handleScroll = throttle(handleResize, 1000);
```

The passed action happens immediately, but doesn't happen again unless the specified interval has elapsed.

**Use Cases**

- Scroll event handlers
- Game loop updates
- Mouse move tracking
- Real-time analytics
- Continuous button clicks
