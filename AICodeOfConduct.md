# AI Code of Conduct
Rules for everybody working at Unigrid or externally contributing code, documentation or other material to the project.

AI tools may be used. They are a way to work faster, not a way to work with less care. Whatever you commit, you wrote, and you
answer for it exactly as if you had typed every character yourself.

## Review everything
 * Read and understand every line before it is committed. If you can't explain why a line is there, it does not go in.
 * Run it. Build the project, run the tests and try the change for real. Output that merely looks right is not evidence.
 * Check what the AI claims against the source. Method names, options, endpoints and library APIs are often invented or taken from
   an older version, so verify them in the code or in the official documentation.
 * Review AI output more critically than a colleague's, not less. It is always confident, whether it is right or wrong.
 * Check changes to tests. A test that was changed to make the build pass proves nothing.

## Recommended tool
We recommend [spyc](https://github.com/adeptum-labs/spyc) for coding with AI and for reviewing what it produces. It is a terminal
viewer for code bases that shows the project overview, the code with syntax colors and the history from git. It is useful whether
AI is used or not, but it is especially important when AI is used, as the amount of code to review grows. For reviewing AI
output, these keys are the most useful:
 * `g` shows all uncommitted changes, staged or not, and new files, as one diff. Go through it before every commit.
 * `c` shows what the tests ran, from the coverage reports, so you can see whether the new code is tested at all.
 * `b` shows who last changed each line and `l` the commit log with the diff of each commit.
 * `G` draws the dependency graph of the project as layered boxes, with what depends on something above it and the number of
   files behind each dependency. Use it to see if a change made a package depend on one it should not know about, and press `c`
   in the graph to find dependency cycles. `Enter` goes into a box and `o` opens its code.
 * `d` and `t` jump to a definition, so you can check that a method the AI calls really exists and does what it claims.

## Banned tools
 * **GitHub Copilot is banned** in this organization. Do not use it for completions, chat or review, and do not let it open or
   comment on pull requests.
 * **Weaker, low-tier models are banned.** Small, fast and cheap models produce plausible code that breaks the design in subtle
   ways. If you use AI, use a current top-tier model.

## Recommended setup
 * Use Fable or Opus 5.5. They are the models we recommend for code in this repository. Other models are generally too weak
   for this code base and produce worse results, which costs the reviewer more than it saves you.
 * Always run them on max effort when you write or change code. The time saved by a lower setting is lost again in review and
   rework.

## Use good judgment
 * AI is a tool for you to think with, not a replacement for thinking. Know what you want to change before you ask for it.
 * Keep changes small and focused, as described in [Committing](Committing.md). A huge generated diff that nobody can review is
   rejected, however good it may be.
 * Follow the style and conventions of the surrounding code. Generated code that looks different from its neighbours is not done.
 * Do not hand over work you don't understand. If the AI solved something you could not have solved or reviewed yourself, ask a
   maintainer first.

## Do not pollute the repository
Keep AI slop out. Do not commit:
 * Code that is not needed: speculative features, extra abstractions, unused helpers, defensive checks for things that cannot
   happen and commented-out code.
 * Comments that repeat the code, stale comments or long explanations of the obvious. Name things well instead.
 * Generated documentation, summaries, plans and notes that nobody asked for.
 * Padded commit messages, pull request descriptions and issue comments. Say what changed and why, in a few plain sentences.
 * Unrelated reformatting, renames or "improvements" that happened to be in the AI's output.
 * Tests that only exercise mocks, or that repeat the implementation.

If a reviewer finds slop in a pull request, expect to clean it up. Repeated slop means that your contributions will no longer be
accepted.

## Protect what is private
 * Never give an AI service secrets, private keys, tokens, passwords or personal data. This includes the contents of local
   configuration files that hold them.
 * Do not paste code or documents that are not public into a tool that stores or trains on its input, unless the Foundation has
   approved the tool.
 * You are responsible for the licence of what you commit. Do not commit output that reproduces code from a project with a
   licence that is incompatible with the licence of this one.
