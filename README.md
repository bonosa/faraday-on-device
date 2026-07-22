# Faraday — private, on-device document analysis

**Read a document without sending it anywhere.**

Faraday runs a real language model *entirely inside your browser*. Drop in a PDF or a photo of a form, and it summarizes the document, extracts the key fields, and flags what matters — with **no upload, no server, and no account**. Turn off your Wi-Fi after the model loads and it keeps working.

Built for the Orion Global Hackathon 2026.

---

## The problem

The documents people most need help understanding are the ones they least want to upload: lab results, medical intake forms, insurance statements, financial records. Every mainstream "AI document assistant" solves this by sending your file to a cloud model (GPT-4o, Claude, Gemini) — which means your most sensitive data leaves your device and lands on someone else's servers.

For a whole class of users — patients, clinics in low-connectivity settings, anyone handling regulated or private data — that tradeoff is a dealbreaker.

## The solution

Faraday moves the model to the data instead of the data to the model. It loads a small language model (Qwen2.5-0.5B) directly into the browser using [transformers.js](https://github.com/huggingface/transformers.js) and runs inference on your own hardware via WebGPU (falling back to CPU where WebGPU isn't available). Your document is read, summarized, and analyzed locally. Nothing is transmitted.

To make that claim verifiable rather than a promise, Faraday **intercepts its own network layer** and shows a live "outbound signals" counter. After the model finishes downloading, analyzing a document adds **zero** network requests — you can watch the counter stay at zero, or open your browser's Network tab and see for yourself.

## What it does

- **Summarizes** any document in plain language for a non-expert.
- **Extracts key fields** (dates, totals, IDs, amounts, contacts) deterministically from the text — reliable and instant, independent of the model.
- **Reads both PDFs and images.** Text PDFs are parsed with pdf.js; photos and scans go through on-device OCR (Tesseract.js). No image ever leaves the browser.
- **Works offline.** Once the model is cached, disconnect the network entirely and it still runs.
- **Proves its privacy claim** with a live outbound-request monitor and a "0 bytes sent" indicator.

## The tradeoff (and why it's the point)

Faraday is slower than a cloud model and its summaries are simpler — because it runs a small model on ordinary consumer hardware instead of a datacenter GPU. That is the deliberate cost of the guarantee that your data never leaves your device. A cloud tool can be fast *or* private; on this hardware, Faraday chooses private. The speed is a visible reminder of where the work is happening: on your machine, not someone else's.

## How to run

Faraday is a single static HTML file. It must be served over `http://localhost` (or any `https://` host) — opening the file directly with `file://` will not work, because browsers block the web workers the model needs under the `file://` origin.

```bash
# clone
git clone https://github.com/bonosa/faraday-on-device.git
cd faraday-on-device

# serve locally (any static server works)
python -m http.server 8000
```

Then open **http://localhost:8000/faraday.html** in Chrome or Edge (version 121+, for WebGPU).
Also hosted on https://faraday-deploy-2.vercel.app/

On first load, the model (~786 MB) downloads once and is cached by the browser. After that it loads instantly and works offline.

### Try the privacy demo

1. Load the page and wait for **"Model ready."**
2. Turn off your Wi-Fi.
3. Drop in one of the sample documents (`demo-lab-report.pdf` or `demo-intake-form.png`).
4. Click **Analyze**. Watch the "Outbound signals" counter — it stays at zero. The analysis happens entirely offline.

## Tech stack

| Component | Role |
|-----------|------|
| [transformers.js](https://github.com/huggingface/transformers.js) | Runs Qwen2.5-0.5B in-browser (WebGPU, CPU fallback) |
| [pdf.js](https://mozilla.github.io/pdf.js/) | Extracts text from PDFs, client-side |
| [Tesseract.js](https://tesseract.projectnaptha.com/) | On-device OCR for photos and scans |
| Vanilla HTML/CSS/JS | No framework, no backend, no build step |

## Sample documents

Synthetic, non-real sample files are included for testing and demonstration:

- `demo-lab-report.pdf` — a blood-work lab report with flagged values (fast PDF path)
- `demo-intake-form.png` — a patient intake form image (OCR path)

Both contain entirely fabricated data and must not be used for any real purpose.

## Roadmap

- **On-device multimodal understanding** — interpret charts, X-rays, and diagrams with a small vision model, not just OCR text.
- **Dedicated edge hardware** — deploy the same pipeline to an NVIDIA Jetson Orin Nano for faster, appliance-style on-device inference.
- **Larger local models** where the user's hardware allows, for richer summaries without giving up the privacy guarantee.

## License

MIT.

## A note on scope

Faraday is a demonstration of private, on-device document analysis. It is not a medical, legal, or financial tool, and its output should not be relied on for decisions in those domains.
