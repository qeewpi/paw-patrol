# MVP Scope Clarifications

The following requirements have simplified MVP implementations with full versions deferred to post-MVP based on development timeline:

### Search and Filter Interaction (FR15)

**MVP Implementation:**
- Search and filters operate independently (mutually exclusive modes)
- User can either search OR apply filters, not both simultaneously
- Switching between search and filter clears the previous operation

**Full Implementation (Post-MVP):**
- Combined search + filter operations (e.g., search "Tabby" AND filter by "Neutered")
- Complex query logic supporting multiple simultaneous criteria
- Persistent filter state while searching

**Rationale:** Independent search/filter is simpler to implement and test; combined operations require complex SQL/backend query building.

---

### Feeding Status Calculation (FR19-FR20)

**MVP Implementation:**
- "Fed Today" defined as feeding activity logged within last 24 hours from current time
- Simple timestamp comparison: `now() - feeding_timestamp < 24 hours`
- No timezone adjustments or day boundary logic

**Full Implementation (Post-MVP):**
- Timezone-aware "Fed Today" using Philippines timezone (GMT+8)
- Resets at midnight Philippines time, not rolling 24-hour window
- Accurate for coordination across different user timezones

**Rationale:** Rolling 24-hour window is simpler and functionally equivalent for single-timezone pilot testing; timezone logic adds complexity.

---

### Temperament and Age Range Tagging (FR22-FR23)

**MVP Implementation:**
- **Temperament:** Free-text field for behavioral notes (no predefined tags)
- **Age Range:** Free-text or simple dropdown with basic options (Kitten, Adult, Senior)
- Filter by text search within behavioral notes field
- No structured tag selection UI

**Full Implementation (Post-MVP):**
- **FR22:** Behavior filter with predefined temperament tags: Friendly, Shy, Aggressive, Feral, Playful, Cautious
- **FR23:** Age Range filter with detailed options: Kitten (0-6mo), Young (6mo-2yr), Adult (2-7yr), Senior (7+yr)
- Multi-select tag UI with checkboxes
- Structured database fields for efficient filtering

**Rationale:** Free-text requires minimal UI work and backend schema; predefined tags need dropdown components, validation, and migration strategy.

---

### Activity Log Editing (FR44-FR45)

**MVP Implementation:**
- Activity logs are immutable once submitted (cannot be edited or deleted)
- If user makes mistake, they add a new corrective activity log with note explaining correction
- Photos are permanently attached to activity at submission time

**Full Implementation (Post-MVP):**
- **FR44:** Activity log creators can edit all details (category, action, notes, timestamp, photos)
- **FR45:** Add or remove photos from existing activity logs
- Edit history tracking (audit trail showing what was changed)

**Rationale:** Immutable logs are simpler (no edit UI, no version control, no conflict resolution) and actually provide better audit trail integrity.

---

### Implementation Strategy

**If Timeline Pressure Occurs:**
1. Implement MVP simplified versions first
2. Validate functionality with personal use
3. Gather pilot user feedback on what's actually needed
4. Prioritize full implementations based on real user pain points

**Upgrade Path:**
- All MVP simplifications can be enhanced without breaking existing data
- Free-text fields can be migrated to structured tags via data migration
- Immutable logs can become editable by adding edit functionality
- Independent search/filter can be combined with query builder

**Testing Note:** Simplified MVP versions are actually easier to test thoroughly, reducing bug risk during aggressive 1-month timeline.
