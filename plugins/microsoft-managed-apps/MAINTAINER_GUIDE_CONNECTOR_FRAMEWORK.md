# Plugin Maintainer Guide: Connector Decision Framework

This guide explains how to use and maintain the **Connector Decision Framework** that was added to the Microsoft Managed Apps plugin.

---

## What Problem Does This Solve?

**Before:** The plugin had a static table of connectors. Users and skills had to manually match their needs to the list, often choosing the wrong connector.

**After:** The plugin applies structured decision logic based on app scenarios, automatically recommending the right connector(s).

---

## Architecture: Where the Decision Logic Lives

### **Core Files**

1. **`shared/connector-decision-guide.md`** (10.8 KB)
   - **The source of truth** for all connector selection logic
   - Contains: decision trees, capability matrix, anti-patterns, examples
   - Referenced by all skills and the Architect agent
   - Update this file when adding connectors or changing recommendations

2. **`shared/shared-instructions.md`**
   - References the Connector Decision Guide
   - Makes it discoverable to all skills

3. **`skills/add-datasource/SKILL.md`**
   - The router skill that applies the decision framework
   - Uses the decision trees to match user scenarios to connectors
   - Invokes the right `/add-*` skills in sequence

4. **`agents/microsoft-apps-architect.md`**
   - Uses the framework for architecture recommendations
   - References decision guide when suggesting connectors

---

## How Connector Selection Works

### **Flow Diagram**

```
User describes app goal
    ↓
/add-datasource skill receives description
    ↓
Skill asks clarifying questions (purpose, data, actions, AI)
    ↓
Skill applies Decision Trees from Connector Decision Guide
    ↓
Skill matches scenario to recommended connector(s)
    ↓
Skill explains why (with reasoning from guide)
    ↓
Skill invokes /add-* skills in the right sequence
    ↓
App has correct connectors
```

### **Decision Trees (From the Guide)**

The framework includes 4 main decision trees:

1. **Scenario 1: SEARCH** — Is the app primarily a search interface?
   - YES → Work IQ
   - NO → Continue to Scenario 2

2. **Scenario 2: SERVICE** — What M365 service does the app work with?
   - Calendar → Office 365
   - Messages → Teams
   - Documents → SharePoint or OneDrive
   - Lists/Custom data → Dataverse
   - Work items → Azure DevOps
   - (etc.)

3. **Scenario 3: AI** — Does the app need AI features?
   - YES → Copilot Studio (for generation/summarization)
   - Also add Work IQ if semantic search is useful

4. **Scenario 4: HYBRID** — Does the app do BOTH search + action?
   - YES → Add multiple connectors in sequence

---

## Adding a New Connector

When a new connector becomes available:

### **Step 1: Document It in the Decision Guide**

Edit `shared/connector-decision-guide.md`:

1. **Add to Capability Matrix** (table at top)
   - Columns: Connector name, Best For, Can Search?, Can Create/Update?, Can Delete?, AI/Semantic?, MCP Server?

2. **Add to Decision Trees**
   - If it's a search connector → add to Scenario 1
   - If it's a CRUD connector → add to Scenario 2
   - If it has AI → add to Scenario 3

3. **Add "When NOT to use" section**
   - When would this connector be a bad choice?
   - What should users use instead?

4. **Add to common app patterns** (if applicable)
   - Does this enable a new app pattern?
   - Add it to "Common App Patterns" section

5. **Add examples** (if complex)
   - Step-by-step decision process using the new connector

### **Step 2: Create the `/add-*` Wrapper Skill**

Create a new skill directory: `skills/add-<connector>/`

With a minimal `SKILL.md` that delegates to `/add-connector`:

```yaml
---
name: add-<connector>
description: Adds <Connector> by delegating to `/add-connector` with api-id=<api-id> and mode=<mode>.
user-invocable: true
allowed-tools: Read, AskUserQuestion, Skill
---

# Add <Connector> (Wrapper)

This skill is a thin wrapper. Use `/add-connector` as the single implementation path.

## Delegation contract

Invoke `/add-connector` with:
- `api-id`: `<api-id>`
- `mode`: `action|table|procedure`

If the user already has a specific connection id, pass it through.
```

### **Step 3: Update the Architect Agent**

Edit `agents/microsoft-apps-architect.md`:

1. Add row to connector table in "Connector-First Principle" section
2. Example:
   ```
   | Use or read <Service>         | <Connector> (`/add-<connector>`) | <Why> |
   ```

---

## Modifying Connector Recommendations

When trade-offs change (e.g., a connector gains new capabilities):

### **Step 1: Identify the Change**

- What capability changed?
- Does it affect decision logic?
- Does it affect existing app patterns?

### **Step 2: Update the Decision Guide**

1. Update **Capability Matrix** if capabilities changed
2. Update **Decision Trees** if logic changed
3. Update **Anti-patterns** if misuse risk changed
4. Update **Common App Patterns** if new patterns emerge
5. Add new **Examples** if complex scenarios arise

### **Step 3: Propagate to Other Files**

- `/add-datasource` SKILL.md: Update examples/recommendations if needed
- Architect agent: Update guidance if recommending patterns changed

### **Step 4: Document the Change**

Add a comment in the Decision Guide explaining when and why the change was made.

---

## Testing Your Changes

After adding/modifying connector logic:

### **Test 1: Decision Tree Coverage**

Run through each decision tree with sample scenarios:
- ✅ Search scenario → Work IQ recommended
- ✅ CRUD scenario → Specific connector recommended
- ✅ AI scenario → Copilot Studio recommended
- ✅ Hybrid scenario → Multiple connectors recommended in sequence

### **Test 2: Anti-Pattern Detection**

Verify anti-patterns are clear:
- ❌ Don't recommend Work IQ for "send email"
- ❌ Don't recommend Office365 for "search all my files"
- ❌ Don't recommend Dataverse for "temporary session state"

### **Test 3: App Pattern Verification**

For each common app pattern:
- ✅ Pattern description makes sense
- ✅ Recommended connectors match the pattern
- ✅ Sequence is correct (actions before searches)

### **Test 4: Examples Are Correct**

Walk through each example in the guide:
- ✅ Questions asked are logical
- ✅ Decision process matches the decision trees
- ✅ Recommended connector is optimal

---

## Maintenance Checklist

**Quarterly review:**
- [ ] Any new connectors available in Power Platform?
- [ ] Any connector capabilities changed?
- [ ] Any new M365 services added?
- [ ] Any user feedback on recommendations?
- [ ] Any emerging app patterns?

**Before release:**
- [ ] All decision trees are current
- [ ] All examples are correct
- [ ] All anti-patterns are documented
- [ ] Capability matrix is complete

**When adding connectors:**
- [ ] Decision Guide updated
- [ ] `/add-*` skill created
- [ ] Architect agent updated
- [ ] Examples added
- [ ] Tests pass

---

## FAQ for Maintainers

### **Q: Why is the Decision Guide separate from individual skill files?**

**A:** Single source of truth. If logic changes, update one file and all skills automatically use the new logic. Avoids duplication and keeps knowledge centralized.

### **Q: When should a new connector get its own `/add-*` skill?**

**A:** Always. Every Power Platform connector should have a wrapper skill, even if it just delegates to `/add-connector`. This makes it discoverable via `/add-datasource` routing.

### **Q: What if a user scenario doesn't fit any decision tree?**

**A:** Add it! The framework is designed to grow. Scenarios that don't fit are signals that the framework needs expansion.

### **Q: How do we handle deprecated connectors?**

**A:** 
1. Mark as deprecated in the Capability Matrix
2. Update decision trees to avoid recommending it
3. Add anti-pattern: "Don't use [Deprecated Connector], use [Replacement] instead"
4. Keep the `/add-*` skill but add deprecation warning

### **Q: Can a connector appear in multiple decision trees?**

**A:** Yes! For example:
- Work IQ appears in both "search" and "AI" scenarios
- Office 365 appears in both "action" (email/calendar) and specific scenarios
- Dataverse appears in "custom data" and "persistent storage" scenarios

This is intentional — same connector, different use cases.

---

## Ownership & Updates

- **Decision Guide maintainer:** Should track M365/Power Platform ecosystem changes
- **Architect agent maintainer:** Should keep recommendations current
- **Skill maintainers:** Should reference the guide, not hardcode logic

**Goal:** Connector selection logic centralizes in one file so it's easy to update and version.
