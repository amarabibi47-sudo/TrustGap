 
# TrustGap

**Detecting Authentication-Passing Business Email Compromise via Contact Baseline and Lookalike Domain Analysis**

An explainable email security prototype for detecting lookalike-domain fraud.

Final Year Project, BS Computer Science, Namal University, Mianwali (Session 2023-2027)

> **Status:** Proposal stage (October 2026). The documents are available in [`docs/`](docs/). Source code will be added as the project progresses.

---

## The Problem: The Trust Gap

SPF, DKIM and DMARC check that an email was sent from a server that is allowed to send for a domain. They do **not** check that the domain is the one you actually trust.

A fraudster can buy a domain that looks almost identical to a real supplier's (for example `supp1ier-pk.com` instead of `supplier-pk.com`), set it up correctly, and pass all three checks. The email looks safe to the system, but it is fake. We call this the **Trust Gap**.

## Our Solution

TrustGap asks the next question: *is this really the person and company we normally talk to?*

It reads saved emails, builds a short history for each contact, compares each new email with that history, and shows a **Low / Medium / High** risk level with the reason in plain words.

## How It Works

For every email, TrustGap runs six checks:

| # | Check | What it looks at |
|---|-------|------------------|
| 1 | Login checks | SPF, DKIM, DMARC result already written by the mail provider (used as one clue, not proof) |
| 2 | Known address? | Is the sender new compared with the contact's history? |
| 3 | Lookalike domain? | Edit distance and look-alike characters (for example `l` vs `1`) |
| 4 | Domain age | How recently the domain was registered |
| 5 | Request type | Payment, bank change, gift cards, passwords, urgent wording |
| 6 | Writing style | Compared only when enough past emails exist |

The signals are combined by a simple machine learning model (logistic regression, then one gradient-boosted model such as XGBoost or LightGBM). Each alert is explained using SHAP values, turned into a short sentence, and shown on a small dashboard.

**Example:** *"The domain is one letter different from your supplier's and was registered 6 days ago, and the email asks to change bank details."*

## Design Principles

- **Read-only.** TrustGap never changes, deletes, moves or sends email. It only shows alerts on its own dashboard.
- **Beside the mailbox, not in the middle.** Mail routing is not changed.
- **Store less.** Small summaries (domain, time, risk) are kept, not full emails.
- **Local processing.** No outside AI service is used.
- **Explainable.** Every alert comes with a reason.

## Project Scope

- **Core deliverable:** an offline engine that reads saved English business emails (`.eml` or mbox), builds contact histories, checks lookalike domains, domain age and behavior, gives a risk level with reasons, and is evaluated on a labelled dataset. A simple dashboard shows the results.
- **Stretch goal:** a read-only live Gmail pilot with volunteer mailboxes (needs approval from Namal IT and the university).
- **Out of scope:** automatic blocking or deleting of email, outgoing email protection, attachment virus scanning, deep link scanning, multi-company service, mobile app, Outlook, and languages other than English.

**Limitation:** TrustGap cannot catch a genuine supplier account that has been hacked, because the domain is then real.

## Deployment and Testing Plan

| Stage | Where | Data | Goal |
|-------|-------|------|------|
| 1. Offline lab | Our laptop | Public emails and synthetic attacks | Prove the method works |
| 2. Replay pilot | Namal computer | Volunteer mail exports (with consent) | Check alerts make sense on real mail |
| 3. Live shadow pilot (if approved) | Namal system | Read-only Gmail | Check live speed and alert volume |

## Datasets

- Enron email corpus (normal business emails)
- Public phishing collections (for example the Nazario corpus)
- Synthetic lookalike-domain attack emails (trust-building, payment change, login request)
- Hard normal emails (new genuine senders, suppliers that change domain, newsletters)
- Small Pakistan scenario set (hand-written, clearly labelled as synthetic)

Training and test data are split by contact or by time, never by random email. Results on real and synthetic data are reported separately.

## Evaluation

TrustGap is compared with three baselines: a domain-only rule, Sublime Security's free lookalike and sender-history rules, and a text-only classifier. The main measure is the **false-positive rate at 90% recall**, reported with confidence intervals. We also test the contribution of each layer (domain, behavior, writing style) separately.

## Tech Stack

| Area | Tools |
|------|-------|
| Engine | Python 3.11, FastAPI |
| Domain checks | RapidFuzz, tldextract, dnspython, RDAP/WHOIS |
| Test domains | dnstwist |
| Text features | scikit-learn, spaCy or NLTK |
| Risk model | scikit-learn (logistic regression), XGBoost or LightGBM |
| Explanation | SHAP |
| Storage | SQLite |
| Dashboard | FastAPI pages or Streamlit |
| Baseline | Sublime Security rules (Docker) |
| Development | Git/GitHub, pytest, Docker, VS Code |
| Stretch only | Gmail API (read-only OAuth 2.0) |

## Hardware Requirements

A normal laptop is enough. **No GPU is needed.**

- Development: 4+ core CPU, 8 GB RAM (16 GB recommended), 30-50 GB free storage
- Pilot server: 2 CPU cores, 4 GB RAM, 50-80 GB disk

## Repository Structure

```
TrustGap/
├── README.md
├── .gitignore
├── docs/
│   ├── TrustGap_FYP_Proposal_Updated.docx
│   └── final_fyp_ppt.pptx
└── src/            (coming soon)
```

## Timeline

October 2026 to March 2027: study and setup, contact history and domain checks, behavior and writing-style signals, model and explanations, evaluation and stress tests, final report and open release.

## Team

| Name | Reg. # |
|------|--------|
| Amara Bibi | NUM-BSCS-2023-14 |
| Sumaira Majeed | NUM-BSCS-2023-33 |

**Supervisor:** Mr. Ramzan Shahid Khan
**Co-supervisors:** Dr. Mudassar Raza, Dr. Muzammil Ahmad
**Industry Supervisor:** Mr. Awais Saeed, Amrood Labs

## Disclaimer

TrustGap is an academic research prototype. It is not a replacement for Gmail, Outlook or a commercial email security product. All attack testing uses made-up suppliers and test domains; no real company is impersonated.

## License

To be decided with the supervisor (for example MIT for code, CC BY 4.0 for the released dataset).
