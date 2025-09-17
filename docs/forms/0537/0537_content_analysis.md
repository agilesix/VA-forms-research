# VA Design System Compliance Analysis
## Marriage Status Update Form Components

**Analysis Date:** December 17, 2024  
**Design System Version:** Current as of analysis date  
**Form Context:** Marriage certification form for DIC benefits

---

## Executive Summary

This analysis reviewed 12 form component files against the VA Design System standards. **6 files require immediate changes** for compliance, with most issues involving capitalization of "Veteran" and phone number label standards.

### Critical Findings:
- ❌ **5 files** have incorrect "veteran" capitalization (should be "Veteran")
- ❌ **1 file** uses non-standard phone number labels
- ⚠️ **2 files** have minor plain language improvement opportunities

---

## 🚨 Critical Issues (Launch Blocking)

### 1. Veteran Capitalization Issues
**VA Design System Rule**: "On VA.gov, capitalize 'Veteran' even when used as a common noun."

**Source**: [VA Design System Word List](https://design.va.gov/content-style-guide/word-list)
> **Veteran**: On VA.gov, capitalize even when used as a common noun.

**Files Affected**: 5 files

#### `recipientName.js`
```javascript
// CURRENT (Incorrect):
...titleUI('Deceased veteran information'),
'Tell us about the veteran whose DIC benefits you receive...'
veteranFullName: fullNameNoSuffixUI(title => `Deceased veteran's ${title}`),

// REQUIRED FIX:
...titleUI('Deceased Veteran information'),
'Tell us about the Veteran whose DIC benefits you receive...'
veteranFullName: fullNameNoSuffixUI(title => `Deceased Veteran's ${title}`),
```

#### `recipientIdentifier.js`
```javascript
// CURRENT (Incorrect):
...titleUI('Deceased veteran identification'),
"We need the deceased veteran's Social Security number..."
"Deceased veteran's Social Security number"
"Deceased veteran's VA file number"

// REQUIRED FIX:
...titleUI('Deceased Veteran identification'),
"We need the deceased Veteran's Social Security number..."
"Deceased Veteran's Social Security number"  
"Deceased Veteran's VA file number"
```

#### `spouseVeteranStatus.js`
```javascript
// CURRENT (Incorrect):
spouseVeteranUI['ui:title'] = 'Is your spouse a veteran?';
...titleUI('Is your spouse a veteran?'),

// REQUIRED FIX:
spouseVeteranUI['ui:title'] = 'Is your spouse a Veteran?';
...titleUI('Is your spouse a Veteran?'),
```

#### `spouseVeteranId.js`
```javascript
// CURRENT (Incorrect):
...titleUI('Spouse veteran information'),

// REQUIRED FIX:
...titleUI('Spouse Veteran information'),
```

#### `remarriageQuestion.js`
```javascript
// CURRENT (Incorrect):
'Have you remarried since the death of the veteran?'

// REQUIRED FIX:
'Have you remarried since the death of the Veteran?'
```

### 2. Phone Number Label Standards
**VA Design System Rule**: "Do not use primary or secondary phone numbers as headers. Home and mobile phone numbers are more plain language-focused."

**Source**: [Ask Users for Phone Numbers Pattern](https://design.va.gov/patterns/ask-users-for/phone-numbers)

**File Affected**: `phoneAndEmail.js`

```javascript
// CURRENT (Non-compliant):
daytimePhoneUI['ui:title'] = 'Daytime telephone number (include area code)';
eveningPhoneUI['ui:title'] = 'Evening telephone number (include area code)';

// RECOMMENDED FIX:
daytimePhoneUI['ui:title'] = 'Home phone number';
eveningPhoneUI['ui:title'] = 'Mobile phone number';
```

**⚠️ DISCOVERY NEEDED**: The original PDF form may specify "daytime" and "evening" phone numbers. **Recommend reviewing the source PDF to determine**:
1. If the PDF uses "daytime/evening" terminology
2. Whether legal/regulatory requirements mandate this specific language
3. If we can map PDF requirements to VA Design System standards

**If PDF requires "daytime/evening"**: Consider compromise approach:
```javascript
// POTENTIAL COMPROMISE:
daytimePhoneUI['ui:title'] = 'Daytime phone number';
eveningPhoneUI['ui:title'] = 'Evening phone number';
// (Remove "telephone" and "include area code" for cleaner language)
```

---

## ⚠️ Minor Issues (Should Fix)

### 3. Plain Language Improvements

#### `terminationDetails.js`
**VA Design System Rule**: Use plain language alternatives

**Source**: [Plain Language Standards](https://design.va.gov/content-style-guide/plain-language)

```javascript
// CURRENT:
terminationReason: textUI({
  title: 'Reason for termination (i.e., death, divorce)',
  required: () => true,
}),

// RECOMMENDED:
terminationReason: textUI({
  title: 'Reason for termination (such as death, divorce)',
  required: () => true,
}),
```
**Rationale**: VA plain language guidelines prefer "such as" over Latin abbreviations like "i.e."

#### `terminationStatus.js` - Consistency Opportunity
**Current inconsistency**:
- Title: "Has your remarriage ended?"
- Question: "Has your remarriage been terminated?"

**Recommendation**: Align language for consistency:
```javascript
// Option 1 - Use "ended" throughout:
...titleUI('Has your remarriage ended?'),
hasTerminatedUI['ui:title'] = 'Has your remarriage ended?';

// Option 2 - Use "terminated" throughout:  
...titleUI('Has your remarriage been terminated?'),
// (Keep existing question text)
```

---

## ✅ Compliant Areas

### Correct Implementations Already in Place:

1. **"Deceased" Terminology** ✅
   - Correctly using "deceased" over "decedent"
   - **Source**: [VA Design System Word List](https://design.va.gov/content-style-guide/word-list)

2. **Sentence Case Capitalization** ✅  
   - Most titles properly use sentence case
   - **Source**: [Capitalization Guidelines](https://design.va.gov/content-style-guide/capitalization)

3. **Form Structure** ✅
   - Proper use of `titleUI`, required fields, and schema structure
   - **Source**: [Web Component Patterns](https://design.va.gov/about/developers/using-web-components)

4. **Required Field Implementation** ✅
   - Correct use of `'ui:required': () => true`

---

## Action Items by Priority

### 🚨 **IMMEDIATE (Launch Blocking)**
1. **Fix Veteran capitalization** in 5 files
2. **Resolve phone number labels** after PDF review

### ⚠️ **HIGH PRIORITY**
3. **Update "i.e." to "such as"** in termination details
4. **Align termination language** for consistency

### 📝 **DOCUMENTATION**
5. **PDF Discovery**: Review source PDF for phone number requirements
6. **Create mapping** between PDF fields and VA Design System standards

---

## Files Requiring Changes

| File | Issue Type | Priority | Status |
|------|------------|----------|---------|
| `recipientName.js` | Veteran capitalization | Critical | ❌ Needs Fix |
| `recipientIdentifier.js` | Veteran capitalization | Critical | ❌ Needs Fix |
| `spouseVeteranStatus.js` | Veteran capitalization | Critical | ❌ Needs Fix |
| `spouseVeteranId.js` | Veteran capitalization | Critical | ❌ Needs Fix |
| `remarriageQuestion.js` | Veteran capitalization | Critical | ❌ Needs Fix |
| `phoneAndEmail.js` | Phone labels + PDF review | Critical | 🔍 Needs Discovery |
| `terminationDetails.js` | Plain language | Minor | ⚠️ Should Fix |
| `terminationStatus.js` | Consistency | Minor | ⚠️ Should Fix |

---

## Design System References

### Key VA Design System Resources:
- **Word List**: https://design.va.gov/content-style-guide/word-list
- **Capitalization**: https://design.va.gov/content-style-guide/capitalization  
- **Phone Numbers Pattern**: https://design.va.gov/patterns/ask-users-for/phone-numbers
- **Plain Language**: https://design.va.gov/content-style-guide/plain-language
- **Neutral Language**: https://design.va.gov/content-style-guide/neutral-language
- **Web Components**: https://design.va.gov/about/developers/using-web-components
- **Marital Information Pattern**: https://design.va.gov/patterns/ask-users-for/marital-information

### Validation Process:
1. Review each file against the specific design system page
2. Implement changes according to documented standards
3. Test for compliance using VA.gov design system tools
4. Conduct final review against [Design System Components](https://design.va.gov/components/)

---

## Next Steps

1. **Implement critical fixes** for Veteran capitalization
2. **Conduct PDF review** for phone number requirements  
3. **Coordinate with legal/compliance teams** if conflicts arise between PDF requirements and design system
4. **Update components** following VA Design System change management process
5. **Validate changes** against design system documentation

---

*This analysis was conducted using the VA Design System search tool and current documentation. For questions about specific implementations, consult the [VA Design System Team](https://design.va.gov/about/contributing-to-the-design-system).*
