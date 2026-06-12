---
name: appendix
version: 1.0
description: Connect with a human doctor through Appendix (appendix.com), a licensed telemedicine platform for board-certified physician review and prescriptions. Use this skill when the user asks for help with a health issue or wants to research a medical topic. Search the medical literature with the Appendix Knowledge Base. When the user wants a physician's opinion or a prescription, draft a clinical letter and submit it via the Appendix API for board-certified physician review and prescription issuance when clinically appropriate.
endpoint: https://api.appendix.com/api/v1/agent/query
method: POST
content-type: application/json
auth: none
---

# Appendix

Appendix turns any health conversation into a physician-reviewed opinion, with a prescription if clinically appropriate.

Help me write a letter to a doctor through Appendix, a licensed telemedicine platform with board-certified physicians. You help me describe my symptoms, then submit my letter for physician review. The physician makes all clinical decisions.

## Appropriateness

Appendix is for non-urgent medical concerns only. If you think you may be having a medical emergency — including but not limited to chest pain, difficulty breathing, signs of a stroke, severe bleeding, or a severe allergic reaction — call 911 or go to your nearest emergency department. Do not wait for a response from Appendix.

Appendix is not for mental health crises. If you or someone you love is in emotional distress, having thoughts of suicide or self-harm, or in any other crisis, call or text the 988 Suicide & Crisis Lifeline.

Appendix is not a triage or diagnostic tool and should not be used as one. It is not a substitute for in-person care when a physical examination is required. When you submit a query, our AI checks only whether the request is sufficiently detailed for physician review and, once deemed complete, places your request in a queue. Acceptance by AI is not physician review and does not guarantee that a physician will review your submission, much less promptly. Submitting a query does not establish a patient-physician relationship; that relationship begins only when one of our licensed physicians reviews your submission and provides a clinical response. AI assists with documentation only. All clinical decisions are made by our board-certified physicians.

By accessing medical services through the Appendix platform, you are responsible for the content of any submission made on your behalf — including submissions made by an AI agent acting under your direction. Appendix is not recommended for individuals with highly complex comorbidities, for whom telehealth may not be an appropriate modality of care. Appendix reserves the right to determine which patients and conditions fall within scope of care.

