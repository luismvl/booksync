# BookSync — Master Plan

> Living plan for learning modern Java and building an open-source, cloud-first reading platform with self-hosting and an optional hosted service.

## 0. Project status

This document is the source of truth for the project until a decision is replaced by an Architecture Decision Record (ADR).

### Current baseline

- **Backend:** Java 25, Spring Boot 4.1, Maven.
- **Web:** React with Next.js and TypeScript.
- **Mobile:** React Native with Expo and TypeScript.
- **Database:** PostgreSQL.
- **Development database:** Docker Compose.
- **Book storage:** Local filesystem behind a storage abstraction.
- **Supported formats at launch:** PDF and EPUB without DRM.
- **Default PDF mode:** Continuous scrolling, with optional paginated mode.
- **Offline:** A downloaded book remains readable offline.
- **Source model:** Open source, self-hostable, with an optional paid hosted service.
- **Initial authentication:** Email and password with short-lived access tokens and rotating refresh-token sessions.
- **Primary promise:** “I can read the same book anywhere and it always remembers exactly where I was.”

---

# 1. Product vision

BookSync is a personal reading platform where a user uploads their own PDF and EPUB files, reads them on the web or mobile, and keeps the complete reading state synchronized:

- Book files.
- Current reading position.
- Highlights.
- Notes.
- Bookmarks.
- Tags and collections.
- Reader preferences.
- Per-device activity and offline changes.

The self-hosted edition and the hosted service should run from the same core codebase. The hosted service provides convenience, storage, backups, updates, and availability; it must not require a separate proprietary reader implementation.

## 1.1 Product principles

1. **The user owns the library.** Files and reading data can be exported.
2. **Offline is a normal state.** It is not treated as an error.
3. **Sync must be understandable.** The application exposes clear states instead of pretending synchronization cannot fail.
4. **No silent data loss.** Conflicting notes or positions are preserved until resolved.
5. **Reading comes first.** Administration, social features, and recommendations must not distract from the reader.
6. **Open formats first.** PDF and EPUB without DRM are the initial scope.
7. **Self-hosting remains practical.** One server, PostgreSQL, and persistent storage should be enough for most installations.
8. **Start as a modular monolith.** No microservices until a measured problem requires them.
9. **Prefer boring infrastructure.** No Kafka, Kubernetes, Redis, or distributed storage for the initial product.
10. **Accessibility is part of the reader.** Keyboard navigation, screen-reader semantics, contrast, font controls, and reduced motion are product requirements.

## 1.2 Explicit non-goals for v1

- Circumventing DRM.
- Kindle/AZW DRM support.
- A bookstore or content marketplace.
- Audiobooks.
- OCR for scanned PDFs.
- Collaborative or public annotations.
- Social feeds, ratings, or recommendations.
- AI summaries or chat with books.
- Real-time multi-user editing.
- Microservices.
- Kubernetes.
- End-to-end encryption.
- Full-text search across every book.
- Automatic metadata matching against external book databases.
- iOS-specific optimization before the Android and web flows are stable.

These may be reconsidered after v1, but they must not expand the initial scope.

---

# 2. Learning strategy

The project has two equal goals:

1. Build BookSync.
2. Relearn Java and become productive with modern Spring Boot.

The learning approach is **vertical and project-driven**, not “finish a Java course and then start the app.” Each project phase introduces only the Java and Spring concepts needed for that phase.

## 2.1 Teaching workflow for each feature

For every feature, follow this cycle:

1. **Context:** Explain what problem is being solved.
2. **Java concepts:** Introduce the language concepts required.
3. **Spring concepts:** Explain the relevant framework behavior and annotations.
4. **Small design:** Define the classes, data flow, and tests before coding.
5. **Implementation:** Build one thin, working slice.
6. **Review:** Review code for correctness, Java style, naming, and unnecessary complexity.
7. **Tests:** Add unit and integration coverage.
8. **Reflection:** Summarize what was learned and what remains unclear.
9. **Commit:** Save a small, understandable commit.

The assistant should explain concepts and review decisions instead of only dumping completed code. Full code can still be provided when requested, but the default is guided implementation.

## 2.2 Java topics to learn through the project

### Foundation

- JDK, JVM, JRE, bytecode, compilation, and runtime.
- Packages, source sets, and classpaths.
- Maven lifecycle and dependency scopes.
- Primitive types versus wrapper types.
- Reference semantics and immutability.
- Classes, interfaces, abstract classes, enums, and records.
- Constructors and factory methods.
- Access modifiers.
- Checked and unchecked exceptions.
- Generics and variance.
- Collections.
- `equals`, `hashCode`, and `toString`.
- Null handling and `Optional`.

### Modern Java

- Records for immutable data carriers.
- Sealed classes and interfaces for closed domain hierarchies.
- Pattern matching.
- Switch expressions.
- Text blocks.
- Streams and collectors.
- `Instant`, `Duration`, and modern date/time APIs.
- Virtual threads and structured concurrency concepts.
- JVM diagnostics and Java Flight Recorder at a basic level.

### Spring

- Dependency injection and the application context.
- Component scanning.
- Configuration properties.
- Controllers and HTTP message conversion.
- Bean validation.
- Services and transaction boundaries.
- Repositories, JPA, Hibernate, and persistence contexts.
- Database migrations with Flyway.
- Spring Security filters and authorization.
- Error handling with Problem Details.
- Actuator, metrics, health checks, and graceful shutdown.
- Testing with Spring Boot and Testcontainers.

## 2.3 Deliberate constraints while learning

- Do not use Lombok initially. Records and explicit code make Java behavior easier to learn.
- Do not add a framework because a tutorial uses it.
- Do not create interfaces for classes that have only one implementation unless they represent a real boundary, such as file storage.
- Do not use inheritance where composition or a sealed hierarchy is clearer.
- Do not force streams into code where a loop is easier to understand.
- Do not use reactive Spring WebFlux for this project initially.
- Do not enable virtual threads until the conventional request model works and is measured.
- Do not hide SQL and transaction behavior behind excessive abstractions.

---

# 3. Technology decisions

## 3.1 Backend

- **Java 25.**
- **Spring Boot 4.1.**
- **Spring MVC**, not WebFlux.
- **Maven**, using the Maven Wrapper committed to the repository.
- **Spring Data JPA and Hibernate** for persistence.
- **PostgreSQL** as the only supported database initially.
- **Flyway** for every schema change.
- **Spring Security** for authentication and authorization.
- **Jakarta Bean Validation** for request and command validation.
- **Spring Boot Actuator** for health and operational endpoints.
- **JUnit 5, AssertJ, and Testcontainers** for testing.
- **OpenAPI** as the API contract and source for the generated TypeScript client.

