# Contact Form and Provider Integration Spec

> **Version:** 1.0
> **Last Updated:** 2026-10-11
> **Status:** Draft for implementation; Web3Forms account created, but endpoint configuration and form credentials are not yet recorded here.

## Goal

Add a usable contact form to the existing static Astro site without introducing a server runtime, while keeping the page visually consistent with the WeCollab8 design and preserving direct email as a fallback.

## Existing Context

- The site is Astro with static output, Tailwind CSS v4, and GitHub Pages hosting at `https://www.wecollab8.co.za`.
- The contact route is `/contact/`. It has introductory copy and a `mailto:hello@wecollab8.co.za` link.
- `src/styles/global.css` defines the site colors and fonts: paper `#f7f8f4`, ink `#1e2c26`, forest `#183d30`, rust `#b96243`, line `#d8dfd8`, and muted `#58665e`.
- There is no server runtime or configured form endpoint in the Astro build or GitHub Pages deployment. The site owner has created a Web3Forms account; endpoint setup remains an implementation prerequisite.
- The privacy page currently describes the site’s consent preferences and Google Fonts request. It must be updated before enabling a form processor.
- `tests/static-output.test.mjs` verifies generated production HTML and should cover the new confirmation route and form markup.

## Provider Decision

**Recommend Web3Forms**, using its HTML submission endpoint and no provider-hosted browser script.

It has an official Astro guide, supports a native HTML `POST` suitable for static hosting, and documents same-domain custom redirects on its free plan. The current pricing page lists 250 monthly submissions on the free tier. The form UI can therefore stay on the existing contact page, with a branded static confirmation route as the non-JavaScript fallback.

This recommendation is conditional on owner review of Web3Forms' current Data Processing Agreement and retention terms before production launch. Its published DPA says submissions may be retained for up to three years, describes processing in India and other jurisdictions, and lists spam subprocessors that may receive submitter IP and email addresses. Update the site privacy notice to match the provider configuration actually enabled; do not imply that form submissions remain only on WeCollab8 systems.

### Alternatives Considered

| Provider | Fit | Reason not selected |
| --- | --- | --- |
| Basin | Has an Astro-specific guide, a native HTML endpoint, basic spam filtering, and 50 submissions/month with 30-day dashboard retention on its current free tier. | Custom redirects are paid. Its hosted BasinJS script can render in-page states, but would add a provider script request to the contact page. |
| Formspree | Supports native HTML and AJAX submissions, validation responses, and honeypot filtering. | Its current free tier is described as for testing/development with 50 submissions/month; custom thank-you pages are paid. Its published privacy policy is dated 2022, so current privacy terms would need additional review. |

Provider pricing and terms can change. Recheck the linked first-party sources when provisioning the endpoint.

## Scope

### In Scope

- Add a contact form to `/contact/` without changing that public route.
- Add a static, branded success route at `/contact/thanks/` for native form submissions when JavaScript is unavailable.
- Deliver submissions to the configured WeCollab8 contact inbox through Web3Forms.
- Use Web3Forms-managed hCaptcha for spam prevention, include its response in AJAX submissions, and load its client script only on the contact route.
- Add accessible validation, sending, success, and failure states.
- Preserve the direct `mailto:hello@wecollab8.co.za` option.
- Update the privacy notice and static-output tests.

### Out of Scope

- Adding a custom server, Astro adapter, database, CRM, mailing list, or automation integration.
- Collecting phone numbers, file attachments, sensitive data, or marketing consent.
- Adding analytics or marketing tags, or loading provider scripts site-wide. The contact route's Web3Forms/hCaptcha script is the explicit spam-protection exception.
- Automatically replying to submitters or promising a response time unless the owner supplies approved copy.

## Form Experience

Keep the existing contact introduction and two-column composition on desktop. Replace the email-only content area with the form, and retain a concise direct-email fallback immediately below it. Stack the content naturally on narrow screens.

The form must look native to the page rather than like an embedded widget or standalone card:

