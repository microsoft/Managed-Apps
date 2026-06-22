# Connector Decision Framework — Complete Documentation Index

This directory now includes a comprehensive **Connector Decision Framework** that enables intelligent, scenario-based connector selection across the plugin.

---

## 🚀 Quick Start

### **For Users (Building Apps)**
1. **Just use `/add-datasource`** — it now applies the framework automatically
2. Describe your app goal (not "which connector?")
3. The plugin will recommend the right connectors and explain why

### **For Developers (Recommending Connectors)**
1. Read: **`CONNECTOR_QUICK_REFERENCE.md`** (5 min)
2. Use the 5-question decision process
3. Apply the decision trees from `shared/connector-decision-guide.md`

### **For Plugin Maintainers (Extending the Framework)**
1. Read: **`MAINTAINER_GUIDE_CONNECTOR_FRAMEWORK.md`** (10 min)
2. When adding connectors: update `shared/connector-decision-guide.md`
3. Reference the maintenance checklist

---

## 📚 Documentation Map

### **Core Framework (The Source of Truth)**
- **`shared/connector-decision-guide.md`** ⭐ **START HERE**
  - All connector selection logic lives here
  - Used by `/add-datasource`, Architect agent, and all skills
  - Update this ONE file when recommendations change

### **User-Focused Guides**
- **`CONNECTOR_QUICK_REFERENCE.md`** (Developer Cheat Sheet)
  - 5-question decision process
  - One-line recommendations
  - Common app patterns
  - Visual decision tree
  - Best for quick reference while coding

- **`FRAMEWORK_SUMMARY.md`** (Overview)
  - What was built and why
  - How the framework works in practice
  - File locations and success metrics
  - Good for understanding the big picture

### **Implementation Guides**
- **`CONNECTOR_SELECTION_IMPROVEMENTS.md`** (What Changed)
  - Before/after comparison
  - Key improvements
  - How the updated system works
  - Files modified

- **`MAINTAINER_GUIDE_CONNECTOR_FRAMEWORK.md`** (How to Maintain)
  - Internal architecture
  - How to add new connectors
  - How to modify recommendations
  - Testing and maintenance checklist
  - FAQ for maintainers

---

## 🎯 Core Concept: The Decision Framework

### **The 5-Question Process**

```
1. Is it primarily SEARCH?           → YES → Work IQ
   NO ↓

2. What M365 SERVICE?               → Calendar, Messages, Files, etc.
   Pick service ↓

3. Need to CREATE/UPDATE/DELETE?    → YES → Use that connector
   NO ↓

4. Need AI features?                → YES → Copilot Studio
   NO ↓

5. Multiple responsibilities?       → YES → Invoke multiple connectors
   NO ↓

DONE → Invoke connectors in sequence
```

### **4 Decision Trees**
1. **Search Scenarios** — Semantic/conversational queries → Work IQ
2. **Service Scenarios** — Specific M365 service → Service-specific connector
3. **AI Scenarios** — Generation/summarization → Copilot Studio
4. **Hybrid Scenarios** — Multiple responsibilities → Multiple connectors in sequence

### **10 Connectors**
| Connector | Best For |
|-----------|----------|
| Work IQ | M365 semantic search |
| Office 365 Outlook | Calendar, email, inbox |
| Teams | Messages, channels |
| SharePoint | Lists, documents |
| OneDrive | Files, versioning |
| Excel | Spreadsheets |
| Azure DevOps | Work items, builds |
| Dataverse | Custom business data |
| Copilot Studio | AI agents, summaries |
| SQL Procedures | Database stored procedures |

---

## 📁 File Structure

```
E:\Repo\playground\plugins\microsoft-managed-apps\
│
├── shared/
│   ├── connector-decision-guide.md ⭐ Core framework (10.8 KB)
│   ├── shared-instructions.md (updated with framework reference)
│   └── [other shared files]
│
├── skills/
│   ├── add-datasource/
│   │   └── SKILL.md (updated to use framework)
│   └── [other skills]
│
├── agents/
│   └── microsoft-apps-architect.md (updated with framework)
│
├── README_CONNECTOR_FRAMEWORK.md ⬅ YOU ARE HERE
├── FRAMEWORK_SUMMARY.md (Overview)
├── CONNECTOR_QUICK_REFERENCE.md (Cheat sheet)
├── CONNECTOR_SELECTION_IMPROVEMENTS.md (What changed)
├── MAINTAINER_GUIDE_CONNECTOR_FRAMEWORK.md (How to maintain)
└── [other plugin files]
```

---

## 🔄 How the Framework Flows Through the Plugin

### **Flow 1: User Uses `/add-datasource`**
```
User: "I want to search meeting transcripts and generate summaries"
    ↓
/add-datasource asks clarifying questions
    ↓
/add-datasource reads connector-decision-guide.md
    ↓
Framework matches: Search + AI scenario
    ↓
Recommendation: Work IQ + Copilot Studio
    ↓
/add-datasource invokes /add-workiq, then /add-mcscopilot
    ↓
User gets both connectors
```

