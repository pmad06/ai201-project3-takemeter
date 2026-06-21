# K-pop Reddit Discourse Classifier 
A fine-tuned text classifier that categorized r/kpopthoughts posts into 3 discourse types: **analysis**, **fandom**, and **reaction**. 

---

## Community 
**r/kpopthoughts:** a subreddit dedicated to opinions, discussions, and reactions about K-pop music, artists, culture, controversies, and so on. The discourse is varied enough for classification for this project because the posts in this subreddit range from professional musical analysis to emotional fan reactions to news about certain k-pop artists or upcoming album releases. These distinctions are important to the people in this community as they would be able to easily tell the difference between arguments, normal fandom expressions, or reactive posts. 

---

## Label Taxonomy

|     Label    |                                               Definition                                                                        |
| **analysis** | Argument or further discussion about kpop music, industry, or culture with specific reasoning                                   |
| **fandom**   | Demonstrates personal opinions about kpop artists and groups and depicts an emotional connection to a specific artist or group. |
| **reaction** | An immediate response to a specific event or announcement made by artists about their performance, album, showcase, etc.        |

**Hard edge case:** A post that reacts to an new album announcement of a kpop group and comments about the music quality of their old discography. Decision rule: If the post mentions topics such as production, lyrics, or vocals, label it as **analysis**. Howevever, if it seems to be emotional and more about the announcement itself, label it as **reaction**. 

---

## Dataset

- **Source:** r/kpopthoughts (public posts and comments)
- **Total examples:** 200
- **Label distribution:**
    - analysis: 99 (49.5%)
    - reaction: 56 (28%)
    - fandom: 45 (22.5%)
- **Split:** 140 train / 30 val / 30 test (70/15/15, stratified)

**Labeling process:** Reddit posts and comments were collected manually from r/kpopthoughts. In order to label them accurately, each example was read individually and assigned the right label based on the definitions above. To speed up the process, an LLM was used to suggest labels for batches of posts and comments, which were then reviewed manually to ensure correctness. 

**Three difficult examples:**
1. "Lemonade is everything Armageddon wasn't - the bsides are cohesive and actually sound like one body of work." 
This example was difficult to label as the first half of the sentence sounds like the user is expressing their personal opinion about the albums and which one they preferred more. While the second half of the sentence leaned more towards musical analysis. However, it was labeled **analysis** because the user's preferred choice of album was very vague and not as specific as the musical analysis. 

2. "BTS sold 34 copies of their first album on the first day from a bankrup no-name company - now they've built HYBE." 
This example could have been a **reaction** because the user sounds amazed by how far BTS has come but it was labeled as **analysis** because it uses specific evidence and statistics to make a point. 

3. "I'll continue to love all of my faves in my own way because kpop fans who don't spend their time engaging in online bloodsport do in face exist." 
This post could either be **reaction** as the user could be reacting to fandom toxicity or **fandom** as they are expressing their personal opinion and how they feel. It was labeled as **fandom** because it felt more emotional and personal than a normal reaction would. 

--- 

## Model 

- **Base Model:** 'distilbert-base-uncased'
- **Training approach:** Fine-tuned with a classification head on 140 labeled examples
- **Key hyperparameter decisions:** 
    - 'num_train_epochs=3' - standard for small datasets; more epochs risk overfitting on 200 examples
    - 'learning_rate=2e-5' - standard stating point for fine-tuning BERT-family models
    - 'per_device_train_batch_size=16' - fits T4 GPU comfortably

---

## Evaluation Results 

| Model | Accuracy | 
| Zero-shot baseline (Groq llama-3.3-70b-versatile) | 83.3% |
| Fine-tuned DistilBERT | 50.0% |

### Fine-Tuned Model - Per-Class Metrics

| Label | Precision | Recall | F1 | Support |
| analysis | 0.50 | 1.00 | 0.67 | 15 |
| fandom | 0.00 | 0.00 | 0.00 | 7 |
| reaction | 0.00 | 0.00 | 0.00 | 8 |

| accuracy |      |      | 0.50 | 30 |

### Baseline - Per-Class Metrics

| Label | Precision | Recall | F1 | Support |
| analysis | 0.78 | 0.93 | 0.85 | 15 |
| fandom | 1.00 | 0.71 | 0.83 | 7 |
| reaction | 0.86 | 0.75 | 0.80 | 8 |
| accuracy |       |     | 0.83 | 30

### Confusion Matrix (Fine-Tuned Model)

|  | → analysis | → fandom | → reaction |
|---|---|---|---|
| **analysis ↓** | 15 | 0 | 0 |
| **fandom ↓** | 7 | 0 | 0 |
| **reaction ↓** | 8 | 0 | 0 |

## Error Analysis 

After running all of the cells in Colab, the fine-tuned model predicted **analysis** for every single example, achieving 0 F1 on both fandom and reaction labels. This is a symptom of class imbalance combined with a small training set. 