- Use the existing CSS variables and Tailwind theme tokens from `src/styles/global.css`.
- Keep the form unframed, using spacing and subtle line borders rather than a shadowed card or colored panel.
- Use paper backgrounds, ink labels, muted helper text, line-colored input borders, forest focus indicators and primary action, and restrained rust accents consistent with existing links and eyebrows.
- Match existing typography. Keep control dimensions stable and make fields full width on mobile.
- Provide visible labels; placeholders must not replace labels. Use clear required indicators and keyboard-visible focus states.

### Questions and Fields

Use the provider's basic name/email/message pattern as the transport starting point, but replace its generic sample copy and styling with the following WeCollab8-specific, accessible form. Keep the form useful for both a well-defined project and an early conversation; do not make every qualification question mandatory.

| Order | Visible label | Field name | Required | Type and guidance |
| --- | --- | --- | --- | --- |
| 1 | Your name | `name` | Yes | Text input; `autocomplete="name"`. |
| 2 | Work email | `email` | Yes | Email input; `autocomplete="email"`. Explain that this is where WeCollab8 will reply. |
| 3 | Organisation | `company` | No | Text input; `autocomplete="organization"`. Label it optional. |
| 4 | What would you like help with? | `area_of_interest` | Yes | Single-choice select. Options: Cloud and platform engineering; Integration and APIs; Enterprise automation; Security; AI consulting; Data engineering; A combination of these; Not sure yet / general enquiry. The last two are valid choices, not errors. |
| 5 | What are you working through? | `message` | Yes | Textarea, maximum 5,000 characters. Prompt for the current situation or opportunity, the main challenge, relevant systems or data at a high level, and what a useful outcome would look like. A short initial description is enough. |
| 6 | Where are you in the process? | `project_stage` | No | Select: Exploring options; Defining requirements; Ready to plan or start; Project already underway; Improving or operating an existing solution; Not sure. |
| 7 | When are you hoping to begin? | `timeframe` | No | Select: As soon as practical; Within 1-3 months; In 3-6 months; More than 6 months away; Just exploring / no fixed date. Add helper text that this is for context and is not a commitment. |
| 8 | Anything else we should know? | `additional_context` | No | Short textarea, maximum 2,000 characters. Invite high-level constraints such as cloud/on-premises environment, integration boundaries, data readiness, security expectations, or delivery/support needs. |

Add contextual guidance below the required project-summary question without requiring visitors to answer every prompt:

- **Cloud and platform:** current environment, migration or reliability goal, and operational constraint.
- **Integration and APIs:** systems that need to work together, the information or process that crosses them, and the main integration difficulty.
- **Enterprise automation:** repetitive or hand-off-heavy process, involved applications, and desired reduction in operational friction.
- **Security:** the high-level service or API concern and environment; never request credentials, exploit details, or confidential architecture.
- **AI consulting:** the business task or user need, high-level data readiness, and how the solution should fit existing workflows. Do not ask visitors to paste proprietary data or prompts.
- **Data engineering:** source types, quality/reliability issue, and intended analytics or AI use; do not ask for sample records or production data.

Keep this guidance as brief examples, not a long questionnaire or per-service wizard. Do not add conditional questions, scoring, budget qualification, file upload, or phone capture in this first version. The selected area plus the two narrative fields should give the team enough context to route and respond, while optional stage/timeframe fields help prioritize without blocking an enquiry.

Use provider field names that are stable and human-readable in notification emails. Confirm the provider's current handling of select values and optional empty fields when the endpoint is configured. Do not use a visitor-controlled email or hidden input to choose the notification recipient.

Include a short note asking visitors not to submit passwords, credentials, financial details, health information, personal identity numbers, production records, or other sensitive/confidential information. Place a link to the privacy notice near the submit button. Do not add a marketing opt-in; an enquiry is not newsletter consent. Do not represent a privacy-notice link or checkbox as legal consent unless the owner obtains jurisdiction-appropriate legal advice.

## Submission and States

Use progressive enhancement:

1. The rendered form is a standard HTML form with the Web3Forms endpoint in `action`, `method="POST"`, correctly named fields, and native browser validation. Include the provider access key and a full absolute `redirect` URL to `https://www.wecollab8.co.za/contact/thanks/` for the no-JavaScript path. Web3Forms documents that this redirect field is for its standard HTML form flow and must be an absolute URL.
2. A small first-party client script may enhance submission with `fetch` to the documented JSON endpoint. It must only send a request after an explicit submit, must not load any third-party script, and must not send the native redirect field in the AJAX payload. Follow the current Web3Forms Astro/API documentation for the request format and response parsing.
3. During submission, expose a polite status, prevent duplicate submission, and disable only the submit action. On success, replace or clearly mark the form as complete without leaving the page. On failure, show an accessible error, preserve the entered values, re-enable submission, and keep the direct email option visible.
4. Use `role="status"`/`aria-live="polite"` for progress and success, and an appropriate alert or field-level errors for failures. Do not inject provider response text as HTML.
5. The `/contact/thanks/` route must use the site layout and offer a return link to `/contact/`. Mark it `noindex` and exclude it from the sitemap and Pagefind, since it is a transaction state rather than a discovery page.

If provider CORS or current response behavior does not support the proposed AJAX enhancement reliably, keep the native form as the required baseline and use the branded redirect; do not add a remote script just to avoid that limitation.

## Provider Configuration and Privacy

- Use the owner's existing Web3Forms account to create a dedicated endpoint and configure its notification recipient as `hello@wecollab8.co.za` in the provider dashboard. Do not create a second account or accept a recipient address from a visitor-controlled field.
- Read the endpoint access key from a build-time setting such as `PUBLIC_WEB3FORMS_ACCESS_KEY`. In a static site, any `PUBLIC_` value embedded in the output is visible to visitors; this key identifies the form and is **not a secret**. Never describe it as secret or use it as an authorization boundary. Do not put the real value in this spec or tests.
- Configure the key through a GitHub repository variable or equivalent build setting. Ensure production deployment is not enabled with an empty key. Pull-request builds without the production variable must still build and test; they may render only the email fallback or use a dummy key for markup tests, but must not send live submissions.
- hCaptcha is enabled for the existing Web3Forms endpoint. Render its managed widget with `data-captcha="true"`, load the Web3Forms client script only on the contact route, include the `h-captcha-response` token in AJAX requests, and prevent submission when the token is empty. Monitor the endpoint. Do not rely on Web3Forms' deprecated honeypot as the sole spam control.
- Do not add a consent toggle for the form itself: the provider receives personal data only when the visitor submits, and the form must not load optional analytics or marketing scripts. The privacy notice must explain the submission processor and data flow.
- Before production launch, the owner must review and accept the provider's current DPA, subprocessors, international processing, retention, and account settings. If those terms are not acceptable, pause launch and select another provider rather than silently changing the processor.
- Update `src/pages/privacy.astro` to identify Web3Forms as the processor for voluntarily submitted enquiries, enumerate the data collected, explain that submission and spam-prevention metadata are sent to the provider and its disclosed subprocessors, and link to the provider privacy notice/DPA. State only retention and transfer details verified against the configured account and current terms.

## Proposed Files and Responsibilities

- `src/pages/contact/index.astro`: retain the route and introduction; render the accessible, styled, website-specific form questions and email fallback.
- `src/pages/contact/thanks.astro`: add the static native-submit confirmation route.
- `src/pages/privacy.astro`: explain the new form processor and submitted data.
- `src/styles/global.css` or a focused contact stylesheet: style controls using existing design tokens without introducing a separate visual system.
- `tests/static-output.test.mjs`: assert form fields/action/fallback, confirmation route, privacy disclosure, `noindex`/Pagefind exclusion, and sitemap exclusion.
- `.github/workflows/deploy.yml`: pass the public build-time endpoint key for production and prevent a production deployment with missing configuration, while keeping PR builds safe without sending real submissions.

Do not add a dependency unless the chosen request handling demonstrably needs one. Prefer a small local script over a remote widget or third-party runtime bundle.

## Acceptance Criteria

### User Experience and Accessibility