### Why Spring MVC

The application performs conventional database and file I/O. A blocking request model is easier to learn, debug, and operate. Java virtual threads can later improve concurrency without adopting the complexity of reactive programming.

### Why Maven

Maven is predictable, convention-based, widely documented, and sufficient for this project. The wrapper removes the need for every contributor to install the same Maven version globally.

## 3.2 Web

- React.
- Next.js.
- TypeScript with strict mode.
- TanStack Query for server state.
- A local database layer using IndexedDB; the exact wrapper is selected during the offline spike.
- A generated API client from OpenAPI.
- PDF.js as the leading PDF renderer candidate.
- EPUB.js or a Readium-compatible web approach as candidates for EPUB.

## 3.3 Mobile

- React Native.
- Expo with development builds, not a permanent dependency on Expo Go.
- TypeScript with strict mode.
- Expo SQLite for synchronized local data.
- Device filesystem storage for downloaded book files.
- Android as the first fully tested mobile target on Ubuntu.
- iOS supported by the architecture and built later through a Mac or cloud build service.

## 3.4 Infrastructure

### Development

- Docker Compose for PostgreSQL.
- Mailpit when email verification and password recovery are introduced.
- Local filesystem for books and generated covers.
- No Redis.
- No message broker.

### Self-hosted production

- Docker Compose.
- Reverse proxy providing HTTPS.
- API container.
- PostgreSQL container or external PostgreSQL.
- Persistent book-data volume.
- Persistent database volume.
- Optional monitoring profile.

### Hosted service

Initially the same architecture can run on one server:

- One API instance.
- Managed or self-hosted PostgreSQL.
- Local persistent volume or an S3-compatible storage implementation.
- Reverse proxy/CDN.
- Background worker may remain inside the API process until load proves otherwise.

---

# 4. Repository strategy

Use a monorepo, but start by generating only the backend application.

```text
booksync/
├── apps/
│   ├── api/                 # Java + Spring Boot
│   ├── web/                 # Next.js
│   └── mobile/              # Expo / React Native
├── packages/
│   ├── api-client/          # Generated TypeScript API client
│   ├── sync-core/           # Shared TypeScript sync logic
│   ├── reader-model/        # Shared locator and annotation types
│   └── design-tokens/       # Optional shared visual tokens
├── infra/
│   ├── compose/
│   ├── reverse-proxy/
│   └── monitoring/
├── docs/
│   ├── adr/                 # Architecture Decision Records
│   ├── api/
│   ├── product/
│   ├── sync/
│   └── self-hosting/
├── scripts/
├── .github/
└── README.md
```

## 4.1 Backend package structure

Use **package by feature**, not global packages such as `controllers`, `services`, and `repositories`.

```text
dev.luismvl.booksync
├── BookSyncApplication.java
├── auth/
├── users/
├── devices/
├── books/
├── reading/
├── annotations/
├── collections/
├── sync/
├── storage/
└── shared/
```

A feature starts simple:

```text
books/
├── BookController.java
├── BookService.java
├── BookRepository.java
├── BookEntity.java
└── dto/
```

Subpackages such as `api`, `application`, `domain`, and `infrastructure` are added only when a feature becomes large enough to justify them.

## 4.2 Architecture style

Use a **modular monolith**:

- One deployable backend.
- Clear feature boundaries.
- Direct in-process calls between modules through intentional public services.
- One PostgreSQL database.
- Transaction boundaries remain simple.
- Modules may later be verified with architecture tests or Spring Modulith.

Do not create microservices in anticipation of scale.

---

# 5. Core domain model

All identifiers use UUIDs generated by the client when offline creation is required, otherwise by the server. All persisted timestamps use UTC and map to Java `Instant`.

## 5.1 User and authentication

### `users`

- `id`
- `email`
- `display_name`
- `status`
- `email_verified_at`
- `created_at`
- `updated_at`

### `user_credentials`

- `user_id`
- `password_hash`
- `password_changed_at`

Keeping credentials separate makes future OAuth identities easier to add without overloading the user table.

### `devices`

- `id`
- `user_id`
- `name`
- `platform`
- `app_version`
- `last_seen_at`
- `created_at`

### `sessions`

- `id`
- `user_id`
- `device_id`
- `refresh_token_hash`
- `token_family_id`
- `expires_at`
- `last_used_at`
- `revoked_at`
- `created_at`

Refresh tokens are stored only as hashes. Rotation and token-family tracking allow reuse detection and session revocation.

## 5.2 Books and files

### `books`

Represents the logical item shown in the user’s library.

- `id`
- `owner_id`
- `title`
- `author`
- `description`
- `language`
- `format`: `PDF` or `EPUB`
- `status`: `UPLOADING`, `PROCESSING`, `READY`, `FAILED`, `DELETED`
- `cover_storage_key`
- `created_at`
- `updated_at`
- `deleted_at`
- `version`

### `book_files`

Represents the stored original file.

- `id`
- `book_id`
- `storage_key`
- `original_filename`
- `media_type`
- `size_bytes`
- `sha256`
- `created_at`

For the initial implementation, a book has one original file. The separation leaves room for file replacement, alternate editions, and generated artifacts later.

### Deduplication rule

Only deduplicate within the same user account initially. Cross-user deduplication can create privacy and deletion complications and is not necessary at the expected scale.

## 5.3 Reading settings

### `reading_preferences`

- `id`
- `user_id`
- `book_id` nullable for global defaults
- `device_id` nullable
- `pdf_mode`: `CONTINUOUS` or `PAGINATED`
- `theme`
- `font_family`
- `font_size`
- `line_height`
- `margin_size`
- `updated_at`

Preference precedence:

1. Device-and-book preference.
2. Book preference.
3. Device preference.
4. User global preference.
5. Application default.

This hierarchy may be simplified in the first implementation.

## 5.4 Reading positions

Do not store only one destructive “current page.” Store the latest position per device and preserve enough recent history to resolve offline divergence.

### `reading_positions`

- `id`
- `user_id`
- `book_id`
- `device_id`
- `locator` as JSONB
- `progression` from `0.0` to `1.0`
- `client_updated_at`
- `server_received_at`
- `session_id` nullable
- `version`

Unique logical key:

```text
(user_id, book_id, device_id)
```

### Optional `reading_position_history`

Store recent checkpoints for debugging and a reading timeline. This is not required for the first slice; it can be introduced when conflict UX is implemented.

## 5.5 Highlights

### `highlights`

- `id`
- `user_id`
- `book_id`
- `locator` as JSONB
- `selected_text`
- `text_before`
- `text_after`
- `color`
- `created_at`
- `updated_at`
- `deleted_at`
- `version`

## 5.6 Notes

### `notes`

- `id`
- `user_id`
- `book_id`
- `highlight_id` nullable
- `locator` nullable when attached to a highlight
- `body`
- `created_at`
- `updated_at`
- `deleted_at`
- `version`

A note may be attached to a highlight or directly to a reading location.

## 5.7 Bookmarks

### `bookmarks`

- `id`
- `user_id`
- `book_id`
- `locator`
- `label` nullable
- `created_at`
- `updated_at`
- `deleted_at`
- `version`

## 5.8 Tags and collections

### `tags`

- `id`
- `user_id`
- `name`
- `created_at`
- `updated_at`
- `deleted_at`
- `version`

### `book_tags`

- `id`
- `user_id`
- `book_id`
- `tag_id`
- `created_at`
- `deleted_at`

A collection can initially be represented as a tag. A separate ordered collection model is added only when the product needs collection-specific behavior.

---

# 6. Locator and annotation model

A locator is a format-independent description of a place in a publication. It is the core of reading progress, bookmarks, highlights, and location notes.

## 6.1 General locator shape

```json
{
  "format": "PDF",
  "progression": 0.427,
  "title": "Chapter 6",
  "position": {
    "pageIndex": 36,
    "offsetRatio": 0.62
  },
  "text": {
    "exact": "selected text",
    "before": "context immediately before",
    "after": "context immediately after"
  }
}
```

The exact schema will be versioned. Every stored locator includes a `schemaVersion` once synchronization begins.

## 6.2 PDF position

For continuous scrolling, the canonical location is:

- Zero-based `pageIndex`.
- `offsetRatio` between `0.0` and `1.0`, representing how far down the page the reading viewport is.
- Overall `progression` for display and rough fallback.

Example:

```json
{
  "format": "PDF",
  "schemaVersion": 1,
  "progression": 0.427,
  "position": {
    "pageIndex": 36,
    "offsetRatio": 0.62
  }
}
```

This position works for continuous and paginated rendering. Paginated mode may ignore the offset when navigating, but it should not destroy it.

## 6.3 PDF highlight target

A PDF highlight stores:

- Page index.
- One or more normalized rectangles relative to the page.
- Selected text.
- Text before and after the selection when available.
- Optional text-layer character indexes as an additional fallback.

```json
{
  "format": "PDF",
  "schemaVersion": 1,
  "position": {
    "pageIndex": 36
  },
  "rectangles": [
    { "x": 0.12, "y": 0.31, "width": 0.55, "height": 0.028 }
  ],
  "text": {
    "exact": "selected text",
    "before": "previous words",
    "after": "following words"
  }
}
```

Coordinates must be normalized against the unrotated page dimensions so zoom and device resolution do not affect them.

## 6.4 EPUB position

An EPUB position stores:

- EPUB CFI when supported by the selected renderer.
- Spine item or resource reference.
- Progression within the resource.
- Overall progression.
- A text quote with context as fallback.

```json
{
  "format": "EPUB",
  "schemaVersion": 1,
  "progression": 0.427,
  "position": {
    "href": "chapter-06.xhtml",
    "cfi": "epubcfi(...) ",
    "resourceProgression": 0.62
  },
  "text": {
    "exact": "selected text",
    "before": "previous words",
    "after": "following words"
  }
}
```

## 6.5 Anchoring rule

Never rely on a single pointer. Store several compatible selectors:

1. Primary format-native pointer: CFI or PDF rectangles.
2. Selected text.
3. Prefix and suffix context.
4. Broad progression fallback.

This gives the reader several ways to recover after a renderer update or minor file variation.

## 6.6 File identity rule

A locator belongs to a specific file hash. If a user replaces a book file with a different hash, BookSync must not silently assume all old locators remain correct. The replacement flow must either:

- Treat it as a new book.
- Attempt a migration and show unresolved annotations.
- Require explicit confirmation.

File replacement is not part of the first release.

---

# 7. Synchronization design

Synchronization is a first-class subsystem, not a collection of unrelated API calls.

## 7.1 Sync principles

- Every offline mutation receives a client-generated operation ID.
- Replaying the same operation must be safe.
- Pull synchronization uses a server cursor, not client timestamps.
- Deletes use tombstones.
- The server never trusts client clocks for ordering.
- Client timestamps are retained for UX and diagnostics only.
- A sync failure must not block local reading or note creation.
- The user can see whether data is local, pending, synchronized, conflicting, or failed.

## 7.2 Client local model

Each client maintains:

- A local database.
- Cached book metadata.
- Local copies of downloaded book files.
- A mutation outbox.
- The last acknowledged server cursor.
- Sync state and retry metadata.

### Mutation outbox

Each local mutation contains:

- `operationId`
- `entityType`
- `entityId`
- `operationType`: create, update, delete
- `baseVersion`
- `payload`
- `createdAt`
- `attemptCount`
- `lastError`

## 7.3 Server sync model

### `processed_sync_operations`

Used for idempotency.

- `operation_id`
- `user_id`
- `processed_at`
- `result_reference`

### `sync_changes`

Append-only user change feed.

- `sequence` as a monotonically increasing server value
- `user_id`
- `entity_type`
- `entity_id`
- `change_type`
- `entity_version`
- `created_at`

The client pulls all changes after its cursor:

```http
GET /api/v1/sync/pull?cursor=1842&limit=500
```

The response contains:

- Changes.
- `nextCursor`.
- `hasMore`.
- Optional server time.

## 7.4 Push flow

```http
POST /api/v1/sync/push
```

The body contains a batch of operations. The server handles each operation transactionally and returns one result per operation:

- Applied.
- Already applied.
- Rejected by validation.
- Conflict.
- Unauthorized.

## 7.5 Conflict policy by entity

### Reading position

Reading positions are not text documents and should not be “merged.” Keep the latest candidate per device.

Normal behavior:

- Automatically resume from the most recently received credible position.
- Do not show a conflict when positions are close.

Conflict behavior:

- When two offline devices report meaningfully different recent positions, show a small reader banner.
- Display device name, time, chapter/page, and percentage.
- Allow “Continue here” or “Jump to other position.”
- Choosing a position creates a new checkpoint rather than deleting history.

### Highlights and bookmarks

These are independent entities with client-generated IDs. Concurrent creation normally merges naturally.

When the same entity is edited on two devices:

- Use optimistic version checks.
- Preserve the conflicting version.
- Mark it for resolution rather than silently overwriting.

### Notes

Never silently overwrite concurrent note edits.

Initial conflict handling:

- Keep the server version.
- Store the incoming conflicting version as a conflict copy.
- Present both versions with actions: keep mine, keep remote, or manually merge.

A CRDT is explicitly deferred. It is unnecessary until collaborative or extremely frequent concurrent note editing becomes a real requirement.

### Deletions

A deletion creates a tombstone with `deleted_at`. Tombstones remain in the sync feed long enough for all active devices to receive them. Physical cleanup is delayed and configurable.

## 7.6 Sync UX states

The application should expose these states consistently:

- **Synced**
- **Syncing**
- **Offline — changes saved locally**
- **Waiting to upload**
- **Conflict requires attention**
- **Sync failed — retrying**
- **Permanent error — action required**

Avoid modal dialogs for normal network transitions. Use subtle status indicators and actionable banners only when user input is required.

---

# 8. Authentication and account design

## 8.1 Initial decision

Implement:

- Email and password registration.
- Login.
- Short-lived JWT access token.
- Rotating opaque refresh token.
- Per-device sessions.
- Logout current session.
- Logout all sessions.
- Password change.
- Password reset after email infrastructure is added.

Do not implement OAuth or magic links in the first vertical slice.

## 8.2 Why this order

Email/password exercises important Spring Security concepts while keeping the first implementation understandable. OAuth and magic links introduce provider configuration, callback flows, email delivery, and account-linking edge cases before the core product exists.

The data model must leave room for future login methods, but unused complexity should not be implemented early.

## 8.3 Token model

- Access token lifetime: approximately 10–15 minutes.
- Refresh session lifetime: configurable, initially around 30 days.
- Refresh token rotates on every successful refresh.
- Refresh token is stored hashed in the database.
- Reuse of an older token revokes the token family.
- Access token carries only stable authorization claims.
- Authorization always verifies ownership in the database for protected resources.

## 8.4 Password storage

Use Spring Security’s password encoding abstraction with a modern adaptive password hash. Store the algorithm identifier with the hash so parameters or algorithms can be upgraded later.

## 8.5 Future authentication phases

1. Email verification.
2. Password recovery through Mailpit in development and SMTP in production.
3. OAuth with one provider.
4. Account linking between password and OAuth identity.
5. Magic-link login only if users request it.

---

# 9. File storage and processing

## 9.1 Storage abstraction

Create a real boundary:

```text
BookStorage
├── store(...)
├── open(...)
├── openRange(...)
├── exists(...)
└── delete(...)
```

Initial implementation:

```text
LocalFilesystemBookStorage
```

Future implementation:

```text
S3BookStorage
```

The domain must never know filesystem paths or S3 APIs.

## 9.2 Local storage layout

Use generated storage keys, not raw filenames:

```text
/data/
├── books/{user-id}/{book-id}/{file-id}.bin
├── covers/{user-id}/{book-id}/{cover-id}.webp
└── temp/{upload-id}.part
```

The original filename is metadata only.

## 9.3 Upload flow

1. Authenticate user.
2. Stream upload to a temporary file.
3. Enforce configurable maximum size.
4. Calculate SHA-256 while streaming.
5. Detect actual media type.
6. Validate PDF or EPUB structure.
7. Check same-user duplicate hash.
8. Move file atomically to permanent storage.
9. Create book and file records.
10. Mark book as processing.
11. Extract metadata and cover.
12. Mark ready or failed with a recoverable error.

The request may process small files synchronously in the first prototype. It should move to a database-backed job once extraction is implemented.

## 9.4 Background jobs

Start with a PostgreSQL job table and a scheduled worker in the same application.

Possible jobs:

- Extract metadata.
- Generate cover.
- Validate content.
- Remove expired tombstones.
- Delete orphaned files.
- Recalculate book progression maps.

Do not introduce Kafka, RabbitMQ, or Redis solely for jobs.

## 9.5 Download and streaming

- Authenticated downloads.
- Ownership validation.
- HTTP Range support.
- `ETag` based on the file hash.
- Correct content type and length.
- Optional content disposition.

Range support matters for large PDFs and readers that fetch content in chunks.

## 9.6 Validation and security

- Never trust file extensions.
- Limit compressed EPUB expansion to reduce archive-bomb risk.
- Reject path traversal entries inside archives.
- Never render uploaded HTML directly in the application origin without isolation and sanitization.
- Apply a Content Security Policy to EPUB rendering.
- Keep uploaded files outside the application source tree and public static directory.
- Add malware scanning only when the hosted service or threat model justifies it.

---

# 10. Reader architecture and technical spikes

The reader is the highest-risk area. It must be validated early, before months are spent on backend infrastructure.

## 10.1 Candidate architecture

A promising architecture is a shared web-based reader core:

- The web app embeds it directly.
- The mobile app embeds the same reader core in a WebView.
- The native shell handles local files, SQLite, authentication, downloads, and system integrations.
- A message bridge transfers locators, selection events, theme settings, and commands.

This may maximize code sharing, but it is not accepted until the technical spikes prove performance, text selection, offline behavior, and accessibility.

## 10.2 PDF spike

Build a disposable prototype that can:

1. Open a local PDF.
2. Render continuous scrolling.
3. Switch to paginated mode.
4. Report the current page and normalized offset.
5. Restore the exact position after restart.
6. Select text.
7. Create a highlight using normalized rectangles.
8. Restore the highlight after restart.
9. Work without network after the file is downloaded.
10. Run on web and Android.

Compare:

- PDF.js in browser and WebView.
- A native React Native PDF library.
- A hybrid approach if necessary.

Measure:

- Rendering speed.
- Memory use on a long document.
- Text selection quality.
- Coordinate stability.
- Accessibility.
- Maintenance activity and license.
- Expo compatibility.

## 10.3 EPUB spike

Build a disposable prototype that can:

1. Open a local EPUB.
2. Render reflowable text.
3. Change font size and theme.
4. Support paginated and continuous modes if feasible.
5. Report a CFI or equivalent locator.
6. Restore location after font and viewport changes.
7. Select text.
8. Create and restore a highlight.
9. Preserve a quote selector fallback.
10. Run on web and Android offline.

Compare EPUB.js and a Readium-compatible approach. Do not choose solely based on GitHub stars.

## 10.4 Spike exit criteria

A renderer is accepted only when:

- It restores progress reliably across two viewport sizes.
- It restores at least 95% of a test set of highlights.
- It handles a representative large file.
- It works offline.
- Its license is compatible with the project.
- The integration does not block future iOS support.

The spike code may be discarded. Its purpose is to remove uncertainty.

