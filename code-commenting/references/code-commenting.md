name: code-commenting-guidelines
description: >
  Provides guidance for writing and reviewing high-quality code comments in any programming language. Use this skill whenever code comments are added, updated, or reviewed. It ensures comments are minimal, clear, and focused on explaining intent ("why") rather than describing code ("what"), while following documentation standards (docstrings, JSDoc, etc.), keeping comments up-to-date, and avoiding common pitfalls (stale or redundant comments, commented-out code, secrets).
High-Quality Code Commenting
Introduction
Code is read far more often than it is written, so it must be easily understood by future readers. Comments are an important tool for communicating intent and context to other developers (and your future self), but they should supplement clear code – not replace it. As MIT’s Hal Abelson famously said, “Programs must be written for people to read, and only incidentally for machines to execute.” Writing clean, self-explanatory code is the first priority, and comments serve as additional clues for aspects that code alone cannot convey[1][2]. In fact, excessive or low-quality comments can be harmful: a bad comment is worse than no comment at all[2] if it misleads or contradicts the code. The goal is to write minimal but meaningful comments that explain the “why” behind the code, while ensuring code remains as self-documenting as possible.
Write Self-Documenting Code First
Strive to make your code self-explanatory through clear naming, simple design, and refactoring, so that little commenting is needed. Don't use comments as a crutch for confusing code. If a piece of code is hard to understand, consider rewriting it or simplifying it rather than layering on comments[3][4]. As Brian Kernighan and P.J. Plauger advised, “Don't comment bad code — rewrite it.”[5]. Senior developers tend to rely less on comments and more on clean code to “tell the story,” whereas junior developers often add commentary instead of improving the code itself[6]. Always assume that no comments will be available, and write your code in the simplest, most straightforward way possible[7]. Only after you've exhausted ways to make the code itself clearer should you add a comment for clarification. This practice ensures that comments serve a high-value purpose (explaining intent or context) rather than compensating for sloppy code. Remember that comments are for humans, so writing good comments requires good writing skills – be clear, concise, and precise in your language[8].
Comment on Intent and Rationale (“Why”), Not Descriptions of Code (“What”)
Use comments to explain why the code exists or does something in a certain way, not to restate what the code is doing. The code itself should illustrate what is happening; the comment should reveal why it’s done that way[9]. In other words: Code tells you how, comments tell you why[10]. This is the most crucial rule of commenting. Comments that only describe the implementation (“increment the counter”, “open the file”) add no value and create noise. Instead, focus on the intent, design decisions, assumptions, or constraints that are not obvious from the code. For example, instead of:
i += 1  # increment i by one (BAD)
use a comment that provides context or reason:
i += 1  # Increment to skip the current element (compensate for removed item) (GOOD)
In the “good” example above, the comment doesn’t repeat the operation but explains why we add one (perhaps due to a removed item affecting an index)[11]. Always ask yourself: Would a reader understand this comment is telling them something the code itself doesn’t? If not, the comment is likely unnecessary. Favor explaining the goal, intent, or any surprising behavior. For instance, rather than // removes stuff from newsletter (which is merely restating a function name), a better comment might note: // Remove user to avoid double-sending if they unsubscribed (avoid issue with external API) – this explains the motivation and links it to a higher-level concern[12].
Include information that isn’t in the code: the “why here and not elsewhere,” reasons for choosing an approach, references to requirements or bug IDs, or any context a future maintainer would need. A useful heuristic is: if a piece of code might prompt a reviewer to ask “why did they do it this way?”, that’s a good place for a clarifying comment.
When (and When Not) to Comment
Comment when it adds clarity or insight. Situations that typically warrant comments include:
•	Complex or non-obvious code logic: If you have code that uses an algorithm or bitwise operation that isn’t immediately clear, a comment explaining the approach or goal can help. This is especially true for clever optimizations or complex business logic that a reader might not grasp quickly.
•	Important intent or decision: Explain the reasoning behind a particular solution, especially if it’s not the most straightforward approach. For example, “// Using algorithm X for better worst-case performance” or “// Using library Y due to compatibility requirement” provides rationale.
•	Workarounds and hacks: If the code implements a workaround for a known issue or an odd hack, document why that is necessary (and possibly reference external sources or tickets). For example: // WORKAROUND: Applied delay because API rate limits cause failure without it (see issue #123).
•	Edge cases and constraints: If the code handles a special case or follows a non-obvious constraint (perhaps from an external system or spec), comment on that. For example, “// Handle leap year logic due to regulatory requirement” or “// Must sort input because downstream service expects ordered data.”
•	Interoperability or system assumptions: Note interactions with other components or systems, such as “// The external API returns null for unknown IDs, so we must check for null here.” These help maintainers understand dependencies.
•	Public APIs and exposed interfaces: (Discussed more below) Public functions, classes, or modules that will be used by others deserve documentation of their behavior, parameters, and usage examples. Use docstrings or structured comments in these cases.
On the other hand, avoid commenting in these scenarios:
•	Restating the code: Do not write comments that literally describe line-by-line what the code does. Example (to avoid): x = x * 2; // multiply x by 2. This is redundant[13] and adds no insight. The reader can see that from the code itself.
•	Explaining obvious language features: Don’t comment things like // loop through list on a for loop or // define class on a class definition – the structure makes it clear.
•	Compensating for poor naming: If you find yourself needing a comment to explain a variable or function, consider renaming it to something more descriptive instead. For instance, instead of:

 	int n; // number of users  
 	prefer:

 	int userCount; // (no comment needed)  
 	Good naming and code structure often eliminate the need for a comment. If a comment is only explaining the purpose of a poorly named variable, fix the name.
•	Every single line or trivial logic: It’s neither necessary nor wise to comment every line. Excessive commentary can clutter the code and actually decrease readability[14]. Save comments for the non-trivial parts.
•	Repeating documentation: If a function name is getUserEmail(), don’t add a comment “// gets the user email.” That’s clear from the name. Comments should add information beyond what identifiers already convey.
Remember that unnecessary comments introduce maintenance cost and noise[15]. Each comment should earn its place by providing value to future readers. When reviewing code, ask “Does this comment tell me anything the code itself doesn’t?” and “Is this comment needed if the code were clean and well-named?” If the answer is no, that comment may be a candidate for removal or avoidance.
Documenting Public APIs and Important Interfaces
Public-facing code elements (APIs, libraries, modules, classes, or functions intended for use by others) should have doc comments or docstrings that describe how to use them. In many languages, these are formal documentation comments that can be extracted by tools. For example:
•	Python: Use triple-quoted docstrings for public modules, functions, classes, and methods. The docstring should describe the function’s purpose, explain parameters and return values, and mention any exceptions or side effects. Python’s guidelines state that “Write docstrings for all public modules, functions, classes, and methods”[16]. Non-public (internal) functions don’t strictly need docstrings, but you should still include a brief comment after the def line explaining what the function does if it's not obvious[16].
•	JavaScript/TypeScript: Use JSDoc comments (/** ... */) for functions, classes, and modules that are part of your public interface or library. Include @param tags for each parameter (with type and description) and @returns (or @return) for the return value, as well as other relevant tags (@throws for errors, @example for usage, etc.). This provides clear usage instructions and can be used by IDEs or documentation generators. For instance:

 	/**
 * Calculate the Fibonacci number at position n.
 * @param {number} n - The position in the Fibonacci sequence.
 * @returns {number} The Fibonacci number.
 */
