# OrthoNow GTM Event Tracking Schema

Tracking schema for Google Tag Manager (GTM) and GA4. Implement this before running any paid marketing campaigns.

## Full Event Schema Table

| Event Name | Trigger Type | Key Parameters (min 3 each) | GA4 Report / Audience |
| :--- | :--- | :--- | :--- |
| **`booking_step_complete`** <br>*(Step 1: Location & Specialty Selection)* | **Custom Event** <br>(Event Name: `booking_step_complete` and filter `step_number` equals `1`) | 1. `step_number` (Integer: `1`) <br>2. `step_name` (String: `"location_specialty_selected"`) <br>3. `clinic_location` (String: e.g., `"Koramangala, Bengaluru"`) <br>4. `specialty` (String: e.g., `"Knee & Joint"`) | **Explore > Funnel Exploration**<br>Used as Step 1 of the "Booking Flow Funnel." <br><br>**Audience Segment**<br>Used to build segment "Step 1 Completers." |
| **`booking_step_complete`** <br>*(Step 2: Patient Details Entry)* | **Custom Event** <br>(Event Name: `booking_step_complete` and filter `step_number` equals `2`) | 1. `step_number` (Integer: `2`) <br>2. `step_name` (String: `"patient_details_entered"`) <br>3. `preferred_date` (String: e.g., `"2025-08-15"`) <br>4. `has_phone` (Boolean: `true`) | **Explore > Funnel Exploration**<br>Used as Step 2 of the "Booking Flow Funnel." <br><br>**Audience Definition**<br>Audience: *"Booking Flow Abandoners (Step 2)"* (Users who completed Step 2 but did not complete Step 3 within 30 minutes). |
| **`booking_step_complete`** <br>*(Step 3: Booking Confirmation)* | **Custom Event** <br>(Event Name: `booking_step_complete` and filter `step_number` equals `3`) | 1. `step_number` (Integer: `3`) <br>2. `step_name` (String: `"booking_confirmed"`) <br>3. `clinic_location` (String: e.g., `"Koramangala, Bengaluru"`) <br>4. `specialty` (String: e.g., `"Knee & Joint"`) <br>5. `booking_id` (String: e.g., `"ONB-20250812-004"`) | **Reports > Engagement > Conversions**<br>Marked as a primary Conversion Event.<br><br>**Explore > Funnel Exploration**<br>The final step (Step 3) in the booking funnel.<br><br>**Audience Definition**<br>Audience: *"Converted Patients (All Locations)"*. |
| **`call_now_clicked`** | **Click - Just Links** <br>(Click URL starts with `tel:`) | 1. `page_location` (String: e.g., `"https://orthonow.in/clinics/koramangala"`) <br>2. `click_text` (String: e.g., `"+91 98765 43210"`) <br>3. `click_element_id` (String: e.g., `"header_call_btn"`, `"location_card_call"`) | **Reports > Engagement > Events**<br>Analyze click counts by element ID to determine which call button performs best.<br><br>**Audience Definition**<br>Audience: *"High-Intent Mobile Leads"* (users who clicked the call button on mobile device types). |
| **`whatsapp_widget_opened`** | **Click - Just Links** <br>(Click URL contains `wa.me` or `api.whatsapp.com`) | 1. `page_location` (String: e.g., `"https://orthonow.in/landing-page"`) <br>2. `widget_position` (String: e.g., `"floating_widget_bottom_right"`) <br>3. `clinic_preference` (String: e.g., `"Bengaluru"`, derived from URL structure) | **Reports > Engagement > Events**<br>Track WhatsApp engagement rates across landing pages.<br><br>**Audience Definition**<br>Audience: *"WhatsApp Prospects"* (users who clicked the widget but did not trigger a booking confirmation). |
| **`patient_guide_form_submitted`** | **Form Submission** <br>(Form ID equals `patient_guide_form`) | 1. `page_location` (String: e.g., `"https://orthonow.in/blog/knee-pain-rehab"`) <br>2. `form_id` (String: `"patient_guide_form"`) <br>3. `guide_name` (String: `"Ultimate Guide to Joint Health 2025"`) | **Reports > Engagement > Events**<br>Track lead generation performance for gated assets.<br><br>**Audience Definition**<br>Audience: *"MQL - Guide Downloaders"* (used for nurturing email/WhatsApp campaigns). |
| **`patient_guide_downloaded`** | **Click - Just Links** <br>(Click URL ends with `.pdf` and contains `"patient-guide"`) | 1. `page_location` (String: e.g., `"https://orthonow.in/thank-you-guide"`) <br>2. `file_name` (String: `"orthonow_patient_guide_v2.pdf"`) <br>3. `file_extension` (String: `"pdf"`) | **Reports > Engagement > Events**<br>Validate that users actually download the PDF after form submission. |
| **`clinic_page_viewed`** | **Page View** <br>(Page Path matches regex `^/clinics/[a-zA-Z0-9_-]+$`) | 1. `page_location` (String: e.g., `"https://orthonow.in/clinics/koramangala"`) <br>2. `clinic_name` (String: e.g., `"Koramangala Clinic"`) <br>3. `city` (String: e.g., `"Bengaluru"`) | **Reports > Engagement > Pages and screens**<br>Identify traffic levels for each of the 9 locations.<br><br>**Audience Definition**<br>Audience: *"Viewed Clinic - No Booking"* (users who viewed a clinic location page but did not book). |
| **`blog_scroll_depth`** | **Scroll Depth** <br>(Vertical scroll depth percentages: `25`, `50`, `75`, `90` on blog paths) | 1. `page_location` (String: e.g., `"https://orthonow.in/blog/causes-of-back-pain"`) <br>2. `scroll_threshold` (Integer: e.g., `90`) <br>3. `article_title` (String: e.g., `"5 Habits Causing Your Chronic Lower Back Pain"`) | **Reports > Engagement > Events**<br>Analyze content engagement. Drill down on the `scroll_threshold` parameter to see article drop-off.<br><br>**Audience Definition**<br>Audience: *"High-Engagement Blog Readers"* (users who scroll to 75% or 90% on blog posts). |

---

## Recommended Google Ads Conversion Action

### Primary Conversion
The **`booking_step_complete` (Step 3: Booking Confirmation)** event must be imported into Google Ads as the **Primary Conversion Action**.

### Why this event?
1.  **Avoid optimizing for low-intent noise**: Smart bidding strategies (like Target CPA) require high-intent signals. If we optimize for Step 1 or Step 2, the algorithm gets bloated optimizing for users who just open the form or begin typing, wasting budget on non-converting clicks.
2.  **Maximize Lead Quality**: Step 3 represents a confirmed booking with validation. This ensures we are training Google Ads to find actual patients, not just window shoppers.

### Conversion Window Configuration
-   **Recommendation**: Set the conversion window to **30 Days**.
-   **Justification**: Orthopaedic treatment is rarely an impulse buy. Patients with back or knee pain typically research clinics, check schedules, and consult family members. The average decision cycle ranges from 1 to 2 weeks. A 30-day conversion window captures these delayed conversions, feeding accurate data back to the Google Ads attribution model.