---

# 11. API design

Use REST under a versioned prefix:

```text
/api/v1
```

Use JSON for domain resources and multipart for uploads. Use consistent Problem Details responses for errors.

## 11.1 Initial endpoints

### System

```http
GET /api/v1/health
```

### Authentication

```http
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
POST /api/v1/auth/logout-all
GET  /api/v1/auth/me
```

### Devices

```http
POST   /api/v1/devices
GET    /api/v1/devices
PATCH  /api/v1/devices/{deviceId}
DELETE /api/v1/devices/{deviceId}
```

### Books

```http
POST   /api/v1/books
GET    /api/v1/books
GET    /api/v1/books/{bookId}
PATCH  /api/v1/books/{bookId}
DELETE /api/v1/books/{bookId}
GET    /api/v1/books/{bookId}/file
GET    /api/v1/books/{bookId}/cover
```

### Reading position

```http
GET /api/v1/books/{bookId}/positions
PUT /api/v1/books/{bookId}/positions/{deviceId}
```

### Highlights

```http
POST   /api/v1/books/{bookId}/highlights
GET    /api/v1/books/{bookId}/highlights
PATCH  /api/v1/highlights/{highlightId}
DELETE /api/v1/highlights/{highlightId}
```

Equivalent resources are added for notes, bookmarks, and tags.

### Synchronization

```http
POST /api/v1/sync/push
GET  /api/v1/sync/pull
GET  /api/v1/sync/bootstrap
```

`bootstrap` returns the complete current synchronized state for a fresh device. Large libraries may later make this endpoint paginated or snapshot-based.

## 11.2 API conventions

- DTOs are Java records where appropriate.
- Entities are never returned directly.
- Requests are validated at the boundary.
- Services enforce ownership and domain rules.
- Database transactions are defined in the application/service layer.
- Pagination uses opaque cursors where ordering can change.
- Mutations support idempotency when retrying is expected.
- Every conflict response includes enough information for the client to resolve it.
- Use `Instant` in Java and ISO-8601 UTC strings over the wire.

## 11.3 OpenAPI workflow

1. Backend exposes the API contract.
2. CI verifies the contract is generated and valid.
3. TypeScript client and types are generated into `packages/api-client`.
4. Web and mobile import the generated client.
5. Hand-written TypeScript types do not duplicate backend request/response contracts.

---

# 12. Web application plan

## 12.1 Main routes

```text
/login
/register
/library
/library/{bookId}
/read/{bookId}
/settings
/settings/devices
/settings/account
```

## 12.2 Main screens

### Authentication

- Register.
- Login.
- Session expiration handling.

### Library

- Upload book.
- Grid/list toggle.
- Processing state.
- Download/offline state.
- Search by title and author.
- Filter by format, tag, and reading status.
- Sort by recently opened, title, and date added.

### Reader

- Distraction-free reading area.
- Table of contents.
- Progress display.
- Search within supported books later.
- Highlight menu.
- Notes panel.
- Bookmarks.
- Reader settings.
- Sync indicator.
- Position conflict banner.

### Book details

- Cover and metadata.
- Reading progress.
- Highlights and notes.
- Tags.
- Download/export actions.
- Delete action with clear consequences.

## 12.3 Web offline strategy

The web application must be installable or at least offline-capable for previously downloaded books.

Research during the offline spike:

- IndexedDB for structured data.
- Cache Storage versus Origin Private File System for large book files.
- Browser quota behavior.
- Service worker update strategy.
- Recovery when browser storage is cleared.

The UI must distinguish:

- Stored on server.
- Downloaded on this browser.
- Pending local changes.

---

# 13. Mobile application plan

## 13.1 Navigation

- Authentication stack.
- Library tab.
- Reader screen.
- Notes/highlights tab or panel.
- Downloads.
- Settings and devices.

## 13.2 Local persistence

- SQLite for metadata, sync cursor, entities, and outbox.
- Filesystem storage for PDFs, EPUBs, and covers.
- Secure storage for refresh token or session secret.
- Memory-only access token.

## 13.3 Download manager

Support:

- Download.
- Pause/cancel where practical.
- Retry.
- File hash verification.
- Storage usage display.
- Remove local copy without deleting from server.
- Automatic cleanup policy later.

## 13.4 Android-first development

Because development occurs on Ubuntu:

- Build and test Android first.
- Keep all platform APIs abstracted.
- Avoid Android-only assumptions in shared packages.
- Add iOS build and test work before beta, not after v1.

---

# 14. UX requirements

## 14.1 Reading continuity

Opening a book should follow this decision order:

1. Current device’s latest local unsynchronized position.
2. User-selected resolved position.
3. Most recent synchronized device position.
4. Beginning of book.

The reader should open immediately from local data, then reconcile with the server without a visible jump unless the user chooses another position.

## 14.2 Progress conflict UX

Example banner:

```text
A newer position was found on “Web — Firefox”.
Page 124 · 62% · 18 minutes ago
[Go there] [Keep this position]
```

Rules:

- Do not show it for tiny differences.
- Do not repeatedly show the same resolved conflict.
- Never move the reader while the user is actively reading.
- Preserve both positions until the choice synchronizes.

## 14.3 Sync communication

Use calm, explicit language:

- “Available offline.”
- “3 changes will sync when you reconnect.”
- “Could not sync this note. Your local copy is safe.”
- “This note was edited on two devices.”

Avoid vague messages such as “Something went wrong.”

## 14.4 Accessibility

- Full keyboard navigation on web.
- Screen-reader labels and semantic controls.
- Sufficient contrast.
- Font scaling.
- Reduced-motion support.
- Touch targets suitable for mobile.
- Reader themes including light, dark, and paper-like options.
- Do not encode sync or annotation state using color alone.

---

# 15. Testing strategy

## 15.1 Backend

### Unit tests

Use for:

- Domain rules.
- Locator validation.
- Conflict policy.
- Token rotation logic.
- Storage key generation.

### Integration tests

Use PostgreSQL through Testcontainers for:

- Repositories.
- Flyway migrations.
- Transactions.
- Authentication flows.
- Sync cursor behavior.
- Idempotent mutation replay.
- Tombstones.
- Ownership isolation.

### HTTP tests

Cover:

- Request validation.
- Status codes.
- Problem Details.
- Authentication and authorization.
- Range requests.
- Multipart upload.

## 15.2 Sync tests

Create deterministic scenarios:

