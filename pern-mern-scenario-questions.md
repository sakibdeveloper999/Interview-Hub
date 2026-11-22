# 50 Scenario-Based PERN & MERN Full-Stack Questions

Each question includes: **Scenario** → **How to Communicate the Answer** (what to say in an interview) → **Explanation** → **Code Example**.

Stack tags: 🟢 MERN (MongoDB) · 🔵 PERN (PostgreSQL) · ⚪ Both

---

## Section 1 — React & Frontend Architecture

### 1. ⚪ A component re-renders on every keystroke and the UI lags. How do you fix it?

**Communicate it as:** "I'd first profile with React DevTools to confirm it's a rendering issue, not a network one, then isolate what's re-rendering and why."

**Explanation:** Uncontrolled re-renders usually come from creating new object/function references on every render, or from a parent re-rendering and dragging children with it. Fix with `useMemo`, `useCallback`, `React.memo`, and by debouncing the input handler itself.

```jsx
import { useState, useCallback, memo } from "react";
import { debounce } from "lodash";

const SearchResults = memo(function SearchResults({ query }) {
  // only re-renders when `query` actually changes
  return <div>Results for: {query}</div>;
});

function SearchBox() {
  const [query, setQuery] = useState("");

  const debouncedSetQuery = useCallback(
    debounce((val) => setQuery(val), 300),
    []
  );

  return (
    <>
      <input onChange={(e) => debouncedSetQuery(e.target.value)} />
      <SearchResults query={query} />
    </>
  );
}
```

---

### 2. ⚪ Product wants a form with 15 fields, some conditionally shown, with validation. How do you architect it?

**Communicate it as:** "I'd avoid manual `useState` per field — I'd reach for a form library so validation, dirty-state, and re-renders are handled predictably, and keep conditional fields declarative."

**Explanation:** `react-hook-form` + `zod` (or `yup`) keeps validation schema-driven and out of JSX, and only re-renders the field that changed (uncontrolled inputs under the hood).

```jsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const schema = z.object({
  employmentType: z.enum(["employed", "freelance"]),
  companyName: z.string().optional(),
}).refine(
  (data) => data.employmentType !== "employed" || !!data.companyName,
  { message: "Company name required", path: ["companyName"] }
);

function JobForm() {
  const { register, watch, handleSubmit, formState: { errors } } =
    useForm({ resolver: zodResolver(schema) });
  const employmentType = watch("employmentType");

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <select {...register("employmentType")}>
        <option value="employed">Employed</option>
        <option value="freelance">Freelance</option>
      </select>
      {employmentType === "employed" && (
        <input {...register("companyName")} placeholder="Company name" />
      )}
      {errors.companyName && <p>{errors.companyName.message}</p>}
    </form>
  );
}
```

---

### 3. ⚪ A dashboard shows stale data after a mutation (e.g., editing a task doesn't update the list). How do you fix it?

**Communicate it as:** "This is a cache-invalidation problem. Instead of manually re-fetching, I'd let a data-fetching library own the cache and invalidate the right query key after the mutation succeeds."

**Explanation:** `@tanstack/react-query` (or RTK Query) separates server state from UI state, so mutations can invalidate exactly the queries that depend on that data.

```jsx
import { useMutation, useQueryClient, useQuery } from "@tanstack/react-query";
import axios from "axios";

function TaskList() {
  const queryClient = useQueryClient();

  const { data: tasks } = useQuery({
    queryKey: ["tasks"],
    queryFn: () => axios.get("/api/tasks").then(r => r.data),
  });

  const updateTask = useMutation({
    mutationFn: (task) => axios.put(`/api/tasks/${task.id}`, task),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ["tasks"] }),
  });

  return tasks?.map(t => (
    <div key={t.id} onClick={() => updateTask.mutate({ ...t, done: true })}>
      {t.title}
    </div>
  ));
}
```

---

### 4. ⚪ A page fetches user data, posts, and comments — currently three sequential `fetch` calls, making it feel slow. How do you speed it up?

**Communicate it as:** "If the requests aren't dependent on each other, they should run in parallel, not sequentially. I'd check the data dependency graph first."

**Explanation:** `Promise.all` (or parallel React Query hooks) fires independent requests concurrently instead of waiting on each one.

```js
async function loadProfilePage(userId) {
  const [user, posts, comments] = await Promise.all([
    fetch(`/api/users/${userId}`).then(r => r.json()),
    fetch(`/api/users/${userId}/posts`).then(r => r.json()),
    fetch(`/api/users/${userId}/comments`).then(r => r.json()),
  ]);
  return { user, posts, comments };
}
```

---

### 5. ⚪ Your app's initial bundle is 3MB and first load is slow. What's your plan?

**Communicate it as:** "I'd start by measuring with a bundle analyzer rather than guessing, then attack the biggest wins: route-based code splitting, lazy-loading heavy libraries, and checking for duplicate dependencies."

**Explanation:** `React.lazy` + `Suspense` for route splitting; dynamic `import()` for heavy one-off libraries (e.g., a charting library used on one page).

```jsx
import { lazy, Suspense } from "react";

const AdminDashboard = lazy(() => import("./pages/AdminDashboard"));

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <AdminDashboard />
    </Suspense>
  );
}
```

---

## Section 2 — Node.js / Express API Design

### 6. ⚪ Your Express API needs to support both web and mobile clients with different auth needs (cookies vs. tokens). How do you structure it?