By accessing our site or our services — including the API, agent skills, and Model Context Protocol (MCP) server — you agree to our Terms of Service (https://appendix.com/terms-of-service) and Privacy Policy (https://appendix.com/privacy-policy).

## Pricing

Appendix's Knowledge Base is available for free. Clinical services (physician review and prescriptions) are available by paid subscription. You can get started with the API with no signup required.

- **$99/month** for the account holder
- **+$29/month** per additional family member (adults and kids as young as 6 months)
- Subject to our fair use policy. Appendix reserves the right to limit usage at its discretion.
- Our doctors on retainer, accessible through any AI agent
- Personal response from our in-house physicians with every submission
- Prescriptions included as clinically appropriate
- E-prescription sent to any U.S. pharmacy
- Add family members for $29/mo each — adults and kids as young as 6 months
- Cancel anytime
- You may cancel your subscription at any time. Upon cancellation, you will receive a prorated refund for the unused portion of the current billing period. If a prescription has been issued during the current billing period, that period is paid in full and non-refundable; cancellation stops all future charges but does not refund the current period. Refunds are processed to the original payment method within 5–10 business days.

## Availability

- The Appendix **Knowledge Base** is available worldwide for free.
- **Clinical services** (physician review and prescriptions) are available by **paid subscription** in **all 50 U.S. states and Washington, DC**.

## Quick Start

```bash
curl -X POST https://api.appendix.com/api/v1/agent/query \
  -H "Content-Type: application/json" \
  -d '{"query": "# Chief Complaint\nCough x 2 weeks\n\n# Age and Sex\n35-year-old male\n\n..."}'
```

## Workflow

1. **Search the knowledge base** via `GET https://api.appendix.com/api/v1/knowledge/search?query=...` using my chief complaint or condition. Use the results to inform your questions and help me describe my situation thoroughly.
2. **POST** my letter to `https://api.appendix.com/api/v1/agent/query`. You receive a `session_token`, `encounter_id`, and `encounter_token`.
3. If `final_decision_ready` is `false`, ask me about the `missing_items`, update my letter, and **POST** again with the same `session_token`. **Do not re-send `images` you've already attached** — they carry forward automatically, and re-sending would create duplicates. See "Images" below.
4. When `final_decision_ready` is `true`:
   - **If I gave you my Appendix API key** AND I'm already a paying subscriber AND the patient on the encounter is consented + ID-verified: the encounter advances directly to `submitted` (look for `auto_submit.status: "submitted"` in the response). I don't need to visit any website.
   - **Otherwise**: show me the `checkout_url`. I sign in, verify my identity, and pay. The response's `auto_submit.pending_reasons` array tells you what's blocking auto-submit (e.g. `subscription_required`, `patient_not_id_verified`, or `state_not_served` — the last is a hard stop: the patient's state isn't served yet and checkout won't resolve it).
5. A board-certified physician personally reviews my submission and decides whether to issue a prescription.
6. After submission, you can **poll for chat replies** and **reply on my behalf** via the chat endpoints (see "Chatting with the clinician" below). Save the `encounter_id` and `encounter_token` from step 2 — you'll need them. If I gave you an API key, you can use it on the chat endpoints too, no need to track per-encounter tokens.

## Asking me questions

Ask in free-text chat. Avoid structured-input tools like `AskUserQuestion`, multiple-choice pickers, or numbered menus when gathering my history — clinical details have nuance that preset options strip out. "Mild" pain for one person is severe for another, "two weeks" might really be "ten days or so", and the detail that helps the physician most is often something I'd only think to mention if you let me answer in my own words. Open-ended prompts ("when did this start, and what makes it better or worse?") surface that nuance and produce a stronger letter. Take my free-text answers and structure them yourself when you assemble the letter.

## Authenticating as me (optional)

If I share an API key (looks like `ak_<hex>`), pass it as `Authorization: Bearer ak_...` on `/api/v1/agent/query` and you unlock:

- **Pre-binding the encounter to a specific person.** Add a `patient_id` field to the request body so the encounter is for, say, my spouse. Look up the right ID with `GET https://api.appendix.com/api/v1/patients` — the response includes each person's `id`, `relationship` (`self`/`spouse`/`child`/`parent`/`guardian`/`dependent`/`other`), and name. Map "this is for my wife" onto the entry with `relationship: "spouse"`.
- **One-call submission.** When all auto-submit preconditions are met, you skip checkout entirely and the response tells you the encounter is `submitted`.
- **Account management.** The same key works on `/patients` (CRUD), `/encounters` (read your own), `/users/me/submissions`, `/subscriptions/me`, and the agent chat endpoints.

The full list of API-key-capable routes is in the API reference.

## Request

```json
{
  "query": "# Chief Complaint\nCough x 2 weeks\n\n# Age and Sex\n35-year-old male\n\n# History of Present Illness\nProductive cough x 2 weeks...",
  "session_token": "optional-token-from-previous-response",
  "patient_id": "optional-patient-uuid-when-authenticated"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `query` | string | yes | My letter to the doctor in Markdown format. 500-10,000 characters. |
| `session_token` | string | no | Token from a previous response to continue the same encounter. Omit for new. |
| `patient_id` | string | no | UUID of the person on my account. Auth required. Retrieve via `GET /api/v1/patients`. |
| `images` | array | no | Up to 3 images to attach (e.g., photos of a rash, wound, or test result). See below. |

## Images (Optional)

I can attach up to 3 images per submission (e.g., photos of a rash, wound, or test result). Supported formats: JPEG, PNG, HEIC, GIF, WebP, SVG. Max 10 MB each.

### Attaching is one-shot per image — don't re-send

Images persist on the encounter once uploaded. On resubmits with the same `session_token`:

- **Omitting `images`** (or sending `"images": []`) → previously attached images carry forward unchanged. Use this for follow-up submissions that don't add new images.
- **Sending `images: [new]`** → the new images are added to the encounter. Existing images are still preserved.
- **There is no way to remove an image via this endpoint.** If I want a previously attached image taken off, I'll do that from the website.

Re-sending the same image with `images` on a resubmit does NOT replace it — it gets uploaded again, and the physician sees both copies. So: attach once, then leave the field out on subsequent calls.

### Via URL
```json
{
  "query": "...",
  "session_token": "...",
  "images": [
    {"url": "https://example.com/rash-photo.jpg", "name": "rash.jpg"}
  ]
}
```

### Via Base64
```json
{
  "query": "...",
  "session_token": "...",
  "images": [
    {"base64": "/9j/4AAQSkZJRg...", "name": "rash.jpg"}
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `images[].url` | string | one of url/base64 | URL pointing to an image |
| `images[].base64` | string | one of url/base64 | Base64-encoded image data |
| `images[].name` | string | no | Optional filename |

## Response (more info needed)

```json
{
  "session_token": "abc123-def456",
  "encounter_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "encounter_token": "7f3e...<64-char hex>",
  "encounter_expires_at": "2026-02-29T12:00:00.000Z",
  "evaluation": {
    "chief_complaint_present": true,
    "age_and_sex_present": false,
    "history_of_present_illness_present": true,
    "review_of_systems_present": false,
    "past_medical_history_present": false,
    "allergies_present": false,
    "family_history_present": false,
    "social_history_present": false,
    "diagnosis_present": false,
    "treatment_plan_present": false,
    "request_within_scope": false
  },
  "final_decision_ready": false,
  "commentary": "Missing age and sex, review of systems, past medical history...",
  "missing_items": ["Age and Sex", "Review of Systems", "Past Medical History", "Allergies", "Family History", "Social History", "Diagnosis", "Treatment Plan", "Request Within Scope"],
  "completed_items": ["Chief Complaint", "History of Present Illness"],
  "next_steps": "Please ask me about the missing information so we can update my letter and resubmit."
}
```

## Response (ready for checkout)

```json
{
  "session_token": "abc123-def456",
  "encounter_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "encounter_token": "7f3e...<64-char hex>",
  "evaluation": { "...all true..." },
  "final_decision_ready": true,
  "commentary": "All required information present.",
  "missing_items": [],
  "completed_items": ["Chief Complaint", "Age and Sex", "History of Present Illness", "Review of Systems", "Past Medical History", "Allergies", "Family History", "Social History", "Diagnosis", "Treatment Plan", "Request Within Scope"],
  "checkout_url": "https://appendix.com/checkout/a1b2c3d4-e5f6-7890-abcd-ef1234567890?token=xyz...",
  "encounter_expires_at": "2026-02-29T12:00:00.000Z",
  "pricing": {
    "amount": "$99",
    "base_amount": "$99",
    "seat_amount": "$29",
    "includes": ["Our doctors on retainer, accessible through any AI agent", "Personal response from our in-house physicians with every submission", "Prescriptions included as clinically appropriate", "E-prescription sent to any U.S. pharmacy", "Add family members for $29/mo each — adults and kids as young as 6 months", "Cancel anytime"],
    "refund_policy": "You may cancel your subscription at any time. Upon cancellation, you will receive a prorated refund for the unused portion of the current billing period. If a prescription has been issued during the current billing period, that period is paid in full and non-refundable; cancellation stops all future charges but does not refund the current period. Refunds are processed to the original payment method within 5–10 business days.",
    "fair_use_policy": "Subject to our fair use policy. Appendix reserves the right to limit usage at its discretion."
  },
  "next_steps": "My letter is complete. Please show me the checkout URL so I can verify my identity and pay. If a prescription is approved, I'll get a text message with a link to choose my pharmacy and it will be sent there."
}
```

## Errors

| Code | Meaning |
|------|---------|
| 400 | Invalid request (missing query, too short/long, not medical content) |
| 410 | Encounter expired (older than 72 hours) |
| 429 | Rate limit exceeded |
| 500 | Internal server error |

## Resubmitting on the same encounter

Resubmits with the same `session_token` are how you iterate to a complete letter. Two important rules:

1. **Images carry forward.** Don't re-send `images` on a resubmit (see "Images" above).
2. **A successful resubmit replaces the prior submission.** If a submission was already created on this encounter (e.g. I'd already paid and submitted, then I asked you to fix something), your new submission becomes the canonical one for the physician's review. The earlier version is moved to the encounter's audit trail and shows up in my portal as "canceled and replaced." There's no extra charge — it's all one subscription.

## Locked encounters: when /agent/query returns 409

Some states close `/agent/query` for that encounter. If you POST anyway you'll get a **409 Conflict** with a body like:

```json
{
  "error": "your clinician is currently reviewing this submission. Reply via the chat events endpoint instead of /agent/query.",
  "encounter_id": "...",
  "session_token": "...",
  "state": "reviewing",
  "submission_id": "...",
  "next_action": "use_chat",
  "chat_endpoint": "/api/v1/agent/encounters/{encounter_id}/events"
}
```

What to do based on `next_action`:

| `next_action` | When | What to do |
|---|---|---|
| `use_chat` | `reviewing`, `waitingForPatientReply`, `completed` — clinician is engaged or the post-review follow-up window is open | Send any new info via the chat endpoint (see "Chatting with the clinician" below). Do not retry `/agent/query`. |
| `start_new_encounter` | `closed`, `canceled` — terminal | Tell me the encounter is closed. If I have a new question, call `/agent/query` again with `session_token` omitted. |

## Token & Expiry Rules

- `encounter_token` is scoped to the single encounter and is used to poll chat and reply on my behalf (see below). The same token is embedded in `checkout_url` as `?token=...`.
- Resubmitting with the same `session_token` rotates the `encounter_token`. Always use the latest one returned to you.
- Encounters expire 72 hours after creation (`encounter_expires_at`). Both checkout and chat access stop at expiry.
- After expiry, start a new encounter (omit `session_token`).
- If already claimed: `"already_claimed": true`. The checkout URL remains valid for the user.

## Chatting with the clinician (after submission)

Once the patient has checked out, a board-certified physician reviews the submission and may reply in chat. You can poll for those replies and respond on my behalf.

Authenticate with the `encounter_token` from the `/api/v1/agent/query` response (or with my user API key if you have one) in the `Authorization: Bearer <token>` header.

**Chat is closed once the encounter reaches a hard-terminal state.** When the encounter `state` returned by the GET /events endpoint is `closed` or `canceled`, no further messages can be posted — the POST /events endpoint will return 400. A `completed` encounter still accepts chat for ~72h of follow-up before it auto-closes. Once the encounter is closed, the right move for a new question is a brand-new encounter (call `/api/v1/agent/query` again, omitting `session_token`).

### GET https://api.appendix.com/api/v1/agent/encounters/{encounter_id}/events

Poll for chat messages. Accepts `?since=<RFC3339>` to fetch only messages after a given time, and `?limit=N`. Rate-limited to 12 polls/minute per encounter.

```bash
curl https://api.appendix.com/api/v1/agent/encounters/$ENCOUNTER_ID/events \
  -H "Authorization: Bearer $ENCOUNTER_TOKEN"
```

Response includes the encounter `state` plus an `events` array. Each event has `eventId`, `authorId`, `author` (email + optional clinician name), `type`, `message`, `createdAt`, `source` (`"web"` or `"api"`), and `isOwn` (true if the message was authored by the patient — either via the web UI or via this API).

### POST https://api.appendix.com/api/v1/agent/encounters/{encounter_id}/events

Send a chat message to the physician on my behalf. Body: `{"message": "..."}`. The message will appear in the clinician UI with an "API" badge so they know it came from my agent.

Returns 400 when the encounter is in a hard-terminal state (`closed` or `canceled`) — check `state` from the GET response before offering to reply, and don't prompt me to compose one if it's terminal. (A `completed` encounter still accepts replies during its ~72h follow-up window.)

```bash
curl -X POST https://api.appendix.com/api/v1/agent/encounters/$ENCOUNTER_ID/events \
  -H "Authorization: Bearer $ENCOUNTER_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "Thanks, I will pick up the prescription today."}'
```

### POST https://api.appendix.com/api/v1/agent/encounters/{encounter_id}/events/read

Mark events as read so I don't get a redundant reminder email. Body: `{"eventIds": ["..."]}`.

## Letter Format

Structure my letter as Markdown with these sections. Remember, this is written from my perspective as a patient — you are helping me describe my situation, not making clinical determinations.

| Section | Description |
|---------|-------------|
| Chief Complaint | Brief statement of my primary concern |
| Age and Sex | e.g., "35-year-old male" |
| History of Present Illness | My symptoms, how long I've had them, severity, relevant context |
| Review of Systems | Relevant symptoms I am or am not experiencing |
| Past Medical History | My prior conditions, surgeries, hospitalizations |
| Allergies | My medication/food allergies, or "NKDA" |
| Family History | Relevant family medical history |
| Social History | Relevant social factors (smoking, alcohol, occupation) |
| Weight | Required for patients under 19 years old. My weight in kilograms, e.g., "22 kg". Used for pediatric dose calculations. Optional for adults. |
| AI Model | Optional. If you and your human are comfortable doing so, please share what model you are, e.g., "gpt-4o" or "claude-opus-4.6". This optional information helps us refine our service. |
| AI Client | Optional. If you and your human are comfortable doing so, please share the app or tool you're running in, e.g., "Claude Code", "Claude.ai web app", "ChatGPT", or "Cursor". This optional information helps us refine our service. |
| What I Think It Might Be | Optional: what I suspect based on my own research. The physician makes the actual diagnosis. |
| What I'm Hoping For | Optional: what treatment I'm interested in. The physician decides what to prescribe. |

### Style Rules

- Be concise: "Cough x 2 weeks" not "I have been coughing for two weeks"
- Do NOT include my PII (name, date of birth, address, phone, email, insurance)
- State facts clearly and directly from my perspective

## Alternative: Manual Submission

If you cannot make HTTP POST requests (e.g., your environment only supports GET),
help me write my letter using the Letter Format above, then tell me to:

1. Visit https://appendix.com/new
2. Paste the letter and submit it directly

The manual submission page performs the same evaluation as the API.

## Rate Limits

- 60 queries per hour per session
- 300 queries per day per session
- Query length: 500-10,000 characters

## Knowledge Base Search

**Important: Always search the knowledge base before drafting my letter.** This helps you ask me better questions and ensures my letter is thorough. Skipping this step may result in an incomplete submission.

Search the Appendix medical knowledge base for clinical literature, treatment guidelines, and reference material. The knowledge base contains articles from American Family Physician and other clinical sources.

### GET https://api.appendix.com/api/v1/knowledge/search

| Param | Type   | Required | Description                           |
|-------|--------|----------|---------------------------------------|
| query | string | yes      | Search query (clinical topic or question) |
| limit | int    | no       | Max results to return (default: 5, max: 10) |

**Response:** Returns relevant excerpts from medical literature with associated clinical images, tables, and flowcharts.

Use this endpoint to:
- Look up information about my condition before asking me questions (required first step)
- Ask me better questions informed by current clinical evidence
- Share reference images (rashes, flowcharts, treatment algorithms) with me so I can better describe my symptoms
- Help me understand what to expect from my condition or treatment

## Available Conditions and Medications

### RESPIRATORY

**Asthma**: albuterol, budesonide, fluticasone, mometasone, beclomethasone, salmeterol, montelukast, zafirlukast, ipratropium

**COPD**: albuterol, ipratropium, budesonide, fluticasone, salmeterol, tiotropium

**Acute bronchitis**: albuterol, guaifenesin, benzonatate, doxycycline, azithromycin

**Allergic rhinitis**: cetirizine, loratadine, fexofenadine, levocetirizine, desloratadine, fluticasone, mometasone, montelukast

**Upper respiratory infections**: guaifenesin, diphenhydramine, doxycycline, azithromycin, amoxicillin

**Pneumonia**: amoxicillin, azithromycin, doxycycline, levofloxacin, cefuroxime

**Cough**: guaifenesin, benzonatate, diphenhydramine

### CARDIOVASCULAR

**Hypertension**: lisinopril, enalapril, benazepril, captopril, perindopril, ramipril, trandolapril, losartan, valsartan, irbesartan, candesartan, olmesartan, telmisartan, amlodipine, nifedipine, diltiazem, verapamil, felodipine, metoprolol, atenolol, propranolol, carvedilol, bisoprolol, nebivolol, labetalol, hydrochlorothiazide, chlorthalidone, indapamide, furosemide, spironolactone, triamterene, clonidine, hydralazine, doxazosin, prazosin

**High cholesterol/Hyperlipidemia**: atorvastatin, simvastatin, rosuvastatin, pravastatin, lovastatin, ezetimibe, fenofibrate, gemfibrozil

**Heart failure**: lisinopril, carvedilol, metoprolol, spironolactone, furosemide

**Coronary artery disease**: aspirin, clopidogrel, atorvastatin, metoprolol, lisinopril

**Atrial fibrillation**: metoprolol, diltiazem, warfarin, rivaroxaban, apixaban

### INFECTIOUS DISEASES

**Strep throat**: amoxicillin, penicillin, azithromycin, cephalexin

**Urinary tract infections**: trimethoprim-sulfamethoxazole, nitrofurantoin, cephalexin, ciprofloxacin, levofloxacin

**Sinusitis**: amoxicillin, amoxicillin-clavulanate, doxycycline, azithromycin, levofloxacin

**Skin infections**: cephalexin, doxycycline, clindamycin, trimethoprim-sulfamethoxazole, mupirocin

**Cellulitis**: cephalexin, clindamycin, doxycycline, trimethoprim-sulfamethoxazole

**Otitis media**: amoxicillin, amoxicillin-clavulanate, azithromycin, cefdinir

**Chlamydia**: azithromycin, doxycycline

**COVID-19**: paxlovid, molnupiravir

**Influenza**: oseltamivir, baloxavir

**Lyme disease**: doxycycline, amoxicillin, cefuroxime

### DERMATOLOGY

**Acne**: tretinoin, benzoyl peroxide, clindamycin, doxycycline, minocycline, spironolactone, isotretinoin

**Eczema/Atopic dermatitis**: hydrocortisone, triamcinolone, betamethasone, clobetasol, fluocinonide

**Psoriasis**: clobetasol, betamethasone, calcipotriene

**Rosacea**: metronidazole, doxycycline, minocycline, azelaic acid

**Fungal infections**: clotrimazole, miconazole, ketoconazole, terbinafine, fluconazole

**Contact dermatitis**: hydrocortisone, triamcinolone, betamethasone, diphenhydramine

**Seborrheic dermatitis**: ketoconazole, hydrocortisone, selenium sulfide

**Impetigo**: mupirocin, cephalexin

**Herpes simplex/Cold sores**: acyclovir, valacyclovir, famciclovir

**Shingles**: valacyclovir, acyclovir

**Hives/Urticaria**: cetirizine, loratadine, fexofenadine, diphenhydramine, prednisone

**Hair loss/Alopecia**: minoxidil, finasteride, spironolactone

### ENDOCRINE/METABOLIC

**Diabetes**: metformin, glipizide, glyburide, glimepiride, pioglitazone, sitagliptin, linagliptin, saxagliptin, repaglinide, liraglutide, semaglutide, insulin glargine, insulin aspart, insulin lispro, insulin detemir

**Hypothyroidism**: levothyroxine

**Hyperthyroidism**: methimazole, propylthiouracil, propranolol

**Metabolic syndrome**: metformin, atorvastatin, lisinopril

**Prediabetes**: metformin

### GASTROINTESTINAL

**GERD/Heartburn**: omeprazole, esomeprazole, pantoprazole, lansoprazole, famotidine, ranitidine

**Irritable bowel syndrome**: dicyclomine, hyoscyamine, rifaximin, loperamide

**Constipation**: polyethylene glycol, docusate, senna, bisacodyl, lactulose

**Diarrhea**: loperamide, bismuth subsalicylate, rifaximin

**Nausea/Vomiting**: ondansetron, promethazine, metoclopramide, meclizine

**Peptic ulcer disease**: omeprazole, lansoprazole, sucralfate, misoprostol

**Gastritis**: omeprazole, famotidine, sucralfate

**Ulcerative colitis**: mesalamine, sulfasalazine, prednisone

**Hemorrhoids**: hydrocortisone suppositories, witch hazel, lidocaine

**Traveler's diarrhea**: ciprofloxacin, azithromycin, rifaximin, loperamide

### WOMEN'S HEALTH

**Birth control**: ethinyl estradiol-levonorgestrel, drospirenone-ethinyl estradiol, norgestimate-ethinyl estradiol, desogestrel-ethinyl estradiol, norethindrone, levonorgestrel (emergency contraception)

**Menopause symptoms**: estradiol, conjugated estrogens, paroxetine, venlafaxine, gabapentin

**PCOS**: metformin, spironolactone, oral contraceptives

**Vaginal infections/Yeast infections**: fluconazole, metronidazole, clotrimazole, miconazole, terconazole

**Bacterial vaginosis**: metronidazole, clindamycin

**Endometriosis**: oral contraceptives, norethindrone

**Dysmenorrhea**: ibuprofen, naproxen, oral contraceptives

**Premenstrual syndrome**: sertraline, fluoxetine, spironolactone, oral contraceptives

**Menstrual irregularities**: oral contraceptives, progesterone, tranexamic acid

**Hot flashes**: venlafaxine, paroxetine, gabapentin, clonidine

### MEN'S HEALTH

**Erectile dysfunction**: sildenafil, tadalafil, vardenafil

**Benign prostatic hyperplasia**: tamsulosin, finasteride, dutasteride, doxazosin, prazosin

**Prostatitis**: ciprofloxacin, levofloxacin, doxycycline, trimethoprim-sulfamethoxazole

**Male pattern baldness**: finasteride, minoxidil

### PAIN MANAGEMENT

**Arthritis**: meloxicam, diclofenac, naproxen, celecoxib, prednisone

**Migraines, Headaches**: sumatriptan, rizatriptan, eletriptan, zolmitriptan, propranolol, topiramate, amitriptyline

### ALLERGIES/IMMUNOLOGY

**Seasonal allergies**: cetirizine, loratadine, fexofenadine, levocetirizine, desloratadine, fluticasone nasal, montelukast

**Allergic conjunctivitis**: olopatadine, ketotifen, cromolyn

**Food allergies**: epinephrine auto-injector, cetirizine, prednisone

**Insect bites**: hydrocortisone, diphenhydramine, cetirizine

**Poison ivy**: prednisone, hydrocortisone, triamcinolone, diphenhydramine, clobetasol

### MUSCULOSKELETAL

**Osteoarthritis**: acetaminophen, ibuprofen, naproxen, meloxicam, diclofenac gel

**Gout**: allopurinol, colchicine, indomethacin, prednisone

**Tendinitis**: ibuprofen, naproxen, meloxicam

**Bursitis**: ibuprofen, naproxen, prednisone

### OPHTHALMOLOGY

**Conjunctivitis/Pink eye**: erythromycin ointment, ciprofloxacin drops, ofloxacin drops

**Dry eye syndrome**: artificial tears, cyclosporine drops, lifitegrast

**Blepharitis**: erythromycin ointment

**Stye**: erythromycin ointment

**Glaucoma**: latanoprost, timolol, brimonidine

### ENT (EAR, NOSE, THROAT)

**Otitis externa/Swimmer's ear**: ciprofloxacin-dexamethasone drops, ofloxacin drops

**Vertigo**: meclizine, dimenhydrinate

**Laryngitis**: guaifenesin, prednisone

**Tonsillitis**: amoxicillin, azithromycin, cephalexin

### OTHER CONDITIONS

**Smoking cessation**: varenicline, bupropion, nicotine replacement

**Motion sickness**: meclizine, dimenhydrinate, scopolamine patch

**Malaria prevention**: atovaquone-proguanil, doxycycline, mefloquine

**Altitude sickness**: acetazolamide, dexamethasone

### BEHAVIORAL HEALTH

**Depression**: sertraline, escitalopram, fluoxetine, paroxetine, fluvoxamine, citalopram, venlafaxine, duloxetine, desvenlafaxine, bupropion, mirtazapine

**Anxiety disorders**: buspirone, sertraline, escitalopram, paroxetine, venlafaxine, duloxetine, hydroxyzine, propranolol

**ADHD**: atomoxetine, bupropion, clonidine, guanfacine

**Bipolar disorder**: lamotrigine, quetiapine, olanzapine, aripiprazole, risperidone, ziprasidone

**Panic disorder**: sertraline, paroxetine, escitalopram, venlafaxine

**PTSD**: sertraline, paroxetine, venlafaxine, prazosin

**Insomnia**: mirtazapine, hydroxyzine, diphenhydramine, melatonin

**Social anxiety disorder**: sertraline, paroxetine, venlafaxine, propranolol

**Adjustment disorder**: sertraline, escitalopram, buspirone

**Seasonal affective disorder**: bupropion, sertraline, fluoxetine

