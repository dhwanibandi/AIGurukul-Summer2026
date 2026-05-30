## 1. Problem Statement 1 (PS1) — Engineering Drawing Verification

### 1.1 Motivation

Rather than working on overused tasks like sentiment analysis or grammar checking, Anupam proposed an **engineering-focused problem** that is:
- Relatively **untouched** in literature
- Directly relevant to students' engineering background
- Practical and applicable

---

### 1.2 Problem Definition

**Goal:** Build a two-agent agentic system that:
1. **Agent 1 (A1):** Generates Python code (using a DXF library) to create a 2D engineering drawing (`.dxf`/`.dwg` file) of a house plot layout.
2. **Agent 2 (A2):** Verifies the generated code against a set of constraints (both logical and semantic), using a solver (Z3), and provides feedback to A1.

```
User Input (plot dimensions + constraints)
            │
            ▼
     ┌──────────────┐
     │   Agent 1    │  ──► Generates Python code (DXF file of plot layout)
     │ (Generator)  │
     └──────────────┘
            │
            ▼  (code output)
     ┌──────────────┐
     │   Agent 2    │  ──► Verifies constraints are met
     │  (Verifier)  │       Uses Z3 Solver for satisfiability
     └──────────────┘
            │
     ┌──────┴──────┐
     │             │
  PASS           FAIL
     │             │
     ▼             ▼
 Final DXF    Feedback → back to A1 (loop)
```

---

### 1.3 Technical Stack

| Component | Tool/Technology |
|---|---|
| 2D Drawing File Format | `.dxf` (DXF Python library — `ezdxf`) |
| Code Generator | LLM (via API — GPT, Claude, Gemini, etc.) |
| Constraint Solver | Z3 Solver (Microsoft Research) |
| Constraint Source | Seattle Building Code (SBC) — well-documented, publicly available |

> **Note:** `ezdxf` is a Python package that can programmatically create DXF/DWG engineering drawing files.

---

### 1.4 Concrete Problem Specification (Plot Layout)

Anupam sketched a specific problem on screen (participants were asked to take a screenshot):

**Plot Dimensions (Land Parcel):**

```
         North
           │
  ─────────────────────
  │                   │ 75 ft
  │    [Tree]         │
  │  (2.5ft x 2ft)    │
  │   at center       │ 80 ft
  │                   │
  ─────────────────────
         South

  Width (East-West): 50 ft
  Other measurements: 10 ft, 5 ft, 40 ft, 20 ft offsets
```

**Key Constraints:**
- The **tree cannot be cut** (Seattle regulation: ~$50,000 fine + 6 months permit).
- **Offsets** must follow **Seattle Building Code (SBC)**.
- Maximize **area coverage** (land utilization).
- The layout must include **only the outer periphery** of the house.
- A **single entry point** (main door + parking) must be on the same side — per SBC fire safety norms.

**Agent 1's Task:**
- Generate Python (DXF) code to draw the outer periphery of a house on this plot, following all constraints.

**Agent 2's Task:**
- Verify the generated code against SBC constraints and geometric rules.
- If constraints are violated, output specific feedback: *"Out of 3 constraints, you missed X — fix and regenerate."*

---

### 4.5 Reference Resources

- **Seattle Building Code (SBC):** Publicly documented, well-structured. Anupam provided a link in chat. *(Prasad Gunjikar to compile all links in a shared Google Doc.)*
- **DXF Python library:** `ezdxf` (or similar package).
- **Z3 Solver:** Available via pip — `pip install z3-solver`.

---

### 4.6 Simplified First Attempt (No Code Required)

For participants who do not yet have LLM API keys, Anupam suggested a **manual iteration approach**:

1. Use **ChatGPT (free)** as **Agent 1** — give it the plot layout + SBC constraints and ask it to generate the layout description/DXF code.
2. Use **Claude or Gemini (free)** as **Agent 2** — paste A1's output and ask it to verify against SBC constraints.
3. Feed A2's feedback back to A1 manually.
4. Repeat for 2–3 iterations.

> **Goal of this step:** Understand the agentic loop conceptually before automating it with code.

---

### 4.7 A1 & A2 LLM Combination Assignments

Each participant must choose a **unique combination** of LLM for A1 and A2. Options suggested:

- ChatGPT (GPT-4o / GPT-4o mini)
- Claude
- Gemini
- DeepSeek
- Moonshot
- Sarvam (Indian LLM)

> Combinations were to be recorded in the **shared Google Sheet (Problem Statement tracker)** — each person picks a unique A1/A2 pairing to allow comparison of results across models.

---

## 5. Deadlines & Next Steps

| Task | Responsible | Deadline |
|---|---|---|
| Complete pre-session lectures (all) | All participants (esp. Saksham) | ASAP |
| Compile all resource links in Google Doc | Prasad Gunjikar | End of meeting / WhatsApp group |
| Attempt PS1 (manual or coded) | All participants | **Sunday, June 1 – 3:00 PM** |
| Record findings in shared Excel sheet | All participants | Before next meeting |
| Next meeting | All participants | **Sunday, June 1, 3:00 PM** |


More details here: https://docs.google.com/document/d/1vI-7MLO7oIiXxsCpduJVfJRKoXYvYoVSM80deqHL4Kw/edit?tab=t.0


*Notes compiled from meeting transcript — AIGurukul PS1, Meet 2, May 28, 2026.*
