# Project 3 Planning: K-pop Reddit Discourse Classifier 

## Community
r/kpopthoughts - an active subreddit about Korean pop music discussion. The discoure is varied enough for classification because posts range from emotional fan reactions to structured musical analysis to event-driven news responses, all within the same community. 

## Label Taxonomy

### Label 1: fandom 
Posts expressing stan behavior, appreciation for their idols or group loyalty. The post tends to be about the artist as a person or their role in a group, but not about the music itself. 

Examples: 
- "BTS RM has been appointed as Goodwill Ambassador for the National Museum of Korea"

### Label 2: analysis
Posts that tend to break down music from their favorite korean artists and explaining the meaning behind lyrics, choreography, and production. 

Examples" 
- "It is very poetic and the English translations really don't do it much justice compared to the Korean lyrics"

### Label 3: reaction
Posts responding to events, such as an artist comeback, chart update or milestone, award, or scandal with an immediate response from the fandom. 

Examples: 
""Suddenly" by I.O.I has taken its first win on this week's episode of MBC Show! Music Core (260613)"

## Hard Edge Cases
A post that reacts to a comeback stage and analyzes the quality of the music and the artists' choreography. 
Decision rule: If the post makes a claim about the music, such as its production, lyrics, or vocals, label it as "analysis". However, if the post is more emotional (happy or sad) with no argument about the music itself, label it as "reaction". 

## Data Collection Plan 
- Source: r/kpop (public posts and comments)
- Target: around 70 examples per label
- If the 3 labels are not evenly represented, the plan would be to actively search for posts that fit underrepresented labels using r/kpop's search or sort by the most popular posts. 

## Evaluation Metrics
- Overall accurate for both fine-tuned and baseline models
- Confusion matrix to see which labels get mixed up 
- Get a separate score for each label to determine if the model is ignoring any category

## Definition of Success
A classifier is useful if it hits at least 0.70 F1 on all three classes 
on the test set, and outperforms the Groq zero-shot baseline by at least 
5 percentage points in overall accuracy.

## AI Tool Plan

### Label stress-testing 
Give Claude context about the labels by providing it with the label definitions and ask it to generate around 10 posts that could be classified as "fandom" or "reaction" or "analysis". Results from this would help determine how the defintions of the labels need to be tightened to be made more classified. 

### Annotation assistance 
An LLM could be used to pre-label batches of around 30 examples and review them myself to verify that they are accurate. Pre-labled examples will be flagged in the CSV for transparency. 

### Failure analysis
After evaluating everything, I will provide Claude with the list of wrong predictions and recognize any patterns before writing the evaluation section. 