**Communicate it as:** "I'd keep the auth strategy pluggable via middleware, so both flows funnel into the same `req.user`, rather than branching logic all through the route handlers."

**Explanation:** A single middleware checks for a JWT in the Authorization header first, falls back to a session cookie, and normalizes the result.

```js
async function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;
  try {
    if (authHeader?.startsWith("Bearer ")) {
      const token = authHeader.split(" ")[1];
      req.user = jwt.verify(token, process.env.JWT_SECRET);
    } else if (req.session?.userId) {
      req.user = await User.findById(req.session.userId);
    } else {
      return res.status(401).json({ error: "Unauthorized" });
    }
    next();
  } catch (err) {
    res.status(401).json({ error: "Invalid token" });
  }
}
```

---

### 7. ⚪ An endpoint occasionally crashes the whole server on bad input. How do you make the API resilient?

**Communicate it as:** "Unhandled errors and unvalidated input are the two usual culprits. I'd add centralized error handling and input validation at the edge, so a bad request never takes down the process."

**Explanation:** Centralized Express error middleware + `async` wrapper prevents unhandled promise rejections from crashing Node; validation (e.g., `zod`/`joi`) rejects malformed input before it reaches business logic.

```js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.post("/api/orders", asyncHandler(async (req, res) => {
  const parsed = orderSchema.safeParse(req.body);
  if (!parsed.success) {
    return res.status(400).json({ errors: parsed.error.flatten() });
  }
  const order = await Order.create(parsed.data);
  res.status(201).json(order);
}));

// Centralized error handler — must be last
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status || 500).json({ error: "Something went wrong" });
});
```

---

### 8. ⚪ Two microservices need to talk to each other, but you want to avoid tight coupling from direct HTTP calls. What are your options?

**Communicate it as:** "For synchronous request/response I'd keep REST, but for anything event-driven — like 'order placed' triggering an email — I'd decouple with a message queue so services don't need to know about each other directly."

**Explanation:** Message queues (RabbitMQ, Redis pub/sub, Kafka) let a producer emit an event without knowing who consumes it.

```js
// Producer (order service)
const redisClient = require("./redisClient");
async function placeOrder(order) {
  await Order.create(order);
  await redisClient.publish("order.created", JSON.stringify(order));
}

// Consumer (email service)
redisSubscriber.subscribe("order.created");
redisSubscriber.on("message", (channel, message) => {
  const order = JSON.parse(message);
  sendOrderConfirmationEmail(order);
});
```

---

### 9. ⚪ Clients are complaining about inconsistent error response shapes across your API. How do you standardize it?

**Communicate it as:** "I'd define one error response contract and funnel every error — validation, auth, not-found, server — through it, using custom error classes instead of ad hoc `res.status().json()` calls."

**Explanation:** A base `AppError` class plus subclasses keeps error creation consistent; middleware serializes them the same way every time.

```js
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}
class NotFoundError extends AppError {
  constructor(resource) { super(`${resource} not found`, 404); }
}

// usage
const task = await Task.findById(id);
if (!task) throw new NotFoundError("Task");

// error middleware
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  res.status(statusCode).json({
    error: { message: err.message, code: statusCode },
  });
});
```

---

### 10. ⚪ You need rate limiting on a public API to prevent abuse, but some partners need higher limits. How do you design it?

**Communicate it as:** "I'd apply a default rate limit globally by IP or API key, then override it per-tier for trusted partners, rather than hardcoding one limit everywhere."

**Explanation:** `express-rate-limit` (backed by Redis in production for multi-instance consistency) with a dynamic `max` based on the requester's tier.

```js
const rateLimit = require("express-rate-limit");
const RedisStore = require("rate-limit-redis");

const limiter = rateLimit({
  store: new RedisStore({ client: redisClient }),
  windowMs: 15 * 60 * 1000,
  max: (req) => (req.apiKeyTier === "partner" ? 1000 : 100),
  keyGenerator: (req) => req.apiKey || req.ip,
});

app.use("/api/", limiter);
```

---

## Section 3 — MongoDB Data Modeling 🟢

### 11. 🟢 You're building a blog where posts have comments. Comments can number in the thousands. Embed or reference?

**Communicate it as:** "I'd base it on access patterns and growth, not a blanket rule — embed when the child data is small and bounded, reference when it's unbounded."

**Explanation:** Thousands of comments per post is unbounded growth — embedding would blow past MongoDB's 16MB document limit and make the post document slow to load. Reference comments instead, and store only a `commentCount` on the post for display.

```js
// Post schema
const postSchema = new mongoose.Schema({
  title: String,
  body: String,
  commentCount: { type: Number, default: 0 },
});

// Comment schema — references post
const commentSchema = new mongoose.Schema({
  postId: { type: mongoose.Schema.Types.ObjectId, ref: "Post", index: true },
  text: String,
  author: String,
});

// fetch paginated comments separately
const comments = await Comment.find({ postId }).skip(page * 20).limit(20);
```

---

### 12. 🟢 A dashboard query aggregating monthly sales per product is taking 8+ seconds. How do you diagnose and fix it?

**Communicate it as:** "First I'd run `.explain('executionStats')` to see if it's doing a collection scan, then add a compound index matching the query's filter and sort, and consider precomputing the aggregate."

**Explanation:** Missing indexes force full collection scans; a compound index on the fields used in `$match` and `$sort` fixes most slow aggregations.

```js
// Add compound index matching the query shape
db.orders.createIndex({ productId: 1, createdAt: -1 });

const salesByMonth = await Order.aggregate([
  { $match: { createdAt: { $gte: startOfYear } } },
  { $group: {
      _id: { product: "$productId", month: { $month: "$createdAt" } },
      totalSales: { $sum: "$amount" },
  }},
  { $sort: { "_id.month": 1 } },
]);
```

---

### 13. 🟢 Two users edit the same document at the same time and the second save silently overwrites the first's changes. How do you prevent this?

**Communicate it as:** "This is a lost-update problem — I'd add optimistic concurrency control with a version field, so the second write fails loudly instead of silently clobbering data."

**Explanation:** Mongoose's built-in `__v` versioning, or a custom `updatedAt` check, rejects a save if the document changed since it was read.

```js
const task = await Task.findById(id);
task.title = "New title";

try {
  await task.save(); // throws VersionError if __v changed underneath
} catch (err) {
  if (err.name === "VersionError") {
    return res.status(409).json({ error: "Task was modified by someone else, please refresh" });
  }
  throw err;
}
```

---

### 14. 🟢 You need to move money between two user "wallet" documents atomically — both updates must succeed or neither should. MongoDB isn't relational — how?

**Communicate it as:** "MongoDB does support multi-document ACID transactions since 4.0, as long as you're on a replica set, so I'd wrap both updates in a session transaction."

```js
const session = await mongoose.startSession();
session.startTransaction();
try {
  await Wallet.updateOne({ userId: fromId }, { $inc: { balance: -amount } }, { session });
  await Wallet.updateOne({ userId: toId }, { $inc: { balance: amount } }, { session });
  await session.commitTransaction();
} catch (err) {
  await session.abortTransaction();
  throw err;
} finally {
  session.endSession();
}
```

---

### 15. 🟢 Search needs to match products by name with typos and partial words (e.g., "runing shoe" → "running shoes"). How?

**Communicate it as:** "A regex `LIKE`-style search won't handle typos or ranking well, so I'd use MongoDB Atlas Search or a text index for relevance-scored full-text search."

```js
// Text index
db.products.createIndex({ name: "text", description: "text" });

const results = await Product.find(
  { $text: { $search: "running shoe" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } });
```

---

## Section 4 — PostgreSQL & SQL Design 🔵

### 16. 🔵 An e-commerce app needs orders with multiple line items, each referencing a product. How do you model this relationally?

**Communicate it as:** "This is a classic many-to-many via a join table — `order_items` — rather than an array column, so I keep referential integrity and can query line items independently."

```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  status VARCHAR(20) DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER REFERENCES orders(id) ON DELETE CASCADE,
  product_id INTEGER REFERENCES products(id),
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  unit_price NUMERIC(10,2) NOT NULL
);
```

---

### 17. 🔵 A report page is running a query that joins 5 tables and takes 12 seconds on a table with 2M rows. How do you optimize it?

**Communicate it as:** "I'd run `EXPLAIN ANALYZE` first to see where the time actually goes — a sequential scan versus an index scan tells very different stories — then index the join and filter columns, and consider a materialized view if it's a recurring report."

```sql
EXPLAIN ANALYZE
SELECT o.id, u.name, SUM(oi.quantity * oi.unit_price) AS total
FROM orders o
JOIN users u ON u.id = o.user_id
JOIN order_items oi ON oi.order_id = o.id
WHERE o.created_at > NOW() - INTERVAL '30 days'
GROUP BY o.id, u.name;

-- add indexes on FK + filter columns
CREATE INDEX idx_orders_created_at ON orders(created_at);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);

-- for a recurring dashboard query, precompute
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT date_trunc('month', o.created_at) AS month, SUM(oi.quantity * oi.unit_price) AS total
FROM orders o JOIN order_items oi ON oi.order_id = o.id
GROUP BY 1;
```

---

### 18. 🔵 You need to prevent two concurrent requests from double-booking the last seat on a flight. How do you enforce this at the DB level?

**Communicate it as:** "Application-level checks have a race condition window, so I'd push the constraint into the database — either a row lock inside a transaction or a check constraint with an atomic decrement."

```sql
BEGIN;

SELECT seats_available FROM flights WHERE id = 42 FOR UPDATE; -- locks the row

UPDATE flights
SET seats_available = seats_available - 1
WHERE id = 42 AND seats_available > 0;

-- if 0 rows affected, no seats left — rollback
COMMIT;
```

```js
// Node/pg example
await pool.query("BEGIN");
try {
  const result = await pool.query(
    `UPDATE flights SET seats_available = seats_available - 1
     WHERE id = $1 AND seats_available > 0 RETURNING *`,
    [flightId]
  );
  if (result.rowCount === 0) throw new Error("No seats available");
  await pool.query("COMMIT");
} catch (err) {
  await pool.query("ROLLBACK");
  throw err;
}
```

---

### 19. 🔵 Product data has some fields that vary wildly by category (a shirt has "size/color", a laptop has "RAM/CPU"). Pure relational columns feel wrong. What do you do?

**Communicate it as:** "I'd avoid EAV tables since they kill query performance — instead I'd use Postgres's native JSONB column for the variable attributes, while keeping core fields relational and indexable."

