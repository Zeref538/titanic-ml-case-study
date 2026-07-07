# Titanic ML — What I Learned (Explained Simply)

*A journey from 0.768 to 0.804 on the Kaggle leaderboard in six versions — including the wrong turns, because that's where the learning happened.*

---

## The Terminology, Explained Like You're Ten

**Machine learning** — Instead of writing rules yourself ("if passenger is female, guess survived"), you show the computer 891 examples and it figures out the rules on its own.

**Feature** — One fact about a passenger: their age, their ticket class, their sex. Features are the clues the model uses to guess.

**Target** — The answer we're trying to guess. Here: survived or not.

**Model** — The "guessing machine" that turns clues into answers. We used three kinds:
- **Logistic regression** — a scorecard. Every clue adds or subtracts points; big total = "survived." Simple and steady.
- **Random forest** — hundreds of decision trees ("is she female? is he in 3rd class?"), each voting, majority wins.
- **Gradient boosting** — trees built one at a time, where each new tree focuses on fixing the mistakes of the previous ones. Our champion.

**Training** — Letting the model study the 891 passengers whose fates we know.

**Prediction** — Asking the model about the 418 passengers whose fates are hidden from us.

**Accuracy** — Out of all guesses, how many were right. 0.80 = 8 out of 10.

**Baseline** — The dumbest possible guess, used as a measuring stick. "Everyone dies" scores 62% because most passengers did die. Any model worth keeping must beat this.

**Cross-validation (CV)** — A practice exam. Split the training data into 5 pieces, train on 4, test on the hidden 5th, repeat 5 times. It predicts how well you'll do on questions you've never seen — *without* wasting a real submission.

**Overfitting** — Memorizing last year's exam answers instead of learning the subject. The model looks brilliant on data it studied and flops on new data.

**Regularization** — Deliberate handicaps that stop memorizing: make trees smaller, show each tree only part of the data, take tiny learning steps. A rule that survives the handicaps is probably a real pattern.

**Imputation** — Filling in missing answers with a sensible guess. 177 passengers had no recorded age, so we guessed from their name title ("Master" = little boy ≈ 4 years old).

**Feature engineering** — Making new clues out of old ones. We extracted titles from names, counted family sizes, computed fare-per-person, and asked "did the rest of your travel group survive?"

**Data leakage** — Accidentally letting the answer sneak into the clues. Like grading your own exam. Our group-survival feature had to *exclude each passenger's own outcome* or it would have "predicted" the answer by peeking at it.

**Hyperparameters** — The model's settings dials (how many trees, how fast to learn). **Tuning** = trying many dial combinations and keeping the best.

**Probability matching** — A human instinct that loses: if 24% survive, guessing "survive" for a random 24% *feels* fair but scores worse than always guessing the majority. Casinos are built on people not knowing this.

**Log loss** — A stricter grading system that scores your *confidence*, not just your answer. Saying "100% sure" and being wrong is punished brutally; honest uncertainty ("63% chance") is rewarded. Under this metric, telling the truth about what you don't know is the winning strategy.

---

## The Journey: Six Versions, Four Mistakes, Two Wins

| Version | What we tried | CV | Leaderboard | Verdict |
|---|---|---|---|---|
| v1 | Solid baseline pipeline | 0.844 | 0.76794 | Good start |
| v2 | Deleted "weak" columns + rows with missing age | 0.824 | 0.75837 | ❌ Made it worse |
| v3 | Deleted only the weakest column (Embarked) | 0.835 | 0.76315 | ❌ Still worse |
| v4 | Added a clever ratio (fare per person) | 0.847 | 0.75598 | ❌ CV up, reality down |
| v5 | Added NEW information (group survival) + regularization | 0.850 | 0.78708 | ✅ +1.9 points |
| v6 | Tuned settings with a persistent search loop | 0.853 | **0.80382** | ✅ +1.7 more |

---

## Mistakes → Lessons

**Mistake 1: "This column looks useless, delete it."**
Deleting Embarked — the weakest feature by every measure — still *lowered* the score. A feature that's weak alone can still contribute in combination with others, and tree models simply ignore genuinely useless columns on their own.
→ **Lesson: with tree models, removal almost never helps. Weak ≠ useless. Test, don't assume.**

**Mistake 2: "Rows with missing values are dirty, delete them."**
Deleting 177 passengers with missing ages threw away a *biased* slice of the data (mostly 3rd class) and made every model worse — and the test set has missing ages too, which you're still required to answer.
→ **Lesson: impute missing values; almost never delete rows. Sometimes missingness itself is a clue (having no cabin number meant you were probably poor).**

**Mistake 3: "My clever new feature raised my practice score, ship it."**
Fare-per-person raised CV by +0.4 and dropped the real leaderboard by −1.2. Rearranging information the model already had gave it new ways to memorize noise, not new knowledge.
→ **Lesson: new *information* beats new *packaging*. And a score that only goes up on your own validation is a warning, not a victory.**

**Mistake 4: "Judge each idea by its solo strength."**
Age alone barely beat the baseline — yet it was crucial in combination (a child in 3rd class lived; an adult man didn't). Meanwhile five nearly-useless features *combined* beat the single strongest traditional feature.
→ **Lesson: features are teammates, not soloists. Judge contributions in context.**

## What Actually Worked

**Win 1: Add information the model has never seen.**
The single biggest gain (+1.9) came from group survival — linking passengers who traveled together and asking whether their companions lived. No existing column contained that; we had to build it. (And build it *honestly*: each passenger's feature used only the *other* members' outcomes — otherwise it's data leakage and the score is fake.)

**Win 2: Restraint.**
Every tuning improvement pointed the same direction: slower learning, smaller trees, less data per tree, minimum group sizes for rules. On small datasets, the best model is the most disciplined one, not the most powerful one. The winning configuration was found at search iteration 19 — after 11 straight failures. **Persistence through no-improvement is part of the method.**

**Win 3: Trust cross-validation, use the leaderboard as a sanity check.**
The public leaderboard is only ~209 passengers — differences of a point are 2–3 people, i.e., luck. Our CV correctly ranked v1 > v2 and v5 > everything before it. The one time CV and leaderboard disagreed (v4), the disagreement itself was the signal.

**Win 4: One change at a time.**
Every version changed one thing and measured it against a control. That's the only reason we *know* which of the four mistakes were mistakes. Change five things at once and you learn nothing from success or failure.

---

## The One-Sentence Summary

> **Data science progress = new honest information + disciplined models + one measured change at a time — and the mistakes, properly measured, teach more than the wins.**

*Final score: 0.80382 — roughly top 5–7% of honest submissions. Everything above ~0.81 on this leaderboard is people looking up the real historical casualty list, which is why "1.00" scores exist and why nobody serious pays attention to them.*
