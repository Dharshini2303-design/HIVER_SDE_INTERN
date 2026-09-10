# AI Customer Support Agent for AppleSupport

## 1. Overview

This project builds an AI-powered customer-support agent using real customer-support conversations from Twitter/X.

The system is designed to perform three main tasks:

1. Classify an incoming customer message into a support intent.
2. Retrieve historically similar customer-support cases and use their resolutions as evidence.
3. Decide whether the message can be automatically handled or should be escalated to human support.

The selected brand for this project is AppleSupport.

---

## 2. Problem Statement

Customer-support teams receive large numbers of messages covering different types of issues.

The goal of this project is to build a support agent that can understand incoming customer messages, retrieve similar historical cases, draft a grounded response, and determine whether human intervention is required.

The system focuses on using historical support conversations as evidence rather than generating unsupported responses.

---

## 3. Dataset

The project uses the Customer Support on Twitter dataset.

Dataset source:

Kaggle:
thoughtvector/customer-support-on-twitter

The dataset contains customer messages and corresponding company responses.

Important fields include:

- tweet_id
- author_id
- inbound
- created_at
- text
- response_tweet_id
- in_response_to_tweet_id

Inbound messages represent customer messages, while outbound messages represent company responses.

Approximately 3 million tweets/replies are available in the original dataset.

For this project, AppleSupport conversations were extracted from the larger dataset.

---

## 4. Brand Selection

AppleSupport was selected because it provided a sufficiently large number of customer-support conversations for building and evaluating a support prototype.

Number of AppleSupport conversations used:

[INSERT ACTUAL NUMBER]

---

## 5. Intent Taxonomy

Eight support intents were identified from the AppleSupport conversations.

The final intent categories are:

1. technical_issue
2. battery_issue
3. general_support
4. icloud_device_issue
5. software_glitch
6. iphone_device_issue
7. ios_version_information
8. ios_update_issue

The initial intent discovery used unsupervised clustering.

KMeans clustering was applied to TF-IDF representations of a representative sample of AppleSupport customer messages.

The resulting clusters were manually inspected and assigned meaningful intent names.

Human annotation was then used to evaluate the usefulness of these categories.

---

## 6. System Architecture

Customer Message
        |
        v
Intent Classifier
        |
        v
Historical Case Retrieval
        |
        v
Top Similar Support Cases
        |
        v
RAG Prompt
        |
        v
FLAN-T5 Response Generator
        |
        +------------------+
        |                  |
        v                  v
Generated Reply      Escalation Decision
                           |
                    AUTO_HANDLE / ESCALATE

---

## 7. Intent Classification

The intent classifier uses:

- TF-IDF vectorization
- Unigrams and bigrams
- Logistic Regression
- Balanced class weights

The classifier predicts one of the eight AppleSupport intents.

---

## 8. Historical Retrieval / RAG

For a new customer message, the system searches the historical AppleSupport dataset for similar customer messages.

TF-IDF vectors are used to represent customer messages.

Cosine similarity is then used to retrieve the top historical cases.

The retrieved cases contain:

- Historical customer message
- Historical brand response
- Intent
- Similarity score

The retrieved historical responses are supplied to the generation model as context.

The golden evaluation messages were removed from the retrieval set using normalized exact text matching to reduce direct evaluation leakage.

---

## 9. Response Generation

The project uses the local open model:

google/flan-t5-small

The model receives:

- Predicted customer intent
- Similar historical customer cases
- Historical brand responses
- Current customer message

The generation prompt instructs the model to:

- Be concise and professional.
- Use historical cases as evidence.
- Avoid inventing policies.
- Avoid inventing refunds, prices, delivery dates or guarantees.
- Avoid claiming that an action was completed without evidence.
- Recommend human review when historical evidence is insufficient.

---

## 10. Escalation Decision

The system uses retrieval similarity as one signal for escalation.

If the maximum similarity between the current message and retrieved historical cases is below the configured threshold, the system escalates the case.

Current threshold:

0.25

The reason is returned together with the decision.

Possible decisions:

- AUTO_HANDLE
- ESCALATE

This rule is intentionally simple and is a prototype-level safety mechanism rather than a complete production escalation policy.

---

## 11. Evaluation Methodology

A 200-example golden evaluation set was created.

The evaluation includes:

- Human-labelled intent
- Model-predicted intent
- Generated response
- Historical retrieval similarity
- Escalation decision
- LLM-as-judge scores
- Human agreement

The golden examples are used consistently for evaluation.

---

## 12. Intent Classification Results

The primary classification metrics are:

Accuracy: [INSERT ACTUAL VALUE]

Weighted Precision: [INSERT ACTUAL VALUE]

Weighted Recall: [INSERT ACTUAL VALUE]

Weighted F1: [INSERT ACTUAL VALUE]

---

## 13. Baseline Comparison

The proposed classifier was compared against simpler baselines.

| System | Accuracy | Weighted Precision | Weighted Recall | Weighted F1 |
|---|---:|---:|---:|---:|
| Majority Class | [VALUE] | [VALUE] | [VALUE] | [VALUE] |
| TF-IDF + Logistic Regression | [VALUE] | [VALUE] | [VALUE] | [VALUE] |
| Final AI Support Agent | [VALUE] | [VALUE] | [VALUE] | [VALUE] |

