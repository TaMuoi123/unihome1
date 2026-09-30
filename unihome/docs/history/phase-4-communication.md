# UniHome — Phase 4: Communication System

> **Document Type:** Phase History / Specification  
> **Status:** ⏳ Not Started  
> **Target Duration:** ~3 weeks  
> **Maintained By:** Module Owner (@dev-4) & Technical Lead  
> **Last Updated:** 2026-09-30  

---

## 1. Phase Goal

Build an integrated in-app messaging system facilitating direct communication between students and landlords (as well as matched roommates). Crucially, this phase generates high-intent behavioral signals (`contact_landlord`, `message_sent`) that power the Recommendation Engine in Phase 5.

---

## 2. Strategic Placement Rationale

Communication is intentionally scheduled **before** Recommendation (Phase 5).  
**Why:** Recommendation engines require rich user behavioral signals. Passive signals (views, searches) represent low-intent browsing. High-intent signals:
- `contact_landlord` (initiating inquiry for a specific room)
- `message_sent` (active engagement in conversation thread)

By building the communication module first, Phase 5 launches with high-quality interaction datasets from day one.

---

## 3. Team Allocation & Responsibilities

| Area | Task | Owner | Key Deliverables |
|---|---|---|---|
| Database | Schema migration for `wp_unihome_conversations` & `wp_unihome_messages` | Dev 4 | Tables with indices on `(conversation_id, created_at)` and participant IDs |
| Backend | `MessageService` — conversation thread lifecycle & message storage | Dev 4 | Thread creation, unread count caching, pagination, message dispatch |
| Backend | `ChatController` — REST API endpoints | Dev 1 | Message sending, inbox retrieval, conversation history REST handlers |
| Backend | Notification — `new_message` event dispatch | Dev 2 | Trigger `NotificationService::notify()` for email/in-app alert |
| Backend | Analytics Tracking — behavioral event emission | Dev 2 | Integrate `EventTracker::track('contact_landlord')` and `'message_sent'` |
| Frontend | Chat interface — thread view & messaging drawer | Dev 3 | Real-time-feeling message feed, auto-scroll, unread badge counter |
| Frontend | Room detail integration — "Contact Landlord" CTA modal | Dev 3 | Direct listing inquiry modal pre-filling room details |
| Testing | MessageService unit & integration tests | Dev 1 | Permission boundary tests, participant isolation, unread counter tests |

---

## 4. REST API Endpoints

| Method | Endpoint | Capability / Auth | Description |
|---|---|---|---|
| `GET` | `/wp-json/unihome/v1/conversations` | Authenticated (`send_messages`) | List user's active conversation threads with latest message & unread count |
| `POST` | `/wp-json/unihome/v1/conversations` | Authenticated (`send_messages`) | Start a new conversation (optionally linked to a `room_id`) |
| `GET` | `/wp-json/unihome/v1/conversations/(?P<id>\d+)/messages` | Authenticated (Participant only) | Paginated message history for a conversation |
| `POST` | `/wp-json/unihome/v1/conversations/(?P<id>\d+)/messages` | Authenticated (Participant only) | Send message in thread; tracks `message_sent` |
| `POST` | `/wp-json/unihome/v1/conversations/(?P<id>\d+)/read` | Authenticated (Participant only) | Mark conversation messages as read |

---

## 5. Behavioral Event Emission

Whenever messaging actions occur, the following events must be emitted via `EventTracker`:

```php
// On conversation initiation from room page:
EventTracker::track('contact_landlord', [
    'room_id'     => $roomId,
    'landlord_id' => $landlordId,
    'user_id'     => $currentUserId,
]);

// On every message sent:
EventTracker::track('message_sent', [
    'conversation_id' => $conversationId,
    'sender_id'       => $currentUserId,
    'recipient_id'    => $recipientId,
]);
```

---

## 6. Completion Criteria & Definition of Done

- [ ] Tables `wp_unihome_conversations` and `wp_unihome_messages` created with appropriate indexing.
- [ ] Students can initiate a conversation directly from any room detail listing page.
- [ ] Users can view conversation lists with accurate unread message counts.
- [ ] Participants can only read/write messages in conversations where they are a registered participant (strict access control).
- [ ] `NotificationService::notify('new_message', ...)` sends alerts when recipient is offline.
- [ ] `contact_landlord` and `message_sent` events reliably write to `wp_unihome_events`.
- [ ] Unit tests ensure non-participants cannot access or inject messages into arbitrary conversations.

---

## 7. Phase Outcomes & Retrospective

*(This section will be populated upon completion of Phase 4)*

### Actual Completion Date:
TBD

### Delivered Changes:
- TBD

### Deviations from Plan:
- TBD

### Lessons Learned:
- TBD
