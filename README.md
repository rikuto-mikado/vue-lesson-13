# Vue Lesson 13

## What I Learned

### JavaScript Proxy - The Foundation of Vue's Reactivity

A **Proxy** is a JavaScript object that wraps another object and intercepts operations on it (like reading or writing properties).

```javascript
const proxy = new Proxy(target, handler);
```

| Parameter | Description |
|-----------|-------------|
| `target`  | The original object to wrap |
| `handler` | An object containing "trap" methods that intercept operations |

### How Vue Uses Proxy

Vue 3 uses Proxy internally to make data reactive. When you change `this.message`, Vue automatically updates the DOM.

```javascript
const data = { message: 'Hello', longMessage: 'Hello World!' };

const handler = {
  set(target, key, value) {
    if (key === 'message') {
      target.longMessage = value + ' World!';
    }
    target.message = value;
  }
};

const proxy = new Proxy(data, handler);
proxy.message = 'Hello!!!!';  // Triggers the set trap
// Now: longMessage = 'Hello!!!! World!'
```

## What Was Difficult

### 1. Understanding What Proxy Actually Is

**Confusion**: "Is Proxy a function? A class? What does it do?"

**Answer**: Proxy is a built-in JavaScript constructor that creates a "middleman" object. It doesn't change the original object; it wraps it.

```
[Your Code] → [Proxy] → [Original Object]
                ↑
         Intercepts operations here
```

### 2. The `set(target, key, value)` Trap

**Confusion**: "What are these parameters? Why this order?"

| Parameter | What It Receives | Example |
|-----------|-----------------|---------|
| `target`  | The original object being proxied | `{ message: 'Hello', longMessage: 'Hello World!' }` |
| `key`     | The property name being set | `'message'` |
| `value`   | The new value being assigned | `'Hello!!!!'` |

**The order is fixed by JavaScript specification** - you cannot change it.

```javascript
set(target, key, value) {
  // target = the original data object
  // key = which property? (e.g., 'message')
  // value = what's the new value? (e.g., 'Hello!!!!')

  console.log(`Setting ${key} to ${value}`);
  target[key] = value;  // Actually set the value
  return true;          // Indicate success
}
```

### 3. Why Use `target[key]` Instead of `target.key`?

```javascript
target[key] = value;   // Correct - key is a variable
target.key = value;    // Wrong - looks for property literally named "key"
```

### 4. How Does `set(target, key, value)` Connect to `new Proxy(data, handler)`?

**Confusion**: "Are these parameters directly mapped to Proxy arguments?"

**Answer**: Only `target` is directly connected. `key` and `value` are determined at operation time.

```javascript
const proxy = new Proxy(data, handler);
//                       ↑      ↑
//                    target  handler (contains set)

proxy.message = 'Hello!!!!';
//     ↑           ↑
//    key        value
```

| Proxy() Argument | set() Parameter | Relationship |
|-----------------|-----------------|--------------|
| `data` | `target` | **Same object** - Proxy passes it automatically |
| `handler` | - | The object containing set itself |
| - | `key` | **Determined at operation time** - property name |
| - | `value` | **Determined at operation time** - assigned value |

**Flow:**
```javascript
// 1. Create Proxy
const proxy = new Proxy(data, handler);

// 2. Set a property
proxy.message = 'Hello!!!!';

// 3. JavaScript automatically calls handler.set()
handler.set(data, 'message', 'Hello!!!!');
//          ↑        ↑           ↑
//       target     key        value
//   (from Proxy) (from op)  (from op)
```

### 5. Common Handler Traps

| Trap | Triggered When |
|------|----------------|
| `get(target, key)` | Reading a property: `proxy.message` |
| `set(target, key, value)` | Writing a property: `proxy.message = 'Hi'` |
| `deleteProperty(target, key)` | Deleting: `delete proxy.message` |