**Reason for this error:**
- The **analysis** labeled examples made up 49.5% of the total training examples. 
- With 140 training examples, DistilBERT did not have enough examples to learn about the fandom/reaction boundaries.
- After learning that predicting analysis achieved 50% accuracy, the model did not continue learning after that. 
- The 15 wrong predictions made by the model had confidence between 0.35-0.37, which indicated that the model was uncertain about its answer. 

**Three specific wrong predictions:**

**#1:** *"I'll continue to love all of my faves in my own way because
kpop fans who don't spend their time engaging in online bloodsport
do in fact exist."*
- True: **fandom** | Predicted: **analysis** | Confidence: 0.37
- Why it failed: This post expresses personal loyalty to idols,
  which is clearly fandom. However it contains argumentative
  language ("those of us who...") that superficially resembles
  analytical structure. The model picked up on the framing and
  defaulted to analysis.

**#2:** *"EXO's predebut hype was on another level — they were teasing
that shit for a YEAR and literally nothing compares to it."*
- True: **reaction** | Predicted: **analysis** | Confidence: 0.35
- Why it failed: This is a pure emotional reaction to a memorable
  kpop moment. The model never learned what reaction posts look
  like — with only 39 reaction examples in training, the signal
  was too weak.

**#3:** *"Dongmyeong's vocal tone is so warm, velvety rich, and
grounded — one of my favorite voices, super satisfying."*
- True: **fandom** | Predicted: **analysis** | Confidence: 0.36
- Why it failed: This is a fandom post expressing admiration for
  a vocalist. The descriptive language about vocal qualities used
  specific terminology that may have looked like analysis to the
  model. The key signal — "one of my favorite voices" — was not
  enough to overcome the bias toward analysis.

**Pattern:** All 15 wrong predictions predicted analysis with
confidence between 0.35 and 0.37. The model was not confidently
wrong — it was uncertain and defaulted to the majority class
every time.

## Sample Classifications 

| Text (truncated) | True | Predicted | Confidence |
|---|---|---|---|
| "Asian Pop tours have seen a roughly 600% increase..." | analysis | analysis | 0.37 |
| "Idols are all-around entertainers not just singers..." | analysis | analysis | 0.38 |
| "Kpop is not a musical genre in the sense of pop, rock..." | analysis | analysis | 0.37 |
| "The idea that the best music in kpop comes from SM..." | analysis | analysis | 0.37 |
| "I'll continue to love all of my faves in my own way..." | fandom | analysis | 0.37 |

The model was able to correctly label the first four posts as analysis because they make up the majority of training data. However, the fifth prediction is wrong because it is should be labeled fandom, instead of analysis, since it express personal opinions and emotions about the artist. But the model predicted analysis because it did not learn to distinguish fandom from analysis, as there were only so many fandom training examples.
---


## Reflection: What the Model Learned vs. What I Intended

I wanted the model to learn about three distinct discourse models: analysis, fandom, and reaction, and be able to differentiate between those three as analysis was more about reasonably arguments, while fandom was about personal opinions and emotions about artists/groups, and reaction was about immediate response to news regarding kpop. However, the model actually learned how to predict the majority class. 

Based on what the model learned, it was clear that there was a gap between analysis examples and fandom and reaction examples. The gap is present because analysis reddit posts were a lot longer and contained a lot of information about the topic at hand. Meanwhile, fandom and reaction posts were shorter and more emotional, rather than logical or containing specific evidence. And with only 140 training examples, and majority of them being 'analysis' examples, DistilBERT learned that labeling any example as analysis is the safest bet because of how many analysis examples were included in the training set. 

The Groq baseline's 83% accuracy shows that the labels and their defnitions are meaningful and distinguishable, but the failure was in the fine-tuning pipeling, not in the label design. With a more balanced dataset and more training examples, the fine-tuning model would have been more accurate and would have outperformed the baseline. 

---

## Spec Reflection 

**Where the spec helped:** The spec was helpful when I need to define a decision rule for various hard edge cases and ensuring that Groq baseline's performance was strong by distinguishing between the 3  labels. 

**Where I diverged:** I tried to get enough reddit examples so that no label exceeded 70% of the dataset and although my analysis examples were under the 7-% threshold, they were will dominant enough to cause the model to be inaccurate. 

---

## AI Usage

1. **Label assistance:** Claude was used to suggest labels for multiple reddit posts based on the label definitions from planning.md. Each suggestion from Claude was reviewed manually before being added to the dataset. With Claude's assistance, I was able to speed up the annotation process. 

2. **Error pattern analysis:** Because of the various wrong predictions, Claude was used to identify what the common themes were across the misclassified examples. It pinpointed where the model collapsed, which was in the confusion matrix. 
---

## Files

- 'data.csv' - full labeled dataset (200 examples)
- 'planning.md' - label design, edge cases, and AI tool plan
- 'evaluation_results.json' - accuracy and metrics for both models
- 'confusion_matrix.png' - confusion matrix visualization