name: documentation-update
description: "Update and validate project documentation whenever code changes (new features, API endpoints, configuration/CLI updates, etc.) require it. Applies consistent Markdown formatting and uses language-appropriate doc generation tools (Python, JS/TS, Go, Java, Rust, etc.) to ensure all documentation is up-to-date and linted."
---
Overview: This skill updates software documentation to reflect the latest code behavior and standards. It is language-agnostic, applying to any project where documentation (e.g. READMEs, docs sites, API references, config guides) needs revision after changes. The skill enforces modern documentation practices and validation across Python, JavaScript/TypeScript, Go, Java, Rust, and more by using appropriate tools in each ecosystem.
When to Use the Skill
Use documentation-update whenever a change in the codebase implies documentation should be added or modified. This includes:
•	Feature Changes: When a new feature is added or an existing behavior changes. For example, if a function’s behavior is updated or a new module is introduced, update usage examples and descriptions.
•	API Updates: When API endpoints (REST/GraphQL routes, public functions, SDK methods) are added, changed, or deprecated. Document new endpoints or changes in request/response formats.
•	Configuration & CLI Changes: When configuration files, environment variables, or command-line interface options are added or modified. Update setup guides, config references, or --help documentation accordingly.
•	Installation or Setup Changes: When the setup process (installation steps, prerequisites, build process) changes. Reflect these in installation docs or README instructions.
•	New Examples or Tutorials: When adding example code, how-to guides, or tutorials for new use cases. Ensure new examples are documented and existing tutorials incorporate the latest best practices.
Triggering conditions: This skill should be invoked for user requests like “update the docs for this change” or when a code diff/PR includes user-facing changes needing docs. The YAML description explicitly captures these triggers so the agent knows when to apply this skill[1].
Documentation Tools & Workflows by Language
To keep the skill general across projects, it uses common documentation generation and validation tools for each language ecosystem:
•	Python: Use tools like Sphinx (with reStructuredText or MyST Markdown) or MkDocs for static site docs. For example, run sphinx-build or mkdocs build to generate HTML and catch warnings. Ensure Python docstrings for public APIs are updated if using autodoc.
•	JavaScript/TypeScript: Use JSDoc or TypeDoc comments for APIs, and static site generators like Docusaurus or VuePress if applicable. For instance, run npx typedoc or npm run docs to build docs and verify no errors.
•	Go: Documentation is often in code comments (godoc). Ensure exported functions/types have updated comments. Use go doc or godoc to review output. If the project uses MkDocs or Hugo for additional docs, update those Markdown files.
•	Java/Kotlin: Use Javadoc (e.g. via Maven or Gradle) for API docs. Update Javadoc comments in code for new or changed APIs and run mvn javadoc:javadoc to verify documentation generates without warnings. Also update any user guides in Markdown if included.
•	Rust: Use Rust’s built-in rustdoc (via cargo doc). Update /// doc comments on public items for new features. If the project has a mdBook or MkDocs site, update those as well. Optionally, enforce #![deny(missing_docs)] in Rust projects to ensure all public items are documented.
•	Other/General: Many projects use static documentation sites (like MkDocs, Docusaurus, Sphinx on ReadTheDocs, etc.). Identify the docs tool in use (check for docs/ folder or config files like mkdocs.yml, docusaurus.config.js, conf.py for Sphinx) and use its workflow. Always run the doc build or generation command to catch formatting issues or warnings.
Each ecosystem often also provides linters or validators for docs. For example, Markdown linting can be done with markdownlint or remark-lint (npm run docs:lint is commonly configured in JS projects to check doc formatting). Sphinx can be run in nit-picky mode (-nW) to treat warnings as errors. Java projects might fail the build if Javadoc has errors. Leverage these tools to validate that documentation updates are correct and complete. (For instance, a CI pipeline may include steps like npm run docs:lint and npm run docs:validate to ensure documentation quality[2].)
Standard Markdown Style Guidelines
All documentation updates must follow consistent Markdown formatting and style:
•	Headings: Use a clear hierarchy. One # top-level title (usually for the document title), then ## for major sections, ### for subsections, etc. Do not skip levels (e.g., don’t jump from an ## heading to ####). Ensure each page has a single H1 and the structure is logical.
•	Code Fences: Use triple backtick code blocks with the language specified for syntax highlighting (e.g., python,json, ```bash). For inline code, use single backticks. Verify that any sample code is up-to-date and formatted correctly.
•	Links and Images: Use descriptive link text and format links as [text](URL). For images, use ![alt text](image.png) with meaningful alt text. Ensure image files exist and consider their size (embedding diagrams or screenshots can aid clarity). Tables should have a header row and use Markdown table syntax properly aligned.
•	Line Length & Whitespace: Keep lines at a reasonable length for readability in diffs (commonly 80-120 characters, unless the project prefers one sentence per line). Include blank lines between headings, paragraphs, or lists as needed for correct Markdown rendering. Remove trailing whitespace and ensure lists are properly indented. Use consistent indentation for nested lists or code examples.
•	Formatting Consistency: Follow any project-specific markdown rules (check if a style guide or .markdownlint.json exists). This includes things like using - for bullet points consistently, or whether to capitalize headings. If available, use the markdown-rules skill or a markdown linter to double-check style compliance.
By adhering to these conventions, the documentation remains clean and easy to maintain. (Refer to the markdown-rules skill for a comprehensive list of Markdown best practices specific to your environment.)
Documentation Update Workflow
To perform a documentation update, follow this step-by-step workflow:
1. Assess the Change: Start by understanding the scope of the code change or feature that necessitated the docs update. Read the feature description, issue, or commit message to identify what needs documenting (new behaviors, new commands, changed outputs, etc.). This ensures you don’t miss any aspect of the update.
2. Identify Affected Documents: Determine which documentation files or sections need updates. Common targets include: - README.md (if installation, usage, or overview needs change). - User guides or tutorials in a docs/ directory. - API reference pages (REST API docs, SDK reference, etc.). - Configuration reference or environment variable tables. - Changelog (if recording changes for release notes). - Any example code files or Jupyter notebooks provided to users. If the project maintains multiple versions of docs (e.g., versioned docs on a website), ensure you update the appropriate version.
3. Apply Updates to Content: Edit the relevant markdown/reStructuredText files with the new information: - Insert new sections or paragraphs for new features. For example, add a subsection in the user guide explaining how to use the new feature or option. - Update existing text that is now inaccurate. For instance, if an API changed a parameter or default value, change that in the docs description and usage examples. - Add or update code samples demonstrating the new behavior. Ensure code samples are valid (they compile/run and produce the described output). - Update any diagrams or sequence charts if needed (some projects use Mermaid or PlantUML for architecture diagrams; update the diagrams if the architecture changed). - If the change is significant, add an entry to the CHANGELOG (see template below) under the “Unreleased” or next version section, categorizing the change (Added, Changed, Fixed, etc.). - Maintain the tone and voice of existing documentation. For developer docs, the tone is usually concise and technical; for user docs, it might be more explanatory.
While editing, apply the Standard Markdown Guidelines mentioned above for formatting. Use clear language and assume the reader might not know the background – provide context for the changes if necessary.
4. Validate the Documentation: After making changes, verify that everything builds and renders correctly: - Build/Generate Docs: Run the documentation generation process. For example, make html or sphinx-build -b html docs/ docs/_build for Sphinx, mkdocs build for MkDocs, or npm run docs for a JS project. Address any warnings or errors (broken links, missing references, formatting issues) that the build logs report. This catches issues like undefined references or malformed Markdown. - Lint Markdown: Run linters/validators. Commonly, markdownlint . (with config) can catch style issues in Markdown. In Python, tools like doc8 can catch reStructuredText issues; in JS, remark or Prettier might be used to format docs. Ensure no lint errors remain. - Review Content Quality: Manually read the updated sections to ensure technical accuracy and clarity. Check that all references to code (function names, CLI flags, file paths) match the actual code (this often entails double-checking the code or tests). Confirm that examples produce the expected output. - Link Check: If possible, run a link checker to verify that all hyperlinks in the documentation (both internal anchors and external URLs) are valid. Some docs tools have a link-check mode (Sphinx has sphinx-build -b linkcheck, Docusaurus/others often rely on third-party link checkers). - Spellcheck/Grammar: Optionally, use a spell checking tool or ensure no obvious typos are introduced. Consistency in terminology is important (e.g. if the UI element was renamed, use the new term everywhere).
By validating these points, we ensure the updated documentation is not only up-to-date but also meets quality standards and will not break the docs site build. If any validation step fails (e.g., broken link, linter error), fix those issues before finalizing. This might involve further edits to correct the problems.
5. Finalize and Communicate: Once the docs are updated and validated: - If this is part of a code change commit, include the documentation changes in the same commit or pull request, so they get reviewed together. Ensure the commit message or PR description mentions that docs were updated. - Double-check if any TODOs or placeholders remain. Remove or resolve them. The documentation should be complete; avoid leaving sections saying “to be updated” or “coming soon.” If certain info is not yet decided, it’s better to describe the current state and note any planned updates in a future release. - Include an entry in the project’s CHANGELOG for this update if it’s user-facing. This helps users see at a glance what was updated (for example, “Added documentation for new Feature X” or “Updated configuration guide for Y change”). - Optionally, notify relevant stakeholders (e.g., if documentation is published to a wiki or an internal portal, make sure to deploy the changes or inform the team of the updates).
Following this workflow ensures a thorough documentation update that coincides with code changes, maintaining consistency between code and docs.
Templates for Common Documentation Updates
To streamline writing, here are templates/examples for typical documentation additions:
Template: New Feature Documentation
When documenting a new feature, create a clear section in the docs (or README) for it. For example:
## New Feature: <Feature Name>

<Brief description of the feature and its purpose in 1-2 sentences.>

**Usage:**
- <How users enable or access this feature>. For example, “Run `tool --featureX` to activate X.”
- <Mention any prerequisites or setup needed for this feature> (e.g., configuration flags).

**Example:**
```bash
# Example of using the feature (commands or code)
$ tool --featureX input.txt
# Expected output or behavior
Processed input.txt with Feature X enabled.
**Impact:** <Explain any important implications, such as performance impact, compatibility notes, or interactions with other features. If the feature replaces an old one, note differences.>
This template includes a feature description, instructions on usage, an example, and an impact note. Adjust the structure as needed (some projects might include subheadings like “Overview”, “How to Use”, “Example” etc.). Ensure the tone matches the rest of the documentation.
Template: API Endpoint Documentation
For a new or changed API endpoint, provide details on its purpose and usage. For example, in API reference:
**Endpoint:** `POST /api/v1/users`

**Description:** Create a new user account. This endpoint registers a new user in the system.

**Request:**
- **URL Parameters:** *None* (or list if any, e.g. `/users/{id}`).
- **Query Parameters:** *None* (or list optional query params).
- **Body:** JSON object with user details:
  ```json
  {
    "email": "user@example.com",
    "password": "securePassword123",
    "name": "John Doe"
  }
  ```
- **Headers:** Requires `Authorization: Bearer <token>` (if authentication is needed).

**Response:**
- **201 Created:** on success, returns JSON:
  ```json
  {
    "id": 123,
    "email": "user@example.com",
    "name": "John Doe",
    "created_at": "2026-02-03T10:39:17Z"
  }
  ```
- **400 Bad Request:** if input is invalid (with an error message in response).
- **409 Conflict:** if email already exists.
- **401 Unauthorized:** if auth token is missing or invalid.

**Notes:** This endpoint requires admin privileges to create certain types of users. Make sure to handle errors as described.
Ensure that each part of the endpoint (method, URL, parameters, body, response, error codes) is documented. Use real examples for requests and responses. If using a tool like OpenAPI/Swagger, ensure the YAML/JSON spec is updated too, but still provide human-readable explanations like above in the docs.
Template: Changelog Entry
Follow a consistent changelog format for logging changes. A common standard is Keep a Changelog style, grouping changes into categories[3]. For example:
## [Unreleased] - 2026-02-03
### Added
- Documentation for new **Feature X** including usage instructions and examples.
- API reference entry for `POST /api/v1/users` (user registration endpoint).

### Changed
- Updated configuration guide to reflect new `MAX_RETRIES` setting.
- Revised CLI help output in README.md for the `--verbose` flag behavior change.

### Fixed
- Corrected a typo in the Introduction section of the User Guide.
Each entry is under a heading like “Added”, “Changed”, “Fixed” (use others like “Deprecated” or “Removed” as needed). Include the date and upcoming version if known. Keep descriptions concise but clear about what was done. This makes it easy for users and contributors to see what documentation (and features) have changed at a glance. When releasing, move these entries under a version number heading and add the release date.
(Refer to the project’s existing CHANGELOG.md for format; the above is an example based on widely-used conventions. The Keep a Changelog format with Added/Changed/Fixed sections is recommended for clarity[4].)
Documentation Quality Checklist
Before considering documentation updates complete, go through this checklist to ensure high quality:
•	Completeness: All new features, changes, or fixes from the code are documented. There are no sections left undocumented or marked "TBD". If a feature is experimental or behind a flag, that is noted.
•	Accuracy: Technical details in the docs match the actual behavior of the latest code. (Double-check function names, CLI flags, default values, error codes, etc. against the code or tests.)
•	Clarity: The writing is clear and understandable. Jargon is explained or kept consistent. Sentences are not overly long. Use active voice and present tense where possible.
•	Consistency: The style and tone match the existing documentation. Terminology is consistent across pages. If the project has a style guide (for example, capitalization of certain terms or UI elements), follow it.
•	Formatting: Markdown formatting is correct (headings, lists, code blocks, tables all render properly). No linting or build warnings remain. All links and references work (no broken links or images).
•	Examples Work: Any code example or CLI snippet has been tested or reasoned through to produce the described outcome. If possible, actually run the example to ensure it’s valid.
•	No Redundancy: Avoid duplicating content in multiple places that could get out-of-sync. Instead, cross-link to canonical sections. For instance, if installation instructions are in the README and a docs site, perhaps have the README link to the docs site to avoid needing to update in two places.
•	Up-to-Date Changelog: If user-facing changes were made, the changelog is updated (or an unreleased section is prepared). This often is checked during release prep but is good to update along with docs.
•	Review & Approval: If working with a team, get a review of the doc changes. Documentation updates should go through the same review process as code. A reviewer might catch unclear phrasing or missing info. (Leverage the code-review skill’s checklists – it typically includes verifying docs are updated in a PR.)
Finally, ensure documentation changes are checked in alongside code changes. Many teams enforce this via code reviews or even CI checks (for example, failing the build if documentation is missing for a PR). This skill’s output should help satisfy those requirements by producing thorough updates.
Related skills: Consider using the planning skill to plan and track documentation tasks for new features as part of your development process. During reviews, the code-review skill can help ensure that documentation updates are not overlooked. For any detailed Markdown style questions, refer to the markdown-rules skill. Each of these skills can complement documentation-update to maintain a robust documentation practice across the project.
(Note: The YAML frontmatter description of this skill is written to clearly indicate when the agent should invoke it, as per OpenAI skill guidelines[1]. This skill is instruction-only; Codex only loads the name and description at runtime until the skill is actually invoked, keeping the context footprint small[5].)
________________________________________
[1] [5] create-skills.md
file://file_000000004710720e80f96773c88c468c
[2] Codex task assigment : r/codex
https://www.reddit.com/r/codex/comments/1qcftww/codex_task_assigment/
[3] [4] Everything you need to know about CHANGELOG.md
https://openchangelog.com/blog/changelog-md
