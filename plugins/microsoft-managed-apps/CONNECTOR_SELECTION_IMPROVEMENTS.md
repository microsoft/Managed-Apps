# Connector Selection Improvements

## Overview

The Microsoft Managed Apps plugin now includes a **comprehensive, scenario-based connector decision framework** to help intelligently select the right connector(s) for any user scenario. This ensures consistent, high-quality connector recommendations across all `/add-*` skills and the Architect agent.

---

## What Was Added

### 1. **New Shared File: `connector-decision-guide.md`**

**Location:** `shared/connector-decision-guide.md`

A comprehensive guide that covers:

- **Connector Capability Matrix** — a table comparing all 10+ connectors across dimensions (search, create/update, delete, AI, MCP)
- **Decision Trees** — by scenario (search, CRUD, AI, hybrid apps)
- **When NOT to use each connector** — anti-patterns to avoid
- **Common App Patterns** — pre-built recommendations:
  - Report/Dashboard apps
  - Meeting Insights apps
  - Document Search & Management apps
  - Task Tracker with AI apps
  - Customer Management Systems
- **Connector Selection Checklist** — 5 key questions to ask when users describe their app
- **Implementation Examples** — step-by-step decision process for 3 real scenarios

**Use this guide when:**
- Recommending connectors to users
- Making connector selection decisions
- Training new developers/skills
- Designing hybrid apps with multiple connectors

---

### 2. **Updated: `/add-datasource` Skill**

**Changes:**
- Added reference to the Connector Decision Guide
- Enhanced the workflow to include an explicit "Apply Decision Guide" step
- Improved the initial questions to gather context for scenario matching
- Now applies structured decision logic instead of static rule-matching

**Impact:** The router now intelligently applies decision trees to match user scenarios to the right connector(s).

---

### 3. **Updated: `shared-instructions.md`**

**Changes:**
- Added new "Connector Decision Guide" section
- Linked the decision guide prominently
- Made it clear that all skills must reference the guide

**Impact:** All skills now know where to find structured connector selection logic.

---

### 4. **Enhanced: Microsoft Apps Architect Agent**

**Changes:**
- Expanded "Your Expertise" section to highlight connector decision-making
- Added explicit reference to the Connector Decision Guide
- Replaced the static connector table with a more detailed version that includes:
  - "Why" column (explanation of when to use each)
  - Note about hybrid apps and sequence
  - Link to the decision guide for detailed patterns
- Emphasized applying decision logic instead of just listing options

**Impact:** The Architect agent now provides more nuanced connector recommendations with explanations.

---

## How It Works

### **For Users Building Apps**

When users describe their app goals to `/add-datasource`:

1. The skill asks about their app's purpose, data, actions, and AI needs
2. The skill applies the **Decision Trees** from the guide to match scenarios
3. The skill recommends the right connector(s) with reasoning
4. The skill invokes the appropriate `/add-*` skills in sequence

**Result:** Users get intelligent, scenario-aware recommendations instead of a static list.

---

### **For Skills & Agents**

When building connector-related logic:

1. **Before recommending:** Check the decision trees and capability matrix
2. **When uncertain:** Apply the "Connector Selection Checklist" (5 questions)
3. **When multiple options exist:** Explain trade-offs using the "Why" column
4. **For complex scenarios:** Reference the "Common App Patterns"

---

## Key Improvements

| Before | After |
|--------|-------|
| "Here are 10 connectors, pick one" | "Based on your scenario (search M365 data), I recommend Work IQ. Here's why..." |
| Static recommendation table | Decision trees that match scenarios |
| No guidance on hybrid apps | Clear patterns for apps needing multiple connectors (with sequence) |
| No "when NOT to use" guidance | Anti-patterns documented to avoid misuse |
| Single decision point | Multi-step decision flow (is it search? → service type? → actions? → AI?) |
| No examples | 5 worked examples showing the decision process end-to-end |

---

## When to Update This Guide

Add or modify guidance when:

1. **New connector added** — document its capability, use cases, and where it fits in decision trees
2. **New app pattern emerges** — add to "Common App Patterns" section with connector sequence
3. **Connector capabilities change** — update the Capability Matrix and decision trees
4. **Users repeatedly ask about a scenario** — add that scenario to the guide to prevent rework
5. **Trade-offs change** — update the "Why" explanations if connector recommendations shift

---

## Example: How the Improved System Works

### **Scenario: "I want to build an app that finds all meetings about Q3 planning and shows action items"**

#### **Before:**
- Plugin shows list of connectors
- User picks one (or guesses)
- Might choose Office 365 (WRONG — not semantic search)

#### **After:**
1. User describes goal
2. `/add-datasource` applies decision tree:
   - "Is it search?" → YES
   - "Semantic M365 search?" → YES
   - "Need to create/update?" → NO
3. Plugin recommends: "Work IQ for semantic search + Copilot Studio for AI summaries"
4. Plugin explains why each is chosen
5. Plugin invokes `/add-workiq` and `/add-mcscopilot` in sequence
6. User gets exactly what they need

---

## Files Modified

- ✅ Created: `shared/connector-decision-guide.md` (10.8 KB, comprehensive reference)
- ✅ Updated: `skills/add-datasource/SKILL.md` (added Decision Guide reference + decision step)
- ✅ Updated: `shared/shared-instructions.md` (added Connector Decision Guide section)
- ✅ Updated: `agents/microsoft-apps-architect.md` (enhanced with decision guidance)

---

## Next Steps

To test the improvements:

1. **Use `/add-datasource`** with a complex scenario (e.g., "Meeting Insights" or "Customer Management System")
2. **Verify the skill applies the decision trees** (asks the right questions, matches scenarios)
3. **Check the recommendations** against the Common App Patterns section
4. **If guidance is missing**, add it to the guide for future users

The system is now **self-documenting** — the decision framework is stored in one place and referenced by all skills, making it easy to update and maintain.
