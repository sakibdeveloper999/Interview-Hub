# Node.js Backend Interview – ১৬০ প্রশ্নের উত্তর (বাংলায়)

*সহজ ও কথ্য বাংলায় লেখা। যেখানে ইংরেজি টার্ম বললে ইন্টারভিউতে সুবিধা, সেখানে টার্মটা ইংরেজিতেই রাখা হয়েছে।*

## ১. JavaScript Fundamentals for Backend (Q1–20)

### Q1. var, let এবং const এর পার্থক্য কী?

- **var**: function-scoped। Hoisted হয় এবং শুরুতে `undefined` থাকে। একই নাম আবার declare করা যায়।
- **let**: block-scoped। Reassign করা যায়, কিন্তু একই scope-এ redeclare করা যায় না।
- **const**: block-scoped। Declare করার সময়ই value দিতে হয়, reassign করা যায় না। তবে const object/array-এর ভেতরের মান বদলানো যায়।

**Best practice:** সবসময় `const` ব্যবহার করুন, বদলাতে হলে `let`, আর `var` এড়িয়ে চলুন।

### Q2. == এবং === এর পার্থক্য কী?

`==` (loose equality) দুই পাশের type আলাদা হলে আগে type coercion করে তারপর তুলনা করে। `===` (strict equality) type আর value দুটোই মেলায়, কোনো coercion করে না।

```js
'5' == 5   // true
'5' === 5  // false
0 == false // true
0 === false // false
```

তাই সবসময় `===` ব্যবহার করা নিরাপদ।

### Q3. Primitive এবং Reference data type কী?

- **Primitive:** string, number, boolean, null, undefined, symbol, bigint। এগুলো immutable এবং copy করলে value-টাই copy হয়।
- **Reference:** object, array, function ইত্যাদি। Variable-এ আসল data থাকে না, memory address (reference) থাকে। তাই একটা variable থেকে আরেকটায় assign করলে দুটোই একই object-কে point করে।

```js
let a = { x: 1 };
let b = a;
b.x = 99;
console.log(a.x); // 99
```

### Q4. null এবং undefined এর পার্থক্য কী?

- **undefined**: variable declare হয়েছে কিন্তু value দেওয়া হয়নি। এটা JavaScript নিজে বসায়।
- **null**: ডেভেলপার ইচ্ছা করে বোঝায় যে এখানে কোনো মান নেই।

`typeof undefined` হলো `'undefined'`, আর `typeof null` হলো `'object'` (এটা JavaScript-এর পুরনো একটা bug)। `null == undefined` true, কিন্তু `null === undefined` false।

### Q5. Hoisting কী এবং কীভাবে কাজ করে?

Code চলার আগে JavaScript engine একটা memory creation phase চালায়। এই সময় declaration গুলো scope-এর উপরে তুলে নেওয়া হয়, একেই hoisting বলে।

- Function declaration পুরোপুরি hoisted হয়, তাই declare করার আগেই call করা যায়।
- `var` hoisted হয় কিন্তু value থাকে `undefined`।
- `let` ও `const` hoisted হয়, কিন্তু initialize হয় না (দেখুন Q6)।

### Q6. Temporal Dead Zone (TDZ) কী?

`let`/`const` declare করার লাইনের আগে থেকে scope-এর শুরু পর্যন্ত যে সময়টা, সেটাই TDZ। এই সময়ে variable-টা access করলে `ReferenceError` আসে।

```js
console.log(a); // ReferenceError
let a = 10;
```

এটা আসলে ভালো, কারণ ভুল করে initialize-এর আগে variable ব্যবহার করলে bug সাথে সাথে ধরা পড়ে।

### Q7. Scope কী? Global, function ও block scope বুঝিয়ে বলুন।

Scope মানে কোনো variable কোথা থেকে access করা যাবে তার সীমানা।

- **Global scope:** সবখান থেকে access করা যায়।
- **Function scope:** function-এর ভেতরে declare করা variable শুধু ওই function-এর ভেতরেই পাওয়া যায় (`var`, `let`, `const` সবই)।
- **Block scope:** `{ }` এর ভেতরে `let`/`const` শুধু ওই block-এই থাকে (যেমন if, for)। `var` block scope মানে না।

JavaScript-এ lexical scoping আছে, মানে ভেতরের scope বাইরের scope-এর variable দেখতে পায়, উল্টোটা নয়।

### Q8. Closure কী? Backend code-এ এটা কোথায় কাজে লাগে?

Closure হলো এমন একটা function যে তার বাইরের scope-এর variable মনে রাখে, এমনকি বাইরের function শেষ হয়ে গেলেও।

```js
function rateLimiter(max) {
  let count = 0;            // private state
  return function () {
    count++;
    return count <= max;
  };
}
const check = rateLimiter(3);
```

**Backend-এ ব্যবহার:** middleware factory (যেমন `authorize('admin')`), rate limiter/counter, memoization বা cache, private state রাখা, config নিয়ে DB helper বানানো।

### Q9. Function declaration এবং arrow function এর পার্থক্য কী?

- Function declaration পুরোপুরি hoisted হয়; arrow function (variable-এ assign করা) হয় না।
- Arrow function-এর নিজের `this` নেই, সে বাইরের scope থেকে `this` নেয় (lexical this)।
- Arrow function-এ নিজের `arguments` object নেই।
- Arrow function-কে `new` দিয়ে constructor হিসেবে ব্যবহার করা যায় না।

Object-এর method বা constructor-এ সাধারণ function, আর callback-এ arrow function ব্যবহার করা ভালো।

### Q10. call(), apply() এবং bind() এর পার্থক্য কী?

তিনটাই function-এর `this` নিজে ঠিক করে দেওয়ার জন্য।

- **call(thisArg, a, b)**: সাথে সাথে function চালায়, argument আলাদা আলাদা দেয়।
- **apply(thisArg, \[a, b\])**: সাথে সাথে চালায়, argument array হিসেবে দেয়।
- **bind(thisArg, a)**: function চালায় না, নতুন একটা function ফেরত দেয় যার `this` fix করা আছে। পরে call করা যায়।

### Q11. this কী এবং এর value কীভাবে ঠিক হয়?

`this` নির্ভর করে function কীভাবে call করা হয়েছে তার উপর, কোথায় লেখা হয়েছে তার উপর নয় (arrow function ছাড়া)।

- Object-এর method হিসেবে call করলে `this` হলো ওই object।
- সাধারণ function call-এ `this` হয় `undefined` (strict mode) বা global object।
- `new` দিয়ে call করলে `this` হলো নতুন তৈরি object।
- `call/apply/bind` দিয়ে manually set করা যায়।
- Arrow function-এ `this` আসে বাইরের scope থেকে।

### Q12. Callback, Promise এবং async/await কী?

- **Callback:** একটা function-কে আরেকটা function-এ argument হিসেবে পাঠানো, কাজ শেষ হলে সেটা call হয়। বেশি nested হলে callback hell তৈরি হয়।
- **Promise:** ভবিষ্যতের কোনো result-এর প্রতিনিধি। তিনটা অবস্থা: pending, fulfilled, rejected। `.then()`, `.catch()` দিয়ে chain করা যায়।
- **async/await:** Promise-এর উপর তৈরি সহজ syntax। Code দেখতে synchronous-এর মতো কিন্তু asynchronous-ই চলে।

```js
async function getUser(id) {
  const user = await User.findById(id);
  return user;
}
```

### Q13. async/await এ error handling কীভাবে হয়?

`try/catch` ব্লক ব্যবহার করে। `await` করা Promise reject হলে সেটা catch-এ চলে আসে।

```js
async function run() {
  try {
    const data = await fetchData();
  } catch (err) {
    console.error(err.message);
  } finally {
    // cleanup
  }
}
```

Express-এ প্রতিটা route-এ try/catch না লিখে একটা `asyncHandler` wrapper বা `.catch(next)` ব্যবহার করা হয় (দেখুন Q50)।

### Q14. Event loop কী? Call stack ও task queue বুঝিয়ে বলুন।

JavaScript single-threaded, কিন্তু event loop এর কারণে non-blocking কাজ করতে পারে।

- **Call stack:** যে function এখন চলছে সেগুলো এখানে থাকে (LIFO)।
- **Async কাজ** (timer, I/O, network) background-এ চলে (browser-এ Web APIs, Node.js-এ libuv)।
- কাজ শেষ হলে callback একটা queue-তে যায়: **microtask queue** (Promise `.then`, `queueMicrotask`) অথবা **macrotask queue** (setTimeout, I/O)।
- Event loop দেখে call stack ফাঁকা কি না। ফাঁকা হলে আগে সব microtask চালায়, তারপর একটা macrotask নেয়।

```js
console.log(1);
setTimeout(() => console.log(2), 0);
Promise.resolve().then(() => console.log(3));
console.log(4);
// output: 1 4 3 2
```

### Q15. Promise.all(), Promise.allSettled() এবং Promise.race() এর পার্থক্য কী?

- **Promise.all(\[...\])**: সব Promise সফল হলে সব result-এর array দেয়। যেকোনো একটা fail করলে সাথে সাথে পুরোটা reject হয়।
- **Promise.allSettled(\[...\])**: সবগুলো শেষ হওয়া পর্যন্ত অপেক্ষা করে, প্রতিটার status (fulfilled/rejected) সহ result দেয়। কখনো reject হয় না।
- **Promise.race(\[...\])**: যেটা সবার আগে শেষ হয় (সফল বা ব্যর্থ), তার ফলাফলই দেয়। Timeout বানাতে কাজে লাগে।

### Q16. Shallow copy এবং deep copy এর পার্থক্য কী?

- **Shallow copy:** শুধু উপরের স্তর copy হয়। ভেতরের nested object গুলো আগের object-এর সাথেই shared থাকে। উদাহরণ: spread `{...obj}`, `Object.assign()`।
- **Deep copy:** nested সব কিছুসহ নতুন করে copy হয়। আধুনিক উপায়: `structuredClone(obj)`। `JSON.parse(JSON.stringify(obj))` ও চলে, কিন্তু এতে Date, function, undefined নষ্ট হয়ে যায়।

### Q17. map(), filter() এবং reduce() কী এবং কখন ব্যবহার করবেন?

- **map()**: প্রতিটা element বদলে নতুন array বানায়। (যেমন user list থেকে শুধু name বের করা)
- **filter()**: শর্ত মেলে এমন element নিয়ে নতুন array বানায়।
- **reduce()**: পুরো array-কে একটা single value-তে নামিয়ে আনে (sum, group, object ইত্যাদি)।

```js
const total = orders
  .filter(o => o.status === 'paid')
  .map(o => o.amount)
  .reduce((sum, a) => sum + a, 0);
```

### Q18. Destructuring কী? Spread ও rest operator এর পার্থক্য কী?

**Destructuring:** array বা object থেকে value সরাসরি variable-এ বের করে আনা।

```js
const { name, email } = user;
const [first, second] = list;
```

Spread ও rest দেখতে একই (`...`) কিন্তু কাজ উল্টো:

- **Spread:** জিনিস ছড়িয়ে দেয়। `const b = [...a, 4]` বা `{...obj, age: 20}`।
- **Rest:** বাকিগুলো জড়ো করে একটা array/object বানায়। `function f(first, ...others)` বা `const { a, ...rest } = obj`।

### Q19. JavaScript module কী এবং import/export কেন ব্যবহার করি?

Module মানে code-কে আলাদা আলাদা ফাইলে ভাগ করা, যেখানে প্রতিটা ফাইলের নিজস্ব scope থাকে। `export` দিয়ে যা বাইরে দিতে চাই তা দেওয়া হয়, `import` দিয়ে অন্য ফাইলে নেওয়া হয়।

**কেন দরকার:** code সুসংগঠিত থাকে, পুনরায় ব্যবহার করা যায়, global scope দূষিত হয় না, আর maintain করা সহজ হয়।

### Q20. Synchronous এবং asynchronous execution এর পার্থক্য কী?

- **Synchronous:** কাজ একটার পর একটা চলে। একটা কাজ শেষ না হওয়া পর্যন্ত পরেরটা শুরু হয় না (blocking)।
- **Asynchronous:** দীর্ঘ কাজ (DB query, file read, API call) শুরু করে দিয়ে অপেক্ষা না করে পরের কাজে যাওয়া যায়। কাজ শেষ হলে callback/Promise-এর মাধ্যমে জানানো হয় (non-blocking)।

Node.js-এ server দ্রুত রাখতে I/O কাজ সবসময় asynchronous করা উচিত।

## ২. Node.js (Q21–40)

### Q21. Node.js কী, এবং browser-এর JavaScript থেকে এটা কীভাবে আলাদা?

Node.js হলো Chrome-এর V8 engine-এর উপর তৈরি একটা JavaScript runtime, যা browser-এর বাইরে (server বা কম্পিউটারে) JavaScript চালাতে দেয়।

**পার্থক্য:**

- Browser-এ `window`, `document`, DOM আছে; Node.js-এ এগুলো নেই। Node-এ আছে `fs`, `http`, `path`, `os`, `process` ইত্যাদি module, যা file system, network আর OS-এর সাথে কাজ করে।
- Global object: browser-এ `window`, Node-এ `global` (দুই জায়গাতেই `globalThis` কাজ করে)।
- Browser-এ মূলত ES Modules; Node-এ CommonJS ও ES Modules দুটোই চলে।

