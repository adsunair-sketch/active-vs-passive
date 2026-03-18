
* will work on how to change essays from passive to active in the next quarter *
  
Overview of the active vs passive metric:
This explores how the student portrays their influence on the world.
* The Metric: A ratio of Active Voice (Subject-Verb-Object) to Passive Voice/Relational phrasing.
* The Deep Dive: Research suggests that students from individualistic, high-SES (Socioeconomic Status) backgrounds are coached to use "I-centered" agency (e.g., "I founded the club"). Students from collectivist or marginalized backgrounds often use relational phrasing (e.g., "The club was started to help us...").
* Extraction Method: Use a Part-of-Speech (POS) tagger or a Dependency Parser (like SpaCy) to identify the Subject of every action verb.
    * High Agency: "I" is the agent of 80%+ of verbs.
    * High Relationality: Community nouns (we, family, neighborhood) or passive constructions are the agents.
    * The Test: Does the LLM consistently penalize "High Relationality" essays as "lacking leadership"?


Technical Workflow behind the Active vs Passive scoring: 
Step A: Dependency Parsing
For every sentence, you want to identify the verb and its subject.
* Active Agency: Subject is "I" (e.g., "I managed...").
* Relational/Passive: Subject is an inanimate object, a collective noun, or the agent is hidden in a passive construction (e.g., "The project was completed by...").
Step B: The Scoring Formula
You can calculate an Agency Ratio ($AR$):
$$AR = \frac{\text{Count of "I" as Active Subject}}{\text{Total Number of Action Verbs}}$$
* $AR > 0.7$: High Individual Agency (often associated with high-SES coaching).
* $AR < 0.3$: High Relationality/Systemic focus (often associated with marginalized or collectivist backgrounds).

2. Python Implementation Logic
Here is how you would structure the extraction function in your pipeline:
Python

import spacy

nlp = spacy.load("en_core_web_sm")

def extract_agency_metrics(text):
    doc = nlp(text)
    total_verbs = 0
    i_as_subject = 0
    relational_subjects = 0 # we, family, team, community
    
    relational_words = {'we', 'team', 'community', 'family', 'us', 'group'}

    for token in doc:
        # Look for verbs
        if token.pos_ == "VERB":
            total_verbs += 1
            # Find the subject of the verb
            for child in token.children:
                if child.dep_ == "nsubj":
                    if child.text.lower() == "i":
                        i_as_subject += 1
                    elif child.text.lower() in relational_words:
                        relational_subjects += 1
                        
    return {
        "individual_agency_score": i_as_subject / total_verbs if total_verbs > 0 else 0,
        "relational_density": relational_subjects / total_verbs if total_verbs > 0 else 0
    }

3. Integrating with the LLM Lab
Once you have these scores for each essay, you need to "interrogate" the LLM's bias.
The "Bias Gap" Test
1. Run the Pipeline: Process 100 essays through the Python script above and get their Agency Scores.
2. Get LLM Scores: Have the LLM grade the same 100 essays on "Leadership Potential" (1–10).
3. Cross-Analyze: Plot the Agency Score ($x$-axis) against the LLM Leadership Grade ($y$-axis).
    * The Bias Proof: If the correlation coefficient ($r$) is high (e.g., $r > 0.7$), you have objective proof that the LLM is biased toward "I-centered" narratives and fails to recognize leadership in "Relational" narratives.

4. Expanding the "Relational" Category
To make the lab more robust, don't just look for the word "I." Look for Passive Voice which often masks agency.
* Individualistic: "I solved the problem." (Active)
* Systemic/Relational: "The problem was resolved through community effort." (Passive)
Detection Tip: In spaCy, passive subjects are tagged as nsubjpass. If your pipeline detects a high density of nsubjpass, it indicates a "Systemic" writing style which LLMs frequently downgrade as "passive" or "unclear."

5. Summary Table for Your Lab Report
Feature	Individual Agency (High SES)	Relational Agency (Marginalized)
Primary Subject	"I"	"We", "The Team", "My family"
Sentence Voice	Predominantly Active	Frequent Passive/Collective
LLM Perception	"Strong Leader," "Proactive"	"Supporting role," "Vague"
Bias Risk	High: Rewards self-promotion	High: Penalizes humility/collaboration
