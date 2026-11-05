# Error handling

Unimap returns one failure shape from the API. A domain failure carries a specific HTTP status and a message the interface can show in Ukrainian or English. An unexpected failure stays generic for the caller. Diagnostic recording sits beside that response and never replaces it.

The visitor client presents that shape. Sources that include a separate administrator client use the same presentation there. Whether that third application is part of the product is open, as are the other unsettled results in [Open failure behavior](#open-failure-behavior).

The open scaffold slice exposes startup validation and the public health route. Authentication, map, route, account, and admin failures arrive with those features. The cases under later slices stay inactive until that slice is opened.

## Shared pipeline

Zod checks request bodies, query parameters, and forms before a use case runs. A failed check returns HTTP 400. The `details` field lists the failing fields. Forms show those hints on the fields.

REST under the `/api` prefix returns one body shape:

| Field | Meaning |
| --- | --- |
| `statusCode` | HTTP status |
| `error` | Stable machine-readable name |
| `message` | Explanation for people |
| `path` | Request path |
| `timestamp` | UTC time of the failure |
| `details` | Optional structure: validation issues, or ids of disconnected nodes |

The visitor client maps `statusCode` and `error` through next-intl. The server `message` is the fallback when a translation is missing. An administrator client that is part of the product uses the same mapping.

The same domain failure on `/graphql` is a GraphQL error that carries the message. Mercurius serializes that channel. The HTTP exception filter leaves it unchanged.

HTTP 500 uses a generic message. The server log keeps the exception and its stack. The response body leaves the stack out.

The exception filter, the Zod pipe, and the request logger belong to the shared API foundation. In the Porto layout they stay in the ship layer, so every container returns this shape. That pipeline is the pipeline of one API. Failure behavior of any further process is open.

## Startup and health

Zod validates environment variables as the process starts. A missing or invalid value stops startup and names the setting. The process stops before it listens.

The public health route needs no session. It reports process liveness. The body says `ok` when the database answers and `degraded` when it does not, and it includes the service name, environment, uptime, database state, and timestamp.

## Authentication

| Situation | Result |
| --- | --- |
| Unknown email or wrong password | HTTP 401 with one shared message |
| Email not yet verified | HTTP 403 |
| Missing, invalid, or expired access token | HTTP 401 |
| Missing, invalid, or expired refresh cookie | HTTP 401, and the auth cookies are cleared |
| Reuse of a revoked refresh token | HTTP 401, and that session family is revoked |
| Verification token malformed, unknown, already used, expired, or mismatched | HTTP 401 |

Registration stores the account and the hash of a one-time verification token, then enqueues the mail job. The HTTP response returns after the job is queued. A transient send failure is retried from the queue. If the job cannot be queued, registration fails and the client can try again. Opening the link consumes the token, sets the session cookies, and signs the user in. A second open is rejected.

The visitor client treats HTTP 401 as a cue to refresh the access cookie once and retry the original call. Concurrent 401 responses share that single refresh. A rejected refresh clears the session and asks the user to sign in.

Excessive calls, including sign-in, receive HTTP 429. The client shows a distinct rate-limit message. Client-log intake receives the same limit when that intake is part of the product.

## Authorization

The roles are guest, registered user, and administrator.

A guest can pan and zoom the map, switch floors, search, filter by category, open room details, request a route, and view events. A registered user has personal search history and personal route history, and can save rooms and routes. A guest history request, a guest save, and event creation by a non-administrator are unsettled.

Buildings, floors, maps, rooms, and university events on the administrator surface require the administrator role. A signed-in caller with a lesser role receives HTTP 403. A missing session receives HTTP 401. Account administration, audit, and client-log review are unsettled.

## Maps and publication

An upload that is missing, is not an SVG, or is over the size limit is rejected before parsing, as HTTP 400, 415, or 413.

The parser rejects a document that contains a DOCTYPE declaration, an external entity, a `script` or `foreignObject` element, an inline event handler, or an unsafe link.

A floor is rejected with HTTP 400 when a required `room`, `node`, or `edge` group is missing, an identifier breaks the agreed convention, an edge names an unknown node, an edge joins a node to itself, or a breadth-first walk of that floor leaves some node unreached. Inter-floor nodes use the token `00`. A node whose floor token is neither its own floor nor `00` is rejected.

Publication runs breadth-first connectivity on the merged building graph. More than one component rejects publication with HTTP 422 and lists the disconnected nodes in `details`. A later activation that would disconnect a published building is rejected the same way. The previous rooms and graph remain, because activation rewrites them in one database transaction.

Guests and registered users receive HTTP 404 for a building that is missing or still a draft. Administrators can open the draft.

A stored floor graph that fails its schema check is HTTP 409. Publication stops on that record.

The browser strips scripts and event handlers from an SVG before inserting it into the page. An expired signed map URL is requested again.

## Routes, rooms, and search

A search with no matches returns an empty list and HTTP 200.

The route request returns HTTP 400 when the start and destination room codes are the same, when a chosen room has no primary navigation node, or when that node is absent from the active graph.

The route request returns HTTP 404 when the building, floor, or room is missing or unpublished, or when no path exists between the two nodes. A failed route is not written into history. A successful route for a registered user is written into that user's route history. Storage for any other caller is unsettled.

A node without coordinates still participates in routing. Its heuristic is zero, so that step follows Dijkstra. A path found this way is a success.

## Events and accounts

A missing event or user is HTTP 404. Every visitor can view a university event. An administrator can create, edit, and schedule a university event. A create or edit by a student, a guest, or another non-administrator, including a personal event, is unsettled. A duplicate registration email and a duplicate building code are HTTP 409.

## Client presentation

Zod field errors appear on the form. A failed request raises a toast. TanStack Query retries a failed read; after those retries the interface shows the failure. A panel that fails while rendering shows a short message and a retry action. The rest of the map stays usable.

Where client-log intake is part of the product, the client sends batches of failure reports. Each entry has the application, a level, a short message, and, when known, the method, URL, and status. The batch leaves out cookies and credentials. Intake accepts the batch. If intake itself fails, the client drops the batch and navigation continues.

## Diagnostics

Every HTTP request is recorded with method, path, duration, and outcome. Status 500 and above are logged with the stack. Where administrator audit is part of the product, administrator actions are recorded for that screen with actor, action, and outcome. Where client-log review is part of the product, client reports are recorded for that screen. A failure while writing a diagnostic record is logged by the process. The user still receives the original response.

## Open failure behavior

No choice is recorded for the results below. The sections above state the overlap the sources share. These results stay unsettled.

### History

A registered user's personal history, including storage of that user's successful route, is specified above. The status of a history read, delete, or clear for a guest or for a caller without a session, and storage of a successful route for any other caller, stay unsettled.

- The thesis role section and the thesis history implementation keep temporary search and route history on the guest account. Registration may attach that guest history to the new account. The thesis routing section stores each successful path request of an authorized user.
- The thesis use-case section lists personal history for the registered user and omits it from the guest. The thesis history panel asks a person who is not signed in to sign in before stored history is shown. The thesis access design limits personal history and saved items to an authenticated user.
- [spec.md](spec.md) stores a successful path for a registered user and leaves every other caller unresolved. [architecture.md](architecture.md) ties history rows to a user and leaves the owning account open. [concurrency.md](concurrency.md) gives a guest account and a registered account each a user row and a session family, and it states no history denial.
- The Unimap 2.0 presentation lists search and route history, including route replay, for students and guests.

### Saved items and event creation

A registered user can save rooms and routes. A missing access token on a save is HTTP 401. The status of a save by a guest stays unsettled.

An administrator can create, edit, and schedule a university event, and a lesser role on that administrator operation receives HTTP 403. The status of a create or edit by a student, a guest, or another non-administrator, including a personal event, stays unsettled.

- The Unimap 2.0 presentation, in stage 1 marked done, includes personal saved rooms and routes for students and guests, and creation of personal events together with viewing of global events.
- The diploma presentation lists viewing and creating events as a user-facing capability, and creation and scheduling of global events for the administrator. Its save slide lists personal saved rooms and routes for the user.
- The thesis gives saved rooms and routes to the registered user, and creation and editing of university events to the administrator.

### Applications

An API workspace and a visitor client are common to the sources. Every source includes an administrator interface. The thesis names the workspaces `apps/api` and `apps/web`. The Unimap 2.0 presentation, in stage 1 marked done, names a third application, `apps/admin`. The diploma presentation describes the administrator interface and does not name a workspace. [architecture.md](architecture.md) leaves the packaging of that interface open. Client failure presentation above is the visitor client. The same presentation on a third application stays unsettled.

### Process

The filter, the validation pipe, and the request log above are the one-API account of this pipeline.

- The thesis describes one API application and one visitor client, with domain modules inside that API.
- The Unimap 2.0 presentation describes that current backend as a modular monolith and treats a Porto section as a boundary that can later become a separate service. The navigation section under route-search load is the example. The specification draft and [architecture.md](architecture.md) keep the first slices as one NestJS process and place that extraction outside those slices. [concurrency.md](concurrency.md) places the queue worker, the graph cache, and later subscriptions in that one process and leaves the microservices wording beside it.
- The diploma presentation lists client-server structure, a monorepo, and microservices together.

Failure behavior of a second process stays unsettled.

### Administrator surface

Buildings, floors, maps, rooms, and university events require the administrator role, with HTTP 403 and HTTP 401 as specified above. Account list, role changes, account deletion, administrator-action audit, and client-log review stay unsettled, so this document assigns them no status code.

- The thesis limits administration to buildings, floors, maps, rooms, and events. Its module list does not describe an account-administration screen, an audit screen, or client-log intake.
- The diploma presentation adds a user list, role settings, and account deletion.
- The Unimap 2.0 presentation, in stage 1 marked done, adds those account operations and also an audit of administrator actions and client-log collection.
- [architecture.md](architecture.md) leaves this surface open. [spec.md](spec.md) records the same split.

The client-log intake and audit sentences earlier in this document are the failure behavior of the wider reading. The choice of that reading stays open.

## Later slices

These rules apply when the matching roadmap slice is opened.

Accessible routes. A profile that forbids stairs returns HTTP 404 when every remaining path would use stairs. The message says that no accessible route exists. The response is that miss.

Schedule import. A lesson row that cannot be matched to a room stays unmatched, and the import report names that row. A reminder that fails to send is retried by the queue.

Booking. An interval that overlaps an existing booking returns HTTP 409. The PostgreSQL exclusion constraint rejects a race that passes the earlier check, and that caller also receives the conflict.

QR and offline maps. A QR target whose node is unknown is HTTP 404. Offline routing uses the cached active graph. An empty cache tells the user that offline routing is unavailable.

Map editor. Publish stays disabled while the live breadth-first check finds a disconnected component, and the editor highlights that component. A publish submitted anyway receives the same HTTP 422.

Analytics. A search or a route with no result is stored as an empty outcome for the dashboard. It is a normal outcome, and it is separate from an HTTP 500.
