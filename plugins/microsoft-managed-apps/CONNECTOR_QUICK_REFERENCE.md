# Connector Quick Reference Card

**Use this to quickly decide which connector(s) to recommend or use.**

---

## The 5-Question Decision Process

1. **Is the app primarily a SEARCH interface?**
   - YES → Use Work IQ (semantic M365 search)
   - NO → Continue to #2

2. **What M365 service does the app work with?**
   - Calendar events → Office 365 (`/add-office365`)
   - Teams messages → Teams (`/add-teams`)
   - Files → OneDrive (`/add-onedrive`)
   - Documents/Lists → SharePoint (`/add-sharepoint`)
   - Custom data → Dataverse (`/add-dataverse`)
   - Work items → Azure DevOps (`/add-azuredevops`)
   - Spreadsheets → Excel (`/add-excel`)
   - Database → SQL Procedures (`/add-procedure`)

3. **Does the app need CREATE/UPDATE/DELETE?**
   - If specific service selected in #2 → use that connector for actions
   - If only READ needed → search connector is sufficient

4. **Does the app need AI features?**
   - Summarization/Content Generation → Copilot Studio (`/add-mcscopilot`)
   - Semantic search across M365 → Work IQ (`/add-workiq`) (if not already added)

5. **Multiple responsibilities in one app?**
   - YES → Add multiple connectors in sequence
   - Example: Office365 (fetch calendar) + Copilot Studio (summarize) + Work IQ (semantic search)

---

## One-Line Recommendations

| I want to...                               | Use this connector  |
|--------------------------------------------|-------------------|
| Search M365 data (semantic, conversational) | Work IQ           |
| List/manage calendar events                 | Office 365        |
| Send/read Teams messages                   | Teams             |
| Search/manage documents                    | SharePoint        |
| Upload/manage files                        | OneDrive          |
| Store custom business data                 | Dataverse         |
| Track work items/bugs                      | Azure DevOps      |
| Read/write spreadsheets                    | Excel Online      |
| Generate summaries or invoke AI agents     | Copilot Studio    |
| Call database stored procedures            | SQL Procedures    |

---

## Common App Patterns (Pre-Made Combinations)

### **Dashboard / Report**
```
Office365 (calendar) + Azure DevOps (work items) + [optional Work IQ for search]
```

### **Meeting Insights**
```
Office365 (meetings) + Teams (transcripts) + Copilot Studio (summaries) + [optional Work IQ for search]
```

### **Document Search**
```
Work IQ (semantic search) + SharePoint (manage) + OneDrive (if personal files)
```

### **Task Tracker with AI**
```
Dataverse (custom tasks) + Copilot Studio (AI suggestions)
```

### **Customer CRM**
```
Dataverse (customer records) + Office365 (emails) + [optional Work IQ for conversations search]
```

---

## ❌ Anti-Patterns (Don't Do This)

- ❌ Use Work IQ to send an email → Use Office365 instead
- ❌ Use Office365 to search files → Use Work IQ instead
- ❌ Use Dataverse for session data → Use React state instead
- ❌ Use Excel for custom business logic → Use Dataverse instead
- ❌ Make direct API calls → Use the connector instead

---

## Decision Tree (Visual)

```
┌─ Is it SEARCH?
│  ├─ YES → Work IQ ✓
│  └─ NO → Continue
│
├─ What SERVICE?
│  ├─ Calendar → Office365 ✓
│  ├─ Teams → Teams ✓
│  ├─ Files → OneDrive ✓
│  ├─ Lists/Docs → SharePoint ✓
│  ├─ Custom → Dataverse ✓
│  ├─ Work Items → Azure DevOps ✓
│  ├─ Sheets → Excel ✓
│  └─ Database → SQL ✓
│
├─ Need ACTIONS (CUD)?
│  ├─ YES → Already covered above ✓
│  └─ NO → Search connector OK ✓
│
├─ Need AI?
│  ├─ YES → Copilot Studio ✓
│  └─ NO → Continue
│
└─ DONE → Invoke selected connectors in sequence
```

---

## For When You Get Stuck

1. **Read the full guide:** `shared/connector-decision-guide.md`
2. **Check examples:** Section "Examples: Applying the Guide"
3. **Check patterns:** Section "Common App Patterns"
4. **Ask:** What is the app's PRIMARY responsibility?
   - Is it mainly for search? → Work IQ
   - Is it mainly for managing data? → Specific connector
   - Is it mainly for AI features? → Copilot Studio
5. **Combine:** If multiple responsibilities → use multiple connectors

---

## File Locations

- **Full Decision Guide:** `shared/connector-decision-guide.md`
- **Maintainer Guide:** `MAINTAINER_GUIDE_CONNECTOR_FRAMEWORK.md`
- **Improvements Summary:** `CONNECTOR_SELECTION_IMPROVEMENTS.md`
- **Skills that use this:** `/add-datasource`, `/add-connector`, `/add-*` wrappers
- **Architect Agent:** `agents/microsoft-apps-architect.md`

---

## Updated: 2026-06-22

Last updated when Connector Decision Framework was formalized.