1. One online client.
2. Two online clients.
3. Client creates data offline and reconnects.
4. Same note edited on two offline clients.
5. Highlight deleted offline while edited elsewhere.
6. Refresh token replay.
7. Push batch retried after response loss.
8. Pull interrupted between pages.
9. Device starts from an old cursor.
10. Tombstone cleanup after inactive devices.

## 15.3 Reader tests

Maintain a legal test corpus containing:

- Small and large PDFs.
- PDFs with selectable text.
- Scanned PDFs with no text layer.
- Rotated pages.
- Multi-column documents.
- Reflowable EPUBs.
- EPUBs with images and unusual CSS.
- Right-to-left or non-Latin sample later.

Test:

- Position round trips.
- Highlight round trips.
- Font and viewport changes.
- Paged/continuous switches.
- Offline opening.
- Performance regressions.

## 15.4 Frontend tests

- Unit tests for pure sync and locator functions.
- Component tests for important UI states.
- Playwright for critical web flows.
- Mobile end-to-end tests added after the core mobile flow stabilizes.

## 15.5 Definition of done

A feature is complete only when:

- Its behavior is documented.
- Migration exists where required.
- Validation and error cases are handled.
- Ownership checks exist.
- Tests cover the important behavior.
- Offline and sync impact is considered.
- API contract is updated.
- Logs contain useful context without secrets.
- The UI has loading, empty, offline, and failure states.

---

# 16. Security and privacy plan

## 16.1 Baseline

- HTTPS in production.
- Passwords are never logged.
- Refresh tokens are never stored in plaintext server-side.
- Access tokens are short-lived.
- File access always validates ownership.
- CORS is restricted.
- Security headers are configured.
- Uploads are validated and isolated.
- SQL is parameterized through JPA or explicit safe queries.
- Secrets come from environment or mounted secret files.
- Production errors do not expose stack traces.

## 16.2 EPUB-specific security

EPUB contains HTML, CSS, and potentially active content. The renderer must:

- Disable or strip scripts by default.
- Isolate content in a sandboxed frame or WebView context.
- Restrict network requests.
- Apply a strict Content Security Policy.
- Block navigation to unexpected schemes.
- Treat external links as user-initiated actions.

## 16.3 Hosted-service privacy

Before public launch, publish:

- What file and reading data is stored.
- Retention after deletion.
- Backup retention.
- Whether administrators can technically access files.
- Export and account-deletion behavior.
- Telemetry behavior.

No analytics or telemetry should be enabled in self-hosted installations by default.

## 16.4 End-to-end encryption

Defer E2EE. It conflicts with server-side metadata extraction, web access, recovery, and background processing. Add it only through a dedicated architecture phase rather than claiming privacy properties the system does not provide.

---

# 17. Observability and operations

## 17.1 Initial operations

- Spring Boot Actuator health endpoints.
- Structured application logs.
- Request correlation ID.
- Startup validation for storage paths and database connectivity.
- Disk-space checks.
- Database migration status.
- Graceful shutdown.

## 17.2 Metrics

Later expose through Micrometer:

- Request count and latency.
- Authentication failures.
- Upload size and duration.
- Metadata-processing duration.
- Sync push/pull count and failures.
- Conflict count.
- Storage usage.
- Database pool health.

## 17.3 Backups

A self-hosted installation is not complete without documented backup and restore.

Backup scope:

- PostgreSQL.
- Book-data directory.
- Configuration and secrets.

Requirements:

- Consistent backup procedure.
- Restore procedure tested automatically or on a schedule.
- Clear compatibility rules between application and database versions.
- No claim that a backup works until a restore has been tested.

---

# 18. Open-source and hosted-service plan

## 18.1 Open-source repository requirements

Before public alpha:

- Clear README.
- Architecture overview.
- Local development setup.
- Self-hosting setup.
- Contribution guide.
- Code of conduct.
- Security policy.
- Issue templates.
- Pull request template.
- Release notes.
- Database migration policy.
- Backup and upgrade documentation.

## 18.2 License decision gate

The license must be decided before accepting external contributions.

Evaluate:

- Whether hosted competitors must publish modifications.
- Compatibility with mobile distribution.
- Whether client SDKs should use a more permissive license.
- Contributor license agreement versus Developer Certificate of Origin.
- Trademark policy for the BookSync name.

Do not casually choose MIT merely because it is common. The license should match the hosted-service strategy.

## 18.3 Hosted plan boundaries

The hosted plan sells operations and convenience:

- Managed deployment.
- Updates.
- Storage.
- Backups.
- Email delivery.
- Monitoring.
- Support.

Core reading and synchronization features should remain available for self-hosting. Commercial limits should focus on hosted resource consumption, not artificial removal of core functionality from the open-source product.

## 18.4 Initial hosted limits

Potential configurable limits:

- Total storage per account.
- Maximum file size.
- Number of active devices.
- Backup retention.

Billing is not part of the initial product. Validate the reader and synchronization first.

---

# 19. Development workflow

## 19.1 Git

- Trunk-based development with short-lived branches.
- Small commits.
- Conventional or clearly descriptive commit messages.
- Pull requests even when working alone for major changes, when review history is useful.
- No long-running “backend” or “frontend” branches.

## 19.2 Architecture Decision Records

Create an ADR for decisions that are expensive to reverse:

- Reader engine.
- Locator schema.
- Sync protocol.
- Authentication token model.
- File storage abstraction.
- License.
- Web offline storage.

ADR format:

```text
Title
Status
Context
Decision
Consequences
Alternatives considered
```

## 19.3 Code quality

Initial backend checks:

- Compile with warnings visible.
- Unit and integration tests.
- Formatting through a Maven plugin once Java 25 compatibility is confirmed.
- Dependency vulnerability scanning in CI.

Later:

- Static analysis.
- Architecture tests.
- Mutation testing for critical sync logic if useful.

## 19.4 Continuous integration

Backend pipeline:

1. Set up JDK 25.
2. Run formatting check.
3. Run `./mvnw verify`.
4. Start Testcontainers as required.
5. Generate and validate OpenAPI.
6. Build application image.

Frontend pipeline later:

1. Install with locked package manager.
2. Typecheck.
3. Lint.
4. Unit tests.
5. Build.
6. E2E smoke tests.

---

# 20. Roadmap

The roadmap is ordered by risk and learning value, not by how visually impressive a feature is.

## Phase 0 — Project foundation

### Goals

- Establish the repository.
- Install and understand the modern Java toolchain.
- Run the first Spring Boot application in VS Code.

### Work

- Install JDK 25 through a Java version manager or trusted distribution.
- Configure VS Code Java and Spring extensions.
- Generate Spring Boot 4.1 Maven project.
- Move it into `apps/api`.
- Add PostgreSQL Docker Compose.
- Add YAML configuration profiles.
- Add `/api/v1/health`.
- Add one Flyway migration.
- Add one Testcontainers integration test.
- Add README development commands.

