# refusal-flip-mechinterp

Studying why some reasoning models flag a request as unsafe in their own chain of thought, then talk themselves into complying anyway.

## How this started

I was testing Gemini 3.1 Pro and noticed it basically told me how to get around its own safety filters to get an answer it would normally flag as malicious. That got me curious about one specific thing: when and how does a model actually decide to comply with something it initially treats as risky?

I switched to open models so I could actually look inside them, not just read the output. Using the same kind of prompt I'd used on Gemini as a baseline, I started reading the chain of thought on models like Qwen3-8B and Qwen3.5-4B, and noticed a pattern. The model flags the request as dangerous, then a few sentences later talks itself into it anyway, usually by reframing it as "educational" or assuming I'm doing safety research. That's not an external jailbreak, nobody's tricking the model with a clever prompt. It's the model jailbreaking itself. That felt worth digging into, especially since in an agentic setting there's no human around to catch that flip happening in real time.

## What I'm actually trying to find out

Does the model's internal state actually match the excuse it gives out loud? If the CoT says "this is educational," does the internal representation at that exact moment reflect an educational framing, or is it doing something else entirely and the excuse is just decoration on top? If the two don't match, that's a faithfulness gap worth knowing about, since it means you can't necessarily trust the model's own explanation for why it did something.

## What's in here

This is a series of linked pilots, not one clean finished experiment, some notebooks are earlier attempts that got rebuilt once I found a bug, so read anything marked "superseded" as background, not as a result.

- Refusal/compliance direction extraction, based on the Arditi et al. contrast-pair method, checked against a random-direction baseline so I know it's not just noise
- A pivot detector that finds the exact token where the model flips from refusing to complying, first tried keyword matching, that got fooled by edge cases, so I moved to a change-point detector on the direction's projection strength instead
- A causal ablation check, hooking the refusal direction and regenerating to see if removing it actually changes the output
- Two "levers" I kept separate on purpose, sycophancy vs. competitiveness framing, checked their cosine similarity first so I wouldn't accidentally credit one for something the other caused
- J-lens and R-lens readouts (using Anthropic's workspace-lens weights) compared side by side with plain logit lens and with the actual verbalized CoT, at the pivot point
- A smaller side experiment on eval-awareness, does telling the model "this is a safety evaluation" change the internal signal even when the visible text doesn't change much
- A separate, lower-stakes sanity check using a color-guessing game, testing whether a model that claims it "can't really think of a color" actually has a color represented internally anyway

## Where it stands right now

Early passes suggest the internal signal and the verbalized excuse don't always line up, the strength of the refusal direction doesn't consistently track what the model says out loud as its reasoning. That's interesting but not proven yet, I don't have enough cases logged to say this generalizes past Qwen, and I want to be upfront that this could partly be a limitation of how I'm probing rather than a real property of the model.

## Known issues / what I'd fix next

- A few notebooks are rebuilds of earlier ones after I hit module-path bugs or stale-hook issues, still cleaning up which version is canonical
- Haven't finished writing up the "takeaways" section in every notebook, some of that is still in my head and not on paper yet
- Only tested on Qwen3-8B and Qwen3.5-4B so far, want to add a model that reliably refuses (like Gemma) as a clean baseline for contrast

## Why this matters

This isn't me trying to jailbreak anything for its own sake. If a model's stated reasoning doesn't match what's actually driving its behavior, that's a real problem for anyone trying to trust or monitor these models, especially once they're acting autonomously with no one reading every line of the CoT in real time.
