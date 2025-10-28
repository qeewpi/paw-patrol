# Epic 4: Activity Logging & Search

**Epic Goal:** Implement comprehensive activity logging system for colonies and cats with category-based actions (Feeding, Health, Environment), photo attachments, and timestamp tracking. Add global search functionality across colonies, cats, and locations with proximity-based results. This epic delivers the coordination and record-keeping capabilities that enable volunteers to track feeding schedules, health interventions, and maintenance activities while finding colonies and cats quickly.

### Story 4.1: Activity Entity & Data Model

As a **developer**,
I want to create the Activity entity with category, action, and association fields,
so that the system can store activity logs for colonies and cats.

**Acceptance Criteria:**

1. `Activity` entity created extending base audit entity with fields: category (FEEDING/HEALTH/ENVIRONMENT enum), action (string), notes (text), activityTimestamp (datetime), colonyId (foreign key, nullable), catId (foreign key, nullable), loggedByUserId (foreign key to User)
2. `ActivityPhoto` entity created with fields: activityId (foreign key), filename, filepath, uploadedAt
3. Activity entity has one-to-many relationship with ActivityPhoto (lazy loading)
4. Database constraints: either colonyId OR catId must be set (not both null), activityTimestamp defaults to now, action max 100 characters
5. Database indexes created on: colonyId, catId, activityTimestamp, category for efficient querying
6. JPA repositories created for Activity and ActivityPhoto entities
7. Database migration script creates Activity and ActivityPhoto tables with proper foreign keys and indexes
8. Activity logs are immutable (no UPDATE/DELETE operations) per MVP scope clarification

---

### Story 4.2: Log Colony-Level Activity

As a **user**,
I want to log activities for an entire colony,
so that I can record feeding, maintenance, or health interventions affecting all cats.

**Acceptance Criteria:**

1. POST `/api/v1/colonies/{colonyId}/activities` endpoint accepts: category (enum), action (string), notes (optional), activityTimestamp (optional, defaults to now), photos (optional multipart files, max 3)
2. Endpoint validates: category is valid enum, action not empty, photos meet validation rules if provided
3. Action dropdown options conditional on category: Feeding (Fed Wet Food, Fed Dry Food, Fed Treats, Refilled Water), Health (Gave Medication, Wound Treatment, Vet Visit, Vaccination, Health Check, Trapped for TNR, Spay/Neuter Surgery), Environment (Cleaned Area, Refilled Water Station, Shelter Maintenance, Waste Removal)
4. Activity created with colonyId set, catId=null, loggedByUserId set to authenticated user
5. ActivityPhoto records created for uploaded photos (optimized and stored like other photos)
6. Returns 201 Created with activity ID and full activity data
7. Returns 400 Bad Request for validation failures
8. Colony Details page displays "Log Activity" FAB button (bottom-right)
9. Tapping FAB opens activity logging modal with: category dropdown, action dropdown (conditional on category), notes text area, photo upload (max 3), timestamp picker (defaults to now)
10. Modal meets NFR2: submission without photos completes in under 2 seconds
11. On successful creation (201 response), modal closes, activity appears in colony activity log, success message shown

---

### Story 4.3: Log Cat-Specific Activity

As a **user**,
I want to log activities for individual cats,
so that I can track cat-specific feeding, health, or behavioral events.

**Acceptance Criteria:**

1. POST `/api/v1/cats/{catId}/activities` endpoint accepts: category, action, notes, activityTimestamp, photos (same structure as colony activities)
2. Endpoint validates inputs identically to colony activities
3. Activity created with catId set, colonyId=null, loggedByUserId set to authenticated user
4. Returns 201 Created with activity data
5. Cat Profile page displays "Log Activity" FAB button
6. Tapping FAB opens activity logging modal (identical UI to colony activities)
7. Modal meets NFR3: submission with photos completes in under 5 seconds
8. On successful creation (201 response), modal closes, activity appears in cat activity log, success message shown
9. Cat-specific activities also appear in parent colony's activity log (see Story 4.5)

---

### Story 4.4: Display Activity Logs on Colony Details

As a **user**,
I want to view all activities for a colony in chronological order,
so that I can see the history of care and interventions.

**Acceptance Criteria:**

