# Epic 5: Advanced Filtering & Admin Tools

**Epic Goal:** Implement comprehensive filter panel with all filter types (Status, Health & Feeding, TNR, Behavior, Basic Info), complete admin verification functionality moved from Epic 1, and finalize remaining MVP features to enable pilot testing. This epic completes the feature set required for real-world volunteer coordination, enabling users to quickly find cats meeting specific criteria and admins to efficiently manage user verification.

### Story 5.1: Filter Panel UI Component

As a **user**,
I want to access a filter panel from the map screen,
so that I can filter colonies and cats by various criteria.

**Acceptance Criteria:**

1. Filter icon displayed in top-right corner of map screen (next to search bar)
2. Tapping filter icon opens filter panel as bottom sheet modal or side drawer
3. Filter panel displays categorized filter sections: Status, Health & Feeding, TNR Status, Behavior, Basic Info
4. Each section has collapsible/expandable header for organization
5. Filter panel has "Apply Filters" button at bottom (primary action)
6. Filter panel has "Clear All" button to reset all filters to defaults
7. Filter panel can be dismissed by tapping outside, close button, or pressing escape key
8. Active filters indicator shown on filter icon (badge with count of active filters)
9. Filter panel styled using Untitled UI components with smooth animations
10. Filter panel responsive on mobile and desktop

---

### Story 5.2: Status Filter Implementation

As a **user**,
I want to filter cats by their status,
so that I can focus on active cats or search for missing/adopted cats.

**Acceptance Criteria:**

1. Status filter section displays checkboxes for: Active, Adopted, Relocated, Missing, Deceased
2. Default selection: Active (checked), all others unchecked
3. Multiple statuses can be selected simultaneously
4. GET `/api/v1/colonies/filtered` endpoint accepts query param: `statuses` (array of status enum values)
5. Endpoint returns colonies that have at least one cat matching selected statuses
6. Frontend applies filter by fetching filtered colonies and updating map pins
7. Map displays only colonies matching filter criteria
8. Filter state persists during session (stored in React state or sessionStorage)
9. Colony count indicator shows: "[X] colonies match filters"
10. Applying Status filter works independently from search (per MVP scope clarification)

---

### Story 5.3: Health & Feeding Filter Implementation

As a **user**,
I want to filter cats by health alerts and feeding status,
so that I can identify cats needing medical attention or feeding.

**Acceptance Criteria:**

1. Health & Feeding filter section displays: Medical Alert toggle, Fed Today toggle
2. Medical Alert toggle filters cats where medicalAlert = true
3. Fed Today toggle filters cats where last feeding activity is within last 24 hours (rolling window per MVP scope)
4. GET `/api/v1/colonies/filtered` endpoint accepts query params: `medicalAlert` (boolean), `fedToday` (boolean)
5. Endpoint calculates Fed Today status: checks if most recent feeding activity timestamp for each cat is within `now() - 24 hours`
6. Backend efficiently queries activities to determine feeding status (indexed query on activityTimestamp)
7. Both toggles can be active simultaneously (AND logic: medical alert AND fed today)
8. Frontend applies filter and updates map with matching colonies
9. Filter indicator shows active health/feeding filters in badge
10. Empty result state message if no colonies match: "No colonies match your filters. Try adjusting criteria."

---

### Story 5.4: TNR Status Filter Implementation

As a **user**,
I want to filter cats by TNR status,
so that I can identify cats that need neutering or track TNR progress.

**Acceptance Criteria:**

1. TNR Status filter section displays checkboxes for: Neutered, Unneutered
2. Both checkboxes can be selected simultaneously
3. Neutered filter matches cats where CatTnrInfo.isNeutered = true
4. Unneutered filter matches cats where CatTnrInfo.isNeutered = false OR CatTnrInfo is null/unknown
5. GET `/api/v1/colonies/filtered` endpoint accepts query param: `tnrStatuses` (array: NEUTERED/UNNEUTERED)
6. Endpoint returns colonies with cats matching selected TNR statuses
7. Frontend applies filter and updates map
8. TNR filter can combine with other filters (Status, Health & Feeding)
9. Filter count badge includes TNR filter in count

---

### Story 5.5: Behavior Filter Implementation (MVP Free-Text)

As a **user**,
I want to filter cats by behavioral characteristics,
so that I can find cats with specific temperaments or behaviors.

**Acceptance Criteria:**

1. Behavior filter section displays text input field with placeholder: "Search behavioral notes" (MVP simplified per scope clarification)
2. GET `/api/v1/colonies/filtered` endpoint accepts query param: `behaviorSearch` (string)
3. Backend performs case-insensitive text search within Cat.behavioralNotes field (using SQL ILIKE or similar)
4. Returns colonies containing cats whose behavioral notes match search term
5. Frontend applies filter and updates map
6. Behavior search works alongside other filters (combined with AND logic)
7. Empty behavior search field means no behavior filtering applied
8. Future enhancement note documented in code comments: "Post-MVP: Replace with predefined temperament tags (Friendly, Shy, Aggressive, Feral, Playful, Cautious) per FR22"

---

### Story 5.6: Basic Info Filter Implementation (MVP Simplified)