### Q22. I/O-heavy application-এর জন্য Node.js কেন উপযুক্ত?

কারণ Node.js **non-blocking, event-driven**। DB query, file read বা API call-এর জন্য thread বসে অপেক্ষা করে না; কাজ শুরু করে দিয়ে অন্য request-এ চলে যায়, কাজ শেষ হলে callback চলে। ফলে একটা ছোট server-ও হাজারো connection সামলাতে পারে। REST API, chat app, streaming, real-time app-এর জন্য এটা দারুণ। উল্টো দিকে, ভারী CPU কাজের (video encoding, বড় calculation) জন্য এটা আদর্শ নয়।

### Q23. Node.js কি single-threaded? পরিষ্কার করে বুঝিয়ে বলুন।

উত্তর: **আংশিক হ্যাঁ।** আপনার JavaScript code চলে একটা main thread-এ (একটা event loop)। কিন্তু Node.js-এর ভেতরে multi-threading আছে:

- **libuv thread pool** (ডিফল্ট ৪টা thread): file system, `crypto`, `zlib`, DNS lookup-এর মতো কাজ এখানে চলে।
- Network I/O চলে OS-এর নিজস্ব async mechanism দিয়ে (epoll, kqueue, IOCP)।
- চাইলে `worker_threads` বা `cluster` ব্যবহার করে আরও thread/process নেওয়া যায়।

ইন্টারভিউতে বলার সহজ লাইন: *"JS execution single-threaded, কিন্তু Node.js পুরোপুরি single-threaded না।"*

### Q24. Node.js-এর event loop কীভাবে কাজ করে?

Event loop হলো libuv-এর একটা অংশ। আপনার main code চলা শেষ হলে event loop শুরু হয় এবং বারবার বিভিন্ন queue ঘুরে দেখে কোনো callback ready কি না। Ready থাকলে সেটা call stack-এ এনে চালায়। কোনো pending কাজ না থাকলে process বন্ধ হয়ে যায়। এভাবেই single thread দিয়েই অনেক async কাজ সামলানো যায়।

### Q25. Event loop-এর প্রধান phase গুলো কী কী?

1. **Timers:** `setTimeout`, `setInterval`-এর callback
2. **Pending callbacks:** কিছু system-level callback
3. **Idle/Prepare:** internal ব্যবহারের জন্য
4. **Poll:** নতুন I/O event আনা ও I/O callback চালানো (সবচেয়ে গুরুত্বপূর্ণ phase)
5. **Check:** `setImmediate`-এর callback
6. **Close callbacks:** যেমন `socket.on('close')`

প্রতিটা phase-এর মাঝখানে **microtask** (`process.nextTick` ও Promise callback) চলে।

### Q26. process.nextTick(), setImmediate() এবং setTimeout() এর পার্থক্য কী?

- **process.nextTick(fn):** current operation শেষ হওয়া মাত্র, event loop এগোনোর আগেই চলে। এটা Promise-এর আগেও চলে। বেশি ব্যবহার করলে I/O আটকে (starve) যেতে পারে।
- **setTimeout(fn, 0):** timers phase-এ চলে, অন্তত \~১ms পরে।
- **setImmediate(fn):** check phase-এ, অর্থাৎ poll phase-এর ঠিক পরে চলে।

```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('promise'));
// nextTick → promise → (timeout / immediate)
```

Main module-এ `setTimeout` আর `setImmediate`-এর order নিশ্চিত না, তবে কোনো I/O callback-এর ভেতরে `setImmediate` সবসময় আগে চলে।

### Q27. CommonJS এবং ES Modules এর পার্থক্য কী?

| বিষয় | CommonJS | ES Modules |
| --- | --- | --- |
| Syntax | `require()`, `module.exports` | `import`, `export` |
| Loading | Synchronous | Asynchronous, static analysis সম্ভব |
| Top-level await | নেই | আছে |
| Enable | ডিফল্ট | `package.json`-এ `"type": "module"` বা `.mjs` ফাইল |
| `__dirname` | আছে | নেই (`import.meta.url` ব্যবহার করতে হয়) |

নতুন প্রজেক্টে ESM ব্যবহার করাই এখন ভালো প্র্যাকটিস, তবে অনেক পুরনো project ও package এখনও CommonJS।

### Q28. package.json এবং package-lock.json এর ভূমিকা কী?

- **package.json:** প্রজেক্টের পরিচয় ও কনফিগারেশন: নাম, version, scripts, dependencies, `type`, `engines` ইত্যাদি। Dependency-র version range (যেমন `^4.18.0`) এখানে থাকে।
- **package-lock.json:** `npm install`-এর পর প্রতিটা package (এবং তার sub-dependency)-র **ঠিক কোন version** install হয়েছে তা লক করে রাখে। ফলে সব মেশিনে ও production-এ একই version install হয়, কোনো সারপ্রাইজ থাকে না। এটা git-এ commit করা উচিত।

### Q29. npm কী? dependencies এবং devDependencies এর পার্থক্য কী?

npm হলো Node Package Manager: package install, update ও script চালানোর টুল, সাথে বিশাল একটা package registry।

- **dependencies:** app production-এ চলার জন্য যা লাগে (express, mongoose, jsonwebtoken)। `npm install express`।
- **devDependencies:** শুধু development বা build-এর সময় লাগে (nodemon, jest, eslint, typescript)। `npm install -D nodemon`।

Production-এ `npm install --omit=dev` দিলে devDependencies install হয় না।

### Q30. Node.js stream কী এবং এর চারটা type কী কী?

Stream হলো data-কে একবারে পুরোটা memory-তে না এনে ছোট ছোট **chunk**-এ পড়া বা লেখার পদ্ধতি। যেমন ২GB ফাইল পুরো RAM-এ না তুলে টুকরো টুকরো করে client-এ পাঠানো। এতে memory কম লাগে এবং দ্রুত শুরু করা যায়।

**চার ধরন:**

- **Readable:** data পড়া যায় (`fs.createReadStream`, HTTP request)
- **Writable:** data লেখা যায় (`fs.createWriteStream`, HTTP response)
- **Duplex:** পড়া ও লেখা দুটোই (TCP socket)
- **Transform:** পড়তে পড়তে data বদলায় (`zlib.createGzip`)

```js
const { pipeline } = require('stream');
pipeline(
  fs.createReadStream('big.txt'),
  zlib.createGzip(),
  fs.createWriteStream('big.txt.gz'),
  (err) => { if (err) console.error(err); }
);
```

### Q31. Buffer কী এবং কখন ব্যবহার করবেন?

Buffer হলো fixed-size raw binary data রাখার জায়গা, যা V8 heap-এর বাইরে থাকে। যেখানে text না, binary data নিয়ে কাজ (file, image, network packet, crypto, base64 convert) সেখানে লাগে।

```js
const buf = Buffer.from('Hello');
buf.toString('base64');   // SGVsbG8=
buf.toString('hex');
```

### Q32. EventEmitter class কী?

`events` module-এর class, যা Observer pattern বাস্তবায়ন করে। একটা অংশ event **emit** করে, অন্য অংশ `on` দিয়ে সেটা **listen** করে। Node-এর অনেক জিনিস (stream, http server) EventEmitter-এর উপর তৈরি।

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();
emitter.on('order', (id) => console.log('Order placed:', id));
emitter.emit('order', 101);
```

`once` (একবার), `off` (listener সরানো) ও আছে। মনে রাখবেন: `'error'` event-এর listener না থাকলে emit করার সময় process crash করে।

### Q33. Synchronous এবং asynchronous file-system method এর পার্থক্য কী?

- **Sync** (`fs.readFileSync`): কাজ শেষ না হওয়া পর্যন্ত পুরো event loop আটকে থাকে। তাই request handle করার সময় অন্য সবার request আটকে যায়।
- **Async** (`fs.readFile` বা `fs.promises.readFile`): কাজ background-এ হয়, event loop free থাকে।

Server চালু হওয়ার সময় একবার config ফাইল পড়ার মতো জায়গায় sync চলে, কিন্তু request-এর ভেতরে সবসময় async ব্যবহার করুন।

### Q34. Uncaught exception এবং unhandled promise rejection কীভাবে handle করবেন?

```js
process.on('uncaughtException', (err) => {
  console.error(err);
  process.exit(1);   // log করে বন্ধ
});
process.on('unhandledRejection', (reason) => {
  console.error(reason);
  process.exit(1);
});
```

**মূল কথা:** এই হ্যান্ডলার শুধু শেষ ভরসা। Exception-এর পর app-এর অবস্থা অনিশ্চিত থাকে, তাই error log করে graceful shutdown করুন এবং PM2/Docker/Kubernetes দিয়ে auto-restart করান। আসল সমাধান হলো আগে থেকেই `try/catch`, `.catch()` ও Express-এর global error middleware ব্যবহার করা। (Node 15+ এ unhandled rejection ডিফল্টে process crash করায়।)

### Q35. Worker thread কী এবং কখন ব্যবহার করবেন?

`worker_threads` দিয়ে একই process-এ আলাদা thread-এ JavaScript চালানো যায়। **CPU-intensive** কাজের জন্য (image processing, বড় data parse, ভারী calculation, encryption) এটা ব্যবহার করুন, যাতে main thread আটকে না যায়। Thread-গুলো `postMessage` দিয়ে data আদান-প্রদান করে, চাইলে `SharedArrayBuffer`-ও share করা যায়। I/O-heavy কাজের জন্য worker লাগে না, কারণ সেটা Node আগে থেকেই ভালো সামলায়।

### Q36. child\_process এবং worker\_threads এর পার্থক্য কী?

- **child\_process** (`spawn`, `exec`, `fork`): সম্পূর্ণ আলাদা **OS process**, আলাদা memory ও V8 instance। যেকোনো external program (python script, ffmpeg) চালানো যায়। ভারী, কিন্তু একটা crash করলে অন্যটায় প্রভাব পড়ে না। Communication হয় IPC দিয়ে।
- **worker\_threads:** একই process-এর ভেতরে আলাদা **thread**, হালকা এবং দ্রুত, memory share করা সম্ভব।

আর `cluster` module একই Node app-এর একাধিক process চালিয়ে সব CPU core কাজে লাগায়।

### Q37. Global process object কী কাজে লাগে?

`process` হলো বর্তমান Node process সম্পর্কে তথ্য ও নিয়ন্ত্রণ দেওয়ার global object:

- `process.env`: environment variable
- `process.argv`: command line argument
- `process.cwd()`, `process.pid`, `process.platform`, `process.uptime()`
- `process.memoryUsage()`: memory ব্যবহার
- `process.exit()`: process বন্ধ করা
- `process.on('SIGTERM', ...)`: signal ও event ধরা (graceful shutdown-এ কাজে লাগে)
- `process.nextTick()`, `process.stdout`

### Q38. Environment variable কী এবং কীভাবে load করবেন?

Environment variable হলো code-এর বাইরে রাখা configuration (port, DB URL, JWT secret, API key)। এতে environment (dev/staging/production) ভেদে code না বদলিয়েই config বদলানো যায় এবং secret code-এ ঢোকে না।

```js
require('dotenv').config();           // .env ফাইল থেকে load
const port = process.env.PORT || 5000;
```

Node 20.6+ এ `node --env-file=.env app.js` দিয়েও load করা যায়। `.env` ফাইল সবসময় `.gitignore`-এ রাখুন এবং startup-এ প্রয়োজনীয় variable আছে কি না যাচাই করুন।

### Q39. Node.js কীভাবে concurrent request handle করে?

প্রতিটা request আসলে Node I/O কাজ (DB query ইত্যাদি) শুরু করে অপেক্ষা না করে ছেড়ে দেয় এবং পরের request ধরে। I/O শেষ হলে event loop সংশ্লিষ্ট callback চালায়। এভাবে এক thread-এ অনেক request interleave হয়ে চলে। তবে কোনো request ভারী CPU কাজ করলে বাকি সবাই আটকে যায়। Scale করতে: `cluster`/PM2 দিয়ে multiple process, load balancer, `worker_threads` ও horizontal scaling।

### Q40. Node.js application-এ memory leak কী কারণে হতে পারে?

সাধারণ কারণ:

- Global variable বা বাড়তেই থাকা array/Map/cache (limit ছাড়া)
- `removeListener` না করা event listener
- `clearInterval`/`clearTimeout` না করা timer
- Closure যা অপ্রয়োজনীয় বড় object ধরে রাখে
- বন্ধ না করা DB connection, stream বা file handle
- বড় data এর reference কোথাও রয়ে যাওয়া

**ধরার উপায়:** `process.memoryUsage()` দেখা, `node --inspect` দিয়ে Chrome DevTools-এ heap snapshot নেওয়া, clinic.js ব্যবহার করা। **সমাধান:** LRU cache, `WeakMap`, listener/timer cleanup।

## ৩. Express.js (Q41–60)

### Q41. Express.js কী এবং Node.js-এর সাথে এটা কেন বহুল ব্যবহৃত?

Express হলো Node.js-এর জন্য একটা minimal ও flexible web framework। Node-এর built-in `http` module দিয়ে server বানানো যায়, কিন্তু routing, body parsing, error handling সব নিজে লিখতে হয়। Express এই কাজগুলো অনেক সহজ করে দেয়।

**কেন জনপ্রিয়:** কম boilerplate, সহজ routing, শক্তিশালী middleware system, বিশাল ecosystem (cors, helmet, multer, passport ইত্যাদি), এবং REST API বানানোর জন্য খুব উপযোগী।

### Q42. Express-এ middleware কী?

Middleware হলো এমন function যা request আর response-এর মাঝখানে বসে এবং `(req, res, next)` তিনটা argument পায়। এটা:

- `req` ও `res` object পরিবর্তন বা পড়তে পারে,
- কোনো কাজ করতে পারে (logging, auth, validation, parsing),
- হয় response পাঠিয়ে cycle শেষ করে, নয়তো `next()` দিয়ে পরের middleware-এ পাঠায়।

```js
const logger = (req, res, next) => {
  console.log(req.method, req.url);
  next();
};
app.use(logger);
```

### Q43. Express middleware কোন ক্রমে execute হয়?

যে ক্রমে `app.use()` বা route define করা হয়েছে ঠিক সেই ক্রমে, উপর থেকে নিচে। কোনো middleware `next()` না ডাকলে বা response না পাঠালে request সেখানেই আটকে থাকে (hang করে)। তাই ক্রম গুরুত্বপূর্ণ: সাধারণত আগে `cors`, `express.json()`, logger; তারপর routes; তারপর 404 handler; সবার শেষে error-handling middleware।

### Q44. app.use() এবং app.get()/app.post() এর পার্থক্য কী?

- **app.use(path, fn):** সব HTTP method-এ কাজ করে এবং path **prefix** match করে। `app.use('/api', fn)` মানে `/api/users`, `/api/orders` সবকিছুর জন্য চলবে। মূলত middleware mount করতে ব্যবহার হয়।
- **app.get/post(path, fn):** শুধু নির্দিষ্ট HTTP method এবং **exact** path match করে। এগুলো route handler।

### Q45. Application-level, router-level এবং error-handling middleware কী?

- **Application-level:** `app.use()` বা `app.get()` দিয়ে পুরো app-এ বাঁধা।
- **Router-level:** `express.Router()` instance-এ বাঁধা (`router.use()`), শুধু ওই router-এর route-এর জন্য।
- **Error-handling:** চারটা parameter নেয় `(err, req, res, next)`। Express এটা দেখে চেনে যে এটা error handler।

এছাড়া **built-in** (`express.json()`) ও **third-party** (`cors`, `helmet`) middleware আছে।

### Q46. req.params, req.query এবং req.body কী?

- **req.params:** URL path-এর অংশ। `/users/:id` এ `/users/42` হলে `req.params.id === '42'`।
- **req.query:** `?` এর পরের অংশ। `/users?page=2&limit=10` হলে `req.query.page === '2'`।
- **req.body:** POST/PUT/PATCH-এ পাঠানো data; `express.json()` মতো parser ছাড়া এটা `undefined` থাকে।

মনে রাখবেন, `params` ও `query`-র সব value **string** হিসেবে আসে।

### Q47. express.json() এর কাজ কী?

এটা একটা built-in middleware যা request-এর JSON body পড়ে parse করে `req.body`-তে রেখে দেয়। এটা না দিলে `req.body` `undefined` হবে। HTML form data-র জন্য `express.urlencoded({ extended: true })` লাগে। Attack ঠেকাতে body size limit দিন: `express.json({ limit: '10kb' })`।

### Q48. Express Router কীভাবে তৈরি ও ব্যবহার করবেন?

Router দিয়ে route গুলোকে আলাদা ফাইলে ভাগ করা যায়, মডুলার রাখা যায়।

```js
// routes/user.routes.js
const router = require('express').Router();
router.get('/', getUsers);
router.post('/', createUser);
module.exports = router;

