# OrthoNow — Namoza Developer Assignment

Welcome to the OrthoNow developer assignment repository. This workspace contains the GTM event schema, high-performance landing page build, and integration design answers developed for OrthoNow's digital expansion.

---

## Repository Structure & Deliverables

This repository is organized as follows:

-   **`task-01-gtm-schema/`**
    -   [`event-schema.md`](./task-01-gtm-schema/event-schema.md): A comprehensive GTM event tracking specification mapping interaction triggers, parameters, and GA4 target reports/audiences for all key site elements, plus Google Ads primary conversion recommendations.
    -   [`booking-form-datalayer.md`](./task-01-gtm-schema/booking-form-datalayer.md): Multi-step `dataLayer.push` JSON schemas and drop-off tracking strategies for the 3-step booking funnel, including a briefing note for the front-end engineering team.
-   **`task-02-landing-page/`**
    -   [`index.html`](./task-02-landing-page/index.html): A single, self-contained, mobile-first landing page. Built with zero external dependencies (system font stacks, inline styling, inline SVGs) to guarantee optimal loading times. It implements client-side validation and GTM dataLayer integration.
    -   [`pagespeed-screenshot.png`](./task-02-landing-page/pagespeed-screenshot.png): A screenshot showing a Mobile Performance score of 98 (with Accessibility, Best Practices, and SEO at 100) from the Chrome DevTools Lighthouse audit.
    -   `robots.txt`: Added to both the repository root and this directory to satisfy search engine crawling and indexing checks in Lighthouse.
-   **`task-03-integration/`**
    -   [`integration-design.md`](./task-03-integration/integration-design.md): A written architectural brief explaining the end-to-end integration flow between the landing page, a Vercel serverless function middleware, HubSpot CRM, and the Karix WhatsApp API. It specifically resolves HubSpot's phone number deduplication limitations and details fallback/monitoring mechanisms for the 2-minute SLA.

---

## How to Run & Verify Task 02 (Landing Page)

### Opening the Page
Because the landing page is designed to be fully self-contained, **no server setup is required**.
-   **Method 1 (Double-Click)**: Navigate to `task-02-landing-page/index.html` in your file explorer and double-click the file to open it in your default web browser.
-   **Method 2 (PowerShell/Terminal)**: Run the following command in your terminal from the project root:
    ```powershell
    Start-Process "task-02-landing-page/index.html"
    ```

### Verifying the GTM dataLayer Push
To verify that the GTM dataLayer push fires correctly, follows validation, and contains no PII:

1.  Open `task-02-landing-page/index.html` in your web browser (Google Chrome recommended).
2.  Right-click anywhere on the page and select **Inspect** (or press `F12` / `Ctrl+Shift+I` on Windows) to open Chrome DevTools.
3.  Click on the **Console** tab at the top of the DevTools panel.
4.  Fill out the form:
    -   Enter a name (e.g., *Ananth Rao*).
    -   Enter a valid 10-digit Indian mobile number starting with 6-9 (e.g., *9876543210*).
5.  Click the **Request Callback** CTA.
6.  Observe the Console. You will see a green-styled log statement printing the executed `dataLayer.push` object:
    ```javascript
    {
      event: "consultation_form_submitted",
      clinic_preference: "Bengaluru",
      lead_source: "google_ads_landing_page"
    }
    ```
7.  Verify in the console that `window.dataLayer` is accessible and contains the event by typing `window.dataLayer` and hitting Enter. Note that **no patient name or phone number** has been written to the dataLayer, preserving user privacy.
8.  Observe that the form has transition to the inline thank-you state immediately **without reloading the page**.

---

## License
This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.
