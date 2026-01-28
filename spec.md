# SPEC-2-Waitlist Page (Google Forms)

## Background

**Goal.** Create a lightweight, branded waitlist page for the P2P car rental MVP that funnels signups into **Google Forms → Google Sheet** for easy operations, no backend needed.

**Who this serves.** Prospective **renters** and **owners** in Pakistan who want early access; ops team needs clean data and segment tags.

**Assumptions (to refine).**

- Single scroll page with hero, value props, and a signup CTA that posts to **Google Forms** (embedded iFrame or direct link with prefill).
- Minimal build so any web dev can ship quickly; optional tracking (GA4/Meta Pixel) and anti‑spam (reCAPTCHA on the Google Form).
- Fields (tentative): Full Name, Email, Phone (Pak +92), City, Role (Renter/Owner/Both), Referral Code, How did you hear about us?, Consent checkbox.

We’ll now capture **Requirements** and then detail the **Method** (form field mapping, tracking, validation, deployment).

## Requirements

### Must‑Have

- Single, responsive landing page (mobile‑first) with:
  - Hero (headline, subhead), key value props, social proof placeholder, and a primary CTA.
  - **Signup CTA** that opens the **Google Form** (embedded iFrame OR new tab).

- Form fields (exactly): **Name, Email, City, Role (Renter/Owner/Both), Source, Consent**.
- Clear privacy/consent copy (Pakistan‑focused) and visible link to Privacy Policy.
- Basic anti‑spam via Google Forms (reCAPTCHA toggle on in Form settings).
- Accessibility: labels for all inputs (when embedded), semantic HTML, 44px tap targets.
- Fast load (<1s TTFB on decent 3G), image assets <200KB total.

### Should‑Have

- Dual‑language copy support (English primary, Urdu secondary) with a simple toggle.
- UTM parameter passthrough to the Google Form (so GA4 source/medium are captured in the Sheet).
- GA4 pageview + outbound click tracking for the form CTA.
- Simple success/thank‑you state after submission (either on Google Forms confirmation screen or a custom redirect page).

### Could‑Have

- Meta Pixel (ViewContent + Lead), LinkedIn Insight Tag.
- Prefill hints (e.g., city from URL param `?city=karachi`, role from `?role=renter`).
- Minimal A/B capability (alternate headline).

### Won’t‑Have (MVP)

- Custom backend or database.
- Complex animation or video backgrounds.
- Owner/renter segmentation logic beyond captured fields.

## Method

### Information Architecture & Flow

1. Visitor lands on `/waitlist` → reads hero & value props.
2. Primary CTA button constructs a **Google Forms prefill URL** with query params from the page (UTM and optional `city`/`role`).
3. CTA opens Google Form in a **new tab**. User submits form.
4. Google Forms success screen confirms submission (optional redirect using link on success screen back to your site `/thank-you`).

### Google Forms Field Mapping

Create a Google Form with these questions **(Short answer unless noted)**:

- Name _(required)_ → **entry.`<NAME_ID>`**
- Email _(required, validate email)_ → **entry.`<EMAIL_ID>`**
- City _(dropdown or short answer)_ → **entry.`<CITY_ID>`**
- Role _(Dropdown: Renter / Owner / Both)_ → **entry.`<ROLE_ID>`**
- Source _(Short answer; we’ll also pass UTM)_ → **entry.`<SOURCE_ID>`**
- Consent _(Checkbox: “I agree to be contacted…”)_ → **entry.`<CONSENT_ID>`**

> **How to get the entry IDs:** In the live form, right‑click → _View page source_ and find `entry.` patterns near your inputs (e.g., `entry.123456789`). Replace placeholders above and in code below.

### URL Construction (Prefill)

Base prefill URL format:

```
https://docs.google.com/forms/d/e/<FORM_ID>/viewform?usp=pp_url&entry.<NAME_ID>=<name>&entry.<EMAIL_ID>=<email>&entry.<CITY_ID>=<city>&entry.<ROLE_ID>=<role>&entry.<SOURCE_ID>=<source>&entry.<CONSENT_ID>=true
```

**Passthrough rules:**

- Read `utm_source`, `utm_medium`, `utm_campaign` from the landing page URL; concatenate into `Source` if present (e.g., `utm_source=meta&utm_medium=cpc → Source="meta:cpc"`).
- Also support friendly params: `?city=karachi` and `?role=renter` to prefill those fields.
- Encode values with `encodeURIComponent`.

### GA4 Tracking

- Fire `page_view` on load.
- Track click on **Join Waitlist** CTA with event `select_content` (or `click_waitlist_cta`).
- (Optional later) Track form success by creating a GA4 event based on visits to `/thank-you` or by using Google Tag Manager on the Form confirmation page.

### Page Structure & Styling

- Single responsive page (Tailwind or lightweight CSS), English copy by default with a small Urdu toggle.
- CTA opens form in a new tab (`target="_blank" rel="noopener"`).
- Accessibility: proper heading hierarchy, focus styles, and alt tags for any images.

### Reference Implementation (Vanilla HTML + minimal JS)

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Join the Waitlist — P2P Car Rental</title>
    <!-- GA4 -->
    <script
      async
      src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"
    ></script>
    <script>
      window.dataLayer = window.dataLayer || [];
      function gtag() {
        dataLayer.push(arguments);
      }
      gtag("js", new Date());
      gtag("config", "G-XXXXXXXXXX");
    </script>
    <style>
      body {
        font-family:
          system-ui,
          -apple-system,
          Segoe UI,
          Roboto,
          Helvetica,
          Arial,
          sans-serif;
        margin: 0;
      }
      .wrap {
        max-width: 960px;
        margin: 0 auto;
        padding: 24px;
      }
      .hero {
        padding: 56px 0;
      }
      .btn {
        display: inline-block;
        padding: 14px 22px;
        border-radius: 10px;
        border: 1px solid #ccc;
        text-decoration: none;
      }
      .btn:focus {
        outline: 3px solid #333;
      }
      .grid {
        display: grid;
        gap: 16px;
        grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
      }
    </style>
  </head>
  <body>
    <main class="wrap">
      <header class="hero">
        <h1>Borrow cars from locals. Earn from your car.</h1>
        <p>
          Launching soon in Karachi. Join the waitlist for early access and
          perks.
        </p>
        <p>
          <a id="cta" class="btn" href="#" target="_blank" rel="noopener"
            >Join the Waitlist</a
          >
        </p>
      </header>

      <section aria-labelledby="why">
        <h2 id="why">Why join?</h2>
        <div class="grid">
          <div>
            <h3>Convenient</h3>
            <p>Pick nearby cars with flexible hours.</p>
          </div>
          <div>
            <h3>Affordable</h3>
            <p>Transparent pricing and protection.</p>
          </div>
          <div>
            <h3>Earn</h3>
            <p>Owners monetize idle time safely.</p>
          </div>
        </div>
      </section>

      <footer style="margin-top:56px">
        <small
          >By joining, you agree to be contacted about launch updates.
          <a href="/privacy" target="_blank">Privacy Policy</a></small
        >
      </footer>
    </main>

    <script>
      // === CONFIG: Replace with your IDs ===
      const FORM_ID = "<FORM_ID>";
      const E = {
        NAME: "<NAME_ID>",
        EMAIL: "<EMAIL_ID>",
        CITY: "<CITY_ID>",
        ROLE: "<ROLE_ID>",
        SOURCE: "<SOURCE_ID>",
        CONSENT: "<CONSENT_ID>",
      };

      function getParam(name) {
        const url = new URL(window.location.href);
        return url.searchParams.get(name) || "";
      }

      function buildSource() {
        const src = getParam("utm_source");
        const med = getParam("utm_medium");
        const camp = getParam("utm_campaign");
        const plain = getParam("source");
        const parts = [];
        if (src) parts.push(`src:${src}`);
        if (med) parts.push(`med:${med}`);
        if (camp) parts.push(`cmp:${camp}`);
        if (plain) parts.push(`tag:${plain}`);
        return parts.join("|");
      }

      function buildPrefillUrl() {
        const base = `https://docs.google.com/forms/d/e/${FORM_ID}/viewform?usp=pp_url`;
        const q = new URLSearchParams();
        // Optional prefill from URL
        const city = getParam("city");
        const role = getParam("role");
        if (city) q.set(`entry.${E.CITY}`, city);
        if (role) q.set(`entry.${E.ROLE}`, role);
        const src = buildSource();
        if (src) q.set(`entry.${E.SOURCE}`, src);
        // Consent default true (checked)
        q.set(`entry.${E.CONSENT}`, "true");
        return `${base}&${q.toString()}`;
      }

      // Wire CTA
      const cta = document.getElementById("cta");
      const prefillUrl = buildPrefillUrl();
      cta.setAttribute("href", prefillUrl);

      // GA4 event on click
      cta.addEventListener("click", function () {
        if (window.gtag) {
          gtag("event", "click_waitlist_cta", {
            link_url: prefillUrl,
            page_location: window.location.href,
          });
        }
      });
    </script>
  </body>
</html>
```

**If you prefer React (Next.js/Vite)**, the same logic goes inside a client component/hooks; use `useEffect` to compute the prefill URL and set it on the button.

### Optional Urdu Toggle (very lightweight)

- Add a toggle button that swaps text nodes from an `en` and `ur` dictionary.

## Implementation

1. **Create Google Form** with the six fields and enable **reCAPTCHA** (Settings → Responses → Collect email off; Limit to 1 response off; CAPTCHA on). Note each field’s `entry.<ID>`.
2. **Publish the page** using the provided HTML or a React equivalent.
3. **Replace placeholders**: `G-XXXXXXXXXX`, `<FORM_ID>`, and each `<*_ID>`.
4. **Set DNS** for `waitlist.yourdomain.pk` → your host (Vercel/Netlify). Configure HTTPS.
5. **QA**:
   - UTM passthrough builds `Source` correctly.
   - City/role prefill works with `?city=karachi&role=owner`.
   - GA4 shows `page_view` and `click_waitlist_cta` events.
   - Accessibility: keyboard navigation and focus rings visible.

6. **Thank‑you route** (optional): create `/thank-you` with simple copy; link to socials.

## Milestones

- **M1 (0.5 day):** Google Form created with correct fields & IDs recorded.
- **M2 (0.5 day):** Static page scaffolded; content and styles added.
- **M3 (0.5 day):** GA4 wired, UTM passthrough verified.
- **M4 (0.5 day):** QA + launch on production domain.

## Gathering Results

- Monitor **Google Sheet** responses; filter by `Role`, `City`, and `Source`.
- In **GA4**, track page views and CTA clicks by UTM to gauge acquisition.
- Create a weekly export from the Sheet for CRM upload (CSV) with columns: Timestamp, Name, Email, City, Role, Source, Consent.