// app.js
app.use('/api/users', require('./routes/user.routes'));
```

### Q49. Centralized error handling কীভাবে implement করবেন?

সব জায়গায় আলাদা করে error response না লিখে একটাই error middleware রাখা হয় যা সবার শেষে বসে।

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
  }
}

// controller-এ
if (!user) return next(new AppError('User not found', 404));

// সবার শেষে
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  res.status(status).json({
    success: false,
    message: status === 500 ? 'Internal Server Error' : err.message,
  });
});
```

Production-এ stack trace বা internal তথ্য client-কে দেখাবেন না, শুধু log করবেন।

### Q50. Async route handler-এ error কীভাবে handle করবেন?

Express 4-এ async function থেকে throw হওয়া error নিজে থেকে error middleware-এ যায় না। তাই:

- প্রতিটা handler-এ `try/catch` করে `next(err)` ডাকুন, অথবা
- একটা wrapper বানান:

```js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/users', asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json(users);
}));
```

Express 5-এ rejected promise আপনা থেকেই error handler-এ যায়।

### Q51. next() এবং next('route') এর উদ্দেশ্য কী?

- **next():** পরের middleware বা handler-এ নিয়ন্ত্রণ পাঠায়।
- **next('route'):** বর্তমান route-এর বাকি handler/middleware বাদ দিয়ে পরের matching route-এ চলে যায়।
- **next(err):** সোজা error-handling middleware-এ চলে যায়।

### Q52. Scalable Express project কীভাবে structure করবেন?

Layered architecture ব্যবহার করুন, যেখানে প্রতিটা অংশের দায়িত্ব আলাদা:

```
src/
  config/        # env, db connection
  routes/        # URL ও middleware map
  controllers/   # request/response handle
  services/      # business logic
  models/        # Mongoose/Sequelize model
  middlewares/   # auth, validate, error
  validators/    # Joi/Zod schema
  utils/         # helper, AppError
app.js           # Express app setup
server.js        # listen + DB connect
```

`app.js` ও `server.js` আলাদা রাখলে testing সহজ হয়। Controller পাতলা রাখুন, business logic service-এ দিন।

### Q53. Incoming request data কীভাবে validate করবেন?

Client-এর data কখনও বিশ্বাস করবেন না। **Joi**, **Zod** বা **express-validator** ব্যবহার করে schema দিয়ে body/query/params যাচাই করুন, ভুল হলে ৪০০ ফেরত দিন।

```js
const validate = (schema) => (req, res, next) => {
  const result = schema.safeParse(req.body);
  if (!result.success) return res.status(400).json({ errors: result.error.issues });
  req.body = result.data;
  next();
};
app.post('/users', validate(userSchema), createUser);
```

### Q54. Express-এ file upload কীভাবে handle করবেন?

**Multer** middleware ব্যবহার করা হয় (`multipart/form-data` parse করে)।

```js
const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 2 * 1024 * 1024 },     // 2MB
  fileFilter: (req, file, cb) =>
    file.mimetype.startsWith('image/') ? cb(null, true) : cb(new Error('Only images')),
});
app.post('/upload', upload.single('avatar'), (req, res) => {
  // req.file.buffer → Cloudinary/S3-এ পাঠান
});
```

File size ও type limit দিন, আর বড় ফাইল server-এ না রেখে Cloudinary বা S3-এ রাখুন।

### Q55. Express-এ CORS কীভাবে enable করবেন?

`cors` package দিয়ে:

```js
const cors = require('cors');
app.use(cors({
  origin: ['https://myapp.com', 'http://localhost:5173'],
  credentials: true,
}));
```

Production-এ `origin: '*'` না দিয়ে নির্দিষ্ট domain-গুলো whitelist করুন। Cookie পাঠাতে হলে `credentials: true` লাগে এবং তখন origin `*` হতে পারে না।

### Q56. Pagination, filtering এবং sorting কীভাবে implement করবেন?

Query parameter দিয়ে: `/products?page=2&limit=10&sort=-price&category=phone`

```js
const page  = Math.max(1, +req.query.page  || 1);
const limit = Math.min(100, +req.query.limit || 10);
const filter = {};
if (req.query.category) filter.category = req.query.category;

const [items, total] = await Promise.all([
  Product.find(filter)
    .sort(req.query.sort || '-createdAt')
    .skip((page - 1) * limit)
    .limit(limit),
  Product.countDocuments(filter),
]);
res.json({ items, page, totalPages: Math.ceil(total / limit), total });
```

User যে field দিয়ে filter/sort করতে পারবে তা whitelist করে দিন এবং `limit`-এ সর্বোচ্চ সীমা রাখুন।

### Q57. Authentication middleware দিয়ে route কীভাবে protect করবেন?

```js
const protect = async (req, res, next) => {
  const header = req.headers.authorization;
  if (!header?.startsWith('Bearer ')) return res.status(401).json({ message: 'Not authenticated' });
  try {
    const decoded = jwt.verify(header.split(' ')[1], process.env.JWT_SECRET);
    req.user = await User.findById(decoded.id).select('-password');
    if (!req.user) return res.status(401).json({ message: 'User not found' });
    next();
  } catch {
    res.status(401).json({ message: 'Invalid or expired token' });
  }
};
app.get('/api/profile', protect, getProfile);
```

### Q58. 404 handling এবং global error handling এর পার্থক্য কী?

- **404 handler:** কোনো route-ই match না করলে চলে। এটা সাধারণ middleware (তিন parameter), সব route-এর **পরে** বসে এবং সাধারণত একটা `404 Not Found` error তৈরি করে।
- **Global error handler:** কোনো handler থেকে error throw বা `next(err)` হলে চলে। এটার **চারটা** parameter `(err, req, res, next)` এবং সবার **শেষে** থাকে।

```js
app.use((req, res, next) => next(new AppError(`Route ${req.originalUrl} not found`, 404)));
app.use(globalErrorHandler);
```

### Q59. Incoming request ও application error কীভাবে log করবেন?

- **Request log:** `morgan` middleware (`app.use(morgan('combined'))`)।
- **Application/error log:** `winston` বা `pino` (log level: error, warn, info, debug; file বা external service-এ পাঠানো যায়)।
- Production-এ structured (JSON) log ও request ID রাখুন যাতে একটা request ট্রেস করা যায়।
- Password, token, কার্ড নম্বরের মতো সংবেদনশীল তথ্য কখনও log করবেন না।

### Q60. Express server gracefully shutdown কীভাবে করবেন?

SIGTERM/SIGINT signal পেলে নতুন connection নেওয়া বন্ধ করে, চলমান request শেষ করে, তারপর DB connection বন্ধ করে process থেকে বের হতে হয়।

```js
const server = app.listen(PORT);

const shutdown = () => {
  server.close(async () => {
    await mongoose.connection.close();
    process.exit(0);
  });
  setTimeout(() => process.exit(1), 10000); // জোর করে বন্ধ
};
process.on('SIGTERM', shutdown);
process.on('SIGINT', shutdown);
```

Docker/Kubernetes-এ deploy করলে এটা বিশেষ জরুরি, নইলে deploy-র সময় user-দের request মাঝপথে কেটে যায়।

## ৪. MongoDB (Q61–80)

### Q61. MongoDB কী এবং relational database থেকে এটা কীভাবে আলাদা?

MongoDB হলো একটা **NoSQL document database**। Data JSON-এর মতো দেখতে document আকারে (আসলে BSON) জমা হয়।

| বিষয় | Relational (MySQL/PostgreSQL) | MongoDB |
| --- | --- | --- |
| Data structure | Table, row, column | Collection, document, field |
| Schema | নির্দিষ্ট ও কড়া | নমনীয় (flexible) |
| Relation | Join ও foreign key | Embedding বা reference |
| Query | SQL | MQL (JSON-style query) |
| Scaling | মূলত vertical | সহজে horizontal (sharding) |

যেখানে data-র গঠন প্রায়ই বদলায় বা nested data বেশি, সেখানে MongoDB ভালো। যেখানে জটিল join ও কড়া transaction দরকার (যেমন accounting), সেখানে SQL এগিয়ে।

### Q62. MongoDB-তে database, collection এবং document কী?

- **Database:** collection-এর ধারক (যেমন `shopDB`)
- **Collection:** একই ধরনের document-এর সমষ্টি, SQL-এর table-এর মতো (যেমন `users`)
- **Document:** একটা record, key-value জোড়ার সমষ্টি, SQL-এর row-এর মতো

```json
{ "_id": "...", "name": "Sakib", "skills": ["React", "Node"] }
```

### Q63. BSON কী?

BSON মানে Binary JSON। MongoDB data JSON আকারে দেখায়, কিন্তু ভেতরে binary format-এ সংরক্ষণ করে। সুবিধা:

- JSON-এর চেয়ে বেশি data type: `ObjectId`, `Date`, `Decimal128`, `Int32/Int64`, `Binary`
- Binary হওয়ায় parse ও traverse করা দ্রুত

একটা document-এর সর্বোচ্চ আকার **১৬MB**।

### Q64. find() এবং findOne() এর পার্থক্য কী?

- **find():** match করা সব document-এর একটা **cursor** দেয় (Mongoose-এ array)। কিছু না মিললে খালি।
- **findOne():** শুধু **প্রথম** match করা document দেয়, না পেলে `null`।

### Q65. insertOne() এবং insertMany() এর পার্থক্য কী?

