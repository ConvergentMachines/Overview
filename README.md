# The Unified State Blueprint

_The foundational framework, technical specification roadmap, and architectural FAQs for a universal computing substrate._

* [1. Who We Are](#1-who-we-are)
* [2. What We Found](#2-what-we-found)
* [3. What Changes](#3-what-changes)
    * [3.1 At the language/runtime layer](#31-at-the-languageruntime-layer)
    * [3.2 At the application layer](#32-at-the-application-layer)
    * [3.3 Across network/protocol boundaries](#33-across-networkprotocol-boundaries)
    * [3.4 Across DSL/query boundaries](#34-across-dslquery-boundaries)


* [4. Reference Papers](#4-reference-papers)
    * [4.1 Computing Theory](#41-computing-theory)
    * [4.2 Language/runtime substrate](#42-languageruntime-substrate)
    * [4.3 System Scope](#43-system-scope)


* [5. Reference Projects](#5-reference-projects)
    * [5.1 Language/runtime primitives](#51-languageruntime-primitives)
    * [5.2 Network/Messaging primitives](#52-networkmessaging-primitives)
    * [5.3 Application-layer primitives](#53-application-layer-primitives)
    * [5.4 Browser/DOM-level work](#54-browserdom-level-work)
    * [5.5 SQL/Database work](#55-sqldatabase-work)
    * [5.6 System/Framework work](#56-systemframework-work)


* [6. The Application Stack](#6-the-application-stack)
    * [6.1 Webflo—the application framework](#61-webflothe-application-framework)
    * [6.2 OOHTML—the rendering layer](#62-oohtmlthe-rendering-layer)
    * [6.3 LinkedQL & FlashQL—the durable layer](#63-linkedql--flashqlthe-durable-layer)


* [7. Infrastructure & Developer Tools](#7-infrastructure--developer-tools)
* [8. Frequently Asked Questions (FAQ)](#8-frequently-asked-questions-faq)

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

## 8. Frequently Asked Questions (FAQ)

---

<details>
<summary><b>1. Is this an AI/deep learning lab or a traditional systems lab?</b></summary>

**We are a systems-first computing substrate lab.** While modern tech often conflates "deep tech" purely with AI, our domain is core computer science and programming language theory.
    
Our focus is entirely on the structural primitive of state itself. We treat state as an infrastructure problem rather than an application problem. However, our universal treatment of state directly intersections with the future of intelligent software; by building a substrate where stateful tracking is handled natively by the runtime, we provide the foundational mechanics required for autonomous agents and complex, reactive distributed environments to run cleanly at scale without legacy boilerplate.
    
</details>

---

<details>
<summary><b>2. How does the lab’s methodology balance research with real-world engineering?</b></summary>

We operate under a strict, bidirectional feedback loop: **Engineering informs the research; research informs the engineering.**

We do not write purely theoretical academic papers in a vacuum, nor do we build ad-hoc, unprincipled software tools. Every specification, language directive, or protocol we design is born from diagnosing systemic friction in production-scale software. Conversely, our formal computer science research mandates the exact architecture of the runtimes and developer infrastructure we implement. We only consider a state problem "solved" when its formal mathematical treatment compiles into stable, product-scale tooling.

</details>

---

<details>
<summary><b>3. How does "State Engineering" differ from traditional State Management?</b></summary>

Traditional state management is a user-land surface concern. It isolates state into specific pockets of an application using local custom stores, variables, and ephemeral frameworks that do not compose with one another.

We have identified and named a recurring, universal shape of work that spans every layer of computing—cache invalidation, UI reactivity, database syncing, and distributed network coordination. We call this macro-discipline State Engineering. State Engineering recognizes that whether you are binding memory mutations to a DOM or reconstructing local state from event streams across a cloud network, you are solving the exact same fundamental problem. Our lab treats this entire continuum as a singular, first-class primitive.

</details>

---

<details>
<summary><b>4. Where does the Unified State paradigm intersect with classic programming language theory?</b></summary>

Historically, programming language design has treated memory management as an implicit, runtime-level concern (e.g., automatic garbage collection), while forcing developers to manage state mutations explicitly through manual, fragile code structures.

The **Unified State Model** alters this division of labor. It answers the fundamental question of who accounts for what, and how, by shifting state tracking from a developer concern to a system concern. In our paradigm, change detection and observability become an intrinsic property of mutation itself, built straight into the runtime's control flow graph. This fundamentally bridges the gap between pure functional programming concepts (which ban shared mutable state) and imperative paradigms, offering a safe, universal protocol for live computation.

</details>

---

<details>
<summary><b>5. How does this model redefine state boundaries across distributed systems?</b></summary>

Modern distributed systems are plagued by the architectural split between stateless protocols (like standard HTTP) and separate, heavy real-time infrastructure (WebSockets, Server-Sent Events) used to push updates. This forces developers to repay the engineering cost of state reconstruction at every network boundary.

Our research redefines these boundaries by extending existing network and query protocols to support Live Contracts. By formalizing change as part of the initial data request model, objects conceptually cross network, protocol, and database boundaries by reference. This eliminates secondary synchronization infrastructure entirely, making state across a global distributed cluster conceptually indistinguishable from a local memory reference.

</details>

---

<details>
<summary><b>6. What is the roadmap for the lab's upcoming publications and specifications?</b></summary>

Our core research is organized into an in-progress, 6-part formal paper series titled **Stateful Computing—Rederived from First Principles.**

The publication roadmap moves sequentially from foundational computation theory up to global cloud execution:

1. **Computing Theory:** Defining the layered account of state and introducing observability as a missing primitive of mutable state.
2. **Language & Runtime Substrate:** Formalizing the low-level Observer API and the implementation semantics of the use live compiler directive.
3. **System Scope:** Delivering specifications for protocol continuity across stateless boundaries and persistent state sessions.

</details>
