# Requirements — Rename event title (L-02)

**Student:** Yael Juliard  **Date:** 2026-10-21  **Fork URL:** https://github.com/Yavigail/event-photo-capstone

## 1. Original request (do not reinterpret)
> I entered the wrong event name. I want to be able to change the event name displayed in the client's gallery.

## 2. Current behavior — verified facts
| Claim | Evidence: file/function/line or HTTP result | Verified? |
| --- | --- | --- |
| Event title exists in data model | `src/domain/types.ts` line 11: `title: string;` inside `EventRecord`. Seed data: `src/data/memory-store.ts` lines 8 and 21 ("Cohen Wedding", "Levi Family Portraits"). | Yes |
| Current client gallery presents title | `src/services/event-service.ts` `getClientGallery`, line 25: `title: event.title`. Served by `GET /client/events/:eventId/gallery` (`src/http/app.ts` lines 27-33). | Yes |
| No rename endpoint exists in inspected route file | `src/http/app.ts` defines only `GET /health`, `GET /api/events`, `GET /api/events/:eventId/gallery`, `GET /client/events/:eventId/gallery`, `POST /api/events/:eventId/selections`. No PUT/PATCH for events. | Yes |
| No service method changes a title | `src/services/event-service.ts` has `listEvents`, `getGallery`, `getClientGallery`, `submitSelection`, `getEventForPhotographer`. None writes `title`. | Yes |
| Data is held in memory only | `src/data/memory-store.ts`: `MemoryStore` with arrays, `export const store = new MemoryStore()`. A change is lost on server restart. | Yes |
| Authorization is a demo header, not real auth | `src/http/app.ts` line 12 comment; photographer identified by `x-photographer-id`, client by `x-client-id`. | Yes |

## 3. Actors / users / use cases / permissions
- Verified:
  - Photographer (`photographerId`, header `x-photographer-id`): lists events and views galleries of own events only (`findEventForPhotographer`, `memory-store.ts` lines 33-37).
  - Client (`clientId`, header `x-client-id`): views gallery and submits selections for own event only (`findClientEvent`, lines 39-41).
  - The event record links one photographer and one client (`types.ts` lines 10 and 12).
- Must clarify:
  - Who may rename: the photographer only, or also the client? The request says "I entered the wrong event name", which suggests the person who created the event (photographer), but this is not stated.
  - Whether the person writing the request is the photographer or an admin.

## 4. Goal, scope, out of scope
- Goal: let an authorized user correct the event title so the corrected title appears in the client gallery.
- In scope (confirmed): changing the `title` of an existing event; the new title shown in the client gallery.
- Out of scope (confirmed by lesson boundaries, slide 13): screen design, choosing an API, writing code. Nothing else was stated by the requester.
- Undecided: whether other views (photographer list and gallery) should also show the new title; title history; notifying the client; persistence beyond memory.

## 5. Functional requirements
- FR-1 (provisional): An authorized requester can submit a new title for an existing event.
- FR-2 (provisional): After a successful change, `GET /client/events/:eventId/gallery` returns the new title.
- FR-3 (provisional): The change is rejected for an unknown event, an unauthorized requester, or an empty title.

## 6. Non-functional requirements / constraints
- NFR-1: Unresolved. No performance, audit or persistence requirement was given.
- Constraint (verified): storage is in-memory only, so the new title is lost on restart.
- Constraint (verified): real authentication is intentionally absent (`app.ts` line 12); only demo header identity exists.

## 7. Ten analysis lenses from the preparation assignment
| Term | Question relevant to this change | Answer known? / evidence |
| --- | --- | --- |
| functional requirements | What must the system do to rename? | Provisional FR-1 to FR-3; no rename exists today (`app.ts`). |
| non-functional requirements | Is persistence, audit or max length required? | Unknown. Store is in-memory. |
| actors / users | Who renames and who sees the result? | Photographer and client exist; who renames is unknown. |
| use cases | Photographer fixes a typo; client sees corrected name. | Partly known from the request. |
| constraints | What limits the solution? | In-memory store; demo identity only. |
| assumptions | What are we assuming? | That the photographer is the one renaming (not confirmed). |
| acceptance criteria | How do we know it works? | Section 9 (provisional). |
| ambiguity | What is unclear in the request? | "Event name" vs `title`; who may rename; which views change. |
| missing information | What do we not know? | Title rules (length, empty), history, client notification. |
| scope | What is in and out? | Only the title change is in scope; the rest is undecided. |

