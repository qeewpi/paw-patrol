# Epic 3: Cat Profiles & Management

**Epic Goal:** Build comprehensive cat profile system with complete CRUD operations, photo galleries, status tracking, and health information management. This epic delivers the detailed individual cat record-keeping capabilities essential for rescue coordination, enabling users to track each cat's identity, health status, TNR information, behavioral traits, and history within their colonies.

### Story 3.1: Cat Entity & Data Model

As a **developer**,
I want to create the Cat entity with comprehensive profile fields,
so that the system can store detailed information about individual cats.

**Acceptance Criteria:**

1. `Cat` entity created extending base audit entity with fields: colonyId (foreign key), name (required), gender (MALE/FEMALE/UNKNOWN enum), status (ACTIVE/ADOPTED/RELOCATED/MISSING/DECEASED enum), distinguishingFeatures (text), medicalAlert (boolean), behavioralNotes (text), ageRange (text for MVP)
2. `CatHealthInfo` entity created with fields: catId (foreign key), lastVetVisit (date), medications (text), knownConditions (text), notes (text)
3. `CatTnrInfo` entity created with fields: catId (foreign key), isNeutered (boolean), neuteredDate (date), tnrOrganization (text), notes (text)
4. `CatPhoto` entity created with fields: catId (foreign key), filename, filepath, isPrimary (boolean), uploadedAt, caption (optional)
5. Cat entity has one-to-one relationships with CatHealthInfo and CatTnrInfo (lazy loading), one-to-many with CatPhoto
6. Database constraints: name max 100 characters, status defaults to ACTIVE
7. JPA repositories created for Cat, CatHealthInfo, CatTnrInfo, CatPhoto entities
8. Database migration script creates all cat-related tables with proper foreign keys and indexes

---

### Story 3.2: Add Cat to Colony (Quick Add)

As a **user**,
I want to quickly add a cat to a colony with minimal required information,
so that I can rapidly document cats I encounter.

**Acceptance Criteria:**

1. POST `/api/v1/colonies/{colonyId}/cats` endpoint accepts: name (required), gender (required), distinguishingFeatures (optional), photo (optional multipart file)
2. Endpoint validates: name not empty, gender is valid enum value, photo meets image validation rules if provided
3. Cat created with status=ACTIVE, colonyId set, default values for optional fields
4. If photo provided, optimized and stored with thumbnail generation (same as colony photos)
5. CatHealthInfo and CatTnrInfo records created with null/default values
6. Returns 201 Created with cat ID and full cat data
7. Returns 400 Bad Request for validation failures
8. Colony Details page displays "Add New Cat" button
9. Tapping button opens quick-add modal with fields: name (required), gender dropdown (Male/Female/Unknown), distinguishing features (text area), photo upload (optional)
10. Modal has camera integration button for instant photo capture on mobile devices
11. On successful creation (201 response), modal closes, cat appears in colony's cat list, success message shown

---

### Story 3.3: Display Cat List on Colony Details

As a **user**,
I want to see all cats in a colony as a scrollable list,
so that I can browse and select cats to view details.

**Acceptance Criteria:**

1. GET `/api/v1/colonies/{colonyId}/cats` endpoint returns all non-deleted cats with fields: id, name, gender, status, primaryPhotoUrl (if exists), medicalAlert
2. Endpoint requires authentication
3. Returns empty array if no cats in colony
4. Colony Details page fetches and displays cat list on page load
5. Cat list rendered as scrollable cards with: circular photo thumbnail (or default icon), name, gender icon, status badge, medical alert icon (if medicalAlert=true)
6. Skeleton loaders displayed while cat data is fetching
7. Empty state shown if no cats: "No cats added yet. Tap 'Add New Cat' to start."
8. Cat cards are tappable/clickable, navigate to Cat Profile page on tap
9. Cat count displayed in colony header updates based on list length

---

### Story 3.4: Cat Profile Page - Basic Display

As a **user**,
I want to view a cat's comprehensive profile,
so that I can see all recorded information about the cat.

**Acceptance Criteria:**