```sql
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  category VARCHAR(50) NOT NULL,
  price NUMERIC(10,2) NOT NULL,
  attributes JSONB
);

CREATE INDEX idx_products_attributes ON products USING GIN (attributes);

-- query products where color = 'red'
SELECT * FROM products WHERE attributes->>'color' = 'red';
```

---

### 20. 🔵 You're migrating a live production database schema (adding a NOT NULL column) without downtime. What's your approach?

**Communicate it as:** "A NOT NULL column with no default locks the whole table on an ALTER in older Postgres, so I'd break it into safe, incremental steps instead of one blocking migration."

```sql
-- Step 1: add nullable column
ALTER TABLE users ADD COLUMN phone_verified BOOLEAN;

-- Step 2: backfill in small batches (application code or scripted)
UPDATE users SET phone_verified = false WHERE phone_verified IS NULL;

-- Step 3: add the NOT NULL constraint once backfilled (Postgres 12+ validates fast if a matching CHECK already exists)
ALTER TABLE users ALTER COLUMN phone_verified SET DEFAULT false;
ALTER TABLE users ALTER COLUMN phone_verified SET NOT NULL;
```

---

## Section 5 — Authentication & Security

### 21. ⚪ How do you implement secure JWT authentication with refresh tokens in a MERN/PERN app?

**Communicate it as:** "I'd keep access tokens short-lived and stateless, and put refresh tokens in an httpOnly cookie so JavaScript can't touch them — that limits the blast radius of an XSS attack."

```js
function generateTokens(userId) {
  const accessToken = jwt.sign({ userId }, process.env.ACCESS_SECRET, { expiresIn: "15m" });
  const refreshToken = jwt.sign({ userId }, process.env.REFRESH_SECRET, { expiresIn: "7d" });
  return { accessToken, refreshToken };
}

app.post("/api/login", async (req, res) => {
  const user = await validateCredentials(req.body);
  const { accessToken, refreshToken } = generateTokens(user.id);

  res.cookie("refreshToken", refreshToken, {
    httpOnly: true, secure: true, sameSite: "strict", maxAge: 7 * 24 * 60 * 60 * 1000,
  });
  res.json({ accessToken });
});

app.post("/api/refresh", (req, res) => {
  const token = req.cookies.refreshToken;
  if (!token) return res.sendStatus(401);
  jwt.verify(token, process.env.REFRESH_SECRET, (err, payload) => {
    if (err) return res.sendStatus(403);
    const { accessToken } = generateTokens(payload.userId);
    res.json({ accessToken });
  });
});
```

---

### 22. ⚪ A pentest reveals your login endpoint is vulnerable to SQL/NoSQL injection. How do you fix it?

**Communicate it as:** "The root cause is building queries from unsanitized user input, so the fix isn't string-escaping hacks — it's always using parameterized queries or an ORM/ODM that does it for you."

```js
// PostgreSQL — vulnerable
const bad = `SELECT * FROM users WHERE email = '${email}'`; // NEVER

// PostgreSQL — safe, parameterized
const { rows } = await pool.query("SELECT * FROM users WHERE email = $1", [email]);

// MongoDB — vulnerable to operator injection if body isn't validated
// e.g. { "email": { "$gt": "" } } bypasses a naive query
const badUser = await User.findOne({ email: req.body.email }); // dangerous if unvalidated

// MongoDB — safe: validate/cast types before querying
if (typeof req.body.email !== "string") return res.status(400).end();
const user = await User.findOne({ email: req.body.email });
```

---

### 23. ⚪ You need role-based access control (admin, editor, viewer) across many routes without repeating logic. How do you structure it?

**Communicate it as:** "I'd centralize permission checks into a reusable middleware factory instead of scattering `if (user.role === ...)` checks across route handlers."

```js
function requireRole(...allowedRoles) {
  return (req, res, next) => {
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: "Forbidden" });
    }
    next();
  };
}

router.delete("/api/posts/:id", authenticate, requireRole("admin", "editor"), deletePost);
router.get("/api/posts", authenticate, requireRole("admin", "editor", "viewer"), listPosts);
```

---

### 24. ⚪ Users report their session gets logged out randomly across tabs/devices when they change their password elsewhere. Expected or bug — and how do you implement "logout everywhere"?

**Communicate it as:** "That should be intentional behavior, and I'd implement it by invalidating a token version/timestamp server-side, since JWTs can't be revoked individually once issued."

```js
// user document/table has tokenVersion
async function logoutEverywhere(userId) {
  await User.updateOne({ _id: userId }, { $inc: { tokenVersion: 1 } });
}

// verify middleware checks tokenVersion matches
async function authenticate(req, res, next) {
  const payload = jwt.verify(req.headers.authorization?.split(" ")[1], process.env.JWT_SECRET);
  const user = await User.findById(payload.userId);
  if (user.tokenVersion !== payload.tokenVersion) {
    return res.status(401).json({ error: "Session expired, please log in again" });
  }
  req.user = user;
  next();
}
```

---

### 25. ⚪ File uploads (profile pictures) are a security concern — a user could upload a malicious script disguised as an image. How do you secure this?

**Communicate it as:** "I'd never trust the client-provided MIME type or extension — I'd validate the actual file signature, limit size, strip metadata, and store uploads outside the web root or in object storage, not serve them from an executable path."

