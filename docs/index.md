---
title: "One Run Is Not Enough: How Much AI Agent Results Vary <br>in Data Science, an XGBoost Optimization Study"
---

## One Run Is Not Enough: How Much AI Agent Results Vary <br>in Data Science, an XGBoost Optimization Study

#### by Szilard Pafka and Eduardo Ariño de la Rubia

**TL;DR** We had AI agents autonomously tune an XGBoost model 10 times each, using the same setup and the same instructions every time, and with three different LLMs. On average the agents deliver large improvements, and the LLMs rank clearly against each other. But the result of a single run varies a lot from run to run, and the ranges of the different LLMs overlap. One run of a given setup is not enough, either to judge how well an agent does or to compare agents.

---

In our [previous project](https://szilard.github.io/xgboost-autoresearch/) we showed that AI coding agents can automate much of a data scientist's trial-and-error work: they research ideas, engineer features, tune the hyperparameters of an XGBoost model, and keep what works. That was essentially one agent doing one run, though, and AI agents are not deterministic. Give the same agent the same task twice and it will try different ideas, in a different order, and end up with a different model. A single run is therefore an anecdote. It cannot tell us how good a setup really is, or whether one LLM is better than another. To answer those questions, we rebuilt the project into a minimal, self-contained building block that can be run many times, fully automatically.

The task is the same as before: predict whether a flight will be delayed, using airline data. The agent has 2 hours to improve a starter XGBoost model on its own. It searches the web for ideas, edits the code, keeps the changes that improve the score and discards the rest. Every model the agent keeps is then scored on a holdout dataset that the agent never sees, so we measure how well the models really generalize, not just the score the agent was optimizing. Compared to the first project, the setup ([xgboost-autoresearch-minimal2](https://github.com/szilard/xgboost-autoresearch-minimal2)) is simpler and stricter. The holdout data is cleanly separated, and automatic checks verify that the agent followed the rules. An orchestrator ([xgboost-autoresearch-minimal2-runs](https://github.com/szilard/xgboost-autoresearch-minimal2-runs)) runs the experiment repeatedly without any human steering: each run starts in a fresh, isolated environment, with identical instructions. With this, we ran three OpenAI models (gpt-6-luna, gpt-6-sol and gpt-6-astra), at maximum reasoning effort, 10 times each.

<img src="holdout_auc.png" style="width: 700px">

<img src="holdout_auc_path.png" style="width: 700px">

The first plot shows the final holdout AUC of each run (higher is better; the starter model scores 0.7155). On average the three LLMs rank clearly: astra comes first, then sol, then luna. But the runs of each LLM are spread widely. The gap between luna's best and worst run is about 0.025 AUC, more than half of the typical improvement the agents achieve over the starter model. The ranges also overlap: luna's best run beats sol's worst run and is on par with astra's weaker runs. If we had compared the three LLMs based on a single run each, we could easily have ranked them wrong. The second plot shows where this variation comes from. It traces the holdout AUC of each run, experiment by experiment. All runs start from the same model, but they take very different paths. Some find a key idea early and climb quickly, while others get stuck on a plateau for dozens of experiments. For example, the best runs discovered that the exact calendar date of the flight is very informative (flights on the same day share weather and congestion), while the weakest run never found this. How far a run gets therefore depends partly on luck: which ideas the agent happens to try, and when. Reassuringly, in every run the holdout score closely tracks the score the agent was optimizing, so the improvements are real and not overfitting.

Our conclusion is that today's AI agents reliably deliver substantial improvements on a data science task like this one, but how large the improvement is varies a lot from run to run, even when the setup and instructions are identical. This has two practical consequences. First, to compare agents, LLMs, prompts or any other part of the setup, one run is not enough. You need repeated runs and should look at the distribution of the results, not a single number. Second, if you just want a good model, it pays to run the agent several times and pick the best run, judged on data the agent has not seen. Both repositories are open source, so you can use them to test your own agents, LLMs and setups, ideally with more than one run each.

License: [CC BY 4.0](https://github.com/szilard/xgboost-autoresearch2/blob/main/docs/LICENSE.md)

<script data-goatcounter="https://szilard.goatcounter.com/count" async src="https://gc.zgo.at/count.js"></script>
