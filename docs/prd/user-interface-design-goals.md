# User Interface Design Goals

### Overall UX Vision

Paw Patrol aims for a **field-optimized, mobile-first experience** that prioritizes speed and simplicity over aesthetic polish. The interface should feel like a practical tool designed for volunteers working outdoors in challenging conditions—think "hiking GPS app" rather than "social media feed."

**Core UX Principles:**
- **One-handed operation:** All critical actions accessible with thumb reach on mobile
- **Glanceable information:** Key data (feeding status, TNR progress) visible without scrolling
- **Fast data entry:** Minimize taps and typing for common actions (logging feeding takes <10 seconds)
- **Forgiving interface:** Large touch targets, confirmation for destructive actions
- **Simple interactions:** Tap-only interface (no swipe gestures) suitable for non-power users

**Visual Language:**
- Clean, utilitarian design inspired by mapping/navigation apps (Google Maps, Waze)
- Photo-forward interface—cat images are primary navigation cues
- Status indicators using color coding (green = fed today, amber = needs attention, red = medical alert)
- Minimal decoration—every pixel serves a functional purpose

### Key Interaction Paradigms

**Map as Home Base:**
- Map is the default landing screen after login (not a dashboard or list view)
- All primary actions flow FROM the map: tap pin → view details → take action
- Persistent search bar and filter icon always visible for quick access

**Bottom Sheet Navigation:**
- Hamburger menu icon (top-left) opens bottom sheet modal with primary navigation
- Bottom sheet contains: User profile, Log Out, Settings (future), Help
- Modal design—sheet slides up from bottom, can be dismissed by tapping outside or dragging down
- Post-MVP: Replace with proper dashboard screen

**Tap-to-Drill Hierarchy:**
- Map → Colony Details → Cat Profile (2 levels of actual navigation)
- Modals used for forms and actions (Add Cat, Log Activity, Filters) - not counted as navigation depth
- Back buttons at every level for predictable navigation
- Breadcrumb context shown in headers ("Colony Name > Cat Name")

**Quick Actions:**
- Long-press map to create colony (500ms)
- Floating action button (FAB) for "Log Activity" on detail screens and "+" for Add Colony on map
- Swipe gestures supported where natural (photo galleries, dismissing modals)
- Primary actions via taps and buttons for clarity

**Photo-Centric Identification:**
- Cat profile photos displayed as circular thumbnails (recognizable at small sizes)
- Gallery view for multiple photos with tap-to-expand
- Camera integration for instant photo capture from forms

**Smart Defaults:**
- Timestamp defaults to "now" but is user-editable
- Colony name auto-suggests from GPS address
- Filters default to "Active" status (hide adopted/deceased cats)

### Core Screens and Views

**1. Interactive Map (Home Screen)**
- Full-screen map with colony pins
- Hamburger menu icon (top-left) opens bottom sheet navigation
- Persistent search bar at top center
- Filter icon (top-right)
- "+" FAB for creating colonies (bottom-right)
- User location indicator
- Tutorial overlay on first launch
- **Post-MVP:** Recent colonies quick access (bottom sheet list or separate dashboard widget)

**2. Bottom Sheet Navigation (Modal)**
- User profile section (name, email, role badge)
- Help button (opens help documentation)
- Log Out button
- Future: Settings
- **Post-MVP:** Replace with dedicated dashboard screen

**3. Colony Details Page**
- Header: Colony name (editable), back button
- Key metrics: Cat count, TNR progress bar, last feeding timestamp
- Cat list (scrollable cards with circular photo thumbnails, skeleton loaders during load, names, key status icons)
- Activity log (chronological feed)
- "Add New Cat" button
- "Log Activity" FAB

**4. Cat Profile Page**
- Header: Cat name, back button
- Hero photo (large, skeleton loader during load, tap-to-view gallery with swipe navigation)
- Status badge (Active/Adopted/etc.)
- Collapsible sections: Basic Info, Health Info, TNR Info, Behavioral Info, History
- Cat-specific activity log
- "Edit Profile" button
- "Log Activity" FAB

**5. Add/Edit Cat Form (Modal)**
- Name field (required)
- Gender dropdown (required)
- Photo upload (optional, up to 10)
- Distinguishing features (text area)
- Additional sections expandable (not required for quick-add)
- Save button

**6. Log Activity Form (Modal)**
- Category dropdown (Feeding/Health/Environment)
- Action dropdown (contextual based on category)
- Notes text area (optional)
- Photo upload (up to 3)
- Timestamp picker (defaults to now)
- Submit button

**7. Search Results Screen**
- List view of colonies/cats matching query
- Distance indicator from user location
- Tap result to navigate to details
- Clear/close search button

**8. Filter Panel (Modal/Drawer)**
- Categorized filter sections (Status, Health & Feeding, TNR, Behavior, Basic Info)
- Checkboxes/toggles for each option
- "Apply Filters" button
- "Clear All" button

**9. Registration/Login Screens**
- Standard email/password forms
- File upload for verification documents
- Pending approval status page (post-registration)

**10. Admin Verification Panel**
- List of pending user applications
- Document viewer for verification materials
- Approve/Reject buttons
- Admin-only access

### Accessibility

**Target Level:** None (no formal WCAG compliance for MVP)

**Rationale:** Prioritizing speed to market for pilot testing. Basic best practices will be followed (semantic HTML, sufficient color contrast for outdoor visibility, keyboard navigation) but no formal accessibility audit or screen reader testing.

**Post-MVP Consideration:** If platform scales to include users with disabilities, implement WCAG AA compliance.

### Branding

**Visual Identity:**
- **No formal branding for MVP** - using Untitled UI component library defaults
- **Color Palette:** Rely on Untitled UI's neutral palette with functional status colors:
  - Green: Positive states (fed, healthy, neutered)
  - Amber/Yellow: Attention needed (medical alert, not fed recently)
  - Red: Urgent/critical (missing, medical emergency)
  - Blue: Informational (audit events, system messages)
- **Typography:** System fonts (San Francisco on iOS, Roboto on Android, -apple-system on web)
- **Iconography:** Standard Material Design or Untitled UI icon set

**Future Branding:** If adopted by rescue organizations, allow white-labeling or custom themes (logo upload, color palette customization).

### Target Device and Platforms

**Primary:** Web Responsive (mobile-first)
- Optimized for smartphone screens (375px-428px width)
- Functional on tablets and desktop but mobile is priority
- Touch-first interactions (large tap targets, no gestures required)

**Browser Support:** Last 2 versions of Chrome, Firefox, Safari, Edge (mobile and desktop)

**No Native Apps:** Web-only for MVP for easier access with no installation required. Users access via browser URL. Progressive Web App (PWA) features could be added later for home screen installation and offline support if needed.
