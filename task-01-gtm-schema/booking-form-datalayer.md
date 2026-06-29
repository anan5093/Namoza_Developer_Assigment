# Booking Form Funnel & dataLayer Tracking Specification

The 3-step appointment booking form is OrthoNow’s primary conversion mechanism. Since a standard GTM setup cannot natively listen to transitions inside a single-page form, we implement a custom dataLayer-driven funnel tracking system.

---

## Step-by-Step Funnel Specification

### Step 1 — Location & Specialty Selection
This step captures the patient's intent regarding which clinic location and clinical specialty they require.

*   **GTM Trigger**: 
    *   **Trigger Type**: Custom Event
    *   **Event Name**: `booking_step_complete`
    *   **Trigger Condition**: `step_number` equals `1`
*   **Exact `dataLayer.push()` JSON**:
    ```json
    {
      "event": "booking_step_complete",
      "step_number": 1,
      "step_name": "location_specialty_selected",
      "clinic_location": "Koramangala, Bengaluru",
      "specialty": "Knee & Joint"
    }
    ```
*   **Abandonment Behavior**: If the user exits the page after completing Step 1 but before starting Step 2, they are categorized as **"Top-of-Funnel Drop-offs."** In GA4, we identify this group as users who fired the `booking_step_complete` event with `step_number: 1` but never fired a subsequent `step_number: 2` event in the same session.

---

### Step 2 — Name / Phone / Date Entry
In this step, the patient enters their contact information (Name, Phone) and preferred date for the appointment.

*   **GTM Trigger**:
    *   **Trigger Type**: Custom Event
    *   **Event Name**: `booking_step_complete`
    *   **Trigger Condition**: `step_number` equals `2`
*   **Exact `dataLayer.push()` JSON**:
    ```json
    {
      "event": "booking_step_complete",
      "step_number": 2,
      "step_name": "patient_details_entered",
      "preferred_date": "2025-08-15",
      "has_phone": true
    }
    ```
    *(Note: To maintain strict privacy and data compliance, we do not push Personal Identifiable Information (PII) like the actual name or phone number. We only push metadata like the preferred date and a boolean `has_phone` indicating validation success.)*
*   **Abandonment Behavior**: If the user leaves the page here, they are flagged as **"High-Intent Drop-offs."** These users were willing to select a location and input their details but stopped short of final confirmation. They are prime targets for remarketing lists.

---

### Step 3 — Booking Confirmation
This is the final step, representing a successful booking.

*   **GTM Trigger**:
    *   **Trigger Type**: Custom Event
    *   **Event Name**: `booking_step_complete`
    *   **Trigger Condition**: `step_number` equals `3`
*   **Exact `dataLayer.push()` JSON**:
    ```json
    {
      "event": "booking_step_complete",
      "step_number": 3,
      "step_name": "booking_confirmed",
      "clinic_location": "Koramangala, Bengaluru",
      "specialty": "Knee & Joint",
      "booking_id": "ONB-20250812-004"
    }
    ```
*   **Conversion Event**: This is the final conversion point. Once this event fires, it is registered in GA4 as a conversion and imported into Google Ads as our primary success metric.

---

## GA4 Funnel Exploration Setup

To visualize and analyze booking drop-offs in Google Analytics 4, configure a **Funnel Exploration** report under the **Explore** tab:

1.  **Create a New Exploration**: Choose **Funnel Exploration**.
2.  **Define the Funnel Steps**:
    *   **Step 1: Selected Location & Specialty**
        *   Event: `booking_step_complete`
        *   Parameter: `step_number` = `1` (or parameter `step_name` = `location_specialty_selected`)
    *   **Step 2: Entered Details**
        *   Event: `booking_step_complete`
        *   Parameter: `step_number` = `2` (or parameter `step_name` = `patient_details_entered`)
    *   **Step 3: Confirmed Booking**
        *   Event: `booking_step_complete`
        *   Parameter: `step_number` = `3` (or parameter `step_name` = `booking_confirmed`)
3.  **Identify Drop-off**:
    *   **Step 1 → Step 2 Drop-off**: Visualized in the funnel bar chart. In the detail table, you can see the percentage of users who selected a location but did not proceed to enter their contact details.
    *   **Step 2 → Step 3 Drop-off**: Represents users who entered details but did not complete the final confirmation. This indicates friction in the final step (e.g., trust anxiety or technical payment/confirmation lag).
4.  **Isolate Bengaluru Clinic Traffic**:
    *   To view this funnel specifically for Bengaluru clinic locations, apply a **Segment** or a **Filter** to the exploration.
    *   Add a filter where the event parameter `clinic_location` **contains** `"Bengaluru"` (or matches the regex `.*Bengaluru.*`). This filters out Chennai and Hyderabad bookings, leaving only Bengaluru location data in the funnel steps.

---

## Who writes the dataLayer push?

### ⚠️ IMPORTANT MARTECH FACT
**Google Tag Manager cannot natively detect step transitions inside a custom multi-step form.** GTM has no way of knowing when validation succeeds and a user moves from Step 1 to Step 2 without custom code. 

Therefore, **the front-end developer must write and trigger the `window.dataLayer.push({...})` block in the website's application code.** GTM simply sits and listens for these pushes.

### Briefing Note for the Front-End Developer Team

Dear Team,

We are implementing funnel tracking for the new 3-step booking form. Please integrate the custom dataLayer pushes at each step transition according to these guidelines:

*   **Timing of the Push**: Trigger the `window.dataLayer.push({...})` statement **immediately after the user clicks the transition button** (e.g., "Next" or "Confirm Appointment") and **only after the step's front-end validation passes**, but **before** the UI changes to the next step or shows the confirmation message.
*   **Privacy & PII Compliance**: Do not push the patient's actual name, email, or phone number to the dataLayer. For Step 2, only push the `preferred_date` formatted as `YYYY-MM-DD` and a boolean `has_phone: true` (which validates that a valid 10-digit Indian phone number was entered).
*   **Push Structure**: Ensure the push uses the exact parameter keys and values defined in this specification. The event name must be `booking_step_complete` across all steps, using the `step_number` parameter to differentiate them.
*   **Pre-initialization Safe-guard**: Always initialize the array before pushing: `window.dataLayer = window.dataLayer || [];` to prevent script crashes if GTM hasn't loaded yet.