1. GET `/api/v1/cats/{catId}` endpoint returns full cat details: basic info, health info, TNR info, photos, behavioral notes
2. Endpoint requires authentication
3. Returns 404 Not Found if cat doesn't exist or is soft-deleted
4. Cat Profile page route: `/cats/{catId}` with back button to Colony Details
5. Page header displays: cat name, status badge, back button
6. Hero section displays primary photo (large, tap-to-expand) or default placeholder
7. Collapsible sections displayed: Basic Info (name, gender, age range, distinguishing features), Health Info (medical alert toggle, vet visits, medications, conditions), TNR Info (neutered status, date, organization), Behavioral Info (notes), History (placeholder for now)
8. Each section has "Edit" button (see Story 3.5)
9. Page loads with skeleton loaders while fetching data
10. Breadcrumb context shown: "Colony Name > Cat Name"

---

### Story 3.5: Edit Cat Profile Information

As a **user**,
I want to edit any cat's profile information,
so that I can update or correct cat details.

**Acceptance Criteria:**

1. PUT `/api/v1/cats/{catId}` endpoint accepts: name, gender, status, distinguishingFeatures, ageRange, behavioralNotes, medicalAlert
2. PUT `/api/v1/cats/{catId}/health` endpoint accepts: lastVetVisit, medications, knownConditions, notes
3. PUT `/api/v1/cats/{catId}/tnr` endpoint accepts: isNeutered, neuteredDate, tnrOrganization, notes
4. All endpoints validate inputs, update audit fields, return 200 OK with updated data
5. Returns 404 if cat not found, 400 for validation errors
6. Endpoints allow any authenticated verified user to edit (MVP - no restriction)
7. Each collapsible section on Cat Profile has "Edit" button opening edit modal
8. Edit modals pre-fill with current values, allow editing all fields in that section
9. Modals have "Save" and "Cancel" buttons
10. On save success (200 response), modal closes, profile data refreshes, success message shown
11. Error messages displayed for validation failures

---

### Story 3.6: Update Cat Status

As a **user**,
I want to update a cat's status (Active, Adopted, Relocated, Missing, Deceased),
so that I can track the cat's current situation.

**Acceptance Criteria:**

1. PUT `/api/v1/cats/{catId}/status` endpoint accepts: status (enum value), notes (optional text for context)
2. Endpoint validates status is valid enum value
3. Status update creates audit event "Cat Status Changed" (visible in activity logs - Epic 4)
4. Returns 200 OK with updated cat data
5. Returns 404 if cat not found, 400 for invalid status
6. Cat Profile page displays prominent "Update Status" button near cat name/header
7. Tapping button opens status update modal with: status dropdown (Active, Adopted, Relocated, Missing, Deceased), optional notes field (e.g., "Adopted by family in Quezon City")
8. Modal has confirmation dialog for destructive status changes (Missing, Deceased)
9. On successful update (200 response), modal closes, status badge updates, success message shown
10. Colony cat list reflects updated status immediately (or on refresh)

---

### Story 3.7: Cat Photo Gallery Upload & Management

As a **user**,
I want to upload and manage multiple photos for a cat,
so that I can visually document the cat's appearance and changes over time.

**Acceptance Criteria:**

1. POST `/api/v1/cats/{catId}/photos` endpoint accepts multipart file uploads (max 10 photos per cat)
2. Endpoint validates: image MIME type, max 5MB per file, max 10 photos total per cat
3. Images optimized on upload: resized to max 1920px width, compressed to 80% JPEG quality
4. Thumbnail generated: 200x200px for display in lists
5. CatPhoto record created with isPrimary=true if first photo, otherwise false
6. Returns 201 Created with photo ID and URLs
7. Returns 400 for validation failures (file type, size, max photos exceeded)
8. Cat Profile page displays photo gallery section showing all photos as grid
9. "Add Photos" button opens upload modal with multiple file selection and camera integration
10. Gallery supports tap-to-expand for full-resolution view with swipe navigation
11. Each photo has "Set as Primary" button (updates primary photo displayed in cat list)
12. On upload success (201 response), photos appear in gallery, success message shown

---

### Story 3.8: Delete Cat Photo

As a **user**,
I want to delete photos from a cat's gallery,
so that I can remove poor quality or outdated images.

**Acceptance Criteria:**