### Java learning

- JDK/JVM/JRE.
- Packages.
- Maven wrapper and lifecycle.
- Classes and records.
- Basic annotations.
- Exceptions.
- JUnit.

### Exit criteria

- Fresh clone can start PostgreSQL and API.
- Health endpoint works.
- Test suite passes from terminal and VS Code.
- No globally installed Maven is required.

---

## Phase 1 — Java refresher through a small domain slice

### Goals

Learn Java language fundamentals without building disposable tutorial exercises.

### Work

Create an in-memory “library prototype” inside tests:

- `BookId` value type.
- `BookFormat` enum.
- `Book` model.
- Validation rules.
- Collection filtering and sorting.
- Errors for unsupported format and invalid metadata.

### Java learning

- Records.
- Enums.
- Sealed interfaces where useful.
- Generics and collections.
- Streams versus loops.
- `Optional`.
- `Instant`.
- Equality and immutability.

### Exit criteria

- Domain code is understandable without Spring.
- Tests demonstrate the language concepts.
- No persistence or HTTP concerns are mixed into domain types.

---

## Phase 2 — Reader feasibility spikes

### Goals

Remove the biggest product risk before building a large backend.

### Work

- Create small web and Expo spike applications.
- Complete PDF spike.
- Complete EPUB spike.
- Test on desktop browser and Android device/emulator.
- Write ADR selecting the initial rendering architecture.
- Finalize locator schema version 1.

### Learning

- Browser rendering and text layers.
- WebView/native bridges.
- File handling on mobile.
- Coordinate normalization.
- EPUB structure and CFI.

### Exit criteria

- PDF and EPUB position round-trip demos work.
- Highlights survive restart.
- Offline opening works.
- Renderer decision is documented.

---

## Phase 3 — Authentication vertical slice

### Goals

Implement a production-shaped login flow without OAuth complexity.

### Work

- User migration.
- Credentials migration.
- Device and session migrations.
- Register.
- Login.
- Refresh rotation.
- Logout.
- Current-user endpoint.
- Authentication filters.
- Authorization foundation.
- Integration tests for all token flows.

### Java/Spring learning

- Dependency injection.
- Controllers, DTO records, and validation.
- Spring Security filter chain.
- Password encoding.
- Transactions.
- JPA entities and repositories.
- Problem Details.

### Exit criteria

- A user can register and authenticate.
- Refresh-token replay is handled.
- Protected endpoints reject invalid access.
- Tests use real PostgreSQL.

---

## Phase 4 — Book upload and library vertical slice

### Goals

Upload a file, store it, list it, and download it.

### Work

- Storage abstraction.
- Local filesystem implementation.
- Book and file migrations.
- Multipart upload.
- SHA-256 calculation.
- PDF/EPUB validation.
- Per-user duplicate detection.
- Library endpoint.
- Book details endpoint.
- Authenticated download with Range and ETag.
- Delete with tombstone and delayed physical cleanup.

### Java/Spring learning

- Streaming I/O.
- Resource handling.
- Configuration properties.
- Transactions crossing database and filesystem boundaries.
- HTTP Range semantics.
- Integration testing temporary files.

### Exit criteria

- Two users cannot access each other’s books.
- File survives restart.
- Range download works.
- Duplicate handling is predictable.

---

## Phase 5 — Metadata and processing jobs

### Goals

Turn uploaded files into usable library items.

### Work

- Processing status.
- Database-backed job queue.
- PDF metadata extraction.
- EPUB metadata extraction.
- Cover extraction/generation.
- Retry and failure state.
- User-facing processing errors.

### Learning

- Scheduled work.
- Transaction claiming and locking.
- Idempotent background processing.
- File parsing safety.
- Operational logs.

### Exit criteria

- Upload returns a library item immediately.
- Processing finishes without blocking the client.
- Failed jobs can be retried.

---

## Phase 6 — Web library and authentication

### Goals

Create the first real user interface against the Java API.

### Work

- Next.js application.
- Generated API client.
- Register/login.
- Access-token refresh flow.
- Library list.
- Upload UI.
- Processing states.
- Book details.

### Exit criteria

- A user can create an account, upload a book, and see it in the browser.
- API errors are presented meaningfully.

---

## Phase 7 — PDF reader on web

### Goals

Deliver the first complete reading experience.

### Work

- PDF renderer integration.
- Continuous default.
- Paginated option.
- Current locator calculation.
- Local immediate progress persistence.
- Reader controls and preferences.
- Download/cache for offline use.

### Exit criteria

- Reopening returns to page and offset.
- Switching modes does not lose the canonical position.
- Previously downloaded PDF opens offline.

---

## Phase 8 — Mobile foundation and PDF reader

### Goals

Read the same PDF on Android.

### Work

- Expo development build.
- Authentication.
- SQLite schema.
- Secure session storage.
- Library screen.
- Download manager.
- PDF reader integration.
- Local progress.

### Exit criteria

- User logs in on Android.
- Downloads a PDF.
- Reads offline.
- Restores local position after restart.

---

## Phase 9 — Progress synchronization

### Goals

Fulfill the core promise across web and mobile.

### Work

- Devices API.
- Reading position endpoint.
- Sync change feed.
- Client outbox.
- Push and pull.
- Retry with backoff.
- Idempotency.
- Position conflict detection.
- Position conflict banner.

### Exit criteria

- Read on web, continue on mobile.
- Read offline on mobile, reconnect, continue on web.
- No progress is silently lost during two-device divergence.

This is the first true product milestone.

---

## Phase 10 — EPUB reader

### Goals

Add EPUB with the same continuity guarantees.

### Work

- EPUB renderer from selected spike.
- CFI/resource locator.
- Font and layout settings.
- Position synchronization.
- Offline download.
- Cross-viewport restore tests.

### Exit criteria

- EPUB position restores after font-size and viewport changes.
- Web and Android agree on a useful position.

---

## Phase 11 — Highlights and bookmarks

### Goals

Synchronize durable annotations.

### Work

- Highlight and bookmark persistence.
- PDF rectangle anchors.
- EPUB CFI/text quote anchors.
- Rendering stored highlights.
- Tombstones.
- Outbox synchronization.
- Export basic JSON/Markdown.

### Exit criteria

- Highlight on one device, view it on another.
- Highlight survives renderer restart and layout changes.
- Deletion does not resurrect from an offline device.

---

## Phase 12 — Notes and conflict resolution

### Goals

