# Waitlist Page Implementation Guide

## Quick Start

### Files Created

- **index.html** – Main waitlist landing page with embedded form
- **SETUP.md** – This file

---

## Step 1: Create Google Form

### 1.1 Create a new Google Form

1. Go to [forms.google.com](https://forms.google.com)
2. Click **"Create"** → **"Blank form"**
3. Set the title to: **"ReCa Waitlist"**

### 1.2 Add Form Fields

Create the following questions in this order:

#### Question 1: Name

- **Question text:** "Full Name"
- **Type:** Short answer
- **Required:** Yes

#### Question 2: Email

- **Question text:** "Email Address"
- **Type:** Short answer
- **Required:** Yes
- **Validation:** Check "Email" under Data validation

#### Question 3: City

- **Question text:** "Which city are you in?"
- **Type:** Short answer
- **Required:** Yes

#### Question 4: Role

- **Question text:** "Are you interested as a..."
- **Type:** Dropdown
- **Options matching our form:**
  - Renter (I want to rent cars)
  - Owner (I want to rent out my car)
  - Both
- **Required:** Yes

#### Question 5: Source

- **Question text:** "How did you hear about us?"
- **Type:** Short answer
- **Required:** No
- **Note:** This will be hidden and auto-filled

#### Question 6: Consent

- **Question text:** "I agree to be contacted about launch updates and offers."
- **Type:** Checkbox
- **Required:** Yes
- **Note:** This handles the checkbox logic

### 1.3 IMPORTANT: Disable Restrict to Users

1. Click the **Settings icon** (gear) at the top right
2. Go to **"Responses"**
3. **TURN OFF "Restrict to users in [Your Org]"** if it exists.
4. **TURN OFF "Limit to 1 response"**. The custom form usage requires these to be OFF for anonymous submission.

### 1.4 Get Form and Field IDs

#### Get Form ID:

1. Look at the form URL: `https://docs.google.com/forms/d/e/1FAIpQLSd...`
2. The ID is the long string between `/d/e/` and `/viewform` or `/edit` in some cases (use the preview link ID)
3. Copy and save it

#### Get Entry IDs:

1. **Share** the form (click Share button) -> **Get pre-filled link**
2. Fill in dummy data for each field (Name: "testname", Email: "testemail").
3. Click "Get Link".
4. Copy link and look for parameters: `&entry.123456=testname`
5. Map each ID to the corresponding field:

```
entry.123456789 = Name field
entry.987654321 = Email field
entry.456789123 = City field
entry.789123456 = Role field
entry.321654987 = Source field
entry.654987321 = Consent field
```

https://docs.google.com/forms/d/e/1FAIpQLSd9iBMVUAMGc3oAXSGEUOzxWfBwB1XKj3KCLxR3NS61TraZ2w/viewform?usp=pp_url&entry.808644764=k&entry.1224357367=jj&entry.958994699=kjdjkd&entry.299895401=Renter+(I+want+to+rent+cars)&entry.1590272573=nk&entry.1639670714=Yes

---

## Step 2: Configure the Page

### 2.1 Update `index.html`

Find this section near the bottom of the file script:

```javascript
// ===== CONFIGURATION (Replace with your actual IDs) =====
const FORM_ID = "REPLACE_WITH_FORM_ID";
const ENTRY_IDS = {
  NAME: "REPLACE_WITH_NAME_ENTRY_ID",
  ...
};
```

Replace the placeholders with your actual IDs.

---

## Step 3: Deploy the Page

### Option A: Vercel (Recommended - Free & Easy)

1. Create account at [vercel.com](https://vercel.com)
2. Import your files
3. Deploy
4. Your site will be live at `https://your-project.vercel.app`

---

## Step 4: QA Checklist

- [ ] **Form Submission**: Fill out the form on the page, submit.
- [ ] **Success Message**: Verify the success screen appears without page reload.
- [ ] **Data Check**: Verify data appears in Google Sheet immediately.
- [ ] **Mobile Responsive**: Test on phone/tablet.
- [ ] **UTM Tracking**: Visit with `?utm_source=test`, verify it lands in the Sheet.

---

## Troubleshooting

### Form submits but nothing happens (no data in sheet)

- Check **FORM_ID** is correct.
- Check **Entry IDs** are correct.
- Ensure Form Settings > Responses > "Restrict to users" is **OFF**.
- Ensure Form Settings > Responses > "Limit to 1 response" is **OFF**.

### Form redirects to Google Page instead of staying

- This usually means the `target="hidden_iframe"` attribute is missing or the iframe ID doesn't match.

### CORS Errors in Console

- This is expected when posting to Google Forms from a custom domain. The submission **still works**, but the browser flags the response. Our code uses the `hidden_iframe` workaround to ignore this error and assume success if the request was sent.

---
