---
description: Save company url to companies.md
argument-hint: [A carrer page url of a company.]
---

# Add Company Command

Save company url to companies.md file

## Requirements

- $ARGUMENTS mustn't be empty
- URL must be a career page
- Career page can be determined by pathname patterns

### Supported Career Page Pathnames

```text
/kariera
/career
/jobs
/join-us
```

## Implementation Steps

### 1. URL Processing & Validation

```bash
# Extract domain from URL (e.g., "https://company.cz/kariera" → "company.cz")
# Validate URL format and accessibility
# Check pathname contains: /kariera, /career, /jobs
```

### 2. Primary Data Extraction

#### Company Name Extraction Strategy

```bash
# Primary: WebFetch with targeted prompt for company name
WebFetch url="$ARGUMENTS" prompt="Extract the company name from this career page. \
Look for: 1) Page title 2) Company logo text 3) Main heading. \
Return only the company name, nothing else."

# Fallback: Parse domain name if WebFetch fails
domain=$(echo "$ARGUMENTS" | sed 's|https\?://||' | sed 's|/.*||' | sed 's|^[^.]*\.||')
company_name=$(echo "$domain" | sed 's|\..*||' | tr '[:lower:]' '[:upper:]' | \
sed 's/.*/\L&/' | sed 's/\b\w/\U&/g')
```

#### Full Company Information

```bash
# Use WebFetch to analyze career page
WebFetch $ARGUMENTS "Extract company information: name, industry, location, \
description, company size, founding date if available"

# If career page lacks details, try main website
WebFetch "https://[domain]" "Get company overview, about us, and basic company details"
```

### 3. Duplicate Detection

```bash
# Check for existing company by domain
Grep pattern="[domain]" file="companies.md" output_mode="files_with_matches"

# Check for existing company by name (case-insensitive)
Grep pattern="[company_name]" file="companies.md" -i=true output_mode="content"

# If matches found, prompt user for confirmation to proceed/update
```

### 4. Data Enrichment (If Needed)

```bash
# If primary extraction yields insufficient data
WebSearch query="[company_name] headquarters industry location"
WebSearch query="[company_name] about company information"
```

### 5. Data Formatting & Writing

```bash
# Format company entry according to companies.md structure from README.md:
- [Company Name](career_page_url) - Brief description of what they do

# Requirements:
# - Each company on one line starting with '-'
# - Description required for all companies
# - Description must be one short sentence describing what the company does
# - Entire entry must fit on one line
# - Use markdown list format: - [Company Name](URL) - Description

# Append to companies.md file
```

### 6. Error Handling & Validation

- **Invalid URL**: Check format, accessibility
- **Non-career page**: Validate pathname patterns
- **Network errors**: Provide fallback prompts for manual input
- **Duplicate handling**: Show existing entry, ask to update/skip
- **Insufficient data**: Prompt user for missing required fields

## Implementation Details

### Search Command Specifications

#### WebFetch Commands

```bash
# Primary career page analysis
WebFetch url="$ARGUMENTS" prompt="Extract only: 1) Company name \
2) Brief description of what the company does (ONE SHORT SENTENCE ONLY)"

# Fallback main website analysis
WebFetch url="[base_domain_url]" prompt="Extract only: 1) Company name \
2) Brief description of what the company does (ONE SHORT SENTENCE ONLY)"
```

#### Grep Commands

```bash
# Domain duplicate check
Grep pattern="[extracted_domain]" path="companies.md" output_mode="files_with_matches"

# Company name duplicate check
Grep pattern="[extracted_company_name]" path="companies.md" -i=true \
output_mode="content" -A=2 -B=2

# Show context around matches for user review
```

#### WebSearch Commands (Fallback)

```bash
# Company description only - must return ONE SHORT SENTENCE
WebSearch query="[company_name] company what does the company do business description"
```

### Data Flow Strategy

1. **URL → Domain Extraction**: Parse URL to get base domain
2. **WebFetch → Company Data**: Extract only company name and ONE SHORT SENTENCE description
3. **Company Data → Grep Search**: Use extracted company name/domain for duplicate detection
4. **Search Results → User Confirmation**: Show duplicates if found, get user decision
5. **Final Data → File Append**: Format and add to companies.md using format: - [Company Name](URL) - Description

### Decision Points

- **Insufficient Data**: If WebFetch returns minimal info, trigger WebSearch
- **Duplicate Found**: Present to user with options: skip, update, or add anyway
- **Search Failures**: Fall back to manual input prompts
- **Data Quality**: Validate required fields (company name, one-sentence description) before adding
- **Format Validation**: Ensure entire entry fits on one line with markdown list format starting with '-'

## Examples

### Example 1: Valid Career Page

**Input:**

```text
https://atlassian.com/company/careers
```

**Expected Output:**

```text
- [Atlassian](https://atlassian.com/company/careers) - Develops collaboration and productivity software for teams
```

### Example 2: Czech Company Career Page

**Input:**

```text
https://avast.com/cs-cz/kariera
```

**Expected Output:**

```text
- [Avast](https://avast.com/cs-cz/kariera) - Provides cybersecurity software and services for consumers and businesses
```

### Example 3: Invalid URL (Non-Career Page)

**Input:**

```text
https://google.com/about
```

**Expected Behavior:**

- Error: URL is not a career page (pathname doesn't match /kariera, /career, /jobs, /join-us)
- Command should exit with error message

### Example 4: Duplicate Company Detection

**Input:**

```text
https://microsoft.com/careers
```

**Expected Behavior (if Microsoft already exists):**

- Error: Company already exists in companies.md
- Show existing entry: `- [Microsoft](https://microsoft.com/jobs) - Develops software, services and solutions`
- Command should exit with error message

### Example 5: URL Validation Error

**Input:**

```text
https://nonexistent-company-12345.com/careers
```

**Expected Behavior:**

- Error: Unable to access URL or extract company information
- Fallback to manual input prompts for company name and description