```js
const multer = require("multer");
const fileTypeChecker = require("file-type-checker");

const upload = multer({
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
  fileFilter: (req, file, cb) => {
    const allowed = ["image/jpeg", "image/png", "image/webp"];
    cb(null, allowed.includes(file.mimetype));
  },
});

app.post("/api/upload", upload.single("avatar"), async (req, res) => {
  const isValidImage = fileTypeChecker.validateFileType(req.file.buffer, ["jpg", "png", "webp"]);
  if (!isValidImage) return res.status(400).json({ error: "Invalid file" });
  // upload req.file.buffer to S3/Cloud storage, never to a public executable path
  const url = await uploadToS3(req.file.buffer, req.file.originalname);
  res.json({ url });
});
```

---

## Section 6 — Real-time Features & WebSockets

### 26. ⚪ You need to add a live chat feature to your MERN/PERN app. What's the architecture?

**Communicate it as:** "REST is request/response, so for a persistent bidirectional channel I'd add Socket.IO alongside Express, authenticate the socket handshake with the same JWT, and use rooms to scope messages per conversation."

```js
const io = require("socket.io")(server, { cors: { origin: process.env.CLIENT_URL } });

io.use((socket, next) => {
  try {
    const token = socket.handshake.auth.token;
    socket.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    next(new Error("Unauthorized"));
  }
});

io.on("connection", (socket) => {
  socket.on("join_room", (roomId) => socket.join(roomId));

  socket.on("send_message", async ({ roomId, text }) => {
    const message = await Message.create({ roomId, text, senderId: socket.user.userId });
    io.to(roomId).emit("new_message", message);
  });
});
```

---

### 27. ⚪ Your Socket.IO app works fine locally but breaks when you scale to multiple server instances behind a load balancer. Why, and how do you fix it?

**Communicate it as:** "Each instance holds its own in-memory socket connections, so a message emitted from instance A never reaches a client connected to instance B. I'd fix it with a shared adapter, like Redis, so events broadcast across all instances."

```js
const { createAdapter } = require("@socket.io/redis-adapter");
const { createClient } = require("redis");

const pubClient = createClient({ url: process.env.REDIS_URL });
const subClient = pubClient.duplicate();

await Promise.all([pubClient.connect(), subClient.connect()]);
io.adapter(createAdapter(pubClient, subClient));
```

---

### 28. ⚪ How do you show "online/offline" presence for users in real time without hammering the database?

**Communicate it as:** "Presence is high-frequency, ephemeral state, so I'd keep it in Redis (or in-memory with pub/sub) rather than writing to Postgres/Mongo on every connect/disconnect."

```js
io.on("connection", async (socket) => {
  const userId = socket.user.userId;
  await redisClient.sAdd("online_users", userId);
  io.emit("presence_update", { userId, status: "online" });

  socket.on("disconnect", async () => {
    await redisClient.sRem("online_users", userId);
    io.emit("presence_update", { userId, status: "offline" });
  });
});
```

---

### 29. ⚪ A live dashboard needs to reflect database changes made outside the app (e.g., a cron job updating stock levels) in real time. How?

**Communicate it as:** "Since the change doesn't originate from a socket event, I'd hook into the database's own change stream so any write — regardless of source — triggers a broadcast."

```js
// MongoDB change streams (MERN)
const changeStream = Product.watch();
changeStream.on("change", (change) => {
  if (change.operationType === "update") {
    io.emit("stock_updated", { productId: change.documentKey._id });
  }
});
```

```sql
-- PostgreSQL LISTEN/NOTIFY (PERN)
CREATE OR REPLACE FUNCTION notify_stock_change() RETURNS trigger AS $$
BEGIN
  PERFORM pg_notify('stock_changes', row_to_json(NEW)::text);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER stock_update_trigger
AFTER UPDATE ON products
FOR EACH ROW EXECUTE FUNCTION notify_stock_change();
```

---

### 30. ⚪ Real-time notifications need to reach a user even if they're not currently connected via WebSocket (e.g., closed the tab). How do you handle both cases?

**Communicate it as:** "I'd decouple 'create the notification' from 'deliver it live' — always persist it, then try a live push, and fall back to the client fetching unread notifications on next load."

```js
async function notifyUser(userId, payload) {
  const notification = await Notification.create({ userId, ...payload, read: false });

  const socketId = onlineUsers.get(userId);
  if (socketId) {
    io.to(socketId).emit("notification", notification);
  }
  // if offline, it's already persisted — client fetches /api/notifications?unread=true on load
}
```

---

## Section 7 — Performance & Scaling

### 31. ⚪ Your API's average response time is fine, but p99 latency is terrible. How do you investigate?

**Communicate it as:** "Averages hide tail latency, so I'd look at what's specifically slow in the p99 — often it's a handful of endpoints hitting an unindexed query, a cold cache, or GC pauses — using APM tracing rather than guessing."

**Explanation:** Tools like New Relic, Datadog APM, or even manual request timing logs with correlation IDs reveal which specific requests/queries drive the tail. Common causes: N+1 queries, missing indexes on rarely-hit filters, synchronous blocking calls (e.g., a slow third-party API called inline).

```js
// simple middleware to log slow requests for triage
app.use((req, res, next) => {
  const start = Date.now();
  res.on("finish", () => {
    const duration = Date.now() - start;
    if (duration > 1000) {
      console.warn(`SLOW: ${req.method} ${req.originalUrl} - ${duration}ms`);
    }
  });
  next();
});
```