- **insertOne(doc):** একটা document যোগ করে।
- **insertMany(\[docs\])**: একসাথে অনেক document যোগ করে, যা একটা একটা করে insert করার চেয়ে দ্রুত। ডিফল্টে `ordered: true`, অর্থাৎ মাঝখানে error হলে বাকিগুলো আর insert হয় না। `ordered: false` দিলে error হওয়া ছাড়া বাকিগুলো ঠিকই ঢোকে।

### Q66. updateOne(), updateMany() এবং replaceOne() এর পার্থক্য কী?

- **updateOne(filter, update):** প্রথম match-এর শুধু নির্দিষ্ট field বদলায়।
- **updateMany(filter, update):** match করা সবগুলো document বদলায়।
- **replaceOne(filter, newDoc):** পুরো document **প্রতিস্থাপন** করে দেয় (শুধু `_id` থাকে), তাই যা দেবেন না তা মুছে যায়।

```js
db.users.updateOne({ _id: id }, { $set: { age: 25 } });
db.users.updateMany({ active: false }, { $set: { archived: true } });
db.users.replaceOne({ _id: id }, { name: 'New', age: 30 });
```

`update` ও `updateMany`-তে `$set`, `$inc`, `$push` মতো operator লাগে; `replaceOne`-তে লাগে না।

### Q67. deleteOne() এবং deleteMany() এর পার্থক্য কী?

`deleteOne(filter)` প্রথম match করা একটা document মোছে। `deleteMany(filter)` match করা সব মোছে। `deleteMany({})` দিলে collection-এর **সব document** মুছে যায়, তাই সাবধান!

### Q68. \_id field এর উদ্দেশ্য কী?

`_id` হলো প্রতিটা document-এর **unique identifier** (primary key)। দেওয়া না হলে MongoDB নিজে একটা `ObjectId` বানায় (১২ বাইট: timestamp, random value, counter)। এটা **immutable** (বদলানো যায় না) এবং স্বয়ংক্রিয়ভাবে indexed। চাইলে নিজের মতো (string, number) `_id` দেওয়া যায়।

### Q69. Index কী এবং এটা কীভাবে query performance বাড়ায়?

Index হলো একটা আলাদা data structure (B-tree) যা নির্বাচিত field-এর মান সাজিয়ে রাখে, যেন MongoDB পুরো collection না ঘেঁটে সরাসরি দরকারি document খুঁজে পায়।

- Index না থাকলে: **COLLSCAN** (সব document স্ক্যান)
- Index থাকলে: **IXSCAN** (শুধু প্রাসঙ্গিক অংশ)

```js
db.users.createIndex({ email: 1 });
db.users.find({ email: 'a@b.com' }).explain('executionStats');
```

**Trade-off:** read দ্রুত হয়, কিন্তু write একটু ধীর হয় এবং বাড়তি storage লাগে। তাই অপ্রয়োজনীয় index বানাবেন না।

### Q70. Compound index কী এবং কখন বানাবেন?

একাধিক field নিয়ে বানানো index: `createIndex({ status: 1, createdAt: -1 })`। যখন query-তে বারবার একসাথে একাধিক field দিয়ে filter ও sort করেন, তখন বানান।

**মনে রাখার বিষয়:**

- Field-এর **ক্রম** গুরুত্বপূর্ণ। সাধারণ নিয়ম **ESR**: আগে Equality, তারপর Sort, তারপর Range।
- **Prefix rule:** `{a, b, c}` index দিয়ে `a`, `a+b`, `a+b+c` query চলে, কিন্তু শুধু `b` বা `c` দিয়ে চলে না।

### Q71. Aggregation pipeline কী?

এটা একাধিক **stage**-এর একটা শৃঙ্খল, যেখানে প্রতিটা stage আগের stage-এর output নিয়ে কাজ করে। Data filter, group, রূপান্তর, join করে report, analytics বা summary বানাতে এটা ব্যবহার হয়। অনেকটা কারখানার assembly line-এর মতো।

### Q72. $match, $group, $project, $lookup stage গুলো বুঝিয়ে বলুন।

- **$match:** শর্ত দিয়ে document filter (SQL-এর WHERE)
- **$group:** `_id` অনুযায়ী group করে sum, avg, count ইত্যাদি হিসাব (GROUP BY)
- **$project:** কোন field রাখবেন/বাদ দেবেন বা নতুন field বানাবেন
- **$lookup:** অন্য collection-এর সাথে join (LEFT JOIN-এর মতো)
- এছাড়া `$sort`, `$limit`, `$unwind` (array ভেঙে আলাদা document করা)

```js
db.orders.aggregate([
  { $match: { status: 'paid' } },
  { $group: { _id: '$customerId', total: { $sum: '$amount' } } },
  { $sort: { total: -1 } },
  { $lookup: { from: 'users', localField: '_id', foreignField: '_id', as: 'user' } },
]);
```

Performance-এর জন্য `$match` যতটা সম্ভব **শুরুর দিকে** রাখুন।

### Q73. Embedding এবং referencing এর পার্থক্য কী?

- **Embedding:** সম্পর্কিত data একই document-এর ভেতরে রাখা (যেমন user-এর ভেতরে `addresses` array)। সুবিধা: এক query-তে সব পাওয়া যায়, দ্রুত read, atomic update।
- **Referencing:** আলাদা collection-এ রেখে `_id` দিয়ে লিংক করা (যেমন order-এ `userId`)। সুবিধা: data বারবার কপি হয় না, document ছোট থাকে।

**কখন কোনটা:** যে data সবসময় একসাথে পড়া হয় এবং সংখ্যায় কম, সেটা embed করুন। যে data বেড়েই চলে (যেমন লক্ষ comment) বা অনেক জায়গায় share হয়, সেটা reference করুন।

### Q74. MongoDB-তে one-to-one, one-to-many এবং many-to-many relationship কীভাবে হয়?

- **One-to-one:** সাধারণত embed (user ও profile)। আলাদা রাখতে চাইলে reference।
- **One-to-many:** "many" কম হলে parent-এর ভেতরে array হিসেবে embed (post ও কয়েকটা tag)। অনেক হলে child-এ parent-এর `_id` রাখা (order-এ `userId`)।
- **Many-to-many:** দুই দিকেই reference-এর array রাখা (student-এ `courseIds`, course-এ `studentIds`) অথবা এক দিকে রেখে অন্য দিকে `$lookup` করা।

### Q75. MongoDB transaction কী এবং কখন লাগে?

Transaction দিয়ে একাধিক operation/document-এ **ACID** (সব হবে নয়তো কিছুই হবে না) নিশ্চিত করা যায়। এটা replica set বা sharded cluster-এ চলে।

**কখন লাগে:** অর্থ স্থানান্তর (এক account থেকে কেটে অন্যটায় যোগ), order তৈরি + stock কমানো, যেখানে একাধিক document একসাথে সফল বা ব্যর্থ হতে হবে।

মনে রাখবেন, **একটা document-এর উপর** যেকোনো operation এমনিতেই atomic। তাই ভালো schema design (embedding) দিয়ে অনেক সময় transaction এড়ানো যায়, কারণ transaction-এ performance খরচ আছে।

### Q76. Unique index এবং normal index এর পার্থক্য কী?

- **Normal index:** শুধু query দ্রুত করে।
- **Unique index:** query দ্রুত করার সাথে সাথে ওই field-এ duplicate value ঢুকতে দেয় না। ঢোকালে `E11000 duplicate key` error আসে (যেমন `email`)।

```js
db.users.createIndex({ email: 1 }, { unique: true });
```

অনেক document-এ field-টা না থাকলে সেটা `null` ধরা হয় এবং duplicate দেখায়। তখন `partialFilterExpression` বা `sparse` ব্যবহার করুন।

### Q77. MongoDB query-তে projection-এর উদ্দেশ্য কী?

Projection দিয়ে নির্ধারণ করা যায় result-এ কোন কোন field আসবে। এতে network ও memory-র ব্যবহার কমে এবং sensitive field (password) লুকানো যায়।

```js
db.users.find({}, { name: 1, email: 1, _id: 0 });   // শুধু name, email
db.users.find({}, { password: 0 });                 // password ছাড়া সব
```

এক projection-এ include ও exclude মেশানো যায় না (শুধু `_id` ব্যতিক্রম)।

### Q78. MongoDB-তে pagination কীভাবে implement করবেন?

**১. skip + limit (offset-based):** সহজ, কিন্তু বড় skip-এর জন্য ধীর, কারণ MongoDB-কে skip হওয়া সব document পেরিয়ে যেতে হয়।

```js
db.posts.find().sort({ createdAt: -1 }).skip((page - 1) * limit).limit(limit);
```

**২. Cursor/range-based:** শেষ দেখা document-এর `_id` (বা sort field) থেকে এগোনো। বড় data-র জন্য অনেক দ্রুত ও স্থিতিশীল।

```js
db.posts.find({ _id: { $lt: lastId } }).sort({ _id: -1 }).limit(limit);
```

### Q79. Slow MongoDB query কীভাবে শনাক্ত ও optimize করবেন?

**শনাক্ত করার উপায়:**

- `explain('executionStats')` দেখুন: COLLSCAN হচ্ছে কি না, `totalDocsExamined` বনাম `nReturned` কত
- Database Profiler (`db.setProfilingLevel`) ও slow query log
- Atlas-এ Performance Advisor

**Optimize করার উপায়:**

- সঠিক index (দরকারে compound) বানানো
- Projection ও `limit` ব্যবহার
- Aggregation-এ `$match` আগে রাখা
- বড় `skip` এড়ানো, শুরুতে wildcard থাকা regex এড়ানো
- Schema নতুন করে ডিজাইন (embed করে join কমানো)
- `.lean()` ব্যবহার (Mongoose-এ)

### Q80. Connection pooling কী এবং কেন গুরুত্বপূর্ণ?

প্রতিটা request-এর জন্য নতুন DB connection খোলা ও বন্ধ করা খুব ব্যয়বহুল। Connection pool আগে থেকে কিছু connection খুলে রাখে এবং request এলে সেখান থেকে একটা নিয়ে কাজ শেষে ফেরত দেয়।

**উপকার:** দ্রুত response, কম resource খরচ, এবং DB-কে অতিরিক্ত connection-এর চাপ থেকে রক্ষা।

```js
mongoose.connect(process.env.MONGO_URI, { maxPoolSize: 10 });
```

অ্যাপ চালুর সময় **একবারই** connect করুন, প্রতিটা request-এ নয়। Pool খুব ছোট হলে request অপেক্ষায় থাকে, খুব বড় হলে DB-তে চাপ পড়ে।

## ৫. Mongoose (Q81–100)

### Q81. Mongoose কী এবং MongoDB-র সাথে এটা কেন ব্যবহার করা হয়?

Mongoose হলো Node.js-এর জন্য একটা **ODM (Object Data Modeling)** library। MongoDB নিজে schema-less, মানে যা খুশি data ঢোকানো যায়। Mongoose তাতে কাঠামো যোগ করে এবং দেয়:

- Schema দিয়ে data-র গঠন নির্ধারণ
- Built-in validation ও default value
- Middleware (hook), virtual, populate
- সহজ ও পড়তে সুবিধাজনক query API

### Q82. MongoDB এবং Mongoose এর পার্থক্য কী?

**MongoDB** হলো আসল database (যেখানে data থাকে)। **Mongoose** হলো একটা Node.js library, যা ওই database-এর সাথে কথা বলার সহজ ও কাঠামোবদ্ধ উপায় দেয়। Mongoose ছাড়াও MongoDB-র official native driver দিয়ে সরাসরি কাজ করা যায়, তবে তাতে schema ও validation নিজেকে সামলাতে হয়।

### Q83. Mongoose schema কী?

Schema হলো একটা collection-এর document-এর কাঠামোর নকশা: কোন field থাকবে, type কী, কোনটা বাধ্যতামূলক, default কী, validation কী।

```js
const userSchema = new mongoose.Schema({
  name:  { type: String, required: true, trim: true },
  email: { type: String, required: true, unique: true, lowercase: true },
  age:   { type: Number, min: 0, default: 18 },
}, { timestamps: true });
```

### Q84. Mongoose model কী?

Model হলো schema থেকে বানানো একটা constructor/class, যা দিয়ে database-এর সাথে সরাসরি কাজ করা হয় (create, find, update, delete)।

```js
const User = mongoose.model('User', userSchema);
```

`'User'` দিলে MongoDB-তে collection-এর নাম হয় `users` (ছোট হাতের ও বহুবচন)।

### Q85. Schema এবং model এর পার্থক্য কী?

- **Schema:** শুধু blueprint বা নকশা। এটা দিয়ে সরাসরি DB-তে কিছু করা যায় না।
- **Model:** ওই নকশা থেকে তৈরি, যা DB-র collection-এর সাথে যোগাযোগের interface দেয়।
- **Document:** model-এর একটা instance, মানে DB-র একটা আসল record।

সহজ উদাহরণ: schema = বাড়ির নকশা, model = নির্মাণ কোম্পানি, document = বানানো একটা বাড়ি।

### Q86. Mongoose-এর সাধারণ schema data type গুলো কী?

`String`, `Number`, `Boolean`, `Date`, `Buffer`, `ObjectId` (`mongoose.Schema.Types.ObjectId`), `Array`, `Mixed` (যেকোনো type), `Map`, `Decimal128`, `UUID`। Nested object ও subdocument-ও schema-তে রাখা যায়।

