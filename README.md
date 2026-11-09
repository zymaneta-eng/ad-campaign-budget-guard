# 🛡️ Ad Campaign Budget Guard (`ad-campaign-budget-guard`)

[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![OpenCV](https://img.shields.io/badge/Computer%20Vision-OpenCV%20Headless-red.svg)](https://opencv.org/)
[![Streamlit](https://img.shields.io/badge/Cockpit-Streamlit-FF4B4B.svg)](https://streamlit.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Code Style: Ruff](https://img.shields.io/badge/code%20style-ruff-000000.svg)](https://github.com/astral-sh/ruff)

An automated **performance marketing ROAS/CPA guardrail and pacing engine** featuring **root-cause creative fatigue diagnosis** (rising frequency + dropping CTR), **OpenCV video hook keyframe extraction** (1 fps, first 5 seconds), and **Human-in-the-Loop (HITL) approval workflows** for high-risk marketing actions.

---

## 🎯 Target Industries & Production Use Cases

- **Performance Marketing Agencies & Media Buyers**: Managing dozens of client ad sets across Meta Ads, TikTok Ads, and Google Ads who need deterministic stop-loss rules to prevent budget bleed without rogue autonomous API blunders.
- **D2C E-Commerce Growth Teams**: Continuously auditing video hook performance, scaling high-ROAS winner ad sets safely, and routing destructive budget changes through human review channels.
- **Enterprise Ad Spend Optimization**: Enforcing strict governance over daily pacing corridors, avoiding end-of-month budget spikes, and preventing runaway costs on broken landing pages.

---

## ✨ Key Features & Architectural Capabilities

- **Causality-Grounded Creative Fatigue Detection**:
  - Distinguishes between normal traffic variance and audience burnout by evaluating the correlation between **rising ad frequency (> 4.0)** and **dropping CTR**.
- **Multimodal Video Hook Keyframe Extractor (`video_hook_extractor.py`)**:
  - Leverages OpenCV (`cv2`) to extract **1 frame per second** for the **first 5 seconds** (the critical viewer retention window).
  - Emits clean Base64 JPEG data URIs formatted for Vision LLM auditing without GPU memory exhaustion.
- **Deterministic 4-Tier Guardrail Engine (`engine.py`)**:
  - `PAUSE_BLEEDING` (Stop-Loss): Immediately halts ad sets exceeding spend thresholds with zero conversions or unacceptable CPA.
  - `CREATIVE_FATIGUE_TRIM`: Decreases budget by 25% when audience saturation is detected.
  - `SCALE_WINNER`: Boosts daily budget (+20%) when ROAS and conversion metrics stay inside target corridors.
  - `HOLD_STEADY`: Preserves existing budget allocation.
- **Human-in-the-Loop (HITL) Safety Gate (`hitl.py`)**:
  - All high-risk destructive actions (`pause_ad_set`, `adjust_budget`) generate `DraftAction` proposals requiring explicit sign-off via Slack or dashboard cards.
  - Low-risk non-destructive tasks (e.g. `generate_ad_copy`) run autonomously.

---

## 🏗 System Architecture

```mermaid
flowchart TD
    subgraph InputSignals ["Ad Network & Creative Signals"]
        METRICS["Ad Network Metrics Stream\n(Meta / TikTok / Google Ads)"]
        VIDEO["Video Creative Files (.mp4)\n(1080x1920 Mobile Ads)"]
    end

    subgraph ProcessingPipeline ["Guardrail Engine & Multimodal Extraction"]
        GUARD["BudgetGuardEngine\n(Pacing & Corridor Evaluation)"]
        CV["OpenCV VideoHookExtractor\n(1 fps, Sec 0..4 Keyframes)"]
        VLLM["Multimodal Vision Auditor\n(Visual Hook Retention Score)"]
    end

    subgraph DecisionMatrix ["Deterministic Action Rules"]
        RULES{"Decision Matrix"}
        STOP["PAUSE_BLEEDING\n(CPA > Threshold)"]
        TRIM["CREATIVE_FATIGUE\n(Freq > 4.0, CTR Down)"]
        SCALE["SCALE_WINNER\n(High ROAS Corridor)"]
        STEADY["HOLD_STEADY\n(Healthy Pacing)"]
    end

    subgraph ExecutionLayer ["HITL Approval & Actuation"]
        HITL["HITL Approval Manager\n(DraftAction Verification)"]
        SLACK["Slack / Streamlit Cockpit\n([Approve] / [Reject])"]
        API["Ad Platform API Actuators\n(Meta Graph / Google Ads API)"]
        DB[("SQLite Audit Trail DB")]
    end

    METRICS --> GUARD
    VIDEO --> CV
    CV --> VLLM
    VLLM --> GUARD
    GUARD --> RULES

    RULES -->|Stop-Loss| STOP
    RULES -->|Burnout| TRIM
    RULES -->|Winner| SCALE
    RULES -->|Normal| STEADY

    STOP & TRIM & SCALE --> HITL
    HITL --> SLACK
    SLACK -->|Operator Approves| API
    GUARD & HITL --> DB
```

---

## 🔄 Guardrail & Approval Flow

```mermaid
sequenceDiagram
    autonumber
    participant Ads as Ad Network API
    participant Guard as BudgetGuardEngine
    participant HITL as Approval Manager
    participant Operator as Media Buyer (Slack/Cockpit)
    participant DB as SQLite Audit Trail

    Ads->>Guard: Ingest Hourly Spend, Impressions, CTR, CPA & ROAS
    Guard->>Guard: Evaluate CPA threshold ($45 max) & Frequency (4.8)
    Guard->>Guard: Trigger Action: CREATIVE_FATIGUE_TRIM (-25% budget)
    Guard->>HITL: Submit DraftAction(ad_set_id="meta_99", action="trim_25")
    HITL->>DB: Persist DraftAction with status=PENDING_APPROVAL
    HITL->>Operator: Send interactive approval card with context metrics
    
    alt Operator Approves
        Operator->>HITL: Click [Approve Action]
        HITL->>Ads: PATCH /adsets/meta_99 (Update budget to $150.00)
        HITL->>DB: Update status=EXECUTED with timestamp & user_id
    else Operator Rejects
        Operator->>HITL: Click [Reject / Override]
        HITL->>DB: Update status=REJECTED (Pacing remains unchanged)
    end
```

---

## 🚀 Quick Start & CLI Usage

### 1. Installation

```bash
cd ad-campaign-budget-guard
pip install -r requirements.txt
pip install -e .
```

### 2. Run CLI Benchmark & Exporters

```bash
# Execute evaluation demo across sample campaigns
python -m ad_campaign_budget_guard.cli --demo --out-csv data/decisions.csv --out-json data/summary.json
```

### 3. Launch Streamlit Operations Cockpit

```bash
streamlit run src/ad_campaign_budget_guard/dashboard.py
```

### 4. Run Automated Tests

```bash
pytest tests/ -v
ruff check .
```


---

## 🖼️ User Interface & Anomaly Alerting Preview

*Visual audit cockpit and instant Telegram emergency alert preview.*

```
+------------------------------------------------------------------------------------+
|  AD CAMPAIGN BUDGET GUARD -- AUDIT & EMERGENCY DISPATCH COCKPIT                    |
+------------------------------------------------------------------------------------+
|  Status: ACTIVE RUNNING  |  Monitored Accounts: 12  |  Burn Spike Guard: ON (3.0x) |
+------------------------------------------------------------------------------------+
|  Campaign ID        | Platform  | Spend / Hr  | Target CPA | Anomaly Status       |
|  cmp_retarget_fb_01 | Meta Ads  | $142.50/hr  | $14.20     | CRITICAL SPEND SPIKE |
|  cmp_brand_search   | Google    | $28.10/hr   | $8.50      | NORMAL (P95 Latency) |
+------------------------------------------------------------------------------------+
|  [ Emergency Killswitch: ENGAGED ]  [ Telegram Dispatch Sent: 2026-11-09 20:45 ]  |
+------------------------------------------------------------------------------------+
```

*(Visual UI screenshots and system workflow previews will be linked here from the `/screenshots` folder).*

---

## 💼 Commercial Licensing & Deployment

This production system is engineered and maintained by **Nataliia Zyma**.

The core engine is enterprise proprietary software available for **commercial licensing**, **custom enterprise integration**, or **turnkey business deployment**:
- **Direct Email:** [`zymaneta@gmail.com`](mailto:zymaneta@gmail.com)
- **GitHub:** [@zymaneta-eng](https://github.com/zymaneta-eng)
- **Turnkey Integration:** Full custom setup, integration with existing company infrastructure, ad accounts, and SLA monitoring within 48–72 hours.
- **SLA & Response:** Enterprise inquiries are reviewed and answered within 2–4 business hours.