---

### 32. ⚪ A list endpoint (`GET /api/posts`) that used to be fast is now timing out as the table grew to millions of rows. How do you fix pagination?

**Communicate it as:** "Offset pagination gets slower the deeper you paginate because the DB still scans and discards all prior rows — I'd switch to cursor-based (keyset) pagination."

```sql
-- Slow at high offsets
SELECT * FROM posts ORDER BY created_at DESC OFFSET 500000 LIMIT 20;

-- Fast: cursor-based
SELECT * FROM posts
WHERE created_at < $1  -- cursor = last seen created_at
ORDER BY created_at DESC
LIMIT 20;
```

```js
// MongoDB equivalent
const posts = await Post.find({ createdAt: { $lt: cursor } })
  .sort({ createdAt: -1 })
  .limit(20);
```

---

### 33. ⚪ Every page load hits the database for data that barely changes (e.g., category list). How do you reduce load?

**Communicate it as:** "This is a textbook caching case — I'd cache it in Redis with a sensible TTL and invalidate on write, rather than caching in application memory which won't work once we scale horizontally."

```js
async function getCategories() {
  const cached = await redisClient.get("categories");
  if (cached) return JSON.parse(cached);

  const categories = await Category.find();
  await redisClient.set("categories", JSON.stringify(categories), { EX: 3600 });
  return categories;
}

async function createCategory(data) {
  const category = await Category.create(data);
  await redisClient.del("categories"); // invalidate on write
  return category;
}
```

---

### 34. ⚪ Uploading and processing a large CSV (100k rows) inside a request handler times out the request. How do you handle it?

**Communicate it as:** "Long-running work doesn't belong inside a synchronous HTTP request — I'd accept the upload, respond immediately with a job ID, and process it asynchronously in a background worker/queue."

```js
const Queue = require("bull");
const csvQueue = new Queue("csv-processing", process.env.REDIS_URL);

app.post("/api/upload-csv", upload.single("file"), async (req, res) => {
  const job = await csvQueue.add({ filePath: req.file.path });
  res.status(202).json({ jobId: job.id, status: "processing" });
});

csvQueue.process(async (job) => {
  await processCsvRows(job.data.filePath); // runs out-of-band
});

app.get("/api/upload-status/:jobId", async (req, res) => {
  const job = await csvQueue.getJob(req.params.jobId);
  res.json({ state: await job.getState() });
});
```

---

### 35. ⚪ Your Node app is a single process and can't use more than one CPU core, and traffic is growing. How do you scale it on one machine before adding more servers?

**Communicate it as:** "Node is single-threaded per process, so before scaling out horizontally I'd scale up with the built-in cluster module (or a process manager like PM2) to use all available cores."

```js
const cluster = require("cluster");
const os = require("os");

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  for (let i = 0; i < numCPUs; i++) cluster.fork();
  cluster.on("exit", () => cluster.fork()); // restart dead workers
} else {
  require("./app"); // each worker runs the Express app
}
```

---

## Section 8 — Testing & Debugging

### 36. ⚪ How do you unit test an Express route that depends on the database, without hitting a real DB?

**Communicate it as:** "I'd isolate the unit under test by mocking the data layer, and keep real-DB tests separate as integration tests, so unit tests stay fast and deterministic."

```js
// controller
async function getUser(req, res) {
  const user = await User.findById(req.params.id);
  if (!user) return res.status(404).json({ error: "Not found" });
  res.json(user);
}

// test with jest + supertest, mocking the model
jest.mock("../models/User");
const User = require("../models/User");

test("returns 404 when user not found", async () => {
  User.findById.mockResolvedValue(null);
  const res = await request(app).get("/api/users/123");
  expect(res.status).toBe(404);
});
```

---

### 37. ⚪ A bug only happens in production and can't be reproduced locally. How do you debug it without local access to prod data?

**Communicate it as:** "I'd rely on structured logging and tracing rather than guesswork — adding correlation IDs, capturing the exact request payload (scrubbed of PII), and checking APM traces around the failure timestamp."

```js
const { v4: uuidv4 } = require("uuid");

app.use((req, res, next) => {
  req.correlationId = req.headers["x-correlation-id"] || uuidv4();
  res.setHeader("x-correlation-id", req.correlationId);
  next();
});

app.use((err, req, res, next) => {
  logger.error({
    correlationId: req.correlationId,
    message: err.message,
    stack: err.stack,
    path: req.originalUrl,
    body: req.body, // scrub sensitive fields before logging in real code
  });
  res.status(500).json({ error: "Internal error", correlationId: req.correlationId });
});
```

---

### 38. ⚪ How do you write an integration test that spins up a real (but isolated) database for each test run?

**Communicate it as:** "I'd use a containerized, ephemeral database per test suite so tests are realistic but don't pollute shared state — `testcontainers` or an in-memory equivalent."

```js
// MongoDB — mongodb-memory-server
const { MongoMemoryServer } = require("mongodb-memory-server");
let mongod;

beforeAll(async () => {
  mongod = await MongoMemoryServer.create();
  await mongoose.connect(mongod.getUri());
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongod.stop();
});
```

