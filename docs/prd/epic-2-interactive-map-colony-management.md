# Epic 2: Interactive Map & Colony Management

**Epic Goal:** Build the core interactive map interface with Google Maps integration, implement colony creation, viewing, editing, and deletion workflows, and establish the primary user navigation pattern (map → colony details). This epic delivers the central hub of the application where users can visualize colony locations, navigate to colony details, and manage colony data, enabling geolocation tracking of cat colonies.

### Story 2.1: Colony Entity & Data Model

As a **developer**,
I want to create the Colony entity with location and metadata fields,
so that the system can store and manage colony information.

**Acceptance Criteria:**

1. `Colony` entity created extending base audit entity with fields: name (required), latitude (decimal), longitude (decimal), address (optional), creatorUserId (foreign key to User)
2. Database constraints enforce: latitude range -90 to 90, longitude range -180 to 180, name max 200 characters
3. Database index created on latitude and longitude fields for efficient spatial queries
4. `ColonyPhoto` entity created with fields: colonyId (foreign key), filename, filepath, isPrimary (boolean), uploadedAt
5. Colony entity has one-to-many relationship with ColonyPhoto (lazy loading)
6. JPA repository created for Colony entity with custom query methods: findAllByDeletedFalse, findByIdAndDeletedFalse
7. Database migration script creates Colony and ColonyPhoto tables with proper foreign keys and indexes

---

### Story 2.2: Google Maps Integration & Map Component

As a **user**,
I want to see an interactive map on the main screen,
so that I can visualize colony locations and navigate the platform.

**Acceptance Criteria:**

1. Google Maps JavaScript API key configured in frontend environment variables
2. React map component created using `@react-google-maps/api` or similar wrapper library
3. Map loads centered on user's current location with zoom level 13 (neighborhood view)
4. Browser geolocation API used to get user's current location (with permission prompt)
5. If location permission denied or unavailable, map defaults to Manila, Philippines (14.5995° N, 120.9842° E)
6. Map restricted to Philippines geographic boundaries (hard bounds: approximately 4.5° N to 21° N, 116° E to 127° E)
7. Map displays user location indicator (blue dot with accuracy circle)
8. Map supports pan and zoom controls (default Google Maps UI)
9. Map placeholder/loading state shown while map initializes
10. Map component meets NFR1: loads in under 3 seconds on 4G connection

---

### Story 2.3: Display Colony Pins on Map

As a **user**,
I want to see colony locations as pins on the map,
so that I can identify where colonies are located.

**Acceptance Criteria:**

1. GET `/api/v1/colonies` endpoint returns all non-deleted colonies with fields: id, name, latitude, longitude, primaryPhotoUrl (if exists), catCount
2. Endpoint requires authentication
3. Frontend fetches colony data on map component mount
4. Colony pins rendered on map at their latitude/longitude coordinates
5. Pin markers use circular photo thumbnail if colony has primary photo, otherwise use default cat icon
6. Map marker clustering implemented using Google Maps MarkerClusterer for dense colony areas (improves performance)
7. Markers are clickable (tap/click event handled)
8. Map automatically adjusts bounds to show all colony pins on initial load (if colonies exist)
9. Skeleton loading state shown while colony data is fetching

---

### Story 2.4: Create Colony via Button & Location Capture

As a **user**,
I want to create a new colony by tapping the '+' button,
so that I can add colonies I discover.

**Acceptance Criteria:**