The comparison shows whether the proposed approach provides improvement over simple alternatives.

---

## 14. Escalation Results

Total evaluated examples:

[VALUE]

AUTO_HANDLE:

[VALUE]

ESCALATE:

[VALUE]

Auto-handle rate:

[VALUE]

Escalation rate:

[VALUE]

The escalation rule is based primarily on retrieval confidence and is therefore intended as a conservative prototype mechanism.

---

## 15. LLM-as-Judge Evaluation

Generated responses were evaluated using five criteria:

1. Relevance
2. Helpfulness
3. Groundedness
4. Professionalism
5. Hallucination avoidance

Each criterion was scored from 1 to 5.

Results:

Relevance: [VALUE]

Helpfulness: [VALUE]

Groundedness: [VALUE]

Professionalism: [VALUE]

Hallucination avoidance: [VALUE]

Overall average:

[VALUE]

The LLM-as-judge provides scalable evaluation, but it should not be treated as equivalent to human evaluation.

---

## 16. Human Agreement

Human annotation was used to assess consistency of the intent taxonomy.

Exact agreement:

[VALUE]

Cohen's Kappa:

[VALUE]

Human agreement provides evidence about how consistently the intent categories can be assigned.

---

## 17. Top 5 Failure Modes

### Failure Mode 1 — Intent Confusion

The classifier may confuse closely related support categories, particularly technical issues, software glitches, device issues and update-related problems.

Real example:

[INSERT REAL EXAMPLE FROM STEP 95]

Why it failed:

The customer message contains vocabulary shared by multiple support intents.

Possible improvement:

Use stronger semantic embeddings and improve the intent taxonomy using additional human-labelled examples.

---

### Failure Mode 2 — Low Retrieval Similarity

Some customer messages do not have sufficiently similar historical examples.

Real example:

[INSERT REAL EXAMPLE FROM STEP 95]

Why it failed:

The historical retrieval system is based on lexical TF-IDF similarity and may fail when the same problem is expressed using different wording.

Possible improvement:

Use sentence embeddings or a neural retrieval model.

---

### Failure Mode 3 — Generic Generated Response

The generation model may produce a response that is too generic or repetitive.

Real example:

[INSERT REAL EXAMPLE FROM STEP 95]

Why it failed:

FLAN-T5-small has limited generation capability and may not fully use the retrieved context.

Possible improvement:

Use a stronger instruction-tuned model and improve prompt structure.

---

### Failure Mode 4 — Weak Grounding

Some generated replies may not be strongly supported by the retrieved historical evidence.

Real example:

[INSERT REAL EXAMPLE FROM STEP 95]

Why it failed:

The retrieved examples may not contain enough information to answer the current customer question.

Possible improvement:

Require stronger evidence before generation and escalate when evidence is insufficient.

---

### Failure Mode 5 — Escalation Decision Error

The similarity threshold may incorrectly auto-handle or escalate certain messages.

Real example:

[INSERT REAL EXAMPLE FROM STEP 95]

Why it failed:

A single similarity threshold does not capture all dimensions of support risk.

Possible improvement:

Tune the threshold on a development set and combine similarity, intent confidence and risk-based rules.

---

## 18. What Is Misleading About My Headline Number?

The headline classification number can make the system appear stronger than it actually is.

The evaluation uses a 200-example golden set, which is relatively small compared with the original dataset.

The intent taxonomy was initially created using unsupervised clustering and therefore contains human-defined interpretations of discovered clusters.

In addition, weighted F1 summarizes intent classification performance but does not measure the quality of generated responses, retrieval quality, or escalation safety.

Therefore, the headline number should not be interpreted as an end-to-end customer-support success rate.

A stronger evaluation would use a larger independently labelled test set and separately evaluate classification, retrieval, response quality and escalation safety.

---

## 19. What I Would Do Next Week

The next improvements would be:

1. Replace TF-IDF retrieval with sentence embeddings.
2. Improve the intent taxonomy using more human annotations.
3. Tune the escalation threshold using a dedicated development set.
4. Use a stronger open-source instruction-tuned language model.
5. Add confidence calibration for intent predictions.
6. Add explicit risk categories for billing, account access and sensitive issues.
7. Build a larger independently labelled evaluation set.
8. Perform more systematic human evaluation of generated replies.
9. Add duplicate and near-duplicate detection to reduce retrieval leakage.
10. Package the system as a simple API or web application.

---

## 20. Decision Log

The major non-obvious design decisions are documented separately in:

step97_decision_log.csv

Important decisions included:

- Selecting AppleSupport based on conversation volume.
- Using a representative sample for clustering.
- Choosing eight intent categories.
- Using KMeans for initial intent discovery.
- Using TF-IDF for classification.
- Using Logistic Regression as the classifier.
- Creating a 200-example golden set.
- Removing golden examples from retrieval.
- Using TF-IDF cosine similarity for retrieval.
- Using FLAN-T5-small for local generation.
- Grounding responses using historical cases.
- Using retrieval similarity for escalation.
- Using human annotation for evaluation.
- Using LLM-as-judge for response evaluation.
- Comparing against baseline systems.

---

## 21. How to Run

### Install dependencies

```bash
pip install pandas numpy scikit-learn transformers sentencepiece accelerate torch tqdm