```js
// PostgreSQL — testcontainers
const { PostgreSqlContainer } = require("@testcontainers/postgresql");
let container, pool;

beforeAll(async () => {
  container = await new PostgreSqlContainer().start();
  pool = new Pool({ connectionString: container.getConnectionUri() });
  await pool.query(fs.readFileSync("./schema.sql", "utf8"));
});

afterAll(async () => { await pool.end(); await container.stop(); });
```

---

### 39. ⚪ A React component's tests pass but the feature breaks in the browser. What's the gap, and how do you close it?

**Communicate it as:** "Unit tests validate logic in isolation but miss integration issues — real network calls, routing, browser APIs. I'd add end-to-end tests with something like Playwright or Cypress to cover the actual user flow."

```js
// Playwright e2e example
import { test, expect } from "@playwright/test";

test("user can log in and see dashboard", async ({ page }) => {
  await page.goto("/login");
  await page.fill('input[name="email"]', "test@example.com");
  await page.fill('input[name="password"]', "password123");
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL("/dashboard");
  await expect(page.locator("h1")).toContainText("Welcome");
});
```

---

### 40. ⚪ Memory usage on your Node server climbs steadily over days until it crashes. How do you find the leak?

**Communicate it as:** "This looks like a memory leak, so I'd take heap snapshots over time under load and diff them to find what's accumulating, rather than restarting the server as a workaround."

**Explanation:** Common Node leak sources: event listeners never removed, growing in-memory caches with no eviction, closures holding references, unbounded arrays/queues.

```js
// Common leak: adding a listener on every request without cleanup
app.get("/api/stream", (req, res) => {
  const listener = () => res.write("data\n");
  eventEmitter.on("tick", listener);
  req.on("close", () => eventEmitter.removeListener("tick", listener)); // fix: clean up
});
```

```bash
# heap snapshot analysis
node --inspect app.js
# then use Chrome DevTools > Memory tab to take & diff heap snapshots
```

---

## Section 9 — Deployment & DevOps

### 41. ⚪ How do you structure environment variables so secrets never end up in your Git repo?

**Communicate it as:** "I'd keep secrets entirely out of the codebase — `.env` files gitignored locally, and real secrets injected via the hosting platform's secret manager in production."

```bash
# .gitignore
.env
.env.local
```

```js
// .env.example (committed — documents required vars, no real values)
DATABASE_URL=
JWT_SECRET=
STRIPE_SECRET_KEY=
```

```js
// app validates required env vars fail fast at boot
const required = ["DATABASE_URL", "JWT_SECRET"];
required.forEach((key) => {
  if (!process.env[key]) throw new Error(`Missing required env var: ${key}`);
});
```

---

### 42. ⚪ How do you containerize a MERN/PERN app for consistent deployment across environments?

**Communicate it as:** "I'd use multi-stage Docker builds to keep the production image small, and docker-compose locally to mirror the production topology (app + db + cache)."

```dockerfile
# Dockerfile — multi-stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports: ["3000:3000"]
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    depends_on: [db]
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: pass
    volumes: ["pgdata:/var/lib/postgresql/data"]
volumes:
  pgdata:
```

---

### 43. ⚪ How do you deploy a schema migration without breaking the currently-running version of the app during rollout?

**Communicate it as:** "I'd make migrations backward-compatible — expand first, migrate, then contract later — so the old app version keeps working during a rolling deploy instead of erroring on a missing/renamed column."

**Explanation:** Renaming a column directly breaks the old app instance mid-deploy. Instead: add new column → deploy code that writes to both → backfill → deploy code that reads only new column → drop old column in a later release.

```sql
-- Step 1 (release N): add new column, keep old
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

-- app writes to both `name` and `full_name` during transition

-- Step 2 (release N+1): backfill, switch reads to full_name
UPDATE users SET full_name = name WHERE full_name IS NULL;

-- Step 3 (release N+2): drop old column once all instances are updated
ALTER TABLE users DROP COLUMN name;
```

---

### 44. ⚪ How do you set up CI so every PR is automatically tested before merge?

**Communicate it as:** "I'd gate merges on a CI pipeline that runs lint, unit tests, and a build, and treat a red pipeline as a hard block, not a suggestion."

```yaml
# .github/workflows/ci.yml
name: CI
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: test }
        ports: ["5432:5432"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

---

### 45. ⚪ Your Node app needs zero-downtime deploys. How do you achieve that?

**Communicate it as:** "I'd deploy behind a load balancer with rolling updates — bring up new instances, health-check them, then drain and remove old ones — rather than stopping and starting the single running process."

```js
// health check endpoint the orchestrator polls before routing traffic
app.get("/health", async (req, res) => {
  try {
    await db.query("SELECT 1"); // or mongoose.connection.readyState === 1
    res.status(200).json({ status: "ok" });
  } catch {
    res.status(503).json({ status: "unhealthy" });
  }
});

// graceful shutdown — finish in-flight requests before exiting
process.on("SIGTERM", async () => {
  server.close(() => {
    db.end();
    process.exit(0);
  });
});
```

---

## Section 10 — System Design & Architecture Scenarios

### 46. ⚪ Design a URL shortener (like bit.ly) — what does the schema and core logic look like?

**Communicate it as:** "I'd focus on the two hot paths — create and redirect — and optimize redirect since it's read-heavy: short code as a primary/indexed lookup, and a cache in front of the DB."

```js
// generate a short, collision-resistant code
const { nanoid } = require("nanoid");