1. POST `/api/v1/colonies` endpoint accepts: name (required), latitude (required), longitude (required), address (optional)
2. Endpoint validates: duplicate prevention (no colony within 1 meter radius using Haversine formula), latitude/longitude ranges, name not empty
3. Colony created with creatorUserId set to authenticated user's ID
4. Returns 201 Created with colony ID and full colony data
5. Returns 400 Bad Request for validation failures
6. Returns 409 Conflict if duplicate colony detected (within 1 meter)
7. Frontend displays '+' FAB button on map (bottom-right corner)
8. Tapping '+' button opens colony creation modal
9. Modal contains: name field (required), address field (optional), location display (latitude/longitude auto-populated from map center or user's current location)
10. "Use Current Location" button in modal updates location to user's GPS coordinates
11. "Use Map Center" button updates location to current map center coordinates
12. Modal validates inputs and displays errors
13. On successful creation (201 response), modal closes, new colony pin appears on map, and success message shown

---

### Story 2.5: Create Colony via Long-Press on Map

As a **user**,
I want to create a colony by long-pressing a location on the map,
so that I can quickly pin colonies at specific locations.

**Acceptance Criteria:**

1. Map component listens for long-press events (500ms touch duration or right-click on desktop)
2. Long-press captures clicked location's latitude and longitude coordinates
3. Long-press triggers colony creation modal with location pre-populated from clicked coordinates
4. Google Maps Geocoding API called to reverse geocode coordinates to address (if available within API budget)
5. Address field pre-populated with geocoded address if successful, otherwise left blank
6. Colony name field uses geocoded address as suggested name (user can edit)
7. Modal flow identical to Story 2.4 after opening (user can edit name/address, submit, etc.)
8. Long-press provides visual feedback (brief animation or temporary marker)
9. Long-press detection works reliably on mobile touch screens and desktop browsers

---

### Story 2.6: Navigate to Colony Details from Map Pin

As a **user**,
I want to tap on a colony pin to view its details,
so that I can see information about the colony and its cats.

**Acceptance Criteria:**

1. Tapping colony pin navigates to Colony Details page with route `/colonies/{colonyId}`
2. Navigation preserves browser history (back button returns to map)
3. GET `/api/v1/colonies/{colonyId}` endpoint returns detailed colony data: id, name, latitude, longitude, address, catCount, createdAt, createdBy (user name), primaryPhotoUrl
4. Endpoint requires authentication
5. Returns 404 Not Found if colony doesn't exist or is soft-deleted
6. Colony Details page displays: colony name (editable inline - see Story 2.7), back button to map, cat count, address (if available), created date/user
7. Colony Details page has placeholder sections for: cat list (empty state: "No cats added yet"), activity log (empty state: "No activities logged yet")
8. Page loads with skeleton loaders for data sections while fetching
9. Error state displayed if colony fetch fails

---

### Story 2.7: Edit Colony Name

As a **user**,
I want to edit a colony's name,
so that I can improve or correct colony identification.

**Acceptance Criteria:**

1. PUT `/api/v1/colonies/{colonyId}` endpoint accepts: name (required), address (optional)
2. Endpoint validates name not empty, max 200 characters
3. Endpoint allows any authenticated verified user to edit (MVP - no Colony Manager restriction)
4. Updates updatedBy and updatedAt audit fields
5. Returns 200 OK with updated colony data
6. Returns 404 if colony not found, 400 for validation errors
7. Colony name on Colony Details page displayed as inline editable field (click to edit) or with "Edit" pencil icon
8. Clicking edit activates text input with current name pre-filled
9. User can save (check icon) or cancel (X icon) edit
10. On save success (200 response), updated name displayed and success message shown
11. Error messages displayed for validation failures

---

### Story 2.8: Delete Colony (Restricted)

As a **Super Admin or colony creator**,
I want to delete colonies that have no dependent data,
so that I can remove erroneous or duplicate colonies.

**Acceptance Criteria:**

1. DELETE `/api/v1/colonies/{colonyId}` endpoint soft-deletes colony (sets deleted=true, deletedAt=now)
2. Endpoint validates: no dependent data exists (no cats, no activities)
3. Endpoint requires: authenticated user is Super Admin OR colony creator (creatorUserId matches authenticated user)
4. Returns 204 No Content on successful deletion
5. Returns 403 Forbidden if user lacks permission
6. Returns 409 Conflict if dependent data exists with message "Cannot delete colony with cats or activities"
7. Returns 404 if colony not found
8. Frontend displays "Delete Colony" button on Colony Details page only for authorized users
9. Delete button triggers confirmation dialog: "Are you sure? This action cannot be undone."
10. On successful deletion (204 response), user redirected to map and colony pin removed
11. Error messages displayed for deletion failures

---

### Story 2.9: Colony Details - Empty State & Tutorial

As a **first-time user**,
I want to see helpful guidance when the map is empty,
so that I understand how to start using the platform.

**Acceptance Criteria:**

1. Map component detects empty colony list (zero colonies returned from API)
2. If empty AND user's first login (check via flag in user profile or session), display tutorial overlay
3. Tutorial overlay explains: "Welcome to Paw Patrol!", "Tap the + button to add your first colony", "Long-press the map to pin a colony at a specific location"
4. Tutorial overlay has "Got it" button to dismiss (sets tutorial completed flag)
5. Tutorial overlay has "Don't show again" checkbox (persists preference)
6. If map is empty but tutorial already seen, show subtle empty state message: "No colonies yet. Tap + to add one."
7. Tutorial overlay is dismissible by tapping outside or pressing escape key
8. Tutorial overlay styled to be prominent but not obstructive

---

### Story 2.10: Bottom Sheet Navigation Menu

As a **user**,
I want to access navigation options via a menu,
so that I can log out, view my profile, or access help resources.

**Acceptance Criteria:**

1. Hamburger menu icon displayed in top-left corner of map screen
2. Tapping hamburger icon opens bottom sheet modal sliding up from bottom
3. Bottom sheet displays: user profile section (name, email, role badges), "Help" button, "Log Out" button
4. Bottom sheet can be dismissed by: tapping outside, dragging down, tapping X close button
5. "Log Out" button clears JWT token from storage and redirects to login page
6. "Help" button navigates to help/documentation page (placeholder for MVP: simple text page with basic instructions)
7. Bottom sheet includes admin-only menu item "Admin Panel" visible only to users with ADMIN or SUPER_ADMIN role
8. Bottom sheet styled using Untitled UI components with smooth slide animation
9. Bottom sheet accessible on mobile and desktop

---

### Story 2.11: Colony Photo Upload & Display

As a **user**,
I want to upload photos for a colony,
so that colonies are easier to identify on the map.

**Acceptance Criteria:**

1. POST `/api/v1/colonies/{colonyId}/photos` endpoint accepts multipart file upload
2. Endpoint validates: image MIME type (JPEG, PNG), max 5MB per file, max 10 photos per colony
3. Image optimized on upload: resized to max 1920px width, compressed to 80% JPEG quality
4. Thumbnail generated: 200x200px for map pin display
5. Files stored with UUID-prefixed filenames
6. ColonyPhoto record created with isPrimary=true if first photo, otherwise false
7. Returns 201 Created with photo ID and URLs (full and thumbnail)
8. Returns 400 for validation failures
9. Colony Details page has "Add Photo" button opening photo upload modal
10. Upload modal supports multiple file selection, shows preview of selected files
11. On upload success (201 response), photos displayed in gallery on Colony Details page
12. First uploaded photo becomes primary photo and displays on map pin
13. Gallery supports tap-to-expand full-resolution view

---

### Story 2.12: Set Primary Colony Photo

As a **user**,
I want to set which photo displays on the map pin,
so that the most recognizable image is shown.

**Acceptance Criteria:**

1. PUT `/api/v1/colonies/{colonyId}/photos/{photoId}/set-primary` endpoint sets isPrimary=true for specified photo
2. Endpoint automatically sets isPrimary=false for all other photos of that colony
3. Returns 200 OK with updated photo data
4. Returns 404 if colony or photo not found
5. Frontend gallery on Colony Details page shows "Set as Primary" button on each photo (except current primary)
6. Current primary photo indicated with badge/icon: "Primary"
7. On set primary success (200 response), gallery updates and map pin updates with new primary photo
8. Map pin thumbnail updates in real-time (or on next data refresh)

---

**Epic 2 Complete:** At this point, users can view an interactive map, create colonies via button or long-press, see colony pins with photos, navigate to colony details, edit colony names, upload photos, and delete empty colonies. The map interface is the functional home base of the application. Ready for Epic 3 to add cat profiles.