function fibonacci(n) { ... }
•	Java/C#: Use Javadoc or XML documentation comments for public classes and methods. Similar to JSDoc, include descriptions for parameters, return values, and any exceptions (e.g., @param, @return, @throws tags in Javadoc, or <param>, <returns>, <exception> in C# XML comments).
•	C/C++: Use Doxygen-style comments (/** ... */ or ///) for functions, classes, and modules in libraries or core code. Document the purpose, parameters (@param tags), return (@return), and any important notes. This allows generating reference docs for your API.
In all cases, write the documentation comments in a consistent format appropriate for your project (reStructuredText, Google style, JSDoc, etc.). Follow any style guide conventions for comment formatting (for example, PEP 257 for Python docstrings, which prescribes how to structure one-line vs. multi-line docstrings). Document the purpose of the function or class (the “what it does and why it’s useful”), not line-by-line implementation details. Think of it as explaining to a user of your code how to call it and what to expect. Additionally, if the function has any important constraints or behaviors, mention them (e.g., “Note: This function is not thread-safe,” or “This must be called after initialization”).
Remember to maintain these API comments as the code evolves. Outdated documentation is especially troublesome in public APIs because users might rely on it. Writing thorough doc comments not only helps users of the code but also forces you as the author to clarify your design.
Explaining Complex Code, Patterns, and Constants
Some parts of code inherently benefit from comments because their intent isn’t obvious from implementation alone. Use comments to demystify complex or non-intuitive code. Examples include:
•	Regular expressions: Regex patterns are notoriously hard to decipher. If you include a complex regex in code, add a comment explaining what it matches or how it works. For example:

 	const pattern = /^(\d{3}-\d{2}-\d{4})$/; // Regex for SSN format (AAA-GG-SSSS)
 	Here the comment describes the pattern in plain language. For very long regexes, you might break them into explanatory fragments or at least provide an overview (“// This pattern validates the input against XYZ format”).
•	Mathematical or algorithmic code: If you implement a non-trivial algorithm (like a particular sorting method, cryptographic function, or an optimization), include a comment about the approach and why this algorithm is used. Mention complexity if relevant or reference the source (e.g., “// Using Fisher–Yates shuffle for unbiased random permutation” or “// Bellman-Ford algorithm used due to possibility of negative weights”). If the algorithm comes from a paper or website, consider citing it in the comment or providing a URL[17].
•	Unusual or “magic” constants: If a constant value is critical (like a specific number, timeout, or bitmask) and its choice isn’t obvious, explain it. For instance:

 	public static final int RETRY_INTERVAL_MS = 15000; // 15 seconds - chosen to match server timeout
 	The comment above clarifies why 15000ms was chosen (perhaps aligning with server-side behavior). Without such a note, a reader might wonder if the value is arbitrary or could be changed. Similarly, any “magic numbers” or strings in code should either be given meaningful names or commented to explain their origin (e.g., “// Port 443 is used because the service requires HTTPS”).
•	Configuration or environment-specific quirks: If code behaves differently based on environment or configuration, and that logic is not obvious, add a comment. For example: “// In production mode, skip caching due to memory constraints” or “// If FEATURE_X is enabled, we bypass this validation (temporary feature flag)”. This informs maintainers of context beyond the code.
•	Legacy or deprecated patterns: If you must use an outdated approach or maintain legacy code for compatibility, document why. For example: “// Using old API v1 response format for backward compatibility with clients” tells the reader that the odd code is intentional and shouldn’t be “fixed” without considering compatibility.
In summary, comments should illuminate the code’s intent especially in complex areas that a reader might find confusing. Google’s code review guide notes that usually “comments are useful when they explain why code exists or does something in a non-obvious way,” with exceptions like regex or complex algorithms where even the “what” may need explanation[18]. Use your judgment: if the code is straightforward and well-named, no comment is needed; if it’s intricate or counter-intuitive, a comment likely helps.
Comment Style and Best Practices
Adopt consistent commenting style and format across the codebase, so comments are easy to read and maintain. Here are some style guidelines and best practices:
•	Clarity and brevity: Write in clear, simple language. Full sentences are often recommended for clarity[19], especially in explanatory or block comments. However, they can be sentence fragments if they remain clear. Aim for concise comments that get to the point, but not so terse that meaning is lost.
•	Proper grammar and punctuation: Treat comments as professional communication. Begin comments with a capital letter (unless referencing an identifier) and end full-sentence comments with a period[19]. Correct spelling and punctuation help readability. Avoid slang or overly informal language; while some humor is acceptable in moderation, remember that not everyone may share the context or find it funny. Clarity trumps wit in comments.
•	Comment format:
•	Inline comments: Use them sparingly and only for short notes on tricky logic. In many languages (C, C++, Java, JavaScript, etc.), inline comments use //. Ensure at least two spaces separate the code and the inline comment[20], and keep them short. In Python, inline comments start with #. Example:

 	value *= scaleFactor;  // Adjust value based on scaling law (non-linear)
•	Block comments: For longer explanations, especially those that span multiple lines, use block comments (for instance, /* ... */ in C-style languages, or multiple # lines in Python). Indent block comments to the same level as the code they describe[21]. If a block comment has multiple paragraphs, separate them with a single # or * on a line by itself (depending on language convention).
•	Documentation comments: Use the language’s standard (as discussed in the Public APIs section) for any formal documentation blocks. E.g., JSDoc comments start with /** and often each line with *; Python docstrings use triple quotes. Maintain whatever structure (like parameter lists or summaries) is expected by those formats.
•	Focus on one topic per comment: Don’t try to explain multiple unrelated things in one comment block. It’s better to separate concerns – e.g., one comment for why a function uses a certain approach, and a separate comment elsewhere for an unrelated configuration detail. This modularity helps when updating comments.
•	Keep line length reasonable: For readability, avoid very long comment lines. Many style guides suggest wrapping comments at 72 or 80 characters[22]. This prevents readers from having to scroll horizontally to read a comment.
•	Use a consistent tone and person: Typically, write comments in third-person descriptive voice or imperative mood. For example, “Initializes the cache on startup” or “Do not call this after dispose()”. Avoid addressing the reader as “you” in most cases; simply describe the code or give instructions. Some projects allow a more conversational tone in comments, but consistency is key.
•	Mark TODOs and notes clearly: If you are including a note to other developers, prefacing a comment with a clear tag (like “TODO”, “FIXME”, “NOTE”) can signal its purpose (we cover these in the next section). Many teams also include their name or initials and date for context, e.g., // TODO(alice): Improve performance here. Only do this if it's part of your team conventions (some teams prefer just a ticket number or nothing personal at all).
•	Avoid unnecessary metadata: In older practices, some would include author names, dates, or history in comments at the top of files or functions. This is usually redundant with version control systems like Git. Modern best practice is to rely on git blame or commit history for authorship and changes, rather than manually maintaining headers in comments. (One exception might be copyright or license notices, which are often required in file headers.)
By following a uniform style, you make comments easier to manage and review. If your project has a style guide, follow its commenting section (for example, Google’s style guides have specific comment rules, and PEP 8 for Python gives detailed comment formatting guidelines[19][11]). Consistency helps everyone know what to expect in comments and reduces ambiguity.
Using Annotations and Tags (TODO, FIXME, etc.)
Special annotation comments flag work that remains or known issues in the code. Common tags include TODO, FIXME, BUG, HACK, and PERF (or OPTIMIZE). These are useful to draw attention to areas that need improvement or revisit. Best practices for using such tags:
•	Use them sparingly and purposefully: A TODO or FIXME should denote something important – e.g., a feature to implement, a temporary workaround to replace, a performance issue to optimize, or a bug to fix. Don’t tag every minor desire; use them for notable technical debt or pending tasks.
•	Follow a standard format: Many teams use a format like TODO([owner]): [description]. For example:

 	// TODO(john): Support localization for this error message.
 	Including a name or identifier can indicate who plans to address it (or who to ask for details). Alternatively, referencing an issue tracker ID is very useful:

 	// TODO: refactor this function (tracking issue #1234)
 	This way, the context or discussion can be found in the issue tracker. A consistent format (e.g., “TODO:” in all caps) allows IDEs or linters to automatically collect these.
•	Be clear about the task or problem: After the tag, concisely state what is remaining or what’s wrong. “FIXME: handle null inputs – crashes if value is null” is much more informative than just “FIXME: bug here.” If it’s something to implement later, describe the intended improvement. If it’s a hack, explain why it’s a hack and what a proper solution would entail. For example:

 	// HACK: Using static credentials here due to missing auth module. Remove once auth is implemented.
•	Avoid leaving them indefinitely: Annotations like TODO are meant to be addressed. Over time, periodically review them. During code reviews, if you see a TODO that is critical (or perhaps already completed but not removed), call it out. One approach is to create an issue in the tracker for every TODO, so they are not forgotten. In fact, a best practice is to add an issue number in the comment and update/close that issue when done[23]. Tools or scripts can help ensure no TODOs slip into production without follow-up.
•	Use appropriate tags: Some distinctions:
•	TODO – something that is not done yet, but planned.
•	FIXME or BUG – a known incorrect behavior that needs fixing.
•	PERF or OPTIMIZE – an area that works but could be made more efficient.
•	HACK – an ugly but necessary code chunk that should be refactored later.
•	NOTE – just an informational comment (often used to highlight an important detail that isn’t a to-do, e.g., NOTE: Module X is deprecated and will be removed in v2.0).
•	UNDONE – sometimes used to indicate a reversal or a feature removed.
Ensure your team agrees on which tags to use and their meaning, as well as the process for resolving them.
Many IDEs and code editors will highlight these tags or list them in a “Tasks” pane. Leverage that to keep track. Also consider that leaving too many TODOs in code can be a sign of accumulating technical debt – try to address them or at least document them in a tracking system. When a TODO or FIXME is resolved, remove the comment (and any associated tag) to avoid confusion. The code’s version history will contain the context if needed, so an old TODO comment hanging around is just clutter.
Keeping Comments Current and Accurate
One of the worst pitfalls is a stale comment – one that was true at some point, but the code changed and the comment did not. Always update or remove comments that no longer reflect reality. Remember: comments that contradict the code are worse than no comments at all[24]. They mislead the reader and can cause more bugs. Therefore:
•	Update comments with code changes: Make it a habit that whenever you modify code, you review nearby comments to see if they need updating too. If you change how a function works, adjust its docstring or comments accordingly. If a comment said “we must do X because of Y” and Y is no longer relevant, change or delete that comment. Keeping comments in sync with code is as important as writing them in the first place.
•	Align comments with tests: If there are comments describing behavior that is also enforced by tests (or vice versa), ensure they don’t conflict. For instance, if a comment says “input can never be null” but you later add tests that allow null, that comment is now wrong. Either the code assumption changed or the comment needs correction. Use your test cases as documentation as well – sometimes tests illustrate usage better than comments. If an edge case is handled in code and described in a comment, ideally your tests cover it too.
•	Remove obsolete comments: After a refactor or a significant change, hunt for any comments that reference the old way. It’s better to have no comment than one that refers to code that no longer exists or conditions that are no longer true. Obsolete comments are often lurking after big redesigns or when a “TODO” got done but the comment remained. If a comment refers to a bug that has since been fixed or a temporary workaround that is no longer needed, eliminate it or rewrite it in past tense to indicate historical context if truly necessary (though usually, you can remove it entirely and rely on version control history if needed).
•	Avoid duplicating information: If documentation (like a README or external docs) has been updated with a new behavior, ensure code comments reflect that too (or vice versa). It’s easy for inconsistencies to arise when the same information is documented in multiple places. Wherever possible, have a single source of truth for a piece of information. For example, if an API is thoroughly described in external documentation, keep code comments brief to avoid having to update it in two places — or better, generate one from the other.
•	Code reviews for comments: When reviewing others’ code, pay attention to comments just as you do to code. Check that comments are accurate, necessary, and clear. Don’t hesitate to question a comment that seems off or redundant. Also verify that any added comments meet the style and guidelines (e.g., not explaining the obvious, not containing personal jargon, etc.). A code review checklist should include commenting: Are comments correct and do they actually help understanding? Are there any comments that should be removed or added for clarity?[18].
•	English or team language: Write comments in a language understood by all project collaborators. If you work in an international team, that usually means English. It’s explicitly recommended in style guides: “Write your comments in English, unless you are 120% sure the code will never be read by people who don’t speak your language.”[25]. A comment that only some team members can read isn’t serving its purpose. Similarly, avoid localized idioms or cultural references that might confuse others. Keep comments universally understandable.
In summary, treat comments as part of the code that needs maintenance. Allocate time for updating comments during refactoring. An outdated comment is technical debt. As a rule, if you encounter a comment that is wrong or irrelevant, fix it or remove it – leaving it in place “just in case” will likely do more harm than good.
Avoiding Common Pitfalls in Comments
Be aware of classic comment anti-patterns and avoid them to maintain a clean and professional codebase:
•	Stale or incorrect comments: As emphasized, these are dangerous. They occur when code changes and comments don’t. Always remove or correct comments that are no longer accurate[24]. For instance, a comment saying “// using algorithm X for speed” when the code now uses a different algorithm is misleading. It’s better to have no comment than a wrong one. When you see such comments, update them immediately.
•	Redundant comments: Comments that repeat the code or add no new information (e.g., i = 0; // set i to 0) only clutter the code[26]. They can make reading harder by adding visual noise without benefit. Rely on the code for the “what” and use comments for the “why” or other insights.
•	Misleading or confusing comments: Sometimes comments are written with good intentions but end up confusing the reader. This can happen if the comment is vague, overly verbose, or uses unclear terminology. Ensure your comments are straightforward and unambiguous. If you find yourself puzzling over what a comment means, that’s a red flag. Simplify it.
•	Commented-out code: Do not leave large blocks of code commented out in the repository. This is a common anti-pattern – often developers comment out old code “just in case” or for reference. This leads to clutter and confusion. Other maintainers might waste time reading that dead code wondering if it’s important[27][28]. Version control already saves old code; you don’t need to preserve it in comments. Committing commented-out code is almost always a bad practice[29]. It can also become outdated quickly, as it won’t be kept in sync with changes. Instead, remove the code. If you truly think you might need it, keep it in a separate branch or mention in a comment that “X was tried and not used, see history at commit <hash>.” But even that is rarely needed. Many linters can be configured to flag large commented sections as warnings. In short, clean up commented-out code before merging – it makes the code cleaner and avoids distracting future readers[30].
•	Sensitive information in comments: Never put passwords, API keys, secret tokens, or other sensitive credentials in comments. It might be tempting to leave a helpful note like “// use API key 12345 for testing,” but remember that even commented-out secrets are still present in your source and version history. They can be extracted and lead to security breaches. Treat comments as publicly visible (especially in open source). Use proper secret management practices (environment variables, config files not in VCS) rather than embedding secrets in comments or code. Automated secret scanners will often flag secrets in comments, and many companies have pre-commit or pre-receive hooks to prevent this[31]. If you find an old comment with a secret, remove it and rotate the secret if it was exposed.
•	Offensive or inappropriate comments: It should go without saying, but keep comments professional and respectful. Do not include insults, profanity, or derogatory remarks about any person, technology, or otherwise. Not only can such comments create a hostile environment, they can also become public (e.g., if your code is open source, or leaks). Even in internal code, always assume your comments could be seen by others (managers, new hires, external partners). Keep the tone civil and focus on the technical aspect. For example, instead of writing “// Stupid hack begins here”, you might say “// TEMPORARY workaround: ...”.
•	Jokes and sarcasm: A lighthearted comment here or there is not forbidden – humor can humanize a codebase – but use caution. Humor is subjective and may not age well. More importantly, jokes should never replace actual explanation. A classic example: // Abandon all hope, ye who enter here is funny but doesn’t help the maintainer understand the code. If you include a joke, ensure there is a real comment explaining the code as well. Avoid sarcasm or irony that could be misinterpreted.
•	Localization issues: As mentioned, prefer English (or your team’s primary language) for comments. Also avoid mixing languages or using non-ASCII characters unnecessarily in comments; these can sometimes cause encoding issues or just confuse readers who don’t speak that language. One exception can be if the domain requires it (e.g., a code dealing with internationalization might have examples of messages in various languages – that can be acceptable if relevant). But generally, stick to one language for all comments.
By steering clear of these pitfalls, you ensure that comments remain a helpful resource rather than a source of technical debt or confusion. Always aim for comments that clarify, not cloud.
Leveraging Tools for Better Comments
Modern development tools can greatly assist in writing and maintaining high-quality comments:
•	IDE Support: Most IDEs/editors (Visual Studio Code, IntelliJ IDEA, PyCharm, VS, etc.) provide shortcuts or templates for commenting. For example, typing /** in many editors will auto-generate a JSDoc or Javadoc template with @param placeholders. Take advantage of these to ensure you don’t forget to document parameters or returns. Some IDEs also highlight mismatches (e.g., if you document a param that doesn’t exist or vice versa). Additionally, IDEs often show doc comments as tooltips or via intellisense – a good incentive to write helpful docs for your functions.
•	Linters and static analysis: Linters can enforce comment-related rules. For instance, ESLint has rules like [no-unused-vars with ignore for args to allow intentionally unused parameters to be documented] or plugins like eslint-plugin-jsdoc to ensure JSDoc comments are complete and well-formed. There are also linters to catch things like TODOs without task references, or flagged words. For Python, pylint can warn if a public function has no docstring, and flake8-docstrings can enforce PEP 257 conventions. Configure these tools to match your team’s standards (e.g., require docstrings on public functions, forbid overly long comments, etc.). This automation helps keep comment quality consistent.
•	Documentation generators: Tools like Doxygen, Sphinx (with reStructuredText in docstrings for Python), JSDoc or TypeDoc for JavaScript/TypeScript, JavaDoc for Java, or DocFX for .NET can convert your in-code comments into human-readable documentation websites or PDFs. Integrating these into your build or CI process can ensure you regularly validate that comments are up-to-date and formatted correctly. For example, running Sphinx can catch references in docstrings that don’t resolve, or JSDoc can warn about missing @param tags. Moreover, knowing that your comments will be published might encourage better writing.
•	Spell checkers: Typos in comments can reduce professionalism and sometimes change meaning. Consider using a spell-check tool on comments (some linters or IDE plugins can highlight misspelled words in comments without touching code). While technical terms or identifiers might be flagged, you can whitelist those. Ensuring correct spelling in comments is part of clear communication.
•	AI-assisted tools: With the advent of AI coding assistants, there are tools that can suggest comments or even automatically generate documentation from code. For example, GitHub Copilot or similar can propose docstring content based on the function implementation. While these can save time, always review and edit AI-generated comments for accuracy and clarity. They might get the intention wrong or be too verbose/concise. Use them as helpers, not authorities. Similarly, AI-based documentation analyzers might identify poorly commented areas or inconsistencies. Keep an eye on emerging tools that can analyze your codebase for documentation coverage or outdated comments.
•	Version control hooks: As mentioned, you can use git hooks or CI checks for certain comment patterns. For example, a pre-commit hook could warn if you are about to commit a line containing API_KEY or password in a comment (to catch secrets)[32]. A pre-receive hook on the git server could reject commits that introduce very large blocks of commented-out code (if that’s a policy) or ensure every TODO comment has an associated issue reference. There are community tools (like Yelp's detect-secrets[31]) and custom scripts to help with this. Setting these up can enforce best practices automatically.
•	Code review templates/checklists: Many teams use a checklist for reviewers that includes a section on documentation/comments. For instance: “✅ Comments: All public methods have appropriate doc comments. Comments are clear, necessary, and correct. No leftover commented-out code or TODOs without issues.” This reminds everyone in the review process to catch comment issues before they hit the main branch. Google’s own review guide explicitly asks reviewers to verify that comments are clear and necessary (and not explaining things that should be self-evident)[18]. Incorporate similar points in your team’s review norms.
By incorporating tooling, you reduce the manual effort required to maintain comment quality. Tools can catch many of the issues discussed (like outdated references, missing documentation, or sensitive info) and encourage a culture where commenting is done properly. Nonetheless, human judgment is irreplaceable for understanding context – so use tools to augment, not replace, careful writing and reviewing of comments.
Examples of Good and Bad Comments
Finally, let's illustrate some examples to solidify the principles:
Example 1: Redundant vs. Meaningful Comment

int total = items.size();  // get size of items (BAD)
Why bad? The comment simply restates what the code does (size() returns the size). It provides no new information. A better approach is either no comment (since the code is clear) or, if context is needed, something like:

int total = items.size();  // Total number of items to process (GOOD)
This comment is somewhat redundant too, but it at least ties the variable to a purpose (“to process”), which might be slightly beyond the obvious. In many cases, even that comment could be omitted if the variable name total or context implies it. The key is: don't repeat the how; only comment if it adds context or intent.
Example 2: Misleading Comment

# Using a binary search (BAD - actually not binary search)
result = linear_search(data, key)
What's wrong? The comment claims a binary search is used, but the code calls linear_search. This is an example of a comment that contradicts the code. It might be a leftover from an earlier implementation. The fix is to remove or correct the comment:

result = linear_search(data, key)
(No comment is actually needed here if the function name is clear. If we wanted a comment, it should correctly state why linear search is acceptable or if the data isn't sorted, etc. But a wrong comment is worse than none[24].)
Example 3: Documenting Intent (“Why”)

// BAD:
if (x < 0) {
    x = 0;  // if x is negative, set it to zero
}
The comment “set it to zero” is describing what the code does, which is obvious. It doesn’t explain the reason. Consider instead:

// If x is negative, clamp to 0 to avoid unexpected behavior downstream
if (x < 0) {
    x = 0;
}
Now the comment (GOOD) explains the intent: we "clamp" negative values to 0 to prevent issues later. It answers why we do this (to avoid unexpected downstream behavior). This is valuable information not evident from the code itself.
Example 4: Public API Documentation

def add_user(user_list, user):
    """Add a user to the list if not already present.

    Args:
        user_list (list[User]): The list of users.
        user (User): The user to add.

    Returns:
        bool: True if the user was added, False if they were already in the list.

    """
    if user in user_list:
        return False
    user_list.append(user)
    return True
In this Python example, the docstring clearly explains what the function does, its parameters, and return value. This is a good comment style for a public function. A bad comment for the same function would be something trivial like:

def add_user(user_list, user):
    # Adds user to list
    ...
This one-liner is inadequate for an API that others might use, because it doesn’t document the conditions or behavior (e.g., what if the user is already there?). Always prefer a complete docstring for such cases.
Example 5: Using TODO with context

// TODO [#99]: remove this hardcoded value when config service is available
const apiUrl = "https://api.example.com/v1/";
// ...
This is a good use of a TODO comment, because it indicates what needs to be done (remove hardcoded value), and even better, references issue #99 which presumably tracks implementing a config service. A bad TODO would be just:

// TODO: fix this
This provides no information on what needs fixing or why. Always be as specific as possible in TODO/FIXME notes and ideally tie them to a tracker.
Example 6: Commented-Out Code
Bad:

// int debug = 1;
// printf("Debug mode on\n");
process(data);
This is a chunk of commented code left in. It’s unclear if it should be removed or will be used later. It clutters the function. The good practice is to delete such lines once they are not needed, or use a debug flag that can be toggled without commenting code. If you absolutely must leave it (which is strongly discouraged), at least add a comment like // DEBUG PRINT (disabled) to explain, but preferably use version control instead of leaving this in source. The above should be:

process(data);
(No commented-out code at all.)
Each of these examples reinforces the guidelines: comments should add insight (not repeat code), remain accurate, explain intent, and avoid anti-patterns. Use them judiciously – not too many, but not too few that readers are lost. Aim for self-explanatory code supplemented by comments that provide rationale, context, and guidance. When in doubt, remember the core principle: “Code tells you how, comments tell you why.”[10]
________________________________________
[1] [2] [4] [5] [10] [14] [15] [23] [26] Best practices for writing code comments - Stack Overflow
https://stackoverflow.blog/2021/12/23/best-practices-for-writing-code-comments/
[3] [6] [7] [8] Coding Without Comments
https://blog.codinghorror.com/coding-without-comments/
[9] [12] [17] Good comments explain WHY, not WHAT, and 3 more rules on writing good comments - DEV Community
https://dev.to/andreasklinger/comments-explain-why-not-what-and-2-more-rules-on-writing-good-comments
[11] [13] [16] [19] [20] [21] [24] [25] PEP 8 – Style Guide for Python Code | peps.python.org
https://peps.python.org/pep-0008/
[18] What to look for in a code review | eng-practices
https://google.github.io/eng-practices/review/reviewer/looking-for.html
[22] PEP-8: Python Naming Conventions & Code Standards | DataCamp
https://www.datacamp.com/tutorial/pep8-tutorial-python-code
[27] [28] [29] [30] Please, don't commit commented out code
https://kentcdodds.com/blog/please-dont-commit-commented-out-code
[31] Best practices for managing & storing secrets like API keys and other credentials : r/programming
https://www.reddit.com/r/programming/comments/h7kmff/best_practices_for_managing_storing_secrets_like/
[32] Best Practices for Using GitHub Secrets - Part 1 - DEV Community
https://dev.to/pwd9000/best-practices-for-using-github-secrets-part-1-596f
