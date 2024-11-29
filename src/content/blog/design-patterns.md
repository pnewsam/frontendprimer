---
title: Design Patterns
description: Design patterns are reusable solutions to common problems in software design.
pubDate: 2024-11-29
---

# Design Patterns

## Module

The **module pattern** makes use of a closure to encapsulate data that should remain private while exposing methods that allow the consumer to take certain prescribed actions.

```javascript
// Create a banking module that manages accounts with private account data and public methods for deposits/withdrawals.
function BankAccount(amount) {
  let balance = amount;

  const deposit = (amount) => {
    balance += amount;
    return balance;
  };

  const withdraw = (amount) => {
    if (amount > balance) {
      throw new Error("Insufficient funds.");
    } else {
      balance -= amount;
      return balance;
    }
  };

  const viewBalance = () => {
    return balance;
  };

  return {
    viewBalance,
    deposit,
    withdraw,
  };
}

const account = BankAccount(500);
account.deposit(500);
account.viewBalance(); // 1000
```

## Observer

The **observer** pattern enables a one-to-many relationship where multiple observers can subscribe/unsubscribe and react to changes in another object (subject). Some common requirements include:

- **Subscribers** can register and unregister their interest in **Topics**
- **Publishers** can broadcast **messages** without knowing about specific subscribers
- **Messages** are distributed to all relevant subscribers

```javascript
// Build a notification system where users can subscribe to topics (sports, news, tech) and receive updates when content is published

const listener1 = (data) => {
  console.log(data);
};

const listener2 = (data) => {
  console.log(data);
};

function NotificationSystem() {
  const listeners = new Map();

  const subscribe = (topic, callback) => {
    if (!listeners.has(topic)) {
      listeners.set(topic, new Set());
    }
    listeners.get(topic).add(callback);

    return () => unsubscribe(topic, callback);
  };

  const unsubscribe = (topic, callback) => {
    if (listeners.has(topic)) {
      listeners.get(topic).delete(callback);
    }
  };

  const notify = (topic, data) => {
    if (!listeners.has(topic)) {
      return;
    }
    listeners.get(topic).forEach((callback) => {
      try {
        callback(data); // Pass optional data
      } catch (error) {
        console.error("Callback error:", error);
      }
    });
  };

  return { subscribe, unsubscribe, notify };
}

const notifier = NotificationSystem();

notifier.subscribe("topic1", listener1);
notifier.subscribe("topic2", listener2);

notifier.notify("topic1", { message: "Hello" }); // Only listener1 will log
```

## Singleton

The **singleton** pattern ensures that only one instance of a given class exists. This is often used for example for managing API clients, or managing global configuration or application state.

```javascript
const createClient = (() => {
  let instance = null;
  return (config) => {
    instance = {
      baseURL: config.baseURL,
      fetch: async (url) => {
        // API logic here
      };
    }
    return instance;
  };
})();

const client = createClient({ baseUrl: "https://www.hey.com" });
```

```javascript
class APIClient {
  static instance = null;

  constructor() {
    // Real initialization
    this.baseURL = "https://api.example.com";
    this.headers = { "Content-Type": "application/json" };
  }

  static getInstance() {
    if (!APIClient.instance) {
      APIClient.instance = new APIClient();
    }
    return APIClient.instance;
  }

  // Actual functionality
  async fetch(endpoint) {
    return fetch(this.baseURL + endpoint, {
      headers: this.headers,
    });
  }
}
```

## Decorator

The **decorator** pattern allows behavior to be added dynamically without modifying the original object. It is often used for adding logging, or implementing access control.

```javascript
const Dog = () => {
  const bark = () => {
    console.log("bark");
  };

  return { bark };
};

const Cat = () => {
  const meow = () => {
    console.log("meow");
  };
  return { meow };
};

const withSpeech = (obj) => {
  const talk = () => {
    console.log("hello");
  };
  return { ...obj, hello };
};

const cat = Cat();
const dog = Dog();

const talkingCat = withSpeech(cat);
const talkingDog = withSpeech(dog);
```