### Q87. required, unique এবং default schema option এর উদ্দেশ্য কী?

- **required:** field না থাকলে validation error। `required: [true, 'Name is required']` দিয়ে message-ও দেওয়া যায়।
- **unique:** unique **index** তৈরি করে। **সতর্কতা:** এটা validator না। তাই duplicate হলে validation error নয়, MongoDB-র `E11000` error আসে।
- **default:** value না দিলে এই মান বসে (`default: Date.now`, `default: 'user'`)।

### Q88. Mongoose-এ custom validation কীভাবে করবেন?

```js
const userSchema = new mongoose.Schema({
  phone: {
    type: String,
    validate: {
      validator: (v) => /^01[3-9]\d{8}$/.test(v),
      message: (props) => `${props.value} is not a valid phone number`,
    },
  },
});
```

Async validator-ও লেখা যায়। মনে রাখবেন, `update` ধরনের query-তে ডিফল্টে validator চলে না, তাই `{ runValidators: true }` দিতে হয়।

### Q89. pre এবং post middleware hook কী?

এগুলো এমন function যা কোনো operation (`save`, `find`, `updateOne`, `deleteOne`) এর **আগে (pre)** বা **পরে (post)** আপনা থেকেই চলে।

```js
userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

userSchema.post('save', function (doc) {
  console.log('User saved:', doc._id);
});
```

সাধারণ ব্যবহার: password hash, slug তৈরি, audit log, cascade delete। মনে রাখবেন, `pre('save')` সাধারণ `updateOne`-এ চলে না, সেজন্য আলাদা `pre('updateOne')` লাগে।

### Q90. save() এবং create() এর পার্থক্য কী?

- **save():** একটা document **instance**-এর method। আগে `new User({...})` বানিয়ে পরে `.save()`। আগের document পরিবর্তন করে `.save()` দিলে update-ও হয়।
- **create():** model-এর static shortcut। ভেতরে `new` + `save` করে দেয়, এবং array দিয়ে একসাথে অনেক document বানাতে পারে।

```js
const u = new User({ name: 'A' }); await u.save();
await User.create({ name: 'B' });
await User.create([{ name: 'C' }, { name: 'D' }]);
```

দুটোতেই validation ও middleware চলে।

### Q91. findById() এবং findOne() এর পার্থক্য কী?

`findById(id)` হলো `findOne({ _id: id })`-এর shortcut। শুধু `_id` দিয়ে খুঁজলে `findById`, অন্য যেকোনো শর্তে (`email`, `username`) `findOne`। Invalid format-এর id দিলে `findById` `CastError` দেয়, তাই আগে `mongoose.isValidObjectId(id)` দিয়ে যাচাই করতে পারেন।

### Q92. populate() কী করে এবং কখন ব্যবহার করবেন?

Schema-তে `ref` দিয়ে অন্য collection-এর `_id` রাখা থাকলে, `populate()` সেই `_id`-র জায়গায় আসল document বসিয়ে দেয় (অনেকটা SQL join-এর মতো, তবে ভেতরে আলাদা query চলে)।

```js
const orders = await Order.find()
  .populate('user', 'name email');   // শুধু name, email আনো
```

যখন related collection-এর data দরকার (order-এর সাথে user-এর নাম) তখন ব্যবহার করুন। কিন্তু অতিরিক্ত বা nested populate performance কমায়, তাই সাবধানে।

### Q93. lean() এবং normal Mongoose query এর পার্থক্য কী?

Normal query **Mongoose document** ফেরত দেয়, যাতে `save()`, virtual, getter ও change tracking থাকে, তাই ভারী। `.lean()` দিলে সাধারণ **plain JavaScript object** ফেরত আসে, যা অনেক দ্রুত এবং কম memory নেয়।

```js
const users = await User.find().lean();
```

শুধু data পড়ে JSON হিসেবে পাঠানোর কাজে (read-only API) `lean()` আদর্শ। কিন্তু ফেরত আসা object-এ `.save()` বা virtual পাবেন না।

### Q94. Mongoose virtual property কী?

Virtual হলো এমন property যা **database-এ জমা হয় না**, বরং অন্য field থেকে হিসাব করে তৈরি হয়।

```js
userSchema.virtual('fullName').get(function () {
  return `${this.firstName} ${this.lastName}`;
});
// JSON-এ দেখাতে:
userSchema.set('toJSON', { virtuals: true });
```

### Q95. Instance method এবং static method এর পার্থক্য কী?

- **Instance method:** একটা নির্দিষ্ট document-এর উপর কাজ করে, `this` হলো ওই document। যেমন `user.comparePassword(pw)`।
- **Static method:** পুরো model-এর উপর কাজ করে, `this` হলো model। যেমন `User.findByEmail(email)`।

```js
userSchema.methods.comparePassword = function (pw) {
  return bcrypt.compare(pw, this.password);
};
userSchema.statics.findByEmail = function (email) {
  return this.findOne({ email });
};
```

### Q96. Mongoose-এ duplicate-key error কীভাবে handle করবেন?

Unique index ভাঙলে error-এর `code` হয় `11000`। এটা ধরে client-কে বন্ধুত্বপূর্ণ message এবং `409 Conflict` দিন।

```js
try {
  await User.create(req.body);
} catch (err) {
  if (err.code === 11000) {
    const field = Object.keys(err.keyValue)[0];
    return res.status(409).json({ message: `${field} already exists` });
  }
  next(err);
}
```

আগে `findOne` দিয়ে চেক করা যথেষ্ট নয়, কারণ দুইটা request একসাথে এলে race condition হয়। তাই unique index + error handling দুটোই রাখুন।

### Q97. দুটো Mongoose model-এর মধ্যে reference কীভাবে define করবেন?

```js
const orderSchema = new mongoose.Schema({
  user:  { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  items: [{ product: { type: mongoose.Schema.Types.ObjectId, ref: 'Product' }, qty: Number }],
});
```

`ref`-এর মান হতে হবে ওই model-এর নাম (যা `mongoose.model('User', ...)`-এ দেওয়া হয়েছে)। পরে `populate('user')` দিয়ে আসল data আনা যায়।

### Q98. Mongoose-এ session এবং transaction কীভাবে ব্যবহার করবেন?

(Replica set বা Atlas cluster লাগে।)

```js
const session = await mongoose.startSession();
try {
  await session.withTransaction(async () => {
    await Account.updateOne({ _id: from }, { $inc: { balance: -100 } }, { session });
    await Account.updateOne({ _id: to },   { $inc: { balance:  100 } }, { session });
  });
} finally {
  session.endSession();
}
```

প্রতিটা operation-এ `{ session }` পাঠাতে ভুলবেন না। কোনো একটা fail করলে পুরো transaction rollback হয়ে যায়।

### Q99. timestamps: true এর উদ্দেশ্য কী?

Schema-র option-এ `{ timestamps: true }` দিলে Mongoose স্বয়ংক্রিয়ভাবে দুটো field যোগ ও পরিচালনা করে: `createdAt` (বানানোর সময়) এবং `updatedAt` (প্রতিবার পরিবর্তনে হালনাগাদ হয়)। নিজে আলাদা করে লিখতে হয় না।

### Q100. অপ্রয়োজনীয় populate() এড়িয়ে query performance কীভাবে বাড়াবেন?

- শুধু **দরকারি field** populate করুন: `populate('user', 'name avatar')`
- List API-তে populate এড়িয়ে চলুন, বিশেষ করে nested হলে (N+1 সমস্যা এড়ান)
- বারবার একসাথে লাগে এমন ছোট data **embed** বা denormalize করুন (যেমন order-এ user-এর নামের একটা কপি)
- Read-only কাজে `.lean()` ব্যবহার করুন
- `select()` দিয়ে শুধু দরকারি field আনুন
- Reference field-এ index বসান
- Filter বা জটিল join দরকার হলে `populate`-এর বদলে aggregation `$lookup` ব্যবহার করুন
- Pagination ও `limit` রাখুন, বারবার লাগা data cache করুন (Redis)

## ৬. Authentication and Authorization (Q101–120)

### Q101. Authentication এবং authorization এর পার্থক্য কী?

- **Authentication (AuthN):** "তুমি কে?" যাচাই করা। যেমন email-password দিয়ে login।
- **Authorization (AuthZ):** "তুমি কী করতে পারবে?" ঠিক করা। যেমন সাধারণ user অন্যের order দেখতে পারবে না, শুধু admin পারবে।

আগে authentication, তারপর authorization। উদাহরণ: বিমানবন্দরে পাসপোর্ট দেখে পরিচয় যাচাই হলো authentication, আর বোর্ডিং পাসের ক্লাস দেখে লাউঞ্জে ঢুকতে দেওয়া হলো authorization।

### Q102. Session-based authentication কীভাবে কাজ করে?

1. User login করলে server credential যাচাই করে।
2. Server একটা **session** বানিয়ে নিজের কাছে (memory, Redis বা DB-তে) রাখে।
3. Client-কে একটা **session ID** দেয়, যা cookie-তে (`httpOnly`) থাকে।
4. প্রতিটা request-এ browser নিজে cookie পাঠায়, server session ID দেখে user চেনে।
5. Logout করলে server সেই session মুছে দেয়।

**সুবিধা:** সহজে revoke করা যায়। **অসুবিধা:** server-এ state রাখতে হয়, একাধিক server থাকলে shared store (Redis) লাগে, এবং CSRF-এর ঝুঁকি আছে।

### Q103. JWT-based authentication কীভাবে কাজ করে?

1. Login সফল হলে server একটা secret key দিয়ে **সই করা token (JWT)** বানায়, যাতে user-এর তথ্য (id, role) থাকে।
2. Client token রাখে এবং প্রতিটা request-এ `Authorization: Bearer <token>` header-এ পাঠায়।
3. Server শুধু signature যাচাই করে, DB-তে খুঁজতে হয় না। তাই এটা **stateless**।

**সুবিধা:** scale করা সহজ, mobile ও SPA-র জন্য ভালো। **অসুবিধা:** একবার দেওয়া token মেয়াদ শেষের আগে সহজে বাতিল করা যায় না (দেখুন Q117)।

### Q104. JWT-এর অংশগুলো কী কী?

তিনটা অংশ, `.` দিয়ে আলাদা: `header.payload.signature`

- **Header:** algorithm ও type (`{ "alg": "HS256", "typ": "JWT" }`)
- **Payload:** claims, যেমন `sub` (user id), `role`, `iat` (কখন বানানো), `exp` (কখন মেয়াদ শেষ)
- **Signature:** `HMAC(header + payload, secret)`, যা প্রমাণ করে token বদলানো হয়নি

**গুরুত্বপূর্ণ:** Payload শুধু Base64 **encode** করা, encrypt করা নয়। যে কেউ পড়তে পারে। তাই এতে password বা গোপন তথ্য রাখবেন না।

### Q105. Access token এবং refresh token এর পার্থক্য কী?

- **Access token:** **স্বল্পমেয়াদী** (৫–১৫ মিনিট), প্রতিটা API request-এ পাঠানো হয়। চুরি হলেও ক্ষতির সময় কম।
- **Refresh token:** **দীর্ঘমেয়াদী** (৭–৩০ দিন), শুধু নতুন access token আনার জন্য ব্যবহার হয়। এটা DB-তে রেখে প্রয়োজনে বাতিল করা যায়।

এই জুটির কারণে user বারবার login না করেও নিরাপদ থাকে।

### Q106. Access ও refresh token কোথায় রাখা উচিত?

- **Access token:** ব্রাউজারে JavaScript variable বা memory-তে (reload হলে refresh token দিয়ে আবার আনা যায়)।
- **Refresh token:** `httpOnly`, `Secure`, `SameSite` cookie-তে।
- `localStorage`-এ token রাখা ঝুঁকিপূর্ণ, কারণ XSS হলে JavaScript সেটা পড়ে ফেলতে পারে। `httpOnly` cookie JavaScript দিয়ে পড়া যায় না, আর `SameSite` দিয়ে CSRF অনেকটা ঠেকানো যায়।
- Mobile app-এ Keychain/Keystore-এর মতো secure storage ব্যবহার করুন।

### Q107. Express middleware-এ JWT কীভাবে verify করবেন?

```js
const protect = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ message: 'No token' });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET, { algorithms: ['HS256'] });
    next();
  } catch (err) {
    const msg = err.name === 'TokenExpiredError' ? 'Token expired' : 'Invalid token';
    res.status(401).json({ message: msg });
  }
};
```

`algorithms` স্পষ্টভাবে দিন যাতে attacker অন্য algorithm চালাতে না পারে।

### Q108. JWT-এর মেয়াদ শেষ হলে কী হয়?

`jwt.verify()` `TokenExpiredError` throw করে এবং server `401` ফেরত দেয়। Client তখন refresh token দিয়ে নতুন access token চায়। Refresh token-ও মেয়াদোত্তীর্ণ বা অবৈধ হলে user-কে আবার login করতে হয়।

### Q109. Refresh-token flow কীভাবে কাজ করে?

