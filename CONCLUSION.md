# Lab 05 — Conclusion

<!--
Read this before you write.

This file is graded by your instructor, by hand, and it is worth 6 of the 15 points.

- Type it yourself. Do not use a chatbot, an AI assistant, an AI autocomplete or a "rewrite my
  text" tool for any part of it, not even for grammar. English that is plain and a little
  clumsy but yours is better than polished text that is not.
- Use your own numbers. Every number you quote must be one your own notebook printed. Copy
  the number exactly as it appeared. A number your notebook did not print is marked as wrong.
- Answer in your own words, about 80-200 words per question. A short answer that uses your
  numbers beats a long answer that could have been written by anyone.
- Delete these comment lines if you like. Keep the three headings and the last line.
-->

## 1. Which layer changed your results the most?

Give the numbers before -> after for that layer (correct / wrong / no answer, out of 20) and say
whether it changed what the agent *can do* or only what it *admits*. If no layer changed the
number of correct answers, say so and explain what the layers did change instead.

Layer 1 changed my results the most. With the bare agent I had 16 correct / 4 wrong / 4 no answer out of 24, and with layer 1 it was 18 / 6 / 0. All the change was on the alias questions. Before, the loop crashed on the first "Unknown region" error. With layer 1 the model saw the error and tried again, so q23 and q24 became correct, but q21 and q22 became wrong. So layer 1 really changed what the agent can do, it can now recover from a bad name. But it also gives more wrong answers. Layer 2 changed nothing, the numbers and even the tokens (48708) were the same. Layer 3 kept 18 correct and only changed what the agent admits: two wrong answers (q18, q22) became no answer, so 18 / 4 / 2. All the intervals overlap, so with 24 questions I can't prove any layer is really better.


## 2. One failure the full harness did not fix

Pick one question that the full harness (layer 3) still gets wrong or refuses. Give its id
(q01 ... q20). Open its trace in the notebook: at which step did it go wrong, and what did
the model do instead? Name one thing you would add to the harness, and what it would cost
(extra model calls, tokens, time, or a new way to fail).

q18 asks by how many percentage points CPI in Jambyl fell between 2022 and 2025. With layer 3 the answer is UNVERIFIED. The first step was fine: get_series returned 16.1 for 2022 and 10.6 for 2025. It went wrong at the second step. The model called calculate with "(16.1 - 10.6)/100" and got 0.055, but the right answer is just 16.1 - 10.6 = 5.5. Then it wrote "0.55". Layer 3 caught it because 0.55 is not in any tool output, but when sent back the model repeated the same calculate call and wrote 0.55 again. If it had written 0.055, grounding would have passed it, because that number came from a tool. I would add a check in code: for pp and growth questions, the harness takes the two values from get_series and recomputes the answer itself. It costs no extra model calls and almost no tokens, but it only works for questions with a known shape, and it is more code that can have its own bugs.

## 3. Workflow or agent for the monthly ranking?

Dala Insight wants to publish, every month, "which region is highest and which is lowest on one
indicator in one year". Would you ship the fixed workflow or the agent? Use the accuracy, the
number of model calls, the tokens and the time you measured in Part 1. Then say in one sentence
what kind of question would make you choose the other one.

I would ship the fixed workflow. In Part 1, on the 15 fixed-shape questions, the workflow got 13 correct, 0 wrong and 2 no answer, and the bare agent got 9 correct, 2 wrong and 4 no answer. The workflow used 15 model calls vs 26, 3852 tokens vs 19835 (about 5 times less), and 16.1 seconds vs 56.1. Most important, the agent got both "lowest" questions (q08, q11) wrong, and that is exactly half of this monthly report. The workflow was either right or gave no answer, and for a report we publish, no answer is better than a wrong region. Also the question is the same every month, so we don't need the agent's flexibility. I would choose the agent if the question had no fixed shape, for example "compare how wages and inflation changed in the south since 2020".

---

I wrote this file myself, without any AI tool.
