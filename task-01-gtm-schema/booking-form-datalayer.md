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

### Front-End Implementation Rules

Implement the custom `dataLayer.push` calls at each step transition using the following rules:

1.  **Timing**: Execute `window.dataLayer.push` immediately after the transition button click (e.g., "Next", "Proceed", or "Confirm") and only after front-end validation succeeds, but prior to updating the form UI state.
2.  **PII Restriction**: Do not include actual names, email addresses, or phone numbers in the parameters. For Step 2, send `preferred_date` formatted as `YYYY-MM-DD` and a boolean flag `has_phone: true` once the phone number passes validation.
3.  **Payload Structure**: Follow the keys and types specified above. The event name must remain `booking_step_complete` for all three steps.
4.  **Array Check**: Initialize the global array safely before executing the push to prevent exceptions:
    ```javascript
    window.dataLayer = window.dataLayer || [];
    window.dataLayer.push({ ... });
    ```