Support notes without silent overwrite.

### Work

- Location notes and highlight notes.
- Version checks.
- Conflict copies.
- Conflict resolution UI.
- Markdown/plain-text export.

### Exit criteria

- Concurrent edits preserve both versions.
- User can resolve conflict clearly.

---

## Phase 13 — Tags, collections, and library polish

### Goals

Make medium-sized personal libraries manageable.

### Work

- Tags.
- Filters and sorting.
- Reading status.
- Recently opened.
- Cover editing.
- Bulk operations where justified.

---

## Phase 14 — Self-hosted alpha

### Goals

Make installation and operation realistic for another person.

### Work

- Production Docker image.
- Docker Compose.
- Reverse-proxy examples.
- Persistent volumes.
- Environment validation.
- Backup and restore guide.
- Upgrade guide.
- Admin bootstrap.
- Storage quota configuration.
- Health checks.

### Exit criteria

- Another developer can install it using documentation only.
- Backup is restored successfully into a fresh installation.

---

## Phase 15 — Public open-source alpha

### Goals

Open the project safely to contributors and testers.

### Work

- License decision.
- Contribution guide.
- Security policy.
- Roadmap and issue labels.
- Example data.
- Architecture documentation.
- Automated releases.

### Exit criteria

- Repository can accept an external contribution.
- Known limitations are documented honestly.

---

## Phase 16 — Beta hardening

### Goals

Prepare for users who are not developers.

### Work

- Email verification and password recovery.
- Rate limiting.
- Session management UI.
- Data export.
- Account deletion.
- Better storage failure handling.
- iOS validation.
- Accessibility audit.
- Security review.
- Performance testing.
- Migration compatibility testing.

---

## Phase 17 — Optional hosted service

### Goals

Offer a managed service without forking the product.

### Work

- Hosted deployment.
- Account quotas.
- Email provider.
- Operational dashboards.
- Automated backups.
- Terms and privacy policy.
- Billing only after usage patterns are understood.

---

# 21. First implementation sequence

The immediate sequence after generating the project is intentionally small:

1. Generate Spring Boot 4.1 + Java 25 + Maven project.
2. Place it under `apps/api`.
3. Open it in VS Code.
4. Confirm `./mvnw test` and `./mvnw spring-boot:run`.
5. Add PostgreSQL through Docker Compose.
6. Add environment-based YAML configuration.
7. Add Flyway and the first migration.
8. Add a health endpoint.
9. Add a Testcontainers integration test.
10. Stop and review every file and concept before implementing authentication.

Do not create all domain tables on the first day. Each migration should belong to the feature being implemented.

---

# 22. Initial Spring Initializr configuration

Use:

```text
Project:        Maven
Language:       Java
Spring Boot:    4.1.0
Group:          dev.luismvl
Artifact:       booksync
Name:           booksync
Description:    Open-source cross-device reading and synchronization platform
Package name:   dev.luismvl.booksync
Packaging:      Jar
Configuration:  YAML
Java:           25
```

Initial dependencies:

- Spring Web.
- Spring Security.
- Spring Data JPA.
- Validation.
- Flyway Migration.
- PostgreSQL Driver.
- Spring Boot Actuator.
- Testcontainers dependencies can be added immediately or manually after generation.

Do not add OAuth client, WebFlux, Redis, messaging, GraphQL, Lombok, or native-image support yet.

---

# 23. Decision backlog

These decisions are deliberately postponed until evidence exists:

- PDF renderer on mobile.
- EPUB renderer.
- Shared WebView reader versus native renderers.
- IndexedDB wrapper and large-file browser storage.
- Exact OpenAPI generation tool.
- Exact Java formatter.
- Metadata extraction libraries.
- License.
- Hosted storage backend.
- OAuth provider.
- E2EE.
- Full-text search.

Every postponed decision should have a trigger: a phase, prototype result, or user need.

---

# 24. Risk register

## High risk

### Reader interoperability

The same locator may behave differently across renderers.

**Mitigation:** early spikes, multiple selectors, file hash binding, round-trip test corpus.

### Mobile text selection

PDF and EPUB selection inside WebView or native components may be inconsistent.

**Mitigation:** prototype before committing architecture.

### Offline synchronization

Retries and concurrent edits can duplicate or overwrite data.

**Mitigation:** operation IDs, idempotency, server cursor, tombstones, version checks, scenario tests.

### Browser storage limits

Large offline libraries may exceed browser quotas.

**Mitigation:** expose storage usage, make downloads explicit, investigate OPFS/Cache Storage, graceful eviction.

## Medium risk

### Large uploads

Connections may fail during upload.

**Mitigation:** start with bounded multipart uploads; add resumable uploads only after real need.

### File parsing security

EPUB is an archive containing web content.

**Mitigation:** archive limits, path validation, sandboxed rendering, CSP.

### Self-host backup inconsistency

Database and files may diverge.

**Mitigation:** documented coordinated backup and periodic orphan reconciliation.

### Scope expansion

Reader apps invite many attractive features.

**Mitigation:** non-goal list and milestone exit criteria.

---

# 25. Success criteria

## Technical alpha

- Web and Android clients.
- PDF and EPUB.
- Upload and offline download.
- Accurate cross-device progress.
- Highlights, notes, and bookmarks.
- Clear conflict handling.
- Docker Compose self-hosting.
- Backup and restore documentation.

## Product success

A user can:

1. Upload a PDF or EPUB on one device.
2. Open it on another device.
3. Read offline.
4. Resume close to the exact previous location.
5. Create a highlight and note.
6. See them on another device.
7. Understand what happens when two devices diverge.
8. Export their reading data.
9. Self-host without needing distributed infrastructure.

## Learning success

At the end of the project, the developer can:

- Start and structure a modern Spring Boot application.
- Explain dependency injection rather than only use annotations.
- Design REST APIs and validation.
- Model relational data and write migrations.
- Implement and test authentication.
- Handle files safely.
- Build reliable offline synchronization.
- Diagnose JVM and Spring behavior.
- Deploy and operate a Java service.
- Make architectural trade-offs without copying a tutorial blindly.

---

# 26. Working agreement

This plan is not a rigid tutorial. It is a living project map.

During implementation:

- Questions can interrupt any phase.
- Confusing Java syntax should be explained by comparison with TypeScript when useful.
- Every major dependency must have a clear reason.
- Decisions may change after a spike, but the change should be recorded.
- We build one working slice at a time.
- We do not skip understanding merely to produce more code.
- We also do not repeat beginner material that an experienced developer already understands.

The next action is Phase 0: generate and inspect the Spring Boot project before adding application code.