1. Login করলে server **access token + refresh token** দেয় (refresh token cookie-তে, এর hash DB-তে)।
2. Client access token দিয়ে API ডাকে।
3. Access token মেয়াদ শেষ হলে API `401` দেয়।
4. Client `/auth/refresh` endpoint-এ ডাকে (cookie নিজে চলে যায়)।
5. Server refresh token যাচাই করে (signature, মেয়াদ, DB-তে আছে কি না)।
6. সফল হলে **নতুন access token** দেয় এবং **refresh token rotation** করে (নতুন refresh token দেয়, পুরনোটা বাতিল)।
7. বাতিল হওয়া refresh token আবার ব্যবহার হলে বুঝতে হবে চুরি হয়েছে, তখন ওই user-এর সব token revoke করুন।

### Q110. Password hashing কী এবং plain text-এ password কেন রাখা উচিত নয়?

Hashing হলো password-কে একটা one-way transformation দিয়ে অপাঠ্য স্ট্রিং বানানো, যা থেকে আসল password ফেরত পাওয়া যায় না। Plain text-এ রাখলে DB leak হলে সবার password সরাসরি প্রকাশ হয়ে যায়, আর বেশিরভাগ user এক password অনেক জায়গায় ব্যবহার করে, ফলে অন্য সাইটও বিপদে পড়ে। এমনকি developer বা admin-ও password দেখতে পারত, যা অনৈতিক ও অনেক জায়গায় আইনবিরুদ্ধ।

### Q111. bcrypt কী এবং salt কী?

**bcrypt** হলো একটা password-নির্দিষ্ট hashing algorithm, যা **ইচ্ছাকৃতভাবে ধীর** এবং `cost factor` (salt rounds, সাধারণত ১০–১২) বাড়িয়ে আরও ধীর করা যায়, ফলে brute-force কঠিন হয়।

**Salt** হলো প্রতিটা password-এর জন্য আলাদা random স্ট্রিং, যা hash করার আগে যোগ হয়। এতে:

- দুই user-এর password এক হলেও hash আলাদা হয়
- আগে থেকে বানানো rainbow table কাজ করে না

Salt hash-এর ভেতরেই জমা থাকে, তাই আলাদা সংরক্ষণ লাগে না।

```js
const hash = await bcrypt.hash(password, 12);
const ok   = await bcrypt.compare(password, hash);
```

বিকল্প: `argon2`, `scrypt`।

### Q112. Hashing এবং encryption এর পার্থক্য কী?

- **Hashing:** one-way, কোনো key লাগে না, আসল data ফেরত আনা যায় না। Password ও integrity check-এর জন্য।
- **Encryption:** two-way, key দিয়ে encrypt করে key দিয়েই decrypt করা যায়। যে data পরে আবার পড়া দরকার (ব্যক্তিগত বার্তা, সংবেদনশীল ফাইল) তার জন্য।

Password কখনও encrypt করে রাখবেন না, hash করুন।

### Q113. Role-Based Access Control (RBAC) কী?

RBAC-তে permission সরাসরি user-কে না দিয়ে **role**-কে দেওয়া হয়, আর user-কে এক বা একাধিক role দেওয়া হয়। যেমন role: `admin`, `editor`, `user`। Admin সব delete করতে পারে, editor শুধু লিখতে ও সম্পাদনা করতে পারে, user শুধু নিজের data দেখতে পারে। এতে permission পরিচালনা সহজ ও পরিষ্কার হয়।

### Q114. কোনো API route শুধু admin user-এর জন্য কীভাবে সীমিত করবেন?

```js
const authorize = (...roles) => (req, res, next) => {
  if (!roles.includes(req.user.role)) {
    return res.status(403).json({ message: 'Forbidden: insufficient permission' });
  }
  next();
};

app.delete('/api/users/:id', protect, authorize('admin'), deleteUser);
```

`protect` আগে চলে `req.user` সেট করে, তারপর `authorize` role দেখে। এটা **closure**-এর একটা বাস্তব উদাহরণ (Q8)।

### Q115. OAuth 2.0 কী এবং কখন ব্যবহার করবেন?

OAuth 2.0 হলো একটা **authorization framework**, যা দিয়ে কোনো thirdparty app user-এর password না জেনেও user-এর অনুমতিতে তার কোনো resource (যেমন Google profile, GitHub repo) সীমিতভাবে ব্যবহার করতে পারে।

**মূল চরিত্র:** Resource Owner (user), Client (আপনার app), Authorization Server (Google), Resource Server (Google API)।

**সাধারণ flow:** Authorization Code flow (সাথে PKCE)। User Google-এ গিয়ে অনুমতি দেয় → আপনার app একটা code পায় → code দিয়ে token নেয়।

**কখন ব্যবহার করবেন:** "Login with Google/GitHub" (এর জন্য OAuth-এর উপর OpenID Connect লাগে), অন্য service-এর API-তে user-এর হয়ে কাজ করা। Node-এ `passport.js` বা `openid-client` কাজে লাগে।

### Q116. 401 Unauthorized এবং 403 Forbidden এর পার্থক্য কী?

- **401:** আপনি **authenticated নন**। Token নেই, ভুল বা মেয়াদোত্তীর্ণ। সমাধান: login করুন।
- **403:** আপনি authenticated, কিন্তু **অনুমতি নেই**। সমাধান: ভিন্ন অনুমতি/role দরকার, আবার login করে লাভ নেই।

নাম "Unauthorized" হলেও 401 আসলে authentication-এর সমস্যা।

### Q117. JWT ব্যবহার করলে logout কীভাবে implement করবেন?

JWT stateless, তাই server চাইলেই সহজে একে বাতিল করতে পারে না। উপায়:

- **Client থেকে মুছে ফেলা:** token ও cookie delete (সবচেয়ে সাধারণ)।
- **Access token ছোট মেয়াদের রাখা** (৫–১৫ মিনিট)।
- **Refresh token DB থেকে মুছে/revoke করা**, ফলে নতুন access token আর পাওয়া যায় না।
- **Denylist/blacklist:** logout করা token-এর `jti` Redis-এ মেয়াদ শেষ হওয়া পর্যন্ত রাখা, প্রতিটা request-এ চেক।
- **Token versioning:** user-এর `tokenVersion` বাড়ালে পুরনো সব token অবৈধ হয়ে যায়।

### Q118. Account lockout বা login rate limiting কীভাবে handle করবেন?

- Login route-এ `express-rate-limit` (যেমন ১৫ মিনিটে ৫ বার), multi-server হলে Redis store।
- প্রতিটা account-এর failed attempt গুনে ধরে নিদিষ্ট সংখ্যার পর **সাময়িক lockout** (১৫ মিনিট), বা ক্রমে বাড়তে থাকা delay।
- বারবার ব্যর্থ হলে CAPTCHA বা MFA, এবং user-কে email-এ সতর্কতা।
- Error message সাধারণ রাখুন ("email বা password ভুল")।
- সতর্কতা: কেউ যেন ইচ্ছা করে অন্যের account lock করে দিয়ে DoS না করতে পারে, তাই IP ও account দুটো মিলিয়ে সীমা ঠিক করুন।

### Q119. নিরাপদে password reset কীভাবে implement করবেন?

1. User email দেয়। Email আছে কি না যাই হোক, response একই রাখুন ("আপনার email-এ লিংক পাঠানো হয়েছে") যাতে কেউ জানতে না পারে কোন email registered।
2. `crypto.randomBytes(32)` দিয়ে random token বানান। DB-তে এর **hash** রাখুন সাথে **ছোট মেয়াদ** (১৫ মিনিট)।
3. Token সহ লিংক email-এ পাঠান।
4. User নতুন password দিলে token-এর hash ও মেয়াদ যাচাই করুন।
5. নতুন password hash করে সেভ করুন, token **একবার ব্যবহারের পরই বাতিল** করুন এবং ওই user-এর সব পুরনো session/refresh token revoke করুন।
6. User-কে "password বদলানো হয়েছে" নোটিফিকেশন পাঠান।

আর রাখুন: HTTPS, rate limiting, এবং URL-এর token log না করা।

### Q120. Registration ও login-এর সময় কোন কোন নিরাপত্তা যাচাই করা উচিত?

**Registration:**

- Input validation (email format, password strength ও দৈর্ঘ্য)
- Email-কে lowercase/trim করে duplicate চেক
- Password `bcrypt`/`argon2` দিয়ে hash
- Mass assignment রোধ: user যেন নিজে `role: 'admin'` পাঠিয়ে admin না হতে পারে
- Email verification এবং rate limiting
- Response-এ password বা hash কখনও না ফেরত দেওয়া

**Login:**

- Generic error message (কোনটা ভুল তা না বলা)
- `bcrypt.compare` দিয়ে যাচাই
- Rate limit ও lockout
- HTTPS, `httpOnly` ও `Secure` cookie
- সন্দেহজনক login log করা, প্রয়োজনে MFA

## ৭. REST API Development (Q121–140)

### Q121. API কী এবং REST API কী?

**API (Application Programming Interface)** হলো দুটো software-এর মধ্যে কথা বলার নিয়ম বা চুক্তি। রেস্টুরেন্টে ওয়েটারের মতো: আপনি (client) অর্ডার দেন, ওয়েটার (API) রান্নাঘরে (server) জানায় এবং খাবার (data) এনে দেয়।

**REST API** হলো এমন API যা HTTP এবং REST-এর নীতি মেনে চলে। এখানে সবকিছু **resource** (user, product, order), প্রতিটার একটা URL থাকে, আর HTTP method (GET, POST...) দিয়ে কাজ করানো হয়। Data সাধারণত JSON আকারে আদান-প্রদান হয়।

### Q122. REST-এর মূল নীতিগুলো কী কী?

- **Client-Server:** frontend ও backend আলাদা এবং স্বাধীন।
- **Stateless:** প্রতিটা request-এ প্রয়োজনীয় সব তথ্য থাকতে হবে, server আগের request মনে রাখে না (তাই token পাঠাতে হয়)।
- **Cacheable:** response cache করা যাবে কি না তা জানানো যায়।
- **Uniform interface:** resource-এর URL (noun, বহুবচন: `/users`), standard HTTP method ও status code।
- **Layered system:** client জানে না সে সরাসরি server-এ নাকি load balancer/proxy-র মাধ্যমে কথা বলছে।
- **Code on demand** (ঐচ্ছিক)।

### Q123. GET, POST, PUT, PATCH এবং DELETE এর পার্থক্য কী?

| Method | কাজ | উদাহরণ |
| --- | --- | --- |
| **GET** | data পড়া (safe, কিছু বদলায় না) | `GET /users/5` |
| **POST** | নতুন resource তৈরি | `POST /users` |
| **PUT** | পুরো resource **প্রতিস্থাপন** | `PUT /users/5` |
| **PATCH** | resource-এর **আংশিক** পরিবর্তন | `PATCH /users/5` |
| **DELETE** | resource মোছা | `DELETE /users/5` |

### Q124. PUT এবং PATCH এর পার্থক্য কী?

- **PUT:** পুরো resource-এর নতুন সম্পূর্ণ রূপ পাঠাতে হয়। যে field পাঠানো হয়নি, সেটা মুছে বা রিসেট হয়ে যেতে পারে।
- **PATCH:** শুধু যে field বদলাতে চান তা পাঠান, বাকিটা যেমন আছে তেমন থাকে।

```js
// user = { name: 'A', age: 20, city: 'Dhaka' }
PUT   { name: 'B', age: 21, city: 'Dhaka' }  // সব দিতে হবে
PATCH { age: 21 }                              // শুধু age বদলাবে
```

### Q125. HTTP request ও response header কী?

Header হলো request বা response-এর **metadata** (key-value জোড়া), মূল body-র বাইরের তথ্য।

- **Request header:** `Authorization` (token), `Content-Type` (body কোন format-এ), `Accept` (কোন format চাই), `User-Agent`, `Cookie`, `Origin`
- **Response header:** `Content-Type`, `Set-Cookie`, `Cache-Control`, `Location` (নতুন resource-এর URL), `Access-Control-Allow-Origin` (CORS), `Retry-After`

### Q126. সবচেয়ে বেশি ব্যবহৃত HTTP status code গুলো কী কী?

Group: **1xx** তথ্য, **2xx** সফল, **3xx** redirect, **4xx** client-এর ভুল, **5xx** server-এর ভুল।

- `200 OK`, `201 Created`, `204 No Content`
- `301/302` redirect, `304 Not Modified`
- `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict` (যেমন duplicate), `422 Unprocessable Entity` (validation), `429 Too Many Requests`
- `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`

### Q127. 200, 201 এবং 204 এর পার্থক্য কী?

- **200 OK:** সফল, সাধারণত response body সহ (GET, PUT, PATCH)।
- **201 Created:** নতুন resource **তৈরি হয়েছে** (POST)। ভালো প্র্যাকটিস: নতুন resource body-তে দিন এবং `Location` header দিন।
- **204 No Content:** সফল, কিন্তু **body নেই** (সাধারণত DELETE-এর পর)।

### Q128. 400, 401, 403, 404 এবং 500 এর পার্থক্য কী?

