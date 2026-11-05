# Concurrency

Unimap serves concurrent map readers, route requests, and administrative writes. PostgreSQL 16 is the source of truth. Redis 7 and BullMQ carry work that continues after the HTTP response. The mechanics below place the queue worker, the in-memory graph cache, and live updates in one Node.js LTS process. The sources disagree on that process shape. That disagreement stays open in the last section.

## Request handling

NestJS 11 on Fastify accepts many connections on the Node.js event loop. The loop is single-threaded and non-blocking: database calls, object-storage calls, and mail I/O yield while other requests proceed.

Route search and the publication connectivity check run on that same thread. A* searches the merged building graph with a Euclidean heuristic. The breadth-first connectivity check walks the same merged graph. The deployment assumption is a campus of a few buildings, with floor graphs small enough to reside in memory.

GraphQL aggregated reads are batched with DataLoader. A throttle guard on the HTTP adapter limits the request rate on that process.

## Publication

Each uploaded floor plan becomes a new versioned map asset. A floor points at one active asset. Activating a version rewrites the denormalized room rows from that asset's graph in the same PostgreSQL transaction. Readers observe the new rooms together with the new graph, or the previous consistent pair.

A building in draft stays visible to administrators. Guests and registered users see it after the merged floor graphs form one connected component and the building is published. A failed connectivity check rejects publication and identifies the disconnected nodes.

REST under `/api` and GraphQL at `/graphql` (Mercurius) share this database. A write committed through either interface is the write the other interface reads.

## Process-local cache

The API merges the published floor graphs of a building and keeps the adjacency structure in process memory. Later route requests on that process reuse the merged graph. The lifetime of that cache is the life of the process: a restart loads the published graphs from PostgreSQL again. Building lists and room categories, which change rarely, are cached in the same process. That cache is local to the API process.

The web client keeps server state in TanStack Query, which caches responses, refetches them in the background, retries failed reads, and invalidates entries after writes. Zustand stores view state: the selected floor, highlights, and open dialogs. Search suggestions start after typing pauses, and one pause sends one request.

Floor SVG files are fetched through time-limited signed URLs from S3-compatible storage. The API process authorizes the URL. Object storage sends the file body.

## Mail queue

Registration stores a one-time email-verification token as a hash and enqueues the message with BullMQ. The HTTP handler returns when the queue accepts the job. SMTP delivery runs on the worker. Redis holds the job, so an API process restart leaves the job in the queue. A transient send failure is retried. When the job cannot be queued, registration fails and the client can try again. Opening the link consumes the token once, sets the session cookies, and signs the user in. A second opening of that same link is rejected.

Queue and mail infrastructure belong to the shared foundation of the API. In the Porto layout, a background job stays beside the container that owns the use case, and containers affect one another through an action, a task, or an event.

## Sessions

Access and refresh tokens travel in HttpOnly cookies. The access token lasts 15 minutes and carries the user id, role, and session id. An ordinary authorized request uses that payload. The refresh token lasts 30 days. The server stores a bcrypt hash of it. Passwords are stored as bcrypt hashes. Sessions that descend from one sign-in form a family.

Rotation revokes the presented session and issues a new session in the same family. A refresh token that is already revoked revokes the whole family.

The web client serializes concurrent refreshes in one tab into one in-flight refresh. The requests that waited then retry with the new access cookie. One rotation therefore reaches the reuse check.

A guest account and a registered account each have their own user row and session family. Family revocation applies to the family that presented the reused token.

## Later live updates and deferred work

The Unimap 2.0 presentation adds the following concurrent behavior. The roadmap schedules it after the open scaffold slice.

Lesson reminders are delayed BullMQ jobs. A job becomes ready a chosen number of minutes before the lesson and sends email and Web Push. A schedule change uses the same notification path. A transient reminder failure is retried from the queue.

Room occupancy (free, occupied, or soon free) is derived from the schedule, bookings, and events. Clients receive changes through Mercurius GraphQL subscriptions. The presentation pairs those subscriptions with WebSocket on the API.

A booking request checks its time interval with a PostgreSQL range and an exclusion constraint, so two overlapping bookings cannot both commit. An administrator approves the request. Approval emits an event that updates the map and sends the notification.

Search and route activity is recorded through BullMQ and aggregated in PostgreSQL materialized views. The analytics dashboard reads those views, so it follows the request that recorded the event.

The service worker stores the active map and graph versions. Offline routing runs A* in the browser on that stored graph.

A deployment that hosts several universities isolates rows by `tenant_id`. A cached building graph stays with the tenant that owns the building.

## Process shape

On the one-process account, the NestJS application holds the queue worker, the graph cache, and, in the later slices, the subscriptions.

The thesis describes one API application. Route search, the in-memory graph, and the mail queue sit in that API. The Unimap 2.0 presentation describes that same current process, and under Porto it describes a path from the monolith toward microservices. Its example is the navigation section, including A*, becoming a separate service when route search dominates the process. The product specification and the architecture document keep the worker, the graph cache, and the subscriptions in the NestJS application and place a split of a section outside the first slices.

The diploma presentation lists the architecture as client-server, a monorepo, and microservices.

These descriptions of the final process shape disagree. This file leaves each of them in place. The mechanics above are the one-process account. The microservices wording stays beside that account.