- `/contact/` retains its current public URL, has a clear form and `mailto:hello@wecollab8.co.za` fallback, and fits the existing visual language on desktop and mobile.
- Every field has an associated visible label, native required/type constraints, usable keyboard focus, and accessible status/error messages. The service choices and prompts reflect all six service areas and allow a general/unsure enquiry.
- Only name, email, area of interest, and the project summary are required. Stage, timeframe, organisation, and additional context remain optional; no budget, phone, file, or marketing fields are present.
- The form is not submitted merely by loading or viewing the page. The Web3Forms/hCaptcha client script is requested only on the contact route; no analytics or marketing script is requested.
- A failed AJAX request preserves entered values and offers retry and direct email. A successful request gives an in-place confirmation; native submission redirects to the same-domain `/contact/thanks/` page.
- No files, phone numbers, sensitive personal data, or marketing subscription are collected.

### Integration, Privacy, and Static Output

- The site remains a static Astro export compatible with GitHub Pages; no server route or adapter is introduced.
- The live endpoint key is supplied through build configuration, is never committed as a secret, and production deployment cannot silently publish an unusable form.
- The privacy notice accurately identifies the configured form processor and the personal data/subprocessor/retention details verified by the owner.
- `dist/contact/thanks/index.html` is generated and complete, but the route is excluded from Pagefind and the sitemap.
- Static tests use no real provider credentials and make no live submission. Browser tests stub the provider endpoint to verify pending, success, failure, retry, and no-JavaScript form markup.
- Before launch, submit one clearly identified test enquiry from the deployed site, verify delivery to `hello@wecollab8.co.za`, verify the same-domain success flow, and remove the test submission according to the provider's controls.
- `npm run build` and `npm test` pass, and the deployed contact flow is manually verified at desktop and mobile widths.

## Suggested Implementation Sequence

1. Owner reviews the current Web3Forms DPA, privacy terms, subprocessor list, retention, and international transfer details. Use the existing account to create the dedicated endpoint, set the notification recipient, and confirm supported field configuration.
2. Add failing static-output tests for the specified question labels/options, required versus optional fields, direct-email fallback, confirmation route, privacy disclosure, and sitemap/Pagefind exclusions.
3. Implement the styled native form and confirmation route, then add the first-party AJAX enhancement only if verified against the current provider API.
4. Add build-time access-key configuration and a production deployment configuration check without exposing a live key in tests.
5. Update the privacy notice, run build/tests, and verify browser validation, responsive layout, stubbed success/failure paths, and no-JavaScript markup.
6. Perform a real test submission only after the endpoint, legal/privacy review, and deployment configuration are approved.

## First-Party References

Provider capabilities and plan limits below were checked on 2026-10-11. Recheck them before implementation because provider plans and terms may change.

- [Web3Forms Astro guide](https://docs.web3forms.com/how-to-guides/static-site-generators/astro.md)
- [Web3Forms basic HTML form](https://docs.web3forms.com/getting-started/examples/basic-html-contact-form.md)
- [Web3Forms custom redirect](https://docs.web3forms.com/getting-started/customizations/redirection.md)
- [Web3Forms spam protection](https://docs.web3forms.com/getting-started/customizations/spam-protection/spam-protection.md)
- [Web3Forms pricing](https://web3forms.com/pricing)
- [Web3Forms privacy policy](https://web3forms.com/privacy)
- [Web3Forms Data Processing Agreement](https://web3forms.com/dpa)
- [Basin Astro guide](https://usebasin.com/guides/static/astro)
- [Basin plan comparison](https://docs.usebasin.com/plan-comparison/)
- [Basin redirects](https://docs.usebasin.com/advanced-features/redirects/)
- [Formspree HTML form guide](https://help.formspree.io/articles/building-your-form/building-an-html-form.md)
- [Formspree AJAX guide](https://help.formspree.io/articles/building-your-form/submit-forms-with-javascript-ajax.md)
- [Formspree plans](https://formspree.io/plans)
- [Formspree privacy policy](https://formspree.io/legal/privacy-policy)