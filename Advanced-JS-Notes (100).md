# Advanced JavaScript (Videos 36–51)

1. Prototype Chain

The prototype chain is JavaScript's built-in inheritance mechanism. Every object has an internal link (`[[Prototype]]`) pointing to another object. When you access a property, JavaScript first looks at the object itself, then travels up the chain until it finds it or reaches `null`.

```js
// Every object links up to null through its prototype chain
// arr → Array.prototype → Object.prototype → null

const arr = [1, 2, 3];
console.log(arr.__proto__ === Array.prototype); // true
console.log(Array.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__); // null
```

```js
// Constructor function + prototype method sharing
function User(username, score) {
  this.username = username;
  this.score = score;
}

// Method lives on prototype — shared by ALL instances (memory efficient)
User.prototype.increment = function () {
  this.score++;
};

User.prototype.printInfo = function () {
  console.log(`${this.username} has score: ${this.score}`);
};

const user1 = new User("maheen", 5);
const user2 = new User("ali", 10);

user1.increment();
user1.printInfo(); // maheen has score: 6
user2.printInfo(); // ali has score: 10
// user2's score is unaffected — each instance has its own data
```

```js
// Object.setPrototypeOf — modern way to set prototype linkage
const teacher = {
  makeAssignment() {
    return "Assignment given";
  },
};

const teachingAssistant = {
  gradeAssignment() {
    return "Assignment graded";
  },
};

Object.setPrototypeOf(teachingAssistant, teacher);
// teachingAssistant now inherits from teacher
console.log(teachingAssistant.makeAssignment()); // "Assignment given"
```

**Key insight:** Classes in JavaScript are syntactic sugar — underneath they use prototypes.

---

 2. ES6+ Syntax

# Destructuring

```js
// Array destructuring
const [first, second, ...rest] = [10, 20, 30, 40, 50];
console.log(first);  // 10
console.log(rest);   // [30, 40, 50]

// Object destructuring
const user = { username: "maheen", email: "maheen@gmail.com", score: 99 };
const { username, email } = user;
console.log(username); // "maheen"

// Rename while destructuring
const { username: name } = user;
console.log(name); // "maheen"

// Default values
const { country = "Pakistan" } = user;
console.log(country); // "Pakistan"

// Destructuring in function parameters
function displayUser({ username, score }) {
  console.log(`${username} scored ${score}`);
}
displayUser(user); // "maheen scored 99"
```
# Spread & Rest Operators

```js
// Spread — expanding iterables
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2];
console.log(combined); // [1, 2, 3, 4, 5, 6]

// Spread with objects
const defaults = { theme: "dark", lang: "en" };
const userPrefs = { ...defaults, lang: "ur" }; // override lang
console.log(userPrefs); // { theme: "dark", lang: "ur" }

// Rest — collecting remaining arguments
function sum(...numbers) {
  return numbers.reduce((acc, num) => acc + num, 0);
}
console.log(sum(1, 2, 3, 4)); // 10
```

# Optional Chaining

```js
// Without optional chaining — can crash
// console.log(user.address.city); // TypeError if address is undefined

// With optional chaining — safe access
const user = { username: "maheen", address: null };
console.log(user?.address?.city);     // undefined (no crash)
console.log(user?.phone?.number);     // undefined (no crash)

// With methods
const arr = null;
console.log(arr?.map(x => x * 2));   // undefined (no crash)

// Practical use with API responses
const apiResponse = {
  data: {
    users: [{ name: "maheen" }]
  }
};
console.log(apiResponse?.data?.users?.[0]?.name); // "maheen"
console.log(apiResponse?.data?.posts?.[0]?.title); // undefined
```

---

 3. Iterators & Generators

# Iterators

An iterator is any object with a `next()` method that returns `{ value, done }`.

```js
// Manual iterator
function createRangeIterator(start, end) {
  let current = start;
  return {
    next() {
      if (current <= end) {
        return { value: current++, done: false };
      }
      return { value: undefined, done: true };
    }
  };
}

const range = createRangeIterator(1, 3);
console.log(range.next()); // { value: 1, done: false }
console.log(range.next()); // { value: 2, done: false }
console.log(range.next()); // { value: 3, done: false }
console.log(range.next()); // { value: undefined, done: true }
```

```js
// Making an object iterable with Symbol.iterator
const playlist = {
  songs: ["Song A", "Song B", "Song C"],
  [Symbol.iterator]() {
    let index = 0;
    return {
      next: () => {
        if (index < this.songs.length) {
          return { value: this.songs[index++], done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
};

for (const song of playlist) {
  console.log(song); // "Song A", "Song B", "Song C"
}
```

# Generators

Generators are functions that can pause and resume execution using `yield`.