- **400:** request-এর data ভুল বা অসম্পূর্ণ (validation fail)।
- **401:** login করা নেই বা token অবৈধ।
- **403:** login আছে, কিন্তু অনুমতি নেই।
- **404:** resource বা route পাওয়া যায়নি।
- **500:** server-এর ভেতরে অপ্রত্যাশিত সমস্যা (bug, DB down)।

4xx মানে client ঠিক করবে, 5xx মানে server-এর developer ঠিক করবে।

### Q129. Path parameter এবং query parameter এর পার্থক্য কী?

- **Path parameter:** URL-এর অংশ, **নির্দিষ্ট একটা resource চিহ্নিত** করে। `/users/42` (42 নম্বর user)। এটা আবশ্যিক।
- **Query parameter:** `?` এর পরে, **ঐচ্ছিক**, filter/sort/pagination-এর জন্য। `/users?role=admin&page=2&sort=-createdAt`।

### Q130. Request-body validation কী এবং কেন গুরুত্বপূর্ণ?

Data প্রসেস করার আগে যাচাই করা যে সব required field আছে কি না, type, format ও সীমা ঠিক আছে কি না।

**কেন জরুরি:** নিরাপত্তা (injection, mass assignment ঠেকানো), data-র গুণগত মান বজায় রাখা, server crash এড়ানো, এবং client-কে স্পষ্ট error message দেওয়া। Joi, Zod বা express-validator ব্যবহার করুন এবং ভুল হলে `400` বা `422` দিন। Client-side validation যথেষ্ট নয়, server-এ validation **অবশ্যই** লাগবে।

### Q131. API versioning কী এবং কীভাবে করা যায়?

API-তে breaking change আনতে হলে পুরনো client যেন ভেঙে না পড়ে, সেজন্য আলাদা version রাখা।

1. **URI versioning (সবচেয়ে সাধারণ):** `/api/v1/users`, `/api/v2/users`
2. **Header versioning:** `Accept: application/vnd.myapp.v2+json`
3. **Query versioning:** `/users?version=2`

```js
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);
```

পুরনো version কবে বন্ধ হবে তা (deprecation policy) আগে থেকে জানান।

### Q132. Consistent API response structure কীভাবে ডিজাইন করবেন?

সব endpoint-এ একই ধরনের কাঠামো রাখুন, যাতে frontend সহজে handle করতে পারে।

```json
// সফল
{ "success": true, "message": "Users fetched", "data": [...],
  "meta": { "page": 1, "limit": 10, "total": 52 } }

// ব্যর্থ
{ "success": false, "message": "Validation failed",
  "errors": [{ "field": "email", "message": "Invalid email" }] }
```

ঠিক status code-ও দিন (সব সময় `200` দিয়ে body-তে error পাঠাবেন না)।

### Q133. API-তে pagination, filtering এবং sorting কী?

বড় data একবারে না পাঠিয়ে ভাগ করে পাঠানো ও প্রয়োজনমতো বাছাই করা:

- **Pagination:** `?page=2&limit=10` (offset-based) অথবা cursor-based (`?cursor=lastId`)
- **Filtering:** `?status=paid&category=phone`
- **Sorting:** `?sort=-price` (`-` মানে descending)

Response-এ `meta` দিন (total, page, totalPages), `limit`-এর সর্বোচ্চ সীমা রাখুন এবং কোন field-এ filter/sort চলবে তার whitelist রাখুন। (কোড-উদাহরণ Q56 ও Q78-এ আছে)

### Q134. Idempotency কী এবং কোন HTTP method গুলো idempotent হওয়ার কথা?

Idempotent মানে একই request একবার দিলেও যা ফল, বহুবার দিলেও server-এর অবস্থা সেই একই থাকবে।

- **Idempotent:** GET, PUT, DELETE, HEAD, OPTIONS
- **Idempotent নয়:** POST (বারবার দিলে বারবার নতুন resource তৈরি হয়)। PATCH-ও নিশ্চিতভাবে নয়।

**কেন গুরুত্বপূর্ণ:** network সমস্যায় client retry করলে যেন দুইবার payment বা order না হয়। তাই POST-এ `Idempotency-Key` header ব্যবহার করে একই key-র দ্বিতীয় request-কে আগের response-ই ফেরত দেওয়া হয়।

### Q135. API-তে authentication এবং authorization এর পার্থক্য কী?

Q101-এর মতোই, তবে API-তে বাস্তবে:

- **Authentication:** token/session যাচাই, ব্যর্থ হলে `401`।
- **Authorization:** role বা ownership যাচাই, ব্যর্থ হলে `403`।

শুধু role দেখলেই হয় না, **ownership**-ও দেখতে হয়। যেমন `GET /orders/123` ডাকলে order-টা এই user-এরই কি না যাচাই করুন, নইলে একজন অন্যজনের data দেখে ফেলবে (IDOR/BOLA vulnerability)।

### Q136. CORS কী এবং browser কেন এটা বলবৎ করে?

ব্রাউজারে **Same-Origin Policy** আছে: এক origin (protocol + domain + port) থেকে চলা JavaScript অন্য origin-এর response পড়তে পারে না। এটা user-কে ক্ষতিকর সাইট থেকে রক্ষা করে (যেমন আপনার ব্যাংক সাইটের লগইন থাকা অবস্থায় অন্য সাইট যেন আপনার data পড়তে না পারে)।

**CORS** হলো এই নিয়মে নিয়ন্ত্রিত ছাড় দেওয়ার উপায়। Server response-এ `Access-Control-Allow-Origin` header দিয়ে জানায় কোন origin অনুমতি পাবে। কিছু request-এর আগে ব্রাউজার `OPTIONS` **preflight** পাঠিয়ে অনুমতি যাচাই করে।

মনে রাখবেন: CORS শুধু **ব্রাউজার** মানে। Postman, curl বা অন্য server CORS-এ আটকায় না, তাই এটা security-র বিকল্প নয়।

### Q137. Rate limiting কী এবং public API-তে কেন ব্যবহার করা উচিত?

Rate limiting হলো একজন client নির্দিষ্ট সময়ে সর্বোচ্চ কত request করতে পারবে তার সীমা বেঁধে দেওয়া (যেমন ১ মিনিটে ১০০টা)।

**কেন:** DoS ও scraping ঠেকানো, brute-force (login) আটকানো, server ও খরচ নিয়ন্ত্রণে রাখা, সবার জন্য ন্যায্য ব্যবহার নিশ্চিত করা। সীমা পেরোলে `429 Too Many Requests` ও `Retry-After` header দিন।

```js
app.use(rateLimit({ windowMs: 60 * 1000, max: 100 }));
```

### Q138. Swagger বা OpenAPI দিয়ে API documentation কীভাবে করবেন?

**OpenAPI** হলো API বর্ণনার একটা standard (YAML/JSON)। **Swagger UI** সেই বর্ণনা থেকে ইন্টারঅ্যাকটিভ documentation পেজ বানায়, যেখানে ব্রাউজার থেকেই API পরীক্ষা করা যায়। Express-এ `swagger-jsdoc` (comment থেকে spec বানায়) ও `swagger-ui-express` (পেজ দেখায়) ব্যবহার হয়।

