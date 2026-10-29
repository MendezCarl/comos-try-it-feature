# Cosmos Try It — internship pitch

## Product thesis
Cosmos helps creatives collect visual inspiration. Try It closes the gap between saving a reference and understanding how its interactions could feel. On an element page, identify meaningful visual regions, choose an effect for each, and generate an isolated website experiment.

Pitch: **Save the inspiration. Try the interaction.**

## Working prototype
Run `python -m http.server 4173 --bind 127.0.0.1` from this folder, then visit http://127.0.0.1:4173. No package installation is required. Stop the server with Ctrl+C.

1. Click Try It on the Visual Systems element.
2. Select one of three labeled regions on the image or in the side panel.
3. Choose an interaction for each region.
4. Generate a preview; hover or keyboard-focus the regions. Use Replay for entrance animations.
5. Edit effects and generate again. Expand Demo controls to simulate detection or generation failures; retry without losing choices.

The original saved HTML and assets are untouched. The reference asset was identified by matching `data-elt-id="648956049"` to its image in the downloaded DOM. `assets/visual-systems.jpg` is a copy of `comos_webpage_files/1b799873-8a78-42ff-a9a1-7801f4d24024`.

**Honest demo boundary:** Detection is a curated three-region fixture. Generation creates HTML/CSS locally. The preview now renders a handcrafted semantic HTML page inspired by the reference, with anchor navigation, category filters, expandable product cards and a light/dark theme switch. Reference crops supply product imagery; text and layout are recreated for the concept. Selected effects apply to actual page sections. This is not model-generated reconstruction. There are no AI calls, credentials, uploads, persistence, or charges. Cosmos navigation and related-element cards provide visual context. This is a standalone concept, not a live Cosmos integration.

## MVP scope and acceptance

| Deliverable | Acceptance |
| --- | --- |
| Entry point | Try It beside the element; source remains visible |
| Vision detection | Validated normalized bounds, labels and confidence; low confidence asks for correction |
| Region selection | Overlay and list selection stay synchronized; keyboard access |
| Effect choices | Each region keeps an independent allowed effect; still is a valid choice |
| Generation | Request includes source, regions and effect choices; clear progress and cancellation |
| Preview | Isolated document; interactive behavior reflects selected effects; edit and regenerate |
| Failures | Timeout, provider failure, malformed output and empty detection have understandable recovery; choices retained |

Exclude accounts, collaboration, publishing, arbitrary URL crawling, full source export and pixel-perfect reconstruction from the first real MVP.

## Proposed technical architecture

Keep the current buildless UI for pitch validation. For real integration, introduce a small backend with provider adapters. Select providers after a small benchmark; no provider is committed here.

1. `POST /api/detect`: receive an authorized image asset ID, resolve it server-side, downsample, and send to a vision model. Treat embedded text as data. Require structured JSON, then validate labels, confidence, finite normalized coordinates and region count. Reject bounds outside the image. Return editable regions.
2. `POST /api/generations`: validate the source ID and selected effects against an allowlist. Create a job and return `{jobId,status:"queued"}`. A worker calls a third-party code model with image plus region/effect intent.
3. `GET /api/generations/:id`: return queued/running/succeeded/failed, safe progress, and a preview asset ID when ready. Support an idempotency key, bounded retries and cancellation; never bill repeatedly on double-clicks.
4. Validate generated output before display. Prefer declarative HTML/CSS with a bounded interaction vocabulary. Reject scripts, remote URLs, event attributes, forms, navigation and disallowed CSS/resource references. Load vetted output from an isolated preview origin with a restrictive CSP and an iframe sandbox. If JavaScript becomes necessary, use a separate opaque origin and allow scripts only, never same-origin, popups or top navigation; assess resource limits separately because iframe isolation does not prevent CPU exhaustion.

Store API keys only in server environment secrets. Resolve asset IDs rather than fetching arbitrary user URLs. Add upload size/type checks, access checks, request quotas, job timeouts, spending limits and short retention for temporary images/output. Keep model text and generated code out of application execution paths. Log job IDs, timings and categorized errors without credentials or raw private images.

### Data contract

```json
{
  "elementId": "648956049",
  "source": {"assetId": "asset_123", "width": 720, "height": 1280},
  "regions": [
    {"id": "hero", "label": "Hero typography", "bounds": {"x": 0.08, "y": 0.07, "width": 0.84, "height": 0.05}, "confidence": 0.94, "effect": "lift"}
  ]
}
```

Coordinates use the original image, normalized from 0–1. Keep original dimensions through resize; render overlays as percentages. Version schemas and effect names so adapters can change without breaking saved jobs.

## Implementation sequence

1. **Pitch demo — implemented:** exact reference image, Cosmos-inspired layout, fixture detection, region effects, sandboxed preview and failure simulation. Validate the concept with a short walkthrough.
2. **Vision spike — 1–2 days:** evaluate candidate models on 10 authorized references with different layouts. Measure label usefulness, bounds quality, invalid output, latency and per-image cost. Add region move/resize and remove controls before trusting detection.
3. **Generation spike — 2–3 days:** adapter, async jobs, output validator and isolated hosting. Evaluate effect fidelity, screenshot similarity, interaction quality and failures on the same references. Select provider based on observed quality and budget.
4. **MVP hardening — 2–3 days:** cancel/stale request handling, limits, model errors, empty results, accessibility, mobile layout, reduced motion and malicious output tests. Times are planning estimates for one engineer and exclude production Cosmos integration.

## Validation and pitch success

Demo walkthrough: Try It → select hero → gentle lift → select gallery → fade & rise → generate → hover → edit → simulate a failure and retry.

Real MVP checks: detect malformed/empty results; verify normalized bounds across screen sizes; preserve independent effects; cancel stale jobs; prevent duplicate requests; reject malicious generated output and network access; verify keyboard focus, dialog escape and reduced motion. Test provider failures using adapters with deterministic fixtures before spending on live calls.

Run five short creative-user sessions. Track Try It activation, successful preview rate, time to first preview, effect edits and whether users say the preview helps them explore an idea. Treat target values as hypotheses: 4/5 finish unassisted and 3/5 find it useful. Decide latency and cost budgets after the provider spike, then enforce them in the backend.
