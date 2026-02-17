________________________________________
name: planning description: Generates a comprehensive plan for implementation or project tasks, covering scope definition, necessary research, stepwise action items with dependencies, as well as validation steps, testing strategies, risk assessment, and final deliverables. Use when a user requests a development plan, architectural blueprint, feature rollout strategy, research plan, or step-by-step execution workflow.
________________________________________
Planning
Goal
Turn a user’s planning request into a structured, actionable plan delivered as the final assistant answer. The plan should cover all key aspects (scope, tasks, validation, etc.) needed to confidently execute the project or feature.
Workflow
1.	Gather context (if available): Quickly review any relevant context such as repository README, design docs, or codebase highlights. Identify project constraints (language, frameworks, requirements) and determine the type of plan needed (implementation, architectural, rollout, research, etc.) to tailor the approach.
2.	Clarify only if needed: If critical information is missing and planning cannot proceed responsibly, ask at most one or two specific clarification questions. Only ask if absolutely blocking; if unsure but not blocked, make a reasonable assumption and proceed with planning.
3.	Create the plan using the template below:
4.	Overview: Begin the plan with a brief introduction (1–3 sentences) stating what is being planned and why, including the high-level approach.
5.	Scope: Clearly define what is in scope and out of scope for this plan. Scope should establish boundaries so it’s obvious what will (and will not) be addressed.
6.	Research: Include a short list of any research, background context, or assumptions that inform the plan. (For example, reference relevant documentation, existing solutions, or performance requirements. If no preliminary research is needed, state "None needed" for this section.)
7.	Action items: Provide an ordered checklist of concrete tasks to execute the plan (aim for ~6–12 items). Each task must be atomic (a single actionable step) and written in verb-first form (e.g. "Setup database schema...", "Implement API endpoint for ...", "Write unit tests for ..."). Include specifics like file names, modules, or commands when applicable (based on context), but do not write any code in these steps. Ensure at least one task covers testing/validation and at least one addresses edge cases or risk mitigation. Avoid overly broad or trivial steps—each item should represent a meaningful, discrete action.
8.	Task breakdown: After the checklist, present a table of tasks with IDs and dependencies. Each task from the action list gets an ID (1, 2, 3, ...) and lists any dependencies (by ID) that must be completed before it. If a task has no prerequisite, mark its dependency as "None". This provides a clear execution order and any parallelization opportunities.
9.	Validation: Describe how the results will be validated or evaluated. Include clear success criteria or acceptance criteria here (e.g. “all unit tests pass”, “feature meets performance targets”, “code review completed”). Mention any specific validation steps, code reviews, or user acceptance tests required to confirm the solution works as intended.
10.	Testing: Outline the testing strategy and specific tests to be performed. This can include unit tests, integration tests, manual testing steps, or automated test frameworks. Be specific about what needs to be tested (functional behavior, edge cases, performance, etc.) and mention any tools or frameworks if relevant (without assuming a particular tech stack unless known).
11.	Risks: Identify potential risks, challenges, or edge cases, and for each, provide a brief mitigation plan. Use a bullet for each significant risk. (For example: "Potential scalability issue with X – Mitigation: design with caching to handle load.") If there are known unknowns or areas of uncertainty, note them here along with how to address them.
12.	Outputs: List the expected deliverables and outcomes of the plan. This should detail what artifacts or changes will result from executing the plan (for example, new modules or features, updated documentation, deployment of a service, etc.). Wherever possible, make deliverables measurable or quantifiable (e.g. version numbers, performance improvements, number of test cases). This section effectively answers "What will we have when the plan is complete?"
13.	Consistency: Use the exact section names and format as in the template. Include every section even if one is not applicable (in that case, give a brief note like "None" or "N/A" so that the plan is explicit and nothing is left implicit).
14.	Deliver the plan: Output the completed plan in Markdown format using the structure above, and nothing else. Do not prepend or append any additional commentary or explanations—provide only the plan itself as the final answer.
Plan template (follow exactly)
# Plan

<Brief introduction describing the goal and overall approach.>

## Scope
- **In scope:** <List aspects that the plan will cover>
- **Out of scope:** <List aspects that are not covered by this plan>

## Research
- <Key research finding or assumption 1>
- <Key research finding or assumption 2>
- *(...or "None needed" if no research required.)*

## Action items
- [ ] **Task 1:** <Description of first task>
- [ ] **Task 2:** <Description of second task>
- [ ] **Task 3:** <Description of third task>
- [ ] **Task 4:** <... and so on for each step in the plan>
- [ ] **Task N:** <Description of last task in the sequence>

## Task breakdown
| ID | Task                         | Depends on |
|----|------------------------------|------------|
| 1  | <Task 1 summary>            | None       |
| 2  | <Task 2 summary>            | 1          |
| 3  | <Task 3 summary>            | 2          |
| ...| <Task ... summary>          | <ID of prerequisite task, or None> |
| N  | <Task N summary>            | <...>      |

## Validation
- <How to validate that the solution meets requirements (e.g. criteria for success, review steps)>

## Testing
- <Tests to perform or testing strategy to ensure quality (unit tests, integration tests, etc.)>

## Risks
- <Risk 1: description of risk – *Mitigation:* plan to mitigate or address this risk>
- <Risk 2: description of risk – *Mitigation:* plan to mitigate or address this risk>
- <Additional risks as needed, or "None" if no significant risks>

## Outputs
- <Deliverable or outcome 1 (e.g. feature X implemented, accessible via ...)>
- <Deliverable or outcome 2 (e.g. documentation updated for Y)>
- <Success criteria or metrics for deliverables, if applicable>
Notes
•	Ensure clarity: The plan should be easy to follow and unambiguous. Each section provides specific information; avoid overlap or mixing concerns between sections.
•	No code or pseudo-code: The output is a planning document, not an implementation. Focus on what to do (and why), not the actual coded solution.
•	Tailor to context: If the planning task is specific (architecture design vs. feature implementation vs. research exploration), adjust the emphasis of the sections accordingly. Always maintain the standard structure, but the content should reflect the nature of the request.
•	Measurable outcomes: Wherever possible, frame outputs and success criteria in measurable terms (quantify improvements, specify targets or acceptance thresholds) so that success can be objectively evaluated.
________________________________________
