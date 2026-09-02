# Charter & Manifesto

## 1. Who We Are

**Unified State Labs** is an engineering and research lab working on state as a first-class problem in computing. The quest is to rederive stateful computing from first principles. The work produces a new substrate for computing with a formal, universal treatment of state—comprising new paradigms and design patterns, new protocol, language, and runtime specifications and implementations—with the infrastructure and developer tools that make it work at product scale. Engineering informs the research; research informs the engineering.

> This Lab continues the work we started at [@webqit](https://github.com/webqit) and [@linked-db](https://github.com/linked-db).
> 
## 2. What We Found

There's a significant share of computing—and of the engineering effort, whether human labour or AI spend—that goes to problems that aren't always named "state problems". They appear unrelated across the various subfields of computing—cache invalidation, sync, state management, reactivity. But underneath them all is the same shape of work: either taming state, or exposing and propagating state changes from point A and reconstructing local state from event streams at point B—with the exact nature of the work changing by the boundary: binding application state to the UI, coordinating distributed states across network or protocol boundaries. State remains the thing often unnamed but constitutes the central problem much of engineering exists to control or construct. We've named the recurring shape of work **State Engineering**.

Having historically been perceived as different problems by different people, state has yet to be established as a first-class problem in computing. The consequence is in solutions that barely talk to each other or compose, the same costs being repaid at every instance of the same problem, more engineering effort than is necessary.

The complete diagnosis is the first of the in-progress paper series: **Stateful Computing—Rederived from First Principles**.

## 3. What Changes

The core question answered by a central treatment of state is: **who accounts for what, and how**.

**In the old world:** state is accounted for by the engineer, through surface machinery, reconstructed at each instance of the same problem. State is a developer concern.

**In the new world:** state is accounted for by the system itself, through a formal protocol that applies universally. State is a system concern.

That shift recurs across the layers and boundaries of computing.

### 3.1 At the language/runtime layer

Where change detection once required special machinery around state, observability becomes a property of mutation itself. State transitions are exposed and directly observed via a shared protocol.

```js
const obj = {};
Observer.observe(obj, handler);

// state transitions 
Observer.set(obj, 'x', value);
```

With observability now being offered by the runtime, applications can maintain a unified state model with the runtime rather than maintain a separate state model through custom stores.

Beyond its realization at the object model, the same underlying protocol also powers another runtime capability. Where the relationship between state and its dependent once had to be maintained by hand, it becomes something the runtime can do by itself based on knowledge of its own transitions and of its own control flow graph. Live computations emerge as a runtime mode.

```js
"use live";
let a = 0;
console.log(a);

// state transitions 
setInterval(() => a++, 1000);
```

State management becomes a runtime-level concern—as memory management now is across most programming languages.

### 3.2 At the application layer

Where two parts of a program or system once shared a static contract—paired with a separate update path for changes—change is formalized as part of the original contract.

```js
function getLiveLocation() {
    const loc = { lng: 3.4, lat: 2.5 };
    setTimeout(() => {
        Observer.set(loc, 'lat', 2.7);
    }, 1000);
    return loc;
}

// this is a live object
const loc = getLiveLocation();
Observer.observe(loc, handler);
```

The model is held as a design pattern named **Live Contracts**.

The same design pattern applied to old problems brings state management—past and present—under a unified model.

```js
// DOM APIs as live contracts 
const result = document.querySelectorAll(css, { live: true });
Observer.observe(result, handler);
```

```js
// URL API as a live contract
const url = new URL('http://example.com/path');
Observer.observe(url, handler);
url.host = 'example2.com';
```

The same underlying protocol makes it all composable.

```js
"use live";
url.search = `?lat=${loc.lat}&lng=${loc.lng}`;
for (let el of result) {
    el.href = url.href;
}
```

With state now being accounted for by the substrate beneath the application layer, the application layer can shrink to its core domain problem.

### 3.3 Across network/protocol boundaries

Where state coordination spanning network or protocol boundaries once required a split architecture—comprising an initial request/response model and a separate update path for changes—change is formalized as part of the original model. The existing protocol is extended to support live requests or live responses where objects conceptually cross the boundary by reference.

```js
// live contracts over HTTP
const response = await fetch(url, { live: true });
const { body } = await response.now();
Observer.observe(body, handler);
```

```js
// live contracts over message channels
const data = {};
postMessage(data, { live: true });
Observer.set(data, 'x', value);
```

With state now being simply shared across boundaries rather than maintained through secondary infrastructure—Web Sockets, SSE, other event systems—state across distributed architectures becomes conceptually indistinguishable from local state; stateful third-party APIs supporting the same model (`fetch(stripeEndpoint, { live: true })`) become conceptually indistinguishable from local Live Contracts.

### 3.4 Across DSL/query boundaries

Where query languages (CSS, SQL, etc.) once constituted an opaque system boundary: "opaque string in, opaque result out"—with no easier way to stay current to the original input than periodic polling—change is formalized as part of the original model. The query engine beneath the language is extended to support live queries.

```js
// live queries over the DOM
const result = document.querySelectorAll(css, { live: true });
Observer.observe(result, handler);
```

```js
// live queries over PostgreSQL, MySQL, etc.
const result = await db.query(sql, { live: true });
Observer.observe(result.rows, handler);
```

With results now being a live computation of the input—and changes delivered differentially as the underlying substrate changes—state across Domain-Specific Language boundaries becomes observable as local state.

## 4. Reference Papers

Our work on state is anchored on an in-progress paper series: **Stateful Computing—Rederived from First Principles**. The series progresses from computing theory, through the language and runtime substrate, to the system scope.

### 4.1 Computing Theory

- *Paper 1*—State Theory: A Layered Account of the Central Problem in Computing
- *Paper 2*—Observability: The Missing Primitive of Mutable State

### 4.2 Language/runtime substrate

- *Paper 3*—The Observer API: Reactivity Over Mutable Store
- *Paper 4*—The UseLive Directive: Live Computations Over Program State

### 4.3 System Scope

- *Paper 5*—Live Sessions: State Continuity Across Stateless Protocols
- *Paper 6*—Threads: State Continuity Across Session Boundaries

## 5. Reference Projects

The new computing substrate is enabled by individual primitives. Each implements a part of the idea at a specific layer or boundary. Projects are developed in the open.

### 5.1 Language/runtime primitives

- **[Observer API](https://github.com/webqit/observer)** — reactivity over mutable store; a direct implementation of Paper #3
- **[UseLive](https://github.com/webqit/use-live)** — live computations over program state; a direct implementation of Paper #4

### 5.2 Network/Messaging primitives

- **[FetchPlus](https://github.com/webqit/fetch-plus)** — the Fetch API, extended; implements Live Sessions (client-side) [Paper #5], the Observability protocol [Paper #2]
- **[PortPlus](https://github.com/webqit/port-plus)** — message-passing APIs—WebSockets, MessagePorts, extended; implements the Observability protocol [Paper #2]

### 5.3 Application-layer primitives

- **[URLPlus](https://github.com/webqit/url-plus)** — the URL interface,  extended; implements Live Contracts [Paper #2]

### 5.4 Browser/DOM-level work

- **[RealDOM](https://github.com/webqit/realdom)** — synchronous DOM Mutation Observer engine; implements Live Contracts [Paper #2]
- **[OOHTML](https://github.com/webqit/oohtml)** — Object-Oriented HTML; applies the Observability protocol to offer live rendering over application state [Paper #2]

### 5.5 SQL/Database work

- **[LinkedQL & FlashQL](https://github.com/linked-db/linked-ql)** — SQL, extended; implements Live Contracts (live queries) [Paper #2]

### 5.6 System/Framework work

- **[Webflo](https://github.com/webqit/webflo)** — a web application framework; implements Live Sessions (server-side) [Paper #5], Threads [Paper #6]; composes other primitives
- **[NodeLiveResponse](https://github.com/webqit/node-live-response)** — node.js & express backends, extended; implements Live Sessions (server-side) [Paper #5]

## 6. The Application Stack

A new application stack follows from the new substrate. The stack derives from the individual primitives the higher-level building blocks for a higher-level problem: an application.

### 6.1 Webflo—the application framework

**[Webflo](https://github.com/webqit/webflo)** is a full-stack web application framework that solves state as a first-class application problem—as fully managed by the system, on the same terms as its underlying substrate. Realtime, multiplayer, and offline-first experiences emerge naturally as consequences. Beyond state, Webflo approaches application architecture differently with a universal model that erases the client/server boundary. Universal routing and rendering emerge as direct consequences.

### 6.2 OOHTML—the rendering layer

**[Object-Oriented HTML (OOHTML)](https://github.com/webqit/oohtml)** is a DOM extension library that polyfills the DOM with data-binding capabilities, a HTML module system, and new scoping behaviours. State is treated as a first-class application problem—with the observability protocol as the shared language of reactivity between the application and the DOM. Reactivity over application state becomes native to the DOM as a consequence—eliminating the need for a UI framework.

### 6.3 LinkedQL & FlashQL—the durable layer

**[LinkedQL](https://github.com/linked-db/linked-ql)** is a superset of SQL that fully specifies the application data contract—incorporating both the application object model (graphs) and the concept of change (online, offline, and schema changes) through live queries, offline sync, and version binding. State becomes a language-level concern rather than something accounted for through secondary infrastructure. LinkedQL works across PostgreSQL and MySQL databases, and universally across server, client, worker, and edge environments.

**FlashQL** is LinkedQL's local, embedded database—a drop-in replacement for SQLite and PGLite. FlashQL makes offline sync possible.

## 7. Infrastructure & Developer Tools

New infrastructure and developer tools make the idea work at product scale.

This is future work.
