________________________________________
name: markdown-best-practices
description: >
Enforces consistent, high-quality Markdown formatting and structure across documentation (headings, lists, code blocks, links, images, tables, front matter metadata, line length, admonitions, etc.).
________________________________________
Markdown Best Practices
This skill outlines standards and best practices for writing Markdown documents in a consistent, high-quality way. It is language-agnostic and applies to general-purpose Markdown usage (README files, developer docs, static site generators like Docusaurus or Jekyll, notebooks, GitHub Flavored Markdown, Pandoc, etc.). By following these guidelines, teams can ensure their Markdown content is easy to read, maintain, and render across different platforms.
Headings
•	Use ATX-style headings (#): Always use the hash (#) syntax for headings rather than underlined headings (Setext style). Underlined headings with === or --- can be confusing and are harder to maintain[1]. For example, prefer ## Section Title over an underlined equivalent.
•	One top-level heading: Use a single H1 as the document title, and do not include more than one H1 in a file[2]. If the environment (e.g., a static site generator or docs platform) automatically generates an H1 from front matter, do not include an H1 in the Markdown content[3]. In such cases, start with an H2 for the first section.
•	Logical hierarchy: Maintain a logical heading order – increment one level at a time (H2 after H1, H3 under H2, etc.)[4]. Do not skip levels (e.g., jumping from H2 to H4) as this disrupts the document structure and can confuse readers and accessibility tools. Each subsection should use the next immediate heading level.
•	Spacing and alignment: Ensure there is a blank line above and below each heading for clarity[5]. Headings should not be indented with spaces or tabs – they must start at the beginning of the line to be recognized as headings[6]. Also, put a single space between the # and the heading text (e.g., use ## Heading not ##Heading)[7].
•	Title casing: Follow a consistent capitalization style for headings. A common choice is "sentence case," meaning only the first word and proper nouns are capitalized[8]. For example, use “## Installing the tool” instead of “## Installing The Tool.” Avoid ending headings with a period.
•	Unique, descriptive titles: Make each heading descriptive and unique[9]. This helps readers scan the content and also ensures that automatically generated anchor links (IDs) are unique. Instead of repetitive headings like “Overview” in multiple sections, include context (e.g., “Overview of Installation” and “Overview of Configuration” rather than two sections just named “Overview”)[10].
•	No trailing punctuation: Do not include a colon, period, or other punctuation at the end of a heading[11] (a question mark is generally acceptable if the heading is a question). Headings should be short phrases or nouns, not full sentences.
•	Example (Proper Structure):
---  <!-- Front matter with title is present, so no H1 needed in content -->
title: "Getting Started"
---  

## Introduction  
... (introductory text)

## Installation  
... (installation instructions)

### Linux Installation  
... (Linux-specific details)

### Windows Installation  
... (Windows-specific details)

## Usage  
... (usage information)
In the above example, the file has a title in YAML front matter which will be used as the H1. The content starts with H2 headings for sections, and H3 for subsections, following a logical hierarchy.
Lists
•	Bullet list style: Use a consistent marker for unordered lists. Common choices are the hyphen (-) or asterisk (*). Pick one and stick to it within a document[12]. For example, if you start a list with - Item 1, use - for all items at that level.
•	Numbered list style: For ordered lists, it’s recommended to use 1. for all items (known as "lazy" or "all 1s" numbering)[13]. Markdown will automatically render them in sequence. This makes maintenance easier if you add or remove items. For example:
1. First step  
1. Second step  
1. Third step  
will render as 1, 2, 3. In short lists that are unlikely to change, you may number them explicitly (1., 2., 3.) for readability[14], but it’s not required.
- Indentation for nesting: When creating nested lists (sub-items), indent by a consistent number of spaces. A 4-space indent (or equivalently 1 tab, if tabs are allowed) for each nesting level is a common standard[15]. This means:
- For a nested sub-list under a numbered list, indent the sub-list by 4 spaces (and continue to use 1. for each sub-item).
- For a nested sub-list under a bullet, indent 4 spaces and use the bullet marker.
Example:
1. First item in ordered list.  
    1. Sub-item in ordered list.  
    1. Another sub-item.  
1. Second item in ordered list.  

- Bullet list item  
    - Sub-item in bullet list  
        - Sub-sub-item in bullet list  
- Another top-level bullet  
In the above, each level is indented consistently by 4 spaces. This alignment also ensures that if an item’s text wraps to a new line, the wrapped lines align under the first character of the item above, improving readability[16].
- One space after markers: Put exactly one space after the list marker (bullet or number). For bullets, - item (not - item with multiple spaces). For numbered, 1. item (ensure a space after the period). Consistent spacing prevents parsing issues.
- Blank lines: Do not insert blank lines between list items, as that will end the list. If you need to put a separate paragraph or a code block inside a list item, indent it properly beneath the list item. Ensure a blank line before the start of a list if it should be separate from preceding text (to avoid it merging with a paragraph). Likewise, after a list, a blank line is recommended before continuing with a normal paragraph, to clearly mark the end of the list.
- Punctuation in lists: If list items are complete sentences, end each with a period (or appropriate punctuation). If they are fragments (short phrases), you can leave them without a terminal period[17]. Be consistent within a list: don’t mix sentences and fragments. Each bullet should form a coherent, parallel structure with the others[18]. For example, if one item is a full sentence, consider making all items full sentences for consistency, or break it into a sub-list or separate paragraph.
- Capitalization: Start each list item with a capital letter (especially if the introductory phrase before the list implies a continuation)[19]. For example: “Steps to install the tool: 1. Download the installer… 2. Run the installer…”. This is a style choice that improves readability.
Code Blocks and Inline Code
•	Inline code: Use backticks ` to denote inline code or command names within text. For example: Use the `--verbose` flag for more details. will render as: Use the --verbose flag for more details. Use inline code for short snippets, file names, and keys or values that appear within paragraphs[20]. This helps distinguish them from regular text.
•	Use inline code to escape Markdown interpretation when needed. If you have text that includes characters that Markdown might parse (like underscores or asterisks) or something that looks like a URL/email that you don’t want to auto-link, wrap it in backticks to ensure it stays literal[21]. For example: An example URL is `https://example.com/?q=\_test\_`.
•	Fenced code blocks: For blocks of code (multiple lines or more complex code), use fenced code blocks with triple backticks. Always put the triple backticks on their own line before and after the code. We strongly recommend fenced code blocks over indented code blocks[22]. Fenced blocks are clearer and have fewer pitfalls (you can specify language, and there’s no ambiguity about when the code block ends)[23].
•	Language specifier: Immediately after the opening backticks, include the language name (or a common alias) for syntax highlighting[24]. For example:
 	```python
def hello(name):
    print(f"Hello, {name}!")
```
 	This will syntax-highlight the code as Python. Use valid language identifiers (e.g., js or javascript, bash, json, html, etc., as appropriate). If the language is unknown or you want no highlighting, you can omit it or use text. Explicitly declaring the language is best practice[24], as it prevents incorrect auto-detection.
- Blank lines: It’s recommended to put a blank line before and after a code block (unless it’s immediately at the start or end of a list item or document) for readability[25]. This ensures the code block is separated from surrounding text.
- No trailing whitespace or extra indentation: Inside code blocks, keep the code exactly as-is (preserve indentation that is meaningful within the code). However, avoid adding trailing spaces on code lines unless absolutely required (some languages use two spaces at end of line, e.g., in Markdown itself for line break). Trailing whitespace in code blocks can be ignored by some linters, but it’s cleaner to remove it[26][27].
- Indented code blocks: Technically, you can also create code blocks by indenting text by 4 spaces, but this is error-prone and not recommended. Indented blocks cannot specify language and may be misinterpreted in some contexts[23]. Always prefer fenced code blocks for clarity and portability.
- Command-line prompts: If showing command-line examples, consider omitting the leading $ or C:\> prompt in code blocks unless you need to distinguish between commands and output. For example, use:

 	ls -la  
git commit -m "Message"  
 	instead of including a literal $ before each command, as that makes copy-pasting harder[28]. If you show sample output, you can include those lines without the prompt or use a separate block with an explanation.
- Long lines in code: If a code line is very long (and not crucial to show in entirety), consider splitting it for readability, or using a horizontal scroll (some renderers do this automatically). However, avoid inserting invisible breaks that might alter the code’s meaning. Usually it’s okay to let code blocks have longer lines since they often represent exact text that shouldn’t be re-wrapped arbitrarily[29][30].
- Escape backticks: If you need to include triple backticks in a code block (rare, except when documenting Markdown itself), you can use four backticks to fence the code or indent the inner triple backticks by a space to prevent termination of the outer block.
Links and Images
•	Link syntax: Use the standard Markdown syntax for hyperlinks: [link text](target-url "Optional Title"). The link text should be descriptive – avoid generic text like “click here” or “link”[31]. Instead, link a meaningful phrase. For example: “See the Installation Guide for more information” is better than “See the guide here.” Descriptive link text improves readability and accessibility[32].
•	If the URL itself is the thing you want to show (e.g., a bare URL), you can just paste the URL and many renderers will auto-link it, but it’s usually better to embed it under a descriptive text.
•	Reference-style links: For very long URLs or frequently repeated links, consider using reference-style links to keep the text tidy[33][34]. For example:
 	Refer to the [research paper][paper] for detailed benchmarks.

... (later in document) ...

[paper]: http://example.com/very/long/url/for_the_research_paper.pdf
 	This separates the long URL from the prose. Reference links can also be helpful in tables where inline URLs would clutter the content[35][36]. Place the reference definitions either at the bottom of the document or at the end of the section where they’re used[37] (to make maintenance easier).
- Relative vs absolute links: When linking to other docs or images in the same repository or site, use relative paths if possible (so the links work in any fork or branch). For example, [User Guide](../guide.md) or [Intro](./README.md). However, avoid complex relative paths with ../ that jump too many directories, as they can become brittle[38]. If the platform supports it, you might use root-relative paths (e.g., (/docs/guide.md) from the site root). Ensure the link will resolve in the context where the file is viewed (GitHub vs a published site might have different base paths).
- Images: Use the syntax ![alt text](path/to/image.png "Optional Title") to embed images. Always provide meaningful alt text inside the [] to describe the image for readers who can’t see it[39]. The alt text should convey the purpose of the image (e.g., ![Architecture diagram showing component interactions](./images/architecture.png)). If the image is purely decorative or explained in surrounding text, you can use a blank alt text (![](image.png)), but generally err on the side of descriptive alt text for accessibility[39].
- Image paths: Prefer using relative paths to images stored in the repository. This ensures they work offline or on GitHub. For example, store images in a folder (like images/ or similar) and reference them relatively: ![UI screenshot](../images/setup_screen.png). Avoid linking to external images (hotlinking) as it can lead to broken images if the external source changes, and may pose privacy/security issues.
- Captions: Markdown doesn’t have a native image caption syntax. If a caption is needed, you can simply add a description below the image in italics or as a regular paragraph. Some static site generators allow adding a caption via HTML or extensions, but in pure Markdown, treat it as normal text.
- Image size: Markdown doesn’t directly support specifying image dimensions. If an image is too large, consider resizing it before committing, or use HTML <img> tag with width/height attributes (as a last resort). But in general, try to keep image file sizes reasonable and use common formats (PNG, JPEG, or SVG for vector).
- Links in images: If an image should be clickable (linking to something), you can nest the image syntax in a link: [![alt text](image.png)](target-url). Use this sparingly (for example, badge images linking to a website). Ensure the alt text still describes the image’s content or purpose.
- Titles for links/images: The title attribute in Markdown ("Optional Title" in the parentheses) is optional. It shows as a tooltip on hover in some browsers. You can use it to add additional context, but keep it short. It’s not required, and many skip it. For images, the title text might be displayed as a caption in some renderers (e.g., some static site generators), but that’s not standard.
- Avoiding dead links: Periodically verify that external links are still valid (they can break over time). For internal links, if files are moved or renamed, update the links accordingly. If using a static site generator, consider enabling a link checker or using a CI tool to catch broken links.
Tables
•	When to use tables: Use tables to organize data that makes sense in a grid or matrix form – typically when you need to present comparable attributes for several items. Do not use tables for layout purposes or for content that is better presented as a list or series of paragraphs[40][41]. If your table has a lot of text in cells, or many empty cells, consider restructuring the information (often a list or subsections might work better)[42][43].
•	Markdown table syntax: A basic table has a header row and separator line, then rows of data. Use pipes | to separate columns. For example:
| Feature          | Description                      |  
| ---------------- | -------------------------------- |  
| Speed            | Fast, with an average of 20 ops/sec. |  
| Portability      | Runs on Windows, Linux, and macOS. |  
The second line of dashes and pipes is required to define the header. You can use one or more dashes for each column, and colons : to indicate alignment. In the above example, the columns will default to left alignment.
- Column alignment: You can control text alignment in columns by placing colons in the separator line:
- :--- (colon on left) for left-align,
- :---: (colons on both ends) for center-align,
- ---: (colon on right) for right-align.
Example: | Column 1 | Column 2 | Column 3 | with a separator line |:--- | :--: | ---:| will make Column 1 left-aligned, Column 2 centered, Column 3 right-aligned. This is purely for formatting; it doesn’t affect the content meaning, but improves readability especially with numbers (which are often right-aligned).
- Keep table text concise: Table cells should ideally contain short phrases or data. Avoid long paragraphs in a table cell. If you find yourself needing multiple sentences or a list inside a table cell, that’s a sign the table might not be the best format. Long text in tables can make the table hard to read and may not wrap well in plain text. Consider extracting that text outside the table or summarizing it.
- Avoid too many columns: Wide tables are hard to read on narrow screens and in plain text form. If you have more than ~4-5 columns, consider if the data can be split into multiple tables or presented differently. In documentation on the web, wide tables may require horizontal scrolling. If using a static site generator or HTML preview, ensure it can handle wide tables gracefully (some add scrollbars).
- No line breaks in cells: Standard Markdown does not support newline characters within table cells (they’ll break the table). Some extended Markdown flavors do, but it’s not universal. If you need a line break in a cell, you might try using <br> HTML tag, but that’s a bit of a hack. It’s usually better to keep content on one line per cell.
- Consistent formatting: It’s not required, but for readability of the raw markdown, you can pad cells so the pipes align (as in the example above). Many editors or linters (like markdownlint) have autofixers to align tables. This makes the raw text table easier to read and edit. However, adding or removing content can require re-adjusting spaces. Whether you align the pipes or not, the rendered output will be the same.
- Use reference links in tables: If you have hyperlinks in table cells, consider using reference-style links to keep the table neat[33][36]. For example, instead of [Project](https://example.com/project) in a cell (which could be very long), use [Project][proj] in the cell, and define [proj]: https://example.com/project elsewhere. This way the table remains clean.
- Header row: Always include a header row to label the columns, even if the content might be self-evident. It helps with readability and is required for some converters to recognize it as a table. If a header doesn’t apply (for instance, if the first column is just labels and second column values), you can use an empty header name but still include the separator line. For accessibility, header cells (<th>) allow screen readers to contextualize the cells.
- Example:
| Item      | Quantity | Price  |
| :-------- | -------: | :----: |
| Apples    |      4   | $3.50  |
| Oranges   |     10   | $5.00  |
In this example, “Item” is left-aligned, “Quantity” is right-aligned (notice the number 10 is right-aligned under the header), and “Price” is center-aligned.
Front Matter (YAML Metadata)
Many documentation pages and blog posts include a YAML “front matter” at the top of the file. This is a section delineated by triple-dashed lines --- that contains metadata about the page. It is not displayed in the output, but is used by static site generators (Jekyll, Hugo, Docusaurus, etc.) or tooling to set titles, dates, categories, etc.
•	Format: The front matter block must be at the very top of the file (no blank lines or content before the opening ---)[44]. It starts with a line containing --- and ends with a matching --- line (sometimes ... is used in certain systems, but --- is most common). Between these lines, you have YAML formatted key-value pairs. For example:
---
title: "My Project Overview"
description: "An introduction to the project and its components."
author: Jane Doe
date: 2026-02-03
tags:
  - Markdown
  - Style Guide
draft: false
---
Each key is followed by a colon and a space, then the value. Strings can be in quotes (especially if they contain special characters or :) but don’t have to be if they’re simple words. Arrays (lists) are indicated by indentation and hyphens (as shown for tags above). Boolean values are typically lowercase true/false.
- Required fields: The required front matter fields depend on the site or usage:
- For documentation pages (e.g., Docusaurus docs, GitHub Docs), a title is usually required[45]. This often serves as the page’s H1 heading and the browser title. Always include a title field to identify the page. If the site generates the H1 from this, do not duplicate the title as a manual # Heading in the content[3].
- Blog posts typically require at least title and a date (and often an author or authors). The date might be used for sorting and display. Format the date in ISO format YYYY-MM-DD (some systems also accept a time). Example: date: 2026-02-03.
- Some static site generators use tags or categories to group content. Provide these if relevant as a YAML list or comma-separated string (check the specific format for your generator)[46][47].
- Other common fields: description (a one-line summary for SEO or previews), slug (to define URL explicitly), layout (for Jekyll, to specify which template to use)[48], draft (to mark a piece as not published), sidebar_position or sidebar_label (for Docusaurus to organize docs)[49], etc. Refer to your platform’s documentation for all available front matter options (for example, Docusaurus and Jekyll have extensive lists).
- Consistency: Use the same ordering of keys for similar documents if possible (e.g., always put title first, then description, then date, etc., in a blog). This makes it easier to scan. Keys are usually all lowercase. Indentation in YAML is critical for nested structures (always use spaces, not tabs, in YAML).
- Quoting: If a value contains special characters, starts with a number sign or other YAML-reserved characters, or has leading/trailing spaces, put it in quotes. When in doubt, quoting strings (especially for titles that have punctuation) is safe. Use double quotes " unless you specifically need single quotes. In the example above, the title is in quotes because it contains spaces and capital letters (not strictly necessary, but a good habit).
- No BOM or stray characters: Ensure there’s no Unicode BOM at the start of the file and no stray characters before the opening ---. This can cause the parser to fail[50]. If using an editor, ensure it’s set to UTF-8 without BOM if possible.
- Effects of front matter: Many tools will treat the presence of front matter as a signal to process the file. For example, GitHub Pages (Jekyll) will only process Markdown files as a Jekyll page if they have front matter. If you want a file to be processed but have no metadata to add, you can include an empty front matter (--- on a line, then another --- on the next line)[51]. This is rarely needed except in advanced scenarios.
- Front matter vs content: Remember, content in the front matter (like the title) is typically not duplicated in the body. So you wouldn’t write a # Title in Markdown if your front matter has title: Title. Many generators will automatically insert the title as an H1 in the rendered page[45]. Likewise, a description in front matter might populate a meta description tag, but if you want that description visible on the page, you should also include it in the content (perhaps as part of an intro paragraph). Know what each field does in your specific context.
- Example: Here is a minimal front matter for a Docusaurus doc page and a Jekyll blog post:
Docusaurus Doc (guide.md):

---
title: "Using the Guide"
sidebar_label: "Guide"
description: "How to use the guide effectively."
---  
(No H1 in content; the content should start with an introduction or an H2.)
Jekyll Blog Post (_posts/2026-02-03-new-features.md):

---
layout: post
title: "New Features in Our Product"
date: 2026-02-03
categories: [Product, Release]
tags: [release, features, 2026]
author: jdoe
---
(The blog post content would typically start with the first paragraph of the post.)
In the Jekyll example, the filename’s date and title also often determine the URL, but the title in front matter is what’s displayed as the heading on the blog page. The layout: post tells Jekyll to use the post template, and categories/tags help with organization.
Admonitions (Notes, Warnings, Tips)
Admonitions (also known as callouts or alerts) are specially formatted blocks that draw attention to important information, such as notes, tips, warnings, or important reminders. Different Markdown flavors implement these differently:
•	GitHub Flavored Markdown (Alerts): GitHub now supports an alert/admonition extension. You create an admonition by using a blockquote that starts with [!KEY] where KEY is one of NOTE, TIP, IMPORTANT, WARNING, or CAUTION[52]. The syntax looks like:
> [!NOTE]
> **Note:** This is a note admonition. It highlights general information.
The first line inside the blockquote has the [!KEY] tag, and subsequent lines (each prefixed with >) contain the content. GitHub will render this with an icon and distinctive styling for the type. For example, [!WARNING] will show a warning icon and highlight, [!TIP] might show a lightbulb icon, etc. This works on GitHub’s web interface and GitHub’s Markdown API (when using GFM mode)[53]. If you preview the raw Markdown elsewhere (like VS Code preview or older tools), you might just see the blockquote with the literal [!NOTE] text, so be aware it’s an extension.
- Docusaurus and others (Triple-colon syntax): Some documentation frameworks like Docusaurus support admonitions using a triple colon syntax. For example:
:::note  
**Note:** This is a note in Docusaurus.  
:::  
The keyword after the colons can be note, tip, info, caution, danger, etc., corresponding to different styles[54][55]. You can optionally add a custom title after the keyword, like :::note My Custom Title to override the default "Note" heading. The content of the admonition is between the start and end ::: lines. Remember to put an empty line after the opening :::note and before the closing ::: to avoid formatting issues[56].
- MkDocs (Triple bang syntax): Another common syntax (used by MkDocs with the Material theme via the pymdownx.superfences extension) uses !!! note or !!! warning etc., followed by indented content. For example:
!!! warning "Low Disk Space"
    Make sure you have at least 1 GB of free space before proceeding.
This is specific to certain Markdown extensions and won’t work in standard Markdown.
- Plain blockquotes: If your environment does not support any special admonition syntax, you can simulate an admonition by using a blockquote with bolded leading text. For example:
> **Note:** Remember to restart the server after making these changes.
This won’t have a fancy colored box or icon by default, but it conveys the intent. This approach is the most compatible fallback – in fact, before native support, many people used blockquotes for callouts, and it degrades gracefully (the content is still readable even if not specially styled)[57]. Some screen readers might read “quote” for each line, which is not ideal, so use this sparingly if native support is absent[58][59].
- Spacing: Always put a blank line before and after an admonition block (no matter which syntax) to ensure it renders correctly and doesn’t get combined with neighboring content[56]. Also, ensure the content inside is properly indented or formatted as required by that syntax. If using the GitHub [!NOTE] style, each line should start with the >. If using triple-colon or triple-bang, the content often needs to be indented or just between the markers depending on the implementation.
- Keep it concise: Admonitions work best for short, crucial pieces of info. If your admonition content is too long (several paragraphs), consider whether all of it is truly “note” or “warning” material, or if some of it should just be normal content. By nature, these callouts are for emphasizing important things.
- Admonition titles: Many systems allow a custom title. Use it to give more specific context (e.g., “Caution: Data Loss” instead of just “Caution”). If not provided, a default like “Note” or “Warning” will be shown. For GitHub’s [!NOTE] style, the title is automatically the type (you can’t customize the word “NOTE” that appears). For Docusaurus, you can specify after the admonition type as shown above.
- Compatibility notes: Admonitions are not part of core Markdown or CommonMark spec. They are extensions in various flavors[60]. If you copy your Markdown to a platform that doesn’t support the syntax, the raw markers ([!NOTE], :::note, etc.) will be visible. For instance, the triple-colon will just show as text on GitHub READMEs (since GitHub only supports the blockquote [!NOTE] syntax as of 2024). Keep this in mind if your docs might be read in plain form in multiple contexts. When in doubt, using the simplest blockquote method is the most portable, albeit without special styling.
Whitespace, Line Length, and Formatting Conventions
•	Trailing whitespace: Remove trailing spaces at the end of lines. They can cause diffs to be noisy and in some cases affect formatting. The only time two trailing spaces are meaningful is to create a forced line break (<br> in HTML) in a paragraph[61][62]. If you intend a line break, use exactly two spaces at end of the line, followed by a newline. Otherwise, ensure no extra spaces follow your text. Many editors and linters (Markdownlint rule MD009) can flag or automatically trim trailing whitespace[26].
•	Hard tabs vs spaces: Do not use literal tab characters for alignment or indentation in Markdown content (except in code blocks where they are part of the code). Use spaces instead[63]. A single tab can render as different widths in different environments, whereas spaces are consistent. Markdownlint (MD010) flags hard tabs as well. Configure your editor to insert spaces when you press Tab in Markdown files. (Standard is 4 spaces per indent level for lists or code blocks, but 2 spaces is also commonly used — just be consistent.)
•	One sentence per line (optional): A recommended practice for documentation is to put each sentence on a new line (soft line breaks). This is not visible in rendered output (Markdown will treat it the same as a space), but it makes diffs and reviews easier because only changed sentences show up as modified. This is optional, but if not following that, try to break long paragraphs into multiple lines (at logical points, e.g., after a clause or at a semicolon). Avoid one giant line for a whole paragraph, as it makes version control diffs harder.
•	Maximum line length: Aim for a line length of around 80 characters (or 100, depending on team preference) for normal text content[64]. This is a soft guideline to improve readability in editors and maintain consistent wrapping. Markdownlint’s MD013 rule defaults to 80 characters. It’s acceptable to exceed this limit for long links, URLs, or if a line has no natural break points (MD013 by default ignores long lines with no spaces)[65]. For example, a Markdown table row or a lengthy URL can exceed 80 chars, and that’s fine. The goal is to break lines at sensible places (spaces or after punctuation) to avoid extremely long lines.
•	Blank lines: Use blank lines to separate block elements (paragraphs, lists, code fences, etc.) for clarity[5]. Generally, insert a blank line:
•	Before and after headings.
•	Before a code block or list, unless it’s directly following a list item (as part of that list).
•	Between paragraphs if you want to start a new paragraph (Markdown requires a blank line to denote separate paragraphs).
Avoid multiple consecutive blank lines. One blank line is enough as a separator (MD012 flags more than one)[66]. Excess blank lines can be cleaned up.
•	Paragraph width and soft wrap: Do not manually insert a line break (two spaces or <br>) in the middle of a sentence just to wrap text. Let text naturally flow and wrap in the rendered output or the editor’s soft-wrap mode. Only insert line breaks where there is a logical break in content (new paragraph, or intentional line break like in an address or poem). If you find yourself wanting to format a list of short items, consider using bullets instead of line breaks.
•	End-of-file newline: Ensure the file ends with a newline character. This is a POSIX standard thing and also just good practice for version control (some tools concatenate files and missing newline can be an issue). Many linters or editors will enforce or warn if the last line does not end in \n.
•	No trailing blank lines: Similarly, don’t leave blank lines at the end of the file after the last content or after the closing front matter ---. End the file cleanly with a newline on the last content line.
•	Avoiding HTML in Markdown: Wherever possible, use Markdown syntax rather than raw HTML. Markdown is designed to be readable as plain text. Using HTML for formatting (like <br> for line breaks, <b> instead of ** for bold, etc.) should be avoided[67]. Most Markdown engines allow some HTML, but it can make the source harder to read and maintain. Exceptions might be if you need to include things Markdown can’t do (like a video embed or a custom styled div). Even then, consider if a plugin or extension is available. Using HTML also reduces portability (for example, some systems may sanitize it or not allow it). So, prefer Markdown constructs for headings, lists, emphasis, links, images, etc.
Workflow: Creating and Editing Markdown Files
Following a clear workflow helps maintain consistency:
1.	Plan the structure: Before writing, outline the document. Decide on the main sections (these will become your H2s, under a top-level title or H1). Having a clear structure prevents heading level mistakes and ensures logical flow.
2.	Start with front matter (if needed): If your project requires YAML front matter, add it first. Include at least the required fields like title (and date/tags for posts). This sets the stage – for example, once you put a title, you know you won’t write an H1 in the content if the site auto-generates it[3].
3.	Write content using the guidelines: Write your paragraphs, lists, and code blocks according to the best practices above. Use proper indentation for lists, add alt text for images as you go, etc. It’s easier to follow the rules during writing than to fix everything later. Keep paragraphs reasonably short. When adding a new heading or section, double-check you’re using the correct level (e.g., if you just finished an H2 section and are adding a sub-section, use H3). Remember the rule of incrementing one level at a time[4].
4.	Use a linter or plugin: Leverage tools like VS Code’s markdownlint extension or linters in your build process. They will catch many issues (spaces, wrong heading levels, line length, etc.) automatically. Configure markdownlint with a config if your standards differ from defaults (for example, if you intentionally allow long lines or want to enforce 4-space list indentation). This can save a lot of manual review effort by pointing out issues as you edit.
5.	Preview the output: If possible, use a Markdown preview (VS Code has one built-in, or use a tool like Grip or a static site’s dev server) to see how it renders. This helps catch issues like a code block not rendering because of a missing fence, a list not appearing as expected due to bad indentation, or a table looking misaligned. It’s especially important for tables and complex elements. For static site generators, run the site locally to ensure things like admonitions or front matter are processed correctly.
6.	Validate links and images: Click through the links in the preview or, if it’s a site, use an automated link checker. Broken links or missing images are common issues. Check that relative links actually point to the right location (especially if using ../). For images, ensure the image files are in the repo at the correct path and that the case matches (on case-sensitive file systems).
7.	Review for consistency: Do a final pass reviewing the styling: Are all your bullet lists using the same marker? Did you consistently capitalize headings? Are there multiple H1s by mistake (there should typically be just one, or none if using front matter for title)[2]? Are all headings surrounded by blank lines? Are list indentation levels consistent[15]? Maybe run the linter one more time to see if anything was missed.
8.	Proofread content: Best practices aside, also proofread the actual content for clarity and correctness. Sometimes in fixing formatting we might introduce a typo or a broken sentence. Ensure the text reads well in addition to being well-formatted.
9.	Commit with a meaningful message: When saving your changes, ensure your commit message reflects what you did (e.g., “Add Markdown style guide (markdown-best-practices skill)”). This helps in version history.
10.	Continuous improvement: If you find new rules or better practices, update this skill/document. Markdown evolves (for example, GitHub added admonitions in 2024), so keep the guidelines up-to-date with new features or decisions your team makes.
Validation and Troubleshooting
Before publishing or merging changes, it’s important to validate the Markdown and troubleshoot any issues:
•	Automated checks: If your project has continuous integration, make sure any Markdown linting or formatting checks pass. For example, a CI job might run Markdownlint or a link checker. Address any failures. Common ones might be line length issues, trailing spaces, or duplicate headings. Use the output from these tools to pinpoint the line and issue.
•	Visual inspection: View the Markdown on GitHub (if applicable) or in the target environment after pushing to a branch. GitHub’s web interface will render the Markdown – check that everything looks correct (headings are properly nested in the TOC if one is generated, lists are rendering, etc.). If using a static site, check the deployed preview. Pay special attention to:
•	Tables (are they formatted correctly? no weird misalignment or HTML showing).
•	Admonitions (if using GitHub’s syntax, do they render with icons? If using another platform, did they convert properly? If you just see the raw syntax, that means it wasn’t recognized).
•	Code blocks (do they appear, and with correct language highlighting? No odd characters or formatting issues, and not getting cut off).
•	Links (click them – ensure they go to the right place and don’t 404).
•	Images (are they visible? If not, check path or case sensitivity).
•	Common issues & fixes:
•	Heading not rendering as heading: Likely there’s no blank line above it or there’s an extra space before the #. Ensure ## Heading is at line start with a blank line above[5][6]. Also, a heading won’t render if it’s inside a code block or a blockquote unintentionally. Make sure the context is correct. If a heading is indented or preceded by 4 spaces, Markdown might think it’s code. Remove indentation for headings.
•	List numbering all “1.” or restarting unexpectedly: This is normal if you used lazy numbering (all 1.). If you see an actual rendered list where every item is "1.", it means the Markdown wasn’t recognized as a list. This could happen if there’s something breaking the list, like an empty line within the list without proper indentation. Ensure continuation lines for a list item are indented 4 spaces or 2 spaces past the bullet. Also, make sure there is a blank line before the list starts, or it might be seen as continuation of a paragraph. Another cause: mixing tabs and spaces in indent – stick to spaces.
•	List not nesting properly: If a sub-list isn’t indented enough, it will either continue the parent list or break entirely. Ensure sub-list items are indented consistently (if using 4 spaces, one level down should have 4 spaces in front of the marker beyond the parent’s indent)[16]. Also, check that you didn’t accidentally use a mixture of tabs and spaces.
•	Table is broken or not showing as a table: This usually means the syntax is off. Check that the header separator line has the correct number of | and that each column has some dashes. Also, there should be no trailing pipe at the end of a line without matching pipes (each row including header should start and end with a pipe | if you’re including the outer pipes). Sometimes an extra pipe or missing pipe can break the table. Counting pipes in each row to ensure they match is a quick way to debug. Also, if you have a pipe character in the cell content, you need to escape it like \| or the table will split there.
•	Table contents overflow: If a table cell has a long word or URL, it might stretch. This isn’t a “breaking” issue, but consider using reference links to shorten the visible text[35]. In HTML output, long strings can cause overflow – some CSS might handle it, but just be mindful.
•	Image not appearing: On GitHub, an image that doesn’t load will show as broken link icon. Check the path: is it correct relative to the Markdown file? If the image is in a subfolder, did you include the folder name? Also check the case – image.PNG vs image.png matters in many cases. If linking to an external image, ensure the URL is correct and accessible (some sites block hotlinking). If the image is large, GitHub might not render it (especially SVGs with certain scripts). As a test, try opening the image URL directly in a browser.
•	Admonition not rendering on GitHub: Remember that GitHub’s syntax requires the > at the start of each line of the blockquote and the [!KEY] tag on the first line[52]. If you missed one of those, it won’t work. Also, the KEY must be uppercase and exactly one of the supported words. If you wrote [!note] in lowercase, it won’t trigger the special formatting. If you use an unsupported word, it will just show normally. Currently supported keys are NOTE, TIP, IMPORTANT, WARNING, CAUTION (and they must be in brackets with exclamation, at the very start of a blockquote line). If all syntax is correct and it still doesn’t render in a preview, ensure you’re previewing on GitHub or using a renderer that supports GFM alerts (some local preview tools might not by default).
•	Admonition not rendering in Docusaurus or others: If triple-colon isn’t working, it could be that the Markdown is being processed by something that doesn’t support it. Docusaurus v2 supports it out of the box. Make sure you didn’t forget the closing ::: or that there are blank lines as needed. If Prettier reformatted your admonition and broke it (concatenated the :::note with content on one line), you’ll need to adjust your formatting or Prettier config[56].
•	Front matter issues: If your page isn’t showing up or building, front matter could be the culprit. A common mistake is a syntax error in YAML. For instance, using a tab for indentation (YAML doesn’t allow that), forgetting to close quotes, or having a colon in a value without quotes. If the site build fails, check the build log – it often points to a line in the front matter. Validate your YAML with an online linter if needed. Also ensure the front matter is bounded by the --- correctly. If you accidentally put an extra --- somewhere in content, it might confuse the parser into thinking front matter ends early.
•	Multiple H1 detected: Some linters (MD025) will complain if more than one # heading is found and front matter title is present[2]. Make sure after using a front matter title, your content uses ## or lower. If your document is standalone (not part of a system that injects a title), having one H1 is fine (like a README might have # Project Name at top). Just avoid numerous H1s – use subsections instead.
•	Reviewer checklist: When someone (or you, wearing a reviewer’s hat) reviews the Markdown, they should check for all the things we’ve discussed. It’s helpful to have a checklist (like the one below) to systematically verify compliance with this style guide.
Reviewer Checklist
Before finalizing a Markdown document, review the following:
•	Headings: Only one H1 (or none, if title is provided via front matter). All other headings are lower-level and properly nested – e.g., no jumping from H2 to H4 (check heading levels increment sequentially)[4]. Headings are preceded and followed by a blank line[5], and start at the beginning of the line (no leading spaces)[6].
•	Heading style: Headings use # syntax (ATX) consistently[1]. No use of underlined heading syntax. Heading text is concise and unique, without trailing punctuation[11]. Consistent capitalization style is applied.
•	Lists: Bullet lists use a consistent marker (- or *)[12]. Numbered lists are either all 1. (preferred for long lists) or correctly ordered if written out[13]. List indentation for sub-items is correct and consistent (e.g., 4 spaces)[15]. No mixing of tabs/spaces. Check that list continuations (multi-line items) are aligned. No blank lines interrupting a single list (unless intentionally separating into multiple lists).
•	List content: If list items are sentences, they end with punctuation; if fragments, they don’t – and this is consistent within each list. No inconsistent mixing of sentence and fragment styles[18].
•	Code blocks: All code blocks are fenced with triple backticks and have a language specified[68][69] (unless the content is generic or you specifically want no highlight). No use of old indented code blocks unless required. Blank lines exist before/after code fences (except when immediately nesting under list items). No stray backticks that could break formatting. In rendered output, verify that the code is rendering (if not, maybe a fence is not closed properly).
•	Inline code: Key technical terms, file names, or commands are appropriately in backticks. But also ensure not every other word is in code font – use it only for actual code/CLI/file references. Check that backticks used for inline code are balanced (opening and closing).
•	Links: All hyperlinks use Markdown syntax and have meaningful link text[31][32]. No bare URLs in text (unless necessary). Check that any reference-style links have their definitions present. No obvious broken URLs (hover to see if the URL looks correct).
•	Images: Every image has alt text describing it[39] (or empty alt if genuinely decorative). Filenames/paths look correct (especially on case-sensitive systems). If images are in a repo, ensure the path is relative and not an absolute file path from someone’s local system. Images are reasonably sized (not huge dimensions causing formatting issues).
•	Tables: Tables have a header row and a separator line. Columns line up properly (in source, or at least have the correct number of columns in each row). No obviously misaligned pipes or missing cells. If any cell has a long phrase or link, consider if it’s still readable. In the rendered preview, ensure the table looks fine (no cells overflowing oddly). If alignment is specified, check the colons in the separator line are in correct places.
•	Front matter: If present, front matter is valid YAML. Check for common errors like unquoted special characters or missing commas in lists (though YAML lists don’t use commas). Ensure required fields (title, date, etc.) are there and correctly formatted[45]. If the front matter title is present, verify that the content does not start with an extra H1. Also ensure the separator --- lines are there and nothing else on those lines.
•	Admonitions: If admonition syntax is used, verify it’s correct for the target platform. For GitHub, check > [!NOTE] usage[52]; for others, check ::: or !!!. Make sure the content inside is appropriately formatted (no missing closing tags). If not using a special syntax, see that faux-admonitions (blockquote with Note:) are used consistently.
•	General formatting: No trailing spaces (enable “show whitespace” in your editor or use the linter)[26]. No tabs (search for \t or again rely on linter)[63]. No more than one blank line in a row[66]. Lines are wrapped at a reasonable length or at semantic breaks. The file ends with a newline, and there’s no random blank lines at the very end.
•	No HTML (unless necessary): Scan for any HTML tags. Occasionally things like <br> or <img> might be used. Flag them and confirm if they’re truly needed (maybe the author didn’t know there’s a Markdown way, or it was the only solution). If not necessary, replace with Markdown. If they are needed, ensure they are correct and will not be stripped by the renderer (some sites might not allow certain HTML).
•	Consistency with team/project style: Beyond these general rules, ensure the document follows any project-specific conventions (maybe the project has a specific glossary, or prefers American vs British spelling, etc., which is outside pure Markdown rules but important for consistency).
By following this comprehensive set of Markdown best practices, you’ll produce documents that are clean, easy to maintain, and render reliably across different viewers. Consistent formatting not only helps tools like linters and converters, but also makes the raw Markdown more readable for collaborators. Happy writing!
________________________________________
[1] [7] [9] [10] [13] [14] [15] [16] [20] [21] [22] [23] [24] [31] [32] [33] [34] [35] [36] [37] [38] [39] [40] [41] [42] [43] [61] [67] [68] [69] Markdown style guide | styleguide
https://google.github.io/styleguide/docguide/style.html
[2] [4] [5] [6] [11] [12] [26] [27] [28] [29] [30] [62] [63] [64] [65] [66] raw.githubusercontent.com
https://raw.githubusercontent.com/DavidAnson/markdownlint/v0.24.0/doc/Rules.md
[3] [8] [17] [18] [19] [49] Markdown writing guide | Datagrok
https://datagrok.ai/help/develop/help-pages/markdown
[25] Creating and highlighting code blocks - GitHub Docs
https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks
[44] [46] [47] [48] [50] [51] Front Matter | Jekyll • Simple, blog-aware, static sites
https://jekyllrb.com/docs/front-matter/
[45] Using YAML frontmatter - GitHub Docs
https://docs.github.com/en/contributing/writing-for-github-docs/using-yaml-frontmatter
[52] [53] TIL: GitHub flavored Markdown admonitions – volatile write
https://knutwalker.codes/blog/til-github-flavored-markdown-admonitions/
[54] [55] [56] Admonitions | Docusaurus
https://docusaurus.io/docs/2.x/markdown-features/admonitions
[57] [58] [59] [60] ⚠️ GitHub is beta testing their own Admonition syntax. We should weigh in - Page 2 - Extensions - CommonMark Discussion
https://talk.commonmark.org/t/github-is-beta-testing-their-own-admonition-syntax-we-should-weigh-in/4173?page=2
