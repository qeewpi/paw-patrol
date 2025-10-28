# Brainstorming Session Results

**Session Date:** 2025-10-26
**Facilitator:** Business Analyst Mary
**Participant:** User

## Executive Summary

**Topic:** An app for tracking stray/community cats, with a Pokemon Go-ish, Google Maps approach. Including details like feeding schedule besides the cats' location/names, etc. The app will also need to implement user registration.

**Session Goals:** Focused ideation on the core mechanics and user experience for a web application prototype using a tech stack of Java/Spring Boot (Backend), Vite (Frontend), Untitled UI (UI), and PostgreSQL (Database).

**Techniques Used:** User Story Mapping, How Now Wow Matrix

**Total Ideas Generated:** 7 major features, 35+ detailed sub-features and user stories.

### Key Themes Identified:
- **Security & Trust:** Manual user verification and a full audit trail are key to creating a safe community.
- **Data-Rich Profiles:** Capturing detailed information about each cat is essential for providing proper care.
- **Efficient UX:** The app must be fast and easy to use for volunteers in the field, with a focus on clear, user-friendly interfaces.
- **Community Collaboration:** Features are designed to help a community of rescuers work together effectively.
- **Gamification:** Simple, engaging mechanics like the "Cat Collection" can drive user motivation.

## Technique Sessions

### User Story Mapping - Complete

**Description:** Mapping the journey of a cat rescuer to focus on their goals and the steps they'll take.

**Ideas Generated (Major Activities/Backbone):**
1. User Registration & Verification
    - User clicks "Register" button.
    - Fills out basic info (Name, Email, Password).
    - Reaches a "Verification" screen to upload documents.
    - Uploads proof of rescue work (photos with cats, vet bills, organization affiliation, etc.).
    - Writes a statement on why they want to join.
    - Submits the application for manual review.
    - Sees a "Pending Approval" status page.

2. View Cats on Map
    - App requests location permission on start.
    - Map centers on the user's current location.
    - Data is scoped to the Philippines, within a set radius.
    - User can pan and zoom the map.
    - Colony pins with photos show the circular image.
    - Colony pins without photos show a default icon.

3. Explore Colony Details
    - Tapping a pin opens the "Colony Details" view.
    - An "Edit" button allows users to change the Colony Name.
    - Displays the total number of cats in the colony.
    - An "Add New Cat" button takes the user to a "quick add" form.
    - Shows a clickable list of individual cats by name.
    - Contains a log of care activities and all system-generated audit events (e.g., Colony Created, Cat Added, Profile Edited).
    - The log can be filtered by "Today", "All Time", or a specific date.
    - Includes a "Status Tags" section for at-a-glance info.
    - A "TNR Progress" indicator shows the ratio of neutered cats (e.g., "5 of 10 cats neutered").

4. View Cat Profile (Accessed from Colony Details)
    - A visible "Status" field displays the cat's current state (e.g., "Active", "Adopted").
    - An "Update Status" button allows users to change the status to Adopted, Relocated, Missing, or Deceased.
    - An "Edit" button allows users to modify all descriptive fields but not historical logs.
    - **Basic Info:** Name, Gender, Estimated Age, Distinguishing Features, Photo/Video Gallery.
    - **Health Info:** Medical Conditions, Special Diet Needs, Vet Visit Log, Vaccination Record, Medication Log.
    - **TNR Info:** Neuter Status, Date of Neuter.
    - **Behavioral Info:** Temperament (tags), Interaction Notes (log).
    - **History:** Date First Sighted, Origin Story (bio).

5. Log a Care Activity
    - User clicks a "Log Activity" button from the colony page.
    - A form appears with a primary dropdown for **Activity Category** (e.g., Feeding, Health, Environment).
    - Selecting a category reveals a secondary field for the specific **Action** (e.g., Gave Medication, Cleaned Shelter).
    - Conditional fields appear where necessary (e.g., a hybrid dropdown/text field for food type if "Fed" is selected).
    - A text area for "Notes" and an option for "Photo Upload" are included.
    - The submission is added to the colony's activity log.

6. Create New Cat/Colony Location
    - User initiates by either tapping a '+' button on the map or long-pressing a location on the map.
    - They can set the location using their device's GPS or by searching for an address.
    - After confirming the pin location, a form appears.
    - The form requires a **Colony Name**, with a suggested name auto-filled from the address.
    - The form also requires adding at least one cat via a "quick add" flow (e.g., providing only a name, photo, and distinguishing features).
    - The full, detailed cat profile can be filled out later.

7. Search & Filter
    - A search bar is always visible at the top of the map.
    - It allows users to search by Colony Name, Cat Name, or Location (address/city).
    - A "Filter" icon next to the search bar opens a panel with categorized filter options.
    - **Filter Options Include:**
        - **Status:** Filter by Active, Adopted, Missing, etc. (Defaults to Active).
        - **Health & Feeding:** Medical Alert (Yes/No), Feeding Status (Today/Not Today).
        - **TNR Status:** Neutered / Unneutered.
        - **Behavior:** Filter by temperament tags.
        - **Basic Info:** Filter by Gender or Age Range.

### How Now Wow Matrix - In Progress

**Description:** Categorizing features by their implementation difficulty and innovation level to define the prototype's scope.

**Now (Easy to do, Normal idea):**

- **User Registration & Verification:** Essential for security and community trust.

- **View Cats on Map:** The core interface for the application.

- **Explore Colony Details:** The main information hub for a colony.

- **View Cat Profile:** Rich, detailed pages for each individual cat.

- **Log a Care Activity:** The primary interaction for users to record their work.

- **Create New Cat/Colony Location:** Allows the community to grow the map.

- **Search & Filter:** Powerful tools to find specific cats or groups of cats.



**Wow (Easy to do, Innovative idea):**

- **Cat Collection / "Cat Dex":** A gamified feature to encourage user engagement.



**How (Hard to do, Normal idea):**

- **Admin Verification Panel:** The backend UI for manually approving new users.



**Later (Hard to do, Innovative idea):**

- **Role-Based Dashboards:** Different interfaces for users vs. admins.

- **AI Cat Recognition:** Using machine learning to identify cats from photos.
