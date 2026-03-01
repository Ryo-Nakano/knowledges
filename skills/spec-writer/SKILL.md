---
name: spec-writer
description: Creates comprehensive specification documents based on user requirements and the project's template. Use this skill when the user wants to implement a new feature or fix a bug and needs to define the specifications first. It enforces a strict "no assumptions, no sycophancy" policy during requirements gathering.
---

# Spec Writer

## Overview

This skill guides the user through the process of defining requirements and creating a specification document. It strictly adheres to the project's `specs/_template.md` structure and follows rigorous interview guidelines to ensure clarity and technical feasibility.

## Core Mandates

1.  **Language Requirement**: All communication with the user and the resulting specification document MUST be in Japanese.
2.  **Reference the Template**: Always start by reading `specs/_template.md` to understand the required sections.
3.  **Strict Interview Process**: Follow the guidelines in `references/interview_guidelines.md`.
    - **Do Not Assume**: Never fill in gaps with assumptions. Ask clarifying questions.
    - **No Sycophancy**: Provide objective, technical feedback. Challenge flawed requests.
    - **Be Specific**: Demand measurable criteria and clear definitions.
4.  **Create the Spec File**: Once requirements are clear, create a new Markdown file in the `specs/` directory using the template structure.

## Workflow

### Phase 1: Preparation

1.  Read the content of `specs/_template.md`.
    - If the file does not exist, ask the user to provide a template or guidance on the desired structure.
2.  Read `references/interview_guidelines.md` to load the interview principles into context.

### Phase 2: The Interview

Conduct a thorough interview to gather all necessary information to fill the template.

- **Objective**: Understand the "Why" and "What" clearly.
- **Scope**: Define what is in scope and what is out of scope.
- **Technical Feasibility**: Assess if the request is technically sound. If not, explain why and propose alternatives.
- **Edge Cases**: Ask about error handling, boundary conditions, and user permissions.

**Example Questions:**
- "What is the primary user goal for this feature?"
- "What should happen if the input data is invalid?"
- "Are there any performance constraints we need to consider?"
- "How does this feature interact with existing module X?"

### Phase 3: Drafting the Specification

1.  Propose a draft of the specification based on the interview.
2.  Ensure all sections from `specs/_template.md` are covered.
3.  Ask the user for confirmation.

### Phase 4: Finalization

1.  Create a new file in the `specs/` directory.
    - Filename format: `specs/{feature_name_in_snake_case}_spec.md`.
    - Example: `specs/add_user_login_spec.md`.
2.  Write the approved content to the file.
3.  Confirm with the user that the file has been created and asks if they are ready to proceed with implementation planning (though implementation itself is outside this skill's scope).
