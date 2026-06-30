# Booking Form - Funnel Drop-off Tracking

This is the part that actually needs real engineering thought, not just naming events.

Here's the thing about a 3-step form — GTM cannot see what's happening *inside* a form on its own. It can fire something when a page loads, or when a generic click happens, but it has no native concept of "user moved from step 2 to step 3 without reloading the page." That only works if the front-end code itself tells GTM that something happened, using a dataLayer push.

So for every single step transition, the front-end developer needs to add a line of JavaScript that pushes an object into window.dataLayer at the exact moment the user completes that step. GTM is just listening for these pushes through a Custom Event trigger - it's not detecting anything on its own.

Here's what each step's push looks like:

**Step 1 - user picks clinic and specialty**

```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Knee and Joint"
}
```

**Step 2 - user enters their details**

I deliberately left phone and name out of this push. There's no reason to put PII into the dataLayer when all GA4 needs is confirmation the step happened.

```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "patient_details_entered",
  "preferred_date": "2025-08-15",
  "has_phone_number": true
}
```

**Step 3 - booking gets confirmed**

```json
{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Knee and Joint",
  "booking_id": "ONB-20250812-004"
}
```

In GTM, I'd set up one Custom Event trigger listening for the event name booking_step_complete, and then use the step_number and step_name variables to differentiate which step fired. One trigger, one tag, three different parameter sets — no need to build three separate triggers.

### Getting this into GA4 Funnel Exploration

Once these three events are flowing into GA4, I'd build a Funnel Exploration with three steps, each one filtered on step_number equalling 1, 2, and 3. GA4 will then show you the percentage of users who completed step 1 but never reached step 2, and the same for step 2 to step 3. That's literally where your drop-off numbers come from — not from guessing, but from the actual step_number values in each push.

If I wanted to dig in further, I'd add a breakdown dimension on clinic_location inside that same funnel, so I can see if drop-off is worse for a specific clinic (maybe their slot availability shown in step 1 is bad, scaring people off before step 2).

### Who actually writes this code

I want to be upfront about this because it's a common misunderstanding — GTM does not write or trigger these pushes by itself. I (or whoever owns the GTM container) can only configure tags and triggers to listen for an event name. The actual act of pushing {event: "booking_step_complete", step_number: 2, ...} into the dataLayer has to be written by the front-end developer, inside the form's JavaScript, at the exact moment step 2 finishes.

If I were briefing the front-end team to add this for step 2 specifically, here's roughly what I'd tell them:

- When the "Continue" button on step 2 is clicked and validation passes (not before validation, otherwise we'd be tracking failed attempts as successes), run a small JS function that pushes the object above into window.dataLayer
- Make sure window.dataLayer = window.dataLayer || [] exists somewhere before this runs, in case GTM hasn't initialized yet
- Don't put name or phone number directly in the push - just a boolean flag if needed
- Use the same field names every time (step_number, step_name) so GTM variables stay consistent across all three steps
- Test it by opening the browser console and typing dataLayer after clicking continue - the object should show up at the bottom of the array