As a **user**,
I want to filter cats by gender and age range,
so that I can find cats matching basic demographic criteria.

**Acceptance Criteria:**

1. Basic Info filter section displays: Gender checkboxes (Male, Female, Unknown), Age Range text input field
2. Multiple genders can be selected simultaneously
3. Age Range uses free-text search within Cat.ageRange field for MVP (per scope clarification)
4. GET `/api/v1/colonies/filtered` endpoint accepts query params: `genders` (array: MALE/FEMALE/UNKNOWN), `ageRangeSearch` (string)
5. Backend filters cats by selected genders (OR logic: Male OR Female)
6. Backend performs text search on ageRange field for age filtering
7. Both gender and age filters can combine (AND logic: matching gender AND age)
8. Frontend applies filter and updates map
9. Filter count badge includes gender and age filters
10. Future enhancement note documented: "Post-MVP: Replace age text search with structured Age Range options (Kitten 0-6mo, Young 6mo-2yr, Adult 2-7yr, Senior 7+yr) per FR23"

---

### Story 5.7: Apply Filters & Update Map Display

As a **user**,
I want to apply selected filters and see only matching colonies on the map,
so that I can focus on relevant locations.

**Acceptance Criteria:**

1. "Apply Filters" button in filter panel triggers filtered data fetch
2. All selected filters combined with AND logic (e.g., Active status AND Medical Alert AND Neutered)
3. GET `/api/v1/colonies/filtered` endpoint accepts all filter query params simultaneously
4. Backend constructs efficient SQL query with JOINs across Colony, Cat, CatHealthInfo, CatTnrInfo, Activity tables
5. Query optimized with proper indexes on filtered fields
6. Returns only colonies with at least one cat matching ALL filter criteria
7. Frontend updates map to display only filtered colony pins (hides non-matching colonies)
8. Filter panel closes after applying filters
9. Active filters summary shown on map: "[X] colonies match filters | Clear Filters" link
10. "Clear Filters" link resets all filters to defaults (Active status only) and refreshes map

---

### Story 5.8: Filter State Persistence

As a **user**,
I want my filter selections to persist during my session,
so that I don't have to re-select filters after navigating away from the map.

**Acceptance Criteria:**

1. Filter state stored in React Context or sessionStorage when filters applied
2. Navigating to Colony Details or Cat Profile and returning to map preserves filter state
3. Map loads with previously applied filters active (not default filters)
4. Active filters indicator badge persists across navigation
5. Page refresh within session restores filter state from sessionStorage
6. Logging out clears filter state
7. Filter state includes: all selected statuses, toggles (medicalAlert, fedToday), TNR statuses, behavior search text, genders, age search text
8. State restoration applies filters automatically on map mount

---

### Story 5.9: Filter and Search Independence (MVP Scope)

As a **user**,
I want to understand that search and filters work independently,
so that I have clear expectations about platform behavior.

**Acceptance Criteria:**

1. Applying filters clears any active search query (per MVP scope clarification: mutually exclusive modes)
2. Performing search clears any active filters (resets to default Active status filter)
3. UI provides clear indication when switching modes: "Filters cleared - now searching" or "Search cleared - now filtering"
4. Search bar disabled or grayed out when filters are active (with tooltip: "Clear filters to search")
5. Filter icon disabled or grayed out when search is active (with tooltip: "Clear search to filter")
6. User can manually clear search or filters to switch modes
7. Documentation/help text explains: "Search and filters work independently in this version. Use search to find specific names/locations, or use filters to narrow by criteria."
8. Future enhancement note: "Post-MVP: Combine search and filters for complex queries (FR15 full implementation)"

---

### Story 5.10: Fed Today Calculation & Display (MVP Rolling Window)

As a **user**,
I want to see which cats were fed today,
so that I can coordinate feeding and avoid duplicate efforts.

**Acceptance Criteria:**

1. Fed Today status calculated as: most recent feeding activity (category=FEEDING) timestamp is within last 24 hours from current time (rolling window per MVP scope clarification)
2. Backend query: `SELECT * FROM activities WHERE catId = ? AND category = 'FEEDING' ORDER BY activityTimestamp DESC LIMIT 1` and check if timestamp >= `now() - 24 hours`
3. Colony Details page displays "Last Fed" badge for each cat: "Fed 2 hours ago" (green) or "Not fed today" (amber/gray)
4. Cat Profile page displays "Last Fed" section with timestamp and color coding
5. Fed Today filter uses same rolling 24-hour logic
6. Status updates in real-time on page refresh (no caching of Fed Today status)
7. Future enhancement note: "Post-MVP: Replace rolling window with timezone-aware day boundary calculation (GMT+8 midnight reset) per FR20 full implementation"
8. Badge colors: Green if fed within 24 hours, amber if 24-48 hours, red if 48+ hours (visual indicator of urgency)

---

**Epic 5 Complete:** At this point, users can apply comprehensive filters (Status, Health & Feeding, TNR, Behavior, Basic Info) to find specific cats and colonies, with filter state persisting across sessions. Fed Today status accurately reflects recent feeding activities using rolling 24-hour calculation. Search and filters operate independently per MVP scope. All core MVP features are implemented and ready for pilot testing.

---