## 8. Questions, assumptions, ambiguities
| Question or assumption | Why it matters | Evidence/status | Who decides? |
| --- | --- | --- | --- |
| "Event name" means the `title` field | Defines what changes | Assumed; only `title` exists in `EventRecord` (`types.ts` line 11) | Requester |
| Who may rename (photographer, client, both)? | Determines the permission check | Open | Product owner |
| Should the photographer views show the new title too? | `listEvents` and `getGallery` also return `title` | Open (they read the same field, so they would change automatically) | Product owner |
| Title validation (empty, max length, duplicates)? | Prevents bad data | Open | Product owner |
| Must the change survive a restart? | Store is in-memory | Open | Product owner / team |
| Should a published gallery allow renaming? | `galleryPublished` flag exists | Open | Product owner |

## 9. Acceptance criteria (provisional if decisions unresolved)
- Given an event owned by the requesting photographer, when a valid new title is submitted, then the event title is updated.
- Given the title was updated, when the client opens `GET /client/events/:eventId/gallery`, then the response shows the new title.
- Given an unknown event id, when a rename is submitted, then the response is 404.
- Given a requester who does not own the event, when a rename is submitted, then the change is rejected and the title is unchanged.
- Given an empty or whitespace-only title, when a rename is submitted, then the change is rejected with an error.

## 10. Error cases and tests
- Unknown event: expect 404 (same pattern as `app.ts` lines 23 and 31).
- Unauthorized requester: missing identity expect 401 (pattern at `app.ts` lines 15, 21); someone else's event expect not found/rejected, as in the existing tests "does not return another photographer's event".
- Empty title: expect 400 with an error message (pattern at `app.ts` line 41).
- Existing client gallery after change: title is new, `photos` and `availableServices` unchanged.

## 11. AI checkpoints: understanding / questioning / spec review
Verified claim in source: there is no rename route in `src/http/app.ts` (only five routes, none that updates an event), and `title` is read in `getClientGallery` at `event-service.ts` line 25.

### AI question and spec review suggestions — reviewed critically
| AI suggestion | Essential / later / irrelevant / unverified | Justification | Evidence |
| --- | --- | --- | --- |
| Check that no endpoint or service method already changes the title | Essential | Confirms the feature is really missing before specifying it | `app.ts` routes and `event-service.ts` methods: none update `title` |
| Ask who is allowed to rename (photographer vs client) | Essential | Affects the permission rule and the tests | Two identities exist: `x-photographer-id`, `x-client-id` |
| Reject empty titles | Essential | Prevents a blank name in the client gallery | Existing 400 validation pattern, `app.ts` line 41 |
| Add title change history / audit log | Later | Not requested; no storage for history exists | Not in `types.ts`; request does not mention it |
| Notify the client by email when the title changes | Irrelevant for now | Not requested; no notification mechanism in the code | No such code found in `src` |
| Persist the title in a database | Unverified / open | Depends on whether data must survive restarts; current store is in-memory | `memory-store.ts` |

## 12. Remaining decisions (do not silently invent answers)
- Who may rename an event.
- Title rules: empty, maximum length, duplicates.
- Whether renaming is allowed after the gallery is published.
- Whether the change must persist across restarts.
- Whether the client is notified.

## 13. Personal reflection
- One fact discovered through code review: the title is stored once in `EventRecord` and read by three methods (`listEvents`, `getGallery`, `getClientGallery`), so a rename would show in the photographer views as well, not only in the client gallery.
- One AI suggestion you rejected or postponed, and why: postponed the audit log of title changes, because the request does not ask for history and the code has nowhere to store it.