1. DELETE `/api/v1/cats/{catId}/photos/{photoId}` endpoint soft-deletes photo
2. If deleted photo was primary, automatically set oldest remaining photo as new primary
3. Returns 204 No Content on successful deletion
4. Returns 404 if cat or photo not found
5. Returns 409 Conflict if attempting to delete last photo (at least one photo should remain for identification)
6. Gallery view shows delete icon/button on each photo
7. Delete button triggers confirmation dialog: "Are you sure you want to delete this photo?"
8. On successful deletion (204 response), photo removed from gallery, success message shown
9. If deleted photo was primary, gallery updates to show new primary photo

---

### Story 3.9: TNR Progress Calculation & Display

As a **developer**,
I want to calculate and display TNR progress for colonies,
so that users can track neutering completion rates.

**Acceptance Criteria:**

1. GET `/api/v1/colonies/{colonyId}/tnr-progress` endpoint calculates: total cats (non-deleted, ACTIVE status), neutered cats (isNeutered=true), percentage
2. Calculation excludes cats with unknown TNR status from neutered count but includes in total count (per FR53)
3. Returns: totalCats, neuteredCats, percentage (calculated as neuteredCats/totalCats * 100)
4. Endpoint requires authentication
5. Colony Details page fetches TNR progress data on load
6. TNR progress displayed as visual progress bar with text: "5 of 10 cats neutered (50%)"
7. Progress bar color-coded: green (80-100%), amber (50-79%), red (0-49%)
8. Progress bar placeholder shown if no cats in colony: "No cats to track"
9. Progress updates when cats added or TNR info changed

---

### Story 3.10: Cat Search Within Colony (Optional Quick Win)

As a **user**,
I want to search for cats by name within a colony,
so that I can quickly find specific cats in large colonies.

**Acceptance Criteria:**

1. Colony Details page cat list has search input field above list
2. Search filters cat list client-side by matching cat name (case-insensitive)
3. Search updates list in real-time as user types
4. Empty state shown if no matches: "No cats found matching '[query]'"
5. Clear button (X icon) in search field resets list to show all cats
6. Search field has placeholder text: "Search cats by name..."
7. Search is local/client-side only (no API call needed for MVP)

---

### Story 3.11: Cat Profile - Medical Alert Indicator

As a **user**,
I want to see prominent medical alert indicators for cats with health concerns,
so that I'm immediately aware of cats needing special attention.

**Acceptance Criteria:**

1. Medical alert toggle (boolean) stored in Cat entity
2. PUT `/api/v1/cats/{catId}` endpoint accepts medicalAlert field
3. Cat Profile page displays prominent medical alert banner if medicalAlert=true: red background, exclamation icon, text "Medical Alert - See Health Info"
4. Banner tappable, expands Health Info section when tapped
5. Cat list cards show red medical alert icon (badge) next to cat name if medicalAlert=true
6. Colony Details cat count shows medical alert count: "3 cats need medical attention"
7. Medical alert can be toggled on/off via Edit Health Info modal
8. Alerts visible at-a-glance for field prioritization

---

### Story 3.12: Last Fed Timestamp Display

As a **developer**,
I want to display "Last Fed" timestamps on Colony Details page,
so that users can see feeding recency at a glance (prepares for Activity Logging in Epic 4).

**Acceptance Criteria:**

1. Database schema preparation: Ensure Activity entity (created in Epic 4) can link to colonies
2. Colony Details page includes placeholder section: "Last Fed: [timestamp or 'Never']"
3. Timestamp calculation service method created (will be implemented fully in Epic 4)
4. For now, display static placeholder: "Activity logging coming soon"
5. UI positioned prominently in colony header: "Last Fed: 8:30 AM (2 hours ago)" format
6. Placeholder styled to match final design (green if recent, amber if >12 hours, red if >24 hours)
7. This story prepares UI/UX for Epic 4 activity logging integration

---

**Epic 3 Complete:** At this point, users can add cats to colonies with detailed profiles (basic info, health, TNR, behavioral notes, photos), view cat lists, edit cat information, update status, manage photo galleries, and see TNR progress. Individual cat tracking is fully functional. Ready for Epic 4 to add activity logging and search.
