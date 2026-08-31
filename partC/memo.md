# Part C - Decision Memo

## Recommendation

I recommend (A), an SFT pass using synthetic casualized response pairs, with
prompt engineering as the baseline.

The main reason is that this needs to be a consistent style change across six
languages. Prompting is useful as a baseline, but I would not rely on it alone.
I would also avoid B for now because it adds another model and another
inference step.

## Assumptions

One A100-80GB is available continuously for two weeks.

Synthetic data can be generated locally using the available model/tooling.
There is no external API budget.

The native-speaker reviewer can review approximately 100 examples per hour.
The review is mainly for naturalness, tone, and obvious meaning errors.

Direct native-speaker review is available only for Hindi and Kannada. The other
four languages will need lighter validation and sampling.

## Back-of-envelope arithmetic

I would start with approximately 12,000 synthetic response pairs:

2,000 examples per language x 6 languages = 12,000 pairs.

If a response pair is about 300 tokens on average:

12,000 x 300 = 3.6M training tokens.

This is small enough for a targeted SFT experiment on one A100 while leaving
time for data generation, validation, and at least one iteration.

For reviewer capacity:

10 hours/week x 100 examples/hour = 1,000 examples/week.

Across two weeks this gives approximately 2,000 reviewed examples. I would use
most of this review budget for Hindi and Kannada and use sampling and automated
checks for the other four languages.

## Success metric

The main metric will be the percentage of responses judged casual and natural
by the reviewer.

On held-out Hindi and Kannada evaluation sets, the SFT model should reach at
least 80% casual/natural ratings while maintaining at least 95%
semantic-preservation accuracy against the original response.

The evaluation set must not be included in the training data.

## Kill criterion

I would stop the SFT experiment if, after the first trained checkpoint and
evaluation by the end of week 1, casual/natural ratings are below 70%, or
semantic-preservation accuracy is below 95%.

If that happens, there is not enough evidence that the synthetic-data approach
will give reliable results within the three-week launch timeline.

## Day-1 experiment

Create a small controlled benchmark of approximately 100 prompts per language
across Hindi, Kannada, Tamil, Telugu, Bengali, and Marathi.

For each prompt, generate:

- the current model response;
- a synthetic casualized target;
- a prompt-engineering-only response.

Compare the outputs on casualness, naturalness, and semantic preservation.
Hindi and Kannada should receive native-speaker review.

The Day-1 result will be used to decide whether SFT is worth the training
time. If prompting is already close to the target, I would keep the simpler
approach. If prompting is clearly below the target and the synthetic examples
are consistently good, I would continue with SFT.