1. GET `/api/v1/colonies/{colonyId}/activities` endpoint returns activities with: id, category, action, notes, activityTimestamp, loggedByUser (name), photoUrls, catName (if cat-specific activity)
2. Endpoint returns both colony-level activities (colonyId matches) AND cat-specific activities (catId matches any cat in colony)
3. Results ordered by activityTimestamp descending (most recent first)
4. Endpoint supports pagination: query params `page` (default 0) and `size` (default 50)
5. Returns paginated response with total count
6. Colony Details page displays activity log section below cat list
7. Activity log rendered as chronological feed with cards showing: timestamp (relative: "2 hours ago"), action, logged by user, notes (truncated with "Read more"), photos (thumbnail grid), cat name badge if cat-specific
8. Activity log initially loads first page (50 items)
9. "Load More" button at bottom fetches next page
10. Empty state shown if no activities: "No activities logged yet. Tap the + button to log an activity."

---

### Story 4.5: Display Activity Logs on Cat Profile

As a **user**,
I want to view all activities for a specific cat,
so that I can see that cat's individual care history.

**Acceptance Criteria:**

1. GET `/api/v1/cats/{catId}/activities` endpoint returns activities where catId matches
2. Results ordered by activityTimestamp descending
3. Supports pagination identical to colony activities
4. Cat Profile page displays activity log section below profile sections
5. Activity log rendered identically to colony activity log (cards with timestamp, action, notes, photos, logged by user)
6. No cat name badge shown (already on cat's profile)
7. "Load More" button for pagination
8. Empty state: "No activities logged for this cat yet."

---

### Story 4.6: Activity Log Filtering by Date Range

As a **user**,
I want to filter activity logs by date range,
so that I can focus on recent activities or specific time periods.

**Acceptance Criteria:**

1. GET `/api/v1/colonies/{colonyId}/activities` and `/api/v1/cats/{catId}/activities` endpoints accept query params: `filterType` (TODAY/ALL_TIME/DATE_RANGE), `startDate` (ISO date), `endDate` (ISO date)
2. TODAY filter: returns activities where activityTimestamp is within current day (00:00 to 23:59 today)
3. DATE_RANGE filter: returns activities between startDate and endDate (inclusive)
4. ALL_TIME filter: returns all activities (no date filtering, default behavior)
5. Endpoints validate: startDate before endDate if DATE_RANGE specified
6. Activity log sections have filter dropdown at top: "Today", "All Time", "Custom Range"
7. Selecting "Custom Range" opens date picker modal with start and end date inputs
8. Applying filter updates activity log display to show only matching activities
9. Filter state persists during session (doesn't reset on page refresh)
10. Activity count badge shows: "[X] activities [filter description]" (e.g., "12 activities today")

---

### Story 4.7: Activity Photo Display & Gallery View

As a **user**,
I want to view photos attached to activity logs,
so that I can see visual documentation of care activities.

**Acceptance Criteria:**

1. Activity cards in activity logs display photo thumbnails as grid (max 3 photos shown inline)
2. Tapping photo thumbnail opens full-resolution gallery view modal
3. Gallery view supports swipe navigation between photos in that activity
4. Gallery view displays photo caption (if any) and activity context (action, timestamp)
5. Gallery view has close button (X) and back gesture support
6. Photos lazy-loaded: thumbnails load first, full resolution on gallery open
7. Photo optimization follows same rules as cat/colony photos (max 5MB upload, compressed to 80% quality)
8. Activity cards show "+[n] more" indicator if more than 3 photos attached

---

### Story 4.8: Last Fed Timestamp Calculation & Display

As a **user**,
I want to see when a colony was last fed,
so that I can prioritize feeding visits.

**Acceptance Criteria:**

1. GET `/api/v1/colonies/{colonyId}/last-fed` endpoint calculates most recent feeding activity timestamp for colony (colony-level OR cat-level feeding activities)
2. Endpoint checks activities where category=FEEDING and activityTimestamp is most recent
3. Returns: timestamp, timeSince (relative string: "2 hours ago"), user who logged it
4. Returns null if no feeding activities exist
5. Colony Details page header displays "Last Fed" section prominently: "Last Fed: 8:30 AM (2 hours ago) by [User Name]"
6. Color-coded based on recency: green if <12 hours, amber if 12-24 hours, red if >24 hours
7. Shows "Never fed" if no feeding activities
8. Updates when new feeding activity logged (refreshes on page load)

---

### Story 4.9: Fed Today Status Calculation (MVP Simplified)

As a **developer**,
I want to calculate "Fed Today" status using rolling 24-hour window,
so that users can filter colonies by feeding recency (MVP simplified per scope clarifications).

**Acceptance Criteria:**

1. Backend service method `isFedToday(colonyId)` checks if most recent feeding activity timestamp is within last 24 hours from current time
2. Calculation: `now() - mostRecentFeedingTimestamp < 24 hours`
3. No timezone adjustment (rolling window, not day boundary)
4. Returns boolean: true if fed within 24 hours, false otherwise
5. Method used by filter functionality in Epic 5
6. Colony list API includes `fedToday` boolean field in response
7. Post-MVP: Upgrade to timezone-aware day boundary calculation using Philippines GMT+8

---

### Story 4.10: Global Search - Backend Implementation

As a **developer**,
I want to implement search across colonies, cats, and locations,
so that users can find entities by name or address.

**Acceptance Criteria:**

1. GET `/api/v1/search` endpoint accepts query params: `q` (search query, required), `userLatitude` (optional), `userLongitude` (optional)
2. Endpoint searches: colony names (LIKE '%query%'), cat names (LIKE '%query%'), colony addresses (LIKE '%query%')
3. Returns combined results array with: type (COLONY/CAT), id, name, colonyId (for cats), colonName (for cats), address (for colonies), photoUrl, distance (if user location provided)
4. Distance calculated using Haversine formula from user location to colony location (straight-line distance)
5. Results ordered by distance ascending (nearest first) if user location provided, otherwise alphabetical by name
6. Results limited to 50 items max for performance
7. Requires authentication
8. Returns 400 if query string empty or too short (<2 characters)

---

### Story 4.11: Global Search - Frontend Implementation

As a **user**,
I want to search for colonies and cats from the map screen,
so that I can quickly find specific locations or animals.

**Acceptance Criteria:**

1. Map screen displays persistent search bar at top center (always visible)
2. Search bar has placeholder text: "Search colonies, cats, or locations..."
3. Typing in search bar (debounced 300ms) triggers search API call with user's current location
4. Search results displayed as overlay panel/modal showing: result type icon (colony/cat), name, address/colony name, distance indicator ("1.2 km away"), photo thumbnail
5. Results list scrollable, ordered by proximity
6. Tapping result navigates to: Colony Details (if colony), Cat Profile (if cat)
7. Search can be cleared via X icon in search bar
8. Empty state if no results: "No matches found for '[query]'"
9. Search overlay dismissible by tapping outside or pressing escape
10. Search state cleared when navigating away from map

---

### Story 4.12: Search Results Navigation

As a **user**,
I want to navigate to search results without leaving search mode,
so that I can explore multiple results before exiting search.

**Acceptance Criteria:**

1. Search results remain visible after tapping a result
2. Tapping result opens Colony Details or Cat Profile in new browser tab/window (desktop) or navigates normally (mobile)
3. User can tap multiple results to explore options
4. "Close Search" button at top of results overlay dismisses search and clears query
5. Back button from Colony/Cat Details returns to map with search cleared
6. Search results cache persists during session (same query returns cached results for 5 minutes)

---

### Story 4.13: System Audit Events Generation

As a **developer**,
I want to generate system audit events for key actions,
so that activity logs include system-generated events alongside user actions.

**Acceptance Criteria:**

1. System audit events created as Activity records with special category: SYSTEM_EVENT
2. Audit events generated for: Colony Created, Cat Added to Colony, Cat Profile Edited (major fields), Cat Status Changed
3. Events have standardized action format: "[Event Type] - [Details]" (e.g., "Cat Added - Fluffy")
4. Events have loggedByUserId set to user who triggered action
5. Events appear in activity logs alongside user-logged activities
6. Events styled differently in UI: gray background, system icon, "System Event" badge
7. Events not editable or deletable (immutable like all MVP activities)
8. Events include relevant context in notes field (e.g., "Status changed from Active to Adopted")

---

**Epic 4 Complete:** At this point, users can log activities for colonies and cats, view activity logs with filtering, see last fed timestamps, search globally for colonies/cats/locations, and navigate to results. System audit events automatically track key actions. Core coordination and discovery features are functional. Ready for Epic 5 to add advanced filtering and complete the MVP.
