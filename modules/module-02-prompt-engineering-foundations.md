<!-- module: 2 | phase: 1 | format: reference | last_verified: 2026-09-15 -->

# Module 2 — Prompt Engineering Foundations

## Writing Prompts and Understanding Tokens

- A **prompt** is the instruction you give a model — everything you type, plus any system instructions, file contents, and prior conversation turns that get sent along with it. The model has no knowledge of your intent beyond what's actually in the prompt; it can't read your mind about what "fix this" means without context.
- A **token** is the basic unit a model reads and generates — roughly a chunk of a word, not a whole word and not a single character. Common short words are often one token; longer or less common words get split into two or more. Code, punctuation, and non-English text tend to tokenize less efficiently than plain English prose.
- Every model call is billed and limited by tokens, not by words or characters — this is why token awareness matters even before you get to prompt structure. A page of source code, a long error log, or a big pasted file all consume tokens whether or not the model needs all of it to help you.
- The **context window** is the maximum number of tokens a model can hold in a single request — your prompt, any attached files, the conversation history, and the model's own output all draw from the same budget. Current Claude models support very large context windows (up to 1M tokens on Sonnet 5, Opus 5, and Fable 5.1, and 200K tokens on Haiku 4.5), but a bigger window doesn't mean you should fill it — irrelevant content still costs money and can distract the model from what actually matters.
- A useful mental model: think of tokens as the currency you're spending on a conversation. Pasting an entire 5,000-line file to ask about one function spends tokens on 4,900 lines the model didn't need to read. Being deliberate about what you include is a core prompting skill, not an afterthought.
- Tokenization is model-specific — the exact chunking Claude uses isn't identical to what another vendor's tokenizer produces. You don't need to memorize tokenizer internals, but knowing that "shorter, more common phrasing tends to tokenize more efficiently" is a fact worth carrying into how you write prompts.

## Structuring and Iterating on Prompts

- **Give clear, direct instructions.** State exactly what you want, including format, scope, and constraints. "Refactor this function" is vague; "refactor this function to remove the duplicate loop, keep the existing function signature, and don't touch the tests" tells the model what success looks like.
- **Provide context, not just commands.** Explain the *why* behind a request when it affects the *how* — for example, telling the model a function is on a hot path changes whether it should optimize for readability or for performance.
- **Use examples (few-shot prompting).** Showing the model one or two examples of the input/output pattern you want is often more effective than describing the pattern in the abstract. This is especially useful for consistent formatting, like a specific commit message style or a particular JSON shape.
- **Use structure to separate parts of a prompt.** XML-style tags (like `<context>`, `<instructions>`, `<example>`) or clear markdown sections help the model tell your background information apart from your actual instructions apart from example content. This matters more as prompts get longer and mix several kinds of material.
- **Break large tasks into steps.** A single sprawling instruction ("build the whole feature") gives the model much more room to misinterpret scope than a sequence of smaller, checkable steps ("first, write the data model; then, write the endpoint; then, write tests for it").
- **Iterate rather than expecting a perfect first prompt.** Prompting is a loop: give an instruction, look at the actual output, and refine your next instruction based on where it fell short — was it too verbose, did it miss a constraint, did it misunderstand the goal? Treat the first response as a draft to correct, not a one-shot bet.
- **Be specific about the role or perspective when it helps.** Asking the model to review code "as a senior backend engineer focused on security" produces a different, often more useful, response than a generic "review this code."

## Getting Concise Outputs and Avoiding Common Mistakes

- **Ask directly for the output length and format you want.** If you want a short answer, say so — "answer in two sentences," "return only the code, no explanation," "respond with a bulleted list." Models default toward being thorough and explanatory unless told otherwise, which is often more tokens than you need.
- **Control formatting explicitly.** Specifying "no markdown headers" or "plain text only" or "valid JSON matching this schema" prevents the model from guessing a format you didn't want and having to redo the request.
- **Avoid vague, open-ended asks when you actually want a specific outcome.** "Make this better" invites the model to guess at what "better" means and often produces a broader change than intended. "Reduce this function's time complexity from O(n²) to O(n log n) without changing its public interface" leaves far less room for misinterpretation.
- **Don't paste more than the task needs.** Pasting an entire file, an entire log, or an entire error trace when only a fragment is relevant wastes tokens and can bury the actually-relevant part in noise the model has to filter through.
- **Don't repeat the same full context across every turn of a conversation** if the model can already see it in history — restate only what's changed, or explicitly reference what came before ("using the schema from earlier").
- **Watch for prompts that ask for two contradictory things** (e.g., "be extremely thorough" and "keep it under 50 words") — the model will pick one to satisfy at the expense of the other, and the result can feel arbitrary if you didn't notice the conflict yourself.
- **A wasted round trip is more expensive than a slightly longer prompt.** If a little more up-front detail avoids a follow-up correction, it's usually worth the extra tokens. The goal isn't to minimize prompt length at all costs — it's to get the right answer in as few turns as possible.

<!-- nav -->
[← Module 1: Understanding Claude and How It Works](module-01-claude-and-how-it-works.md) | [Module 3: Understanding Agentic AI Systems →](module-03-agentic-ai-systems.md)
