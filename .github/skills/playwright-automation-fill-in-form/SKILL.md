---
name: playwright-automation-fill-in-form
description: 'Automate filling in a form using Playwright MCP'
---

# Automating Filling in a Form with Playwright MCP

Your goal is to automate the process of filling in a form using Playwright MCP.

## Specific Instructions

Navigate to the form URL provided by the user. If no URL is provided, ask for one.

### Fill in the form with the details the user provides:

Use accessible locators to find each field — prefer `getByLabel`, `getByRole`, `getByPlaceholder`.

For each field:
1. Locate the field using its label or role
2. Fill it using `fill()` for text, `selectOption()` for dropdowns, `setInputFiles()` for file uploads
3. Verify the value was entered correctly before moving to the next field

**DO NOT SUBMIT THE FORM.**

Ask the user to review all filled values before submitting. Show a summary of what was entered.

> Source: [github/awesome-copilot](https://github.com/github/awesome-copilot)
