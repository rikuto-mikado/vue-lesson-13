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

### 6. Are `target`, `key`, `value` Fixed Names?

**No. They are just variable names.** You can name them anything:

```javascript
// All of these work exactly the same:
set(target, key, value) { target[key] = value; }
set(obj, prop, val) { obj[prop] = val; }
set(a, b, c) { a[b] = c; }
set(banana, apple, orange) { banana[apple] = orange; }
```

**What matters is the ORDER, not the name:**

| Position | Always Receives | Common Names |
|----------|----------------|--------------|
| 1st | Original object | `target`, `obj` |
| 2nd | Property name | `key`, `prop`, `property` |
| 3rd | New value | `value`, `val`, `newValue` |

### 7. Can I Omit Parameters?

**Yes, you can omit parameters you don't need** (from right to left):

```javascript
// Use all 3
set(target, key, value) {
  console.log(`Setting ${key} to ${value}`);
  target[key] = value;
}

// Omit value (rarely useful)
set(target, key) {
  console.log(`Someone tried to set ${key}`);
  // Can't actually set anything without value
}

// Just target (almost never useful)
set(target) {
  console.log('Something was set');
}
```

**But you CANNOT skip middle parameters:**

```javascript
// WRONG - can't skip key to get value
set(target, value) { ... }  // value will actually be the key!

// If you need value but not key, you must still declare key:
set(target, key, value) {
  // Just don't use key
  console.log(value);
}
```

**There's also a 4th parameter (receiver):**

```javascript
set(target, key, value, receiver) {
  // receiver = the proxy itself (or object inheriting from it)
  // Rarely needed, safe to omit
}
```

**Practical rule:** Always include all parameters you need, in order. Omit only from the end.

---

## 補足

**Proxyは「割り込み」の仕組み。** 通常のオブジェクト操作を横取りして、自分の処理を優先的に実行できる。Vueはこれを使ってデータ変更を検知し、自動でDOMを更新している。