```js
// Basic generator
function* simpleGenerator() {
  console.log("Start");
  yield 1;
  console.log("After first yield");
  yield 2;
  console.log("After second yield");
  yield 3;
  console.log("Done");
}

const gen = simpleGenerator();
console.log(gen.next()); // "Start" → { value: 1, done: false }
console.log(gen.next()); // "After first yield" → { value: 2, done: false }
console.log(gen.next()); // "After second yield" → { value: 3, done: false }
console.log(gen.next()); // "Done" → { value: undefined, done: true }
```

```js
// Infinite sequence generator (practical use case)
function* idGenerator() {
  let id = 1;
  while (true) {
    yield id++;
  }
}

const getId = idGenerator();
console.log(getId.next().value); // 1
console.log(getId.next().value); // 2
console.log(getId.next().value); // 3
// never runs out — generates on demand
```

```js
// Generator with for...of
function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

const fib = fibonacci();
for (let i = 0; i < 7; i++) {
  process.stdout.write(fib.next().value + " ");
}
// 0 1 1 2 3 5 8
```

---

 4. ES Modules vs CommonJS

# CommonJS (Node.js traditional — `.js` files)

```js
// math.js — CommonJS export
const add = (a, b) => a + b;
const multiply = (a, b) => a * b;

module.exports = { add, multiply };
// OR export one thing:
// module.exports = add;
```

```js
// app.js — CommonJS import
const { add, multiply } = require("./math");
// Loaded synchronously — file is read and executed immediately
console.log(add(2, 3));      // 5
console.log(multiply(2, 3)); // 6
```

# ES Modules 

```js
// math.mjs — ES Module export
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;

// Default export
export default function greet(name) {
  return `Hello, ${name}!`;
}
```

```js
// app.mjs — ES Module import
import greet, { add, multiply } from "./math.mjs";
// Loaded asynchronously — resolved before execution

console.log(add(2, 3));        // 5
console.log(multiply(2, 3));   // 6
console.log(greet("maheen"));      // "Hello, maheen!"

// Dynamic import (lazy loading)
async function loadModule() {
  const { add } = await import("./math.mjs");
  console.log(add(10, 20));
}
loadModule();
```

# Key Differences

| Feature           |          CommonJS              | ES Modules                  |
|                   |                                |                             |
| Syntax            | `require()` / `module.exports` | `import` / `export`         |
| Loading           |  Synchronous                   | Asynchronous                |
| When parsed       |  At runtime                    | At parse time (static)      |
| Tree shaking      |  Not supported                 | Supported                   |
| Top-level `await` |  Not supported                 |  Supported                  |
| File extension    | `.js`                          | `.mjs` or `"type":"module"` |
| Used in           | Node.js (legacy)               | Browsers + Modern Node.js   |

---

# 5. Bonus: Classes (ES6 Syntactic Sugar over Prototypes)

```js
class User {
  constructor(username, email) {
    this.username = username;
    this.email = email;
  }

  // Instance method
  getInfo() {
    return `${this.username} — ${this.email}`;
  }

  // Static method (called on class, not instance)
  static createGuestUser() {
    return new User("guest", "guest@example.com");
  }
}

class Teacher extends User {
  constructor(username, email, subject) {
    super(username, email); // calls User constructor
    this.subject = subject;
  }

  addCourse(course) {
    return `${this.username} added: ${course}`;
  }
}

const t = new Teacher("raza", " raza@chai.com", "JavaScript");
console.log(t.getInfo());          // raza — raza@chai.com
console.log(t.addCourse("React")); // raza added: React
console.log(t instanceof User);    // true
```

---

# 6. Bonus: Closures & Lexical Scope

```js
// Closure = inner function retains access to outer function's variables
// even after outer function has finished executing

function makeCounter() {
  let count = 0; // lives in closure memory

  return {
    increment() { count++; },
    decrement() { count--; },
    getCount() { return count; }
  };
}

const counter = makeCounter();
counter.increment();
counter.increment();
counter.increment();
counter.decrement();
console.log(counter.getCount()); // 2
// count is private — can't access it directly from outside
```

---

# 7. Bonus: Promises & Async/Await

```js
// Creating a promise
const fetchUser = (id) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) {
        resolve({ id, username: "raza", email: "raza@chai.com" });
      } else {
        reject(new Error("Invalid user ID"));
      }
    }, 1000);
  });
};

// Consuming with async/await
async function getUser(id) {
  try {
    const user = await fetchUser(id);
    console.log(user);
  } catch (error) {
    console.error("Error:", error.message);
  }
}

getUser(1);  // { id: 1, username: 'raza', email: 'raza@chai.com' }
getUser(-1); // Error: Invalid user ID




