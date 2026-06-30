# Task 01 - GTM Event Schema for OrthoNow

Before jumping into the table, here's how I thought about this. OrthoNow has zero event tracking right now, so the marketing team basically can't see what's working and what isn't. My job here is to map out every interaction on the site that matters for conversions, give each one a clean event name, decide how GTM should catch it, and tell GA4 what to do with that data once it shows up.

I've grouped this by the interactions listed in the brief: the booking form, call buttons, WhatsApp widget, the patient guide download, clinic page views, and blog scroll depth.

## Full Event Table

| Event Name | Trigger Type | Key Parameters | GA4 Report / Audience |
|---|---|---|---|
| booking_step_complete | Custom Event (dataLayer push) | step_number, step_name, clinic_location | Funnel Exploration - Booking Flow |
| call_now_clicked | Click - Just Links (tel: links) | page_location, clinic_name, button_position | Engagement > Events report, also feeds an audience for "intent but no booking" |
| whatsapp_widget_opened | Click - All Elements (matching widget button class/id) | page_location, device_category, time_on_page | Engagement > Events, audience for retargeting WhatsApp clickers who didn't book |
| patient_guide_form_submitted | Form Submission | page_location, form_id, lead_source | Conversions report, audience for nurture email/WhatsApp follow-up |
| patient_guide_downloaded | Click - Just Links (PDF link, fires after form_submitted) | file_name, file_extension, page_location | Engagement > File Downloads, feeds a "warm lead" audience |
| clinic_page_viewed | Page View (9 separate clinic pages) | clinic_name, city, page_location | Engagement > Pages and Screens report, audience segmented by city interest |
| blog_scroll_depth | Scroll Depth (25/50/75/90 thresholds) | scroll_percentage, article_title, time_on_page | Engagement > Events, audience for "engaged readers" to retarget with booking ads |

A few notes on why I made some of these calls:

For Call Now buttons, I used "Click - Just Links" instead of "Click - All Elements" because these are tel: links and GTM handles that cleanly without needing extra JS. I'm also tracking button_position because the same button shows up on homepage, clinic pages, and the landing page — knowing where someone clicked from tells you a lot about intent.

The patient guide is actually two events, not one. The form gets submitted first (name + phone), and only after that does the PDF download happen. If I just track the download, I lose the lead capture moment, and if I just track the form, I don't know if they actually got the resource. Splitting them gives marketing both signals.

Clinic page views could technically just be regular GA4 pageviews since GA4 tracks those automatically, but since there are 9 of them and the brief specifically calls them out, I'm treating them as a named custom event so they're easy to filter as a group in reports rather than digging through 9 separate URL paths.

## Google Ads Conversion Recommendation

Out of everything in this schema, I would import **booking_step_complete where step_number equals 3** (booking_confirmed) as the Google Ads conversion action, not the earlier steps and not the call/WhatsApp clicks.

The reason is pretty simple - Google Ads' automated bidding (Target CPA or Target ROAS) is only as good as the signal you feed it. If I imported step 1 (just picking a clinic) as the conversion, the system would start optimising toward people who barely showed intent, and you'd end up with a campaign that looks like it's performing but is actually just collecting browsers, not patients. A confirmed booking is the only point in this whole flow where you know for certain someone is actually going to show up at a clinic.

I'd set the conversion window to 30 days click-through, since healthcare decisions (especially around an injury) usually involve a few days of research before someone books, not an instant decision.

I would, however, keep booking_step_complete (step 1 and 2) and call_now_clicked as **secondary/micro conversions** that Google Ads can see in reporting but doesn't actively bid toward - those are still useful for understanding the funnel even if they're not the optimisation target.