```js
const swaggerUi = require('swagger-ui-express');
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

Spec-এ endpoint, parameter, request/response schema, auth পদ্ধতি লেখা থাকে। এতে frontend টিমের সাথে সমন্বয় সহজ হয়।

### Q139. Order তৈরি ও পরিচালনার জন্য API কীভাবে ডিজাইন করবেন?

| Endpoint | কাজ | অ্যাক্সেস |
| --- | --- | --- |
| `POST /api/v1/orders` | নতুন order (`201`) | logged-in user |
| `GET /api/v1/orders` | নিজের order list (pagination, filter সহ) | user (admin সব দেখে) |
| `GET /api/v1/orders/:id` | একটা order-এর বিস্তারিত | মালিক বা admin |
| `PATCH /api/v1/orders/:id/status` | status বদল (processing, shipped...) | admin |
| `POST /api/v1/orders/:id/cancel` | order বাতিল | মালিক (শর্ত সাপেক্ষে) |

**গুরুত্বপূর্ণ ডিজাইন সিদ্ধান্ত:**

- Client থেকে শুধু `productId` ও `quantity` নিন; **দাম ও মোট টাকা server-এ হিসাব** করুন (client-এর পাঠানো দাম বিশ্বাস করবেন না)।
- Stock যাচাই ও কমানো, order তৈরি, একসাথে transaction-এ (Q75, Q98)।
- `Idempotency-Key` দিয়ে দুইবার order রোধ।
- Status-এর জন্য নির্দিষ্ট enum ও অনুমোদিত পরিবর্তনের ক্রম (pending → paid → shipped → delivered)।
- Validation, authentication, ownership check, pagination।

### Q140. API error-এ ভেতরের তথ্য ফাঁস না করে কীভাবে error handle করবেন?

- Client-কে **সাধারণ ও নিরাপদ message** দিন (`500`-এ "Something went wrong")।
- আসল error, stack trace ও সংশ্লিষ্ট তথ্য **server-এর log-এ** রাখুন, সাথে একটা **request ID** যা client-কেও দেওয়া যায়, যাতে সাপোর্টে সেই ID দিয়ে খুঁজে পাওয়া যায়।
- Production-এ stack trace, SQL query, file path, DB নাম বা library version কখনও response-এ পাঠাবেন না।
- Operational error (যেমন "email already exists") আর programmer error (bug) আলাদা করুন: প্রথমটার message client-কে দেখানো যায়, দ্বিতীয়টা নয়।

```js
app.use((err, req, res, next) => {
  logger.error({ err, requestId: req.id });
  const status = err.statusCode || 500;
  res.status(status).json({
    success: false,
    message: err.isOperational ? err.message : 'Something went wrong',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
});
```

## ৮. Backend Security (Q141–160)

### Q141. Node.js application-এর সবচেয়ে সাধারণ security ঝুঁকিগুলো কী কী?

- **Injection:** SQL, NoSQL, command injection
- **Broken authentication:** দুর্বল password নীতি, token-এর ভুল ব্যবহার
- **Broken access control:** অন্যের data দেখা বা বদলানো (IDOR), role যাচাই না করা
- **XSS ও CSRF**
- **Sensitive data exposure:** password, token, secret ফাঁস; HTTPS না থাকা
- **Security misconfiguration:** debug mode চালু, ডিফল্ট credential, খোলা CORS
- **Vulnerable dependencies:** পুরনো বা ক্ষতিকর npm package
- **Mass assignment, SSRF, ReDoS, prototype pollution, DoS**
- **অপর্যাপ্ত logging ও monitoring**

এগুলো মোটামুটি **OWASP Top 10**-এর তালিকার সাথে মেলে।

### Q142. SQL injection কী এবং কীভাবে ঠেকাবেন?

User-এর input সরাসরি SQL query-র সাথে জুড়ে দিলে attacker query-র অর্থই বদলে দিতে পারে।

```js
// ঝুঁকিপূর্ণ
const q = `SELECT * FROM users WHERE email = '${email}' AND password = '${pw}'`;
// email = ' OR 1=1 --  দিলে login ছাড়াই ঢুকে যাবে

// নিরাপদ: parameterized query
const r = await pool.query('SELECT * FROM users WHERE email = $1', [email]);
```

**প্রতিরোধ:** parameterized query/prepared statement, ORM (Sequelize, Prisma) ঠিকভাবে ব্যবহার, input validation, DB user-কে সর্বনিম্ন অনুমতি (least privilege) দেওয়া, এবং error-এ SQL না দেখানো।

### Q143. NoSQL injection কী এবং MongoDB query-কে কীভাবে প্রভাবিত করে?

MongoDB query-তে operator (`$ne`, `$gt`, `$where`) থাকে। Attacker string-এর বদলে object পাঠিয়ে query-র অর্থ বদলে দিতে পারে।

```js
// ঝুঁকিপূর্ণ: req.body সরাসরি query-তে
User.findOne({ email: req.body.email, password: req.body.password });
// attacker পাঠালো: { "email": "admin@x.com", "password": { "$ne": null } }
// ফল: password না জেনেই login!
```

**প্রতিরোধ:**

- Input-এর **type** যাচাই করুন (email ও password অবশ্যই string)
- `express-mongo-sanitize` দিয়ে `$` ও `.` দেওয়া key সরান
- Mongoose schema ও `strictQuery` ব্যবহার করুন
- `$where` ও অযাচাই করা JavaScript query এড়ান
- Password আগে `bcrypt.compare` দিয়ে মিলান, query-তে নয়

### Q144. Cross-site scripting (XSS) কী?

XSS হলো এমন আক্রমণ যেখানে attacker ক্ষতিকর JavaScript কোনো পেজে ঢুকিয়ে দেয় এবং সেটা অন্য user-এর ব্রাউজারে চলে। এতে cookie/token চুরি, user-এর হয়ে কাজ করা বা ভুয়া ফর্ম দেখানো সম্ভব।

**ধরন:** Stored (DB-তে জমা হওয়া comment-এ script), Reflected (URL-এর input সাথে সাথে পেজে), DOM-based।

**প্রতিরোধ:**

- Output **escape/encode** করা (React ডিফল্টে করে; `dangerouslySetInnerHTML` এড়ান)
- HTML গ্রহণ করতে হলে `sanitize-html`/DOMPurify দিয়ে পরিষ্কার করা
- **Content-Security-Policy** header (Helmet দিয়ে)
- Token/cookie `httpOnly` রাখা
- Input validation

### Q145. Cross-site request forgery (CSRF) কী?

User কোনো সাইটে login করা থাকা অবস্থায় attacker-এর সাইট তার ব্রাউজার দিয়ে ওই সাইটে লুকিয়ে request পাঠায়। যেহেতু ব্রাউজার **cookie নিজে থেকেই** পাঠায়, server ভাবে এটা আসল user-এর কাজ (যেমন টাকা পাঠানো বা email বদলানো)।

**প্রতিরোধ:**

- `SameSite=Lax/Strict` cookie
- CSRF token (প্রতিটা form/request-এ একটা গোপন token)
- `Origin`/`Referer` header যাচাই
- Token `Authorization` header-এ পাঠানো, কারণ ব্রাউজার এটা নিজে থেকে পাঠায় না

CSRF মূলত **cookie-ভিত্তিক** authentication-এর সমস্যা।

### Q146. CORS কী এবং এটা কি authentication-এর বিকল্প?

**না, একেবারেই নয়।** CORS শুধু ব্রাউজারে কাজ করা একটা নিয়ম যা ঠিক করে কোন website-এর JavaScript আপনার API-র response পড়তে পারবে। Postman, curl, script বা অন্য server CORS মানে না, তাই যে কেউ সরাসরি আপনার API ডাকতে পারে। তাই authentication, authorization ও validation সবসময় আলাদাভাবে লাগবে। (বিস্তারিত Q136)

### Q147. Rate limiting কীভাবে API-কে রক্ষা করে?

একজন client-এর request সংখ্যা সীমিত করে:

- Brute-force (password/OTP অনুমান) আটকায়
- DoS ও ক্রমাগত scraping কমায়
- Server ও খরচের উপর চাপ ঠেকায়

**ভালো প্র্যাকটিস:** সবার জন্য সাধারণ সীমা, আর সংবেদনশীল route-এ (login, OTP, password reset) অনেক কড়া সীমা। IP, user ID বা API key ধরে সীমা নির্ধারণ করুন। Reverse proxy-র পেছনে থাকলে `app.set('trust proxy', 1)` দিন। একাধিক server হলে Redis store ব্যবহার করুন।

```js
app.use('/api/auth/login', rateLimit({ windowMs: 15 * 60 * 1000, max: 5 }));
```

### Q148. Express-এ Helmet কী এবং এটা কী রক্ষা করতে সাহায্য করে?

Helmet হলো একটা middleware যা অনেকগুলো নিরাপদ **HTTP security header** একসাথে সেট করে:

- `Content-Security-Policy`: XSS ও অনুমোদনহীন resource লোড ঠেকায়
- `Strict-Transport-Security` (HSTS): ব্রাউজারকে সবসময় HTTPS ব্যবহার করতে বাধ্য করে
- `X-Content-Type-Options: nosniff`: MIME sniffing বন্ধ করে
- `X-Frame-Options`/`frame-ancestors`: clickjacking ঠেকায়
- `Referrer-Policy`, আর `X-Powered-By` header লুকায়

```js
app.use(helmet());
```

Helmet একা সম্পূর্ণ নিরাপত্তা দেয় না, এটা কেবল একটা স্তর।

### Q149. Password কেন শক্তিশালী password-hashing algorithm দিয়ে hash করা উচিত?

SHA-256 বা MD5-এর মতো সাধারণ hash খুব **দ্রুত**, আধুনিক GPU সেকেন্ডে শত কোটি অনুমান করতে পারে। DB leak হলে এগুলো দিয়ে hash করা password খুব দ্রুত ভেঙে ফেলা যায়।

**bcrypt, argon2, scrypt** ইচ্ছাকৃতভাবে ধীর ও resource-ঘন, **আলাদা salt** ব্যবহার করে (rainbow table অকেজো), এবং cost বাড়িয়ে সময়ের সাথে শক্তিশালী রাখা যায়। ফলে leak হলেও প্রতিটা password ভাঙা অত্যন্ত ব্যয়বহুল হয়ে যায়।

### Q150. API key, database credential ও JWT secret কীভাবে সংরক্ষণ করা উচিত?

- **কোডে বা git-এ কখনও নয়।** `.env` ফাইল `.gitignore`-এ রাখুন।
- Environment variable বা **secret manager** (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager, Doppler) ব্যবহার করুন।
- প্রতিটা environment (dev, staging, production)-এর জন্য আলাদা secret।
- JWT secret দীর্ঘ ও random হবে (কমপক্ষে ৩২ বাইট): `crypto.randomBytes(32).toString('hex')`
- নিয়মিত **rotate** করুন, আর ফাঁস হলে সঙ্গে সঙ্গে বদলান।
- Log-এ কখনও secret ছাপবেন না; `gitleaks`-এর মতো টুল দিয়ে repo স্ক্যান করুন।
- DB user-কে শুধু দরকারি অনুমতি দিন।

### Q151. Input validation কী এবং server-এ কেন করতে হবে?

Input validation হলো আসা data আমাদের প্রত্যাশামতো কি না যাচাই করা: type, দৈর্ঘ্য, format, অনুমোদিত মান। সবচেয়ে নিরাপদ পদ্ধতি **whitelist**: যা অনুমোদিত শুধু তা-ই নেওয়া, বাকি সব বাদ।

Client-side validation শুধু UX-এর জন্য। Attacker DevTools, Postman বা script দিয়ে সহজে সেটা এড়িয়ে সরাসরি API-তে যা খুশি পাঠাতে পারে। তাই **server-side validation বাধ্যতামূলক** (Joi, Zod, express-validator)।

### Q152. Client থেকে পাঠানো data কেন বিশ্বাস করা উচিত নয়?

Client-এর পাঠানো **সবকিছু** জাল করা যায়: body, query, header, cookie, hidden field, ফাইলের নাম ও type, এমনকি token-এর বাইরের সব দাবি।

- Order-এ client-এর পাঠানো `price` বিশ্বাস করবেন না, DB থেকে নিন।
- Body-র `userId` বিশ্বাস করবেন না, token থেকে আসা `req.user.id` ব্যবহার করুন।
- Body-র `role` বিশ্বাস করবেন না।
- ফাইলের `Content-Type` header বিশ্বাস করবেন না।

নীতি হলো: **Never trust, always verify।**

### Q153. Mass-assignment vulnerability কীভাবে ঠেকাবেন?

Request body সরাসরি model-এ বসিয়ে দিলে attacker এমন field পাঠাতে পারে যা তার পাঠানোর কথা নয়।

```js
// ঝুঁকিপূর্ণ: attacker { ..., "role": "admin" } পাঠালেই admin!
await User.create(req.body);

// নিরাপদ: শুধু অনুমোদিত field নিন
const { name, email, password } = req.body;
await User.create({ name, email, password });
```

**প্রতিরোধ:** allow-list (শুধু দরকারি field নেওয়া), validation schema-তে অজানা field বাতিল করা (`.strict()`), sensitive field (`role`, `isAdmin`, `isVerified`) শুধু server থেকে সেট করা, আলাদা admin endpoint রাখা।

### Q154. File upload কীভাবে নিরাপদে handle করবেন?

- **Size ও সংখ্যা** সীমিত করুন।
- **Extension ও MIME type whitelist** করুন এবং আসল content যাচাই করুন (magic bytes, `file-type` library)।
- ফাইলের **নাম নিজে random** বানান (UUID); user-এর দেওয়া নাম বিশ্বাস করবেন না (path traversal: `../../etc`)।
- Web root-এর **বাইরে** বা cloud storage-এ (S3, Cloudinary) রাখুন, execute permission দেবেন না।
- সম্ভব হলে virus scan, ছবি হলে re-process (resize/re-encode)।
- Private ফাইল দেখাতে authentication চেক করুন এবং সঠিক `Content-Type`/`Content-Disposition` দিন।

### Q155. Brute-force login attack কীভাবে ঠেকাবেন?

- Login route-এ **rate limiting** এবং ব্যর্থ চেষ্টার পর **সাময়িক lockout** বা ক্রমে বাড়তে থাকা delay
- বারবার ব্যর্থ হলে **CAPTCHA**
- **MFA/2FA**
- শক্তিশালী password নীতি ও ফাঁস হওয়া password-এর তালিকার সাথে মিলিয়ে বাতিল
- Generic error message ("email বা password ভুল")
- ধীর hashing (bcrypt/argon2)
- সন্দেহজনক চেষ্টা log করা ও সতর্কতা পাঠানো

### Q156. API error response-এ কোন তথ্য প্রকাশ করা উচিত নয়?

- **Stack trace** ও internal error message
- **SQL query বা DB-র ভেতরের error**, collection/table-এর নাম
- **File path**, server বা internal IP, library ও framework-এর version
- **Environment variable, secret, token**
- **User enumeration:** "এই email registered নেই" বা "password ভুল" আলাদা করে জানানো, কারণ attacker তখন কোন email সঠিক তা জানতে পারে

ভেতরের বিবরণ log-এ রাখুন, client-কে সাধারণ message ও request ID দিন (Q140)।

### Q157. Production application-এ HTTPS কেন ব্যবহার করা উচিত?

HTTPS-এ TLS দিয়ে traffic encrypt হয়। এতে:

- **গোপনীয়তা:** মাঝপথে কেউ (public Wi-Fi, ISP) password, token বা data পড়তে পারে না
- **অখণ্ডতা:** পথে data বদলানো যায় না
- **সত্যতা:** আপনি সত্যিই আসল server-এর সাথে কথা বলছেন (MITM ঠেকায়)

`Secure` cookie, HTTP/2, service worker এবং অনেক browser feature HTTPS ছাড়া কাজ করে না। Let's Encrypt বা Cloudflare দিয়ে বিনামূল্যে পাওয়া যায়। HTTP থেকে HTTPS-এ redirect করুন ও HSTS চালু রাখুন।

### Q158. Secure, HttpOnly এবং SameSite cookie flag কী?

- **Secure:** cookie শুধু HTTPS-এ পাঠানো হবে।
- **HttpOnly:** JavaScript (`document.cookie`) দিয়ে পড়া যাবে না, তাই XSS হলেও token চুরি কঠিন।
- **SameSite:** অন্য সাইট থেকে আসা request-এ cookie যাবে কি না ঠিক করে (`Strict`, `Lax`, `None`), CSRF ঠেকাতে সাহায্য করে।

```js
res.cookie('refreshToken', token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
  maxAge: 7 * 24 * 60 * 60 * 1000,
});
```

### Q159. Dependency vulnerability কীভাবে Node.js application-কে প্রভাবিত করে?

একটা Node প্রজেক্টে নিজের লেখা ছাড়াও শতশত (transitive সহ) npm package থাকে। এর যেকোনো একটাতে দুর্বলতা বা ক্ষতিকর code (supply chain attack, typosquatting) থাকলে সেটা সরাসরি আপনার app-এর দুর্বলতা হয়ে যায়।

**প্রতিরোধ:**

- `npm audit` নিয়মিত চালান, CI-তে রাখুন
- Dependabot/Renovate/Snyk দিয়ে স্বয়ংক্রিয় update
- `package-lock.json` commit করুন, CI/production-এ `npm ci` ব্যবহার করুন
- অপ্রয়োজনীয় package কম রাখুন; নতুন package নেওয়ার আগে জনপ্রিয়তা, maintenance ও নাম যাচাই করুন
- পরিত্যক্ত package এড়ান

### Q160. Backend API deploy করার আগে কোন নিরাপত্তা যাচাইগুলো করবেন?

**Pre-deploy checklist:**

- [ ] সব route-এ authentication ও authorization (ownership সহ) আছে
- [ ] সব input server-এ validate করা, mass assignment বন্ধ, NoSQL/SQL injection প্রতিরোধ
- [ ] Password `bcrypt`/`argon2` দিয়ে hash করা
- [ ] সব secret environment variable/secret manager-এ, git-এ কিছু নেই
- [ ] `npm audit` পরিষ্কার, dependency update করা
- [ ] `helmet()` চালু, CORS-এ নির্দিষ্ট origin whitelist
- [ ] Rate limiting (বিশেষ করে login/OTP/reset)
- [ ] HTTPS ও HSTS, cookie-তে `Secure`, `HttpOnly`, `SameSite`
- [ ] Body size limit সেট করা, file upload-এ size/type/নাম যাচাই
- [ ] Production-এ `NODE_ENV=production`, error-এ stack trace/ভেতরের তথ্য নেই
- [ ] Logging ও monitoring/alert চালু, লগে সংবেদনশীল তথ্য নেই
- [ ] DB-তে least-privilege user, network সীমিত, backup আছে
- [ ] Debug endpoint ও test ডেটা সরানো
- [ ] Security test, code review ও (সম্ভব হলে) penetration test

---

*শুভকামনা! উত্তরগুলো মুখস্থ না করে বুঝে নিজের ভাষায় বলার অভ্যাস করুন, আর সম্ভব হলে প্রতিটা বিষয়ে ছোট একটা কোড নিজে চালিয়ে দেখুন। ইন্টারভিউয়াররা সবচেয়ে বেশি পছন্দ করেন বাস্তব উদাহরণ সহ ব্যাখ্যা।*
