# neo-**τ-bench**

neo-τ-bench is an fork of [tau-bench](https://github.com/sierra-research/tau-bench) that improves on the original benchmark in the following areas:

- testing corpus
- tools
- system prompts

These improvements stem from my experimentations with the airline environment, so some changes only apply to or are optimized for the (more challenging) airline environment. Furthermore, most of the implementation is optimized for legibility and comprehension when diffing against the original benchmark, not for clean abstraction and efficiency. The reason for this is to facilitate the appreciation for the merits of the changes without prescribing specific solutions. The ultimate hope is that these changes, or a version of them, are adopted by the original benchmark.

## Testing corpus

Most of the test cases in the original tau-bench were AI generated. While this enabled automatic test generation at scale, it resulted in many suboptimal test definitions. Since the benchmark relies on AI-simulated users to converse with the agents, it becomes imperative to have clear and assertive instructions & definitions for the agents to minimize hallucinations. To this end, a new manually curated and verified test corpus `revised_test` was created for the airline environment. `revised_test` directly borrows and improves on a subset of the original `test`. 

Here are the results between the two test corpi

| Test corpus       | Pass^1 | Pass^2 | Pass^3 | Pass^4 |
| -------------- | ------ | ------ | ------ | ------ |
| revised_test      | **0.52**     | **0.38**     | **0.28**     | **0.22**     |
| test     | 0.37     | 0.28     | 0.25     | 0.22    |

The command for `revised_test` was
```
python run.py --agent-strategy tool-calling --env airline --model openrouter/anthropic/claude-3.7-sonnet --model-provider openrouter --user-model openrouter/anthropic/claude-3.7-sonnet --user-model-provider openrouter --user-strategy llm --max-concurrency 33 --task-split revised_test --num-trials 4
```
and the command for `test` was
```
python run.py --agent-strategy tool-calling --env airline --model openrouter/anthropic/claude-3.7-sonnet --model-provider openrouter --user-model openrouter/anthropic/claude-3.7-sonnet --user-model-provider openrouter --user-strategy llm --max-concurrency 33 --task-split test --num-trials 4 --task-ids 0 3 6 11 20 23 32 1 10 25 4 12 14 15 17 21 22 41 42 43 47 48 49 19 18 26 27
```

More tests will be converted into `revised_test` over time. You can find more information at https://github.com/yiphei/new-tau-bench/wiki/revised_tasks_test.py-changelog. 

## Tools

### Tool descriptions

Some tools had unclear, incorrect, or incomplete descriptions, so they were improved. For instance, the `SearchOnestopFlight` tool had the incorrect description `Search direct flights between two cities on a specific date.`

### Tool addition

A new `SortFlights` tool was added. Many test cases required the AI to sort long lists of flights, where AI often committed errors and hallucinated. I don’t believe a model’s native sorting abilities are critical to evaluating its agentic performance, so I decided to introduce a sorting tool. Likewise, I modified the tools `SearchDirectFlight` and `SearchOnestopFlight` to have built-in sorting features.

### Tool API and business logic

The business logic of some tools were either corrected or improved. For instance, the original `UpdateReservationBaggages` tool expected a `payment_id` even when there were zero nonfree baggages and thus no payment was expected. This created ambiguity for what `payment_id` to use. Therefore, `UpdateReservationBaggages` was updated to expect a null `payment_id` when there are zero nonfree baggages.

## System prompts

### User LLM-strategy system prompt

One of the biggest problems I encoureted with the AI-simulated user was premature conversation termination via `'###STOP###'`. Often, the conversation was not over, but the user hallucinated and terminated it. Therefore, the system prompt was improved to reduce these incidents. Better formatting was also applied.

### Policy (wiki.md)

The change flights and cabin sections of the policy are confusing because it first states that basic economy flights cannot be modified, but it then separately states that basic economy flights can upgrade cabin. Once upgraded, they could be changed like any other flight. Since cabin changes are instrinsically part of flight changes (both conceptually and at the tool API level), the two sections were merged into one and the overall exposition was improved for clarity.