### **Flow 2: Architect Agent Recommends Connectors**
```
App design needs connectors
    ↓
Architect reads connector-decision-guide.md
    ↓
Architect applies decision trees
    ↓
Architect recommends with reasoning
    ↓
Architect explains trade-offs
    ↓
Developer has confident, informed choice
```

### **Flow 3: Developer Building Custom Logic**
```
Developer needs to recommend a connector
    ↓
Developer reads CONNECTOR_QUICK_REFERENCE.md
    ↓
Developer applies 5-question process
    ↓
Developer uses decision trees from connector-decision-guide.md
    ↓
Developer recommends right connector with reasoning
```

---

## ✅ What This Enables

### **For Users**
- ✅ Don't have to choose between 10 connectors
- ✅ Just describe your app goal
- ✅ Get guided to the right choice automatically
- ✅ See reasoning for each recommendation
- ✅ Multiple connectors work together automatically

### **For Developers**
- ✅ Consistent framework across all skills
- ✅ Decision logic in one place (easy to update)
- ✅ Clear guidance when recommending connectors
- ✅ Examples of common app patterns

### **For Plugin Maintainers**
- ✅ All connector logic centralized
- ✅ Easy to add new connectors (update one file)
- ✅ Easy to modify recommendations (update one file)
- ✅ Maintenance checklist provided
- ✅ No scattered guidance across multiple files

---

## 🎓 Learning Path

### **5-Minute Overview**
1. Read this file (you're reading it now ✓)
2. Skim `CONNECTOR_QUICK_REFERENCE.md`

### **30-Minute Deep Dive**
1. Read `FRAMEWORK_SUMMARY.md`
2. Scan `shared/connector-decision-guide.md` (sections 1-3)
3. Understand the capability matrix and decision trees

### **Full Mastery**
1. Read `shared/connector-decision-guide.md` (entire file)
2. Study the worked examples (section "Examples: Applying the Guide")
3. Review common app patterns
4. Read `MAINTAINER_GUIDE_CONNECTOR_FRAMEWORK.md` if maintaining

---

## 🔧 Common Tasks

### **"I want to recommend a connector"**
→ Use `CONNECTOR_QUICK_REFERENCE.md` (5 min)

### **"I'm building a Meeting Insights app"**
→ Look up pattern in `shared/connector-decision-guide.md` section "Common App Patterns"

### **"I need to add a new connector to the plugin"**
→ Follow steps in `MAINTAINER_GUIDE_CONNECTOR_FRAMEWORK.md` section "Adding a New Connector"

### **"How do I decide between Work IQ and Office 365?"**
→ Read `shared/connector-decision-guide.md` section "Decision Trees" → "Scenario 1: SEARCH vs ACTION"

### **"What's a bad use of this connector?"**
→ Check `shared/connector-decision-guide.md` section "When NOT to Use a Connector"

### **"How do multiple connectors work together?"**
→ Read `shared/connector-decision-guide.md` section "Common App Patterns"

---

## 🚨 Important Notes

### **Single Source of Truth**
All connector selection logic is in `shared/connector-decision-guide.md`. 
- If you're recommending a connector → reference the guide
- If you're changing recommendations → update the guide ONCE
- All skills automatically use the updated logic

### **When to Update**
- New connector available → Update decision guide
- Connector capabilities change → Update capability matrix + decision trees
- New app pattern emerges → Add to "Common App Patterns"
- Users ask about a scenario → Add to examples

### **Never**
- ❌ Don't hardcode connector lists in skills
- ❌ Don't scatter decision logic across multiple files
- ❌ Don't add new connectors without updating the guide
- ❌ Don't create separate router logic — use the unified framework

---

## 📞 Need Help?

### **"How does this work?"**
- Start: `FRAMEWORK_SUMMARY.md`
- Deep dive: `shared/connector-decision-guide.md`

### **"How do I use it?"**
- Quick: `CONNECTOR_QUICK_REFERENCE.md`
- Full: `shared/connector-decision-guide.md`

### **"How do I maintain it?"**
- Guide: `MAINTAINER_GUIDE_CONNECTOR_FRAMEWORK.md`
- Checklist: Same file, "Maintenance Checklist" section

### **"I found a problem with a recommendation"**
1. Note what scenario failed
2. Check `shared/connector-decision-guide.md`
3. Update the decision guide
4. All skills automatically apply the fix

---

## 📊 Stats

- **Decision Trees:** 4
- **Connectors Covered:** 10+
- **Common App Patterns:** 5
- **Worked Examples:** 3
- **Anti-Patterns Documented:** 8+
- **Single Source of Truth:** Yes ✓
- **Maintenance Burden:** Reduced ✓

---

## 🎉 Ready to Go!

The connector selection framework is now fully integrated into the plugin. 

**Next time you use the plugin:**
1. Run `/add-datasource`
2. Watch it intelligently select connectors based on your scenario
3. See the reasoning behind each recommendation
4. Get multiple connectors working together automatically

**No more guessing. Just describe your app goal.**

---

**Created:** June 22, 2026
**Framework Version:** 1.0
**Status:** Production Ready ✓