app.post("/api/shorten", async (req, res) => {
  const code = nanoid(7);
  await Url.create({ code, longUrl: req.body.url });
  res.json({ shortUrl: `https://short.ly/${code}` });
});

app.get("/:code", async (req, res) => {
  const cached = await redisClient.get(req.params.code);
  if (cached) return res.redirect(cached);

  const entry = await Url.findOne({ code: req.params.code });
  if (!entry) return res.status(404).send("Not found");

  await redisClient.set(req.params.code, entry.longUrl, { EX: 86400 });
  res.redirect(entry.longUrl);
});
```

---

### 47. ⚪ Design a notification system that supports email, SMS, and push, without hardcoding a channel into every feature.

**Communicate it as:** "I'd decouple 'what happened' from 'how it's delivered' using a strategy pattern per channel, driven by user preferences, so adding a new channel doesn't touch existing feature code."

```js
class EmailChannel { send(user, payload) { /* nodemailer logic */ } }
class SmsChannel { send(user, payload) { /* twilio logic */ } }
class PushChannel { send(user, payload) { /* fcm logic */ } }

const channels = { email: new EmailChannel(), sms: new SmsChannel(), push: new PushChannel() };

async function notify(userId, payload) {
  const user = await User.findById(userId);
  const enabledChannels = user.notificationPreferences; // e.g. ["email", "push"]
  await Promise.allSettled(
    enabledChannels.map((ch) => channels[ch].send(user, payload))
  );
}
```

---

### 48. ⚪ Design the backend for a multi-tenant SaaS app (each company's data must be isolated). What's your approach?

**Communicate it as:** "The three standard options are separate databases per tenant, separate schemas, or a shared schema with a `tenant_id` column — I'd pick based on tenant count and isolation requirements, and default to shared-schema-with-tenant_id for most SaaS scale."

```sql
-- shared schema approach
CREATE TABLE projects (
  id SERIAL PRIMARY KEY,
  tenant_id INTEGER NOT NULL REFERENCES tenants(id),
  name VARCHAR(255)
);
CREATE INDEX idx_projects_tenant ON projects(tenant_id);
```

```js
// enforce tenant scoping at the middleware level so it's never forgotten in a query
function scopeToTenant(req, res, next) {
  req.tenantId = req.user.tenantId;
  next();
}

app.get("/api/projects", authenticate, scopeToTenant, async (req, res) => {
  const projects = await Project.find({ tenantId: req.tenantId }); // never trust a client-passed tenantId
  res.json(projects);
});
```

---

### 49. ⚪ Design a feature flag system so product can toggle features per user segment without a redeploy.

**Communicate it as:** "I'd keep flags out of code and in a fast-lookup store, evaluated per-request against user/segment rules, so toggling doesn't need a deploy — and cache evaluations to avoid a DB hit on every request."

```js
async function isFeatureEnabled(flagKey, user) {
  const flag = await redisClient.get(`flag:${flagKey}`) 
    ?? JSON.stringify(await FeatureFlag.findOne({ key: flagKey }));
  const parsed = JSON.parse(flag);
  if (!parsed?.enabled) return false;
  if (parsed.rolloutPercentage) {
    const hash = hashUserId(user.id) % 100;
    return hash < parsed.rolloutPercentage;
  }
  return parsed.enabledSegments?.includes(user.segment) ?? true;
}

app.get("/api/dashboard", async (req, res) => {
  const showNewUI = await isFeatureEnabled("new-dashboard-ui", req.user);
  res.json({ layout: showNewUI ? "v2" : "v1" });
});
```

---

### 50. ⚪ Your PERN/MERN app needs to support offline-first behavior in the frontend (works with spotty connectivity, syncs when back online). How do you architect it?

**Communicate it as:** "I'd separate local-first writes from server sync — persist changes locally immediately for a responsive UI, queue them, and reconcile with the server (including conflict resolution) once connectivity returns."

```js
// simplified offline queue using IndexedDB (via idb-keyval) + navigator.onLine
import { get, set } from "idb-keyval";

async function saveTaskOffline(task) {
  const queue = (await get("syncQueue")) || [];
  queue.push({ ...task, localId: crypto.randomUUID(), synced: false });
  await set("syncQueue", queue);
  renderLocally(task); // optimistic UI update
}

window.addEventListener("online", async () => {
  const queue = (await get("syncQueue")) || [];
  const unsynced = queue.filter((t) => !t.synced);
  for (const task of unsynced) {
    try {
      const res = await fetch("/api/tasks", {
        method: "POST", body: JSON.stringify(task), headers: { "Content-Type": "application/json" },
      });
      task.synced = res.ok;
    } catch { /* stays unsynced, retried on next online event */ }
  }
  await set("syncQueue", queue.filter((t) => !t.synced));
});
```

---

## Quick Reference — What "Good Communication" Looks Like

Across all 50 answers, the pattern for interview communication is:

1. **Name the underlying problem first** ("this is a caching problem," "this is a race condition") before jumping to a tool.
2. **State the trade-off**, not just the solution — why this approach over an obvious alternative.
3. **Anchor in access patterns / growth / failure modes**, not memorized rules ("embed vs reference" isn't a rule, it's a judgment call).
4. **Mention how you'd verify** (profiler, `EXPLAIN ANALYZE`, heap snapshot) rather than guessing — shows debugging discipline.
5. **Keep code as the last step**, illustrating the explanation rather than replacing it.
