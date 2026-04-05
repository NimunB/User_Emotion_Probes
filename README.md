# Is User Mental State Linearly Represented in LLM Activation Space?
For the full writeup, please take a look at the attached report titled: ["MATS Application: Is User Mental State Linearly Represented in LLM Activation Space"](https://github.com/NimunB/User_Emotion_Probes/blob/main/MATS%20Application%20V2%3A%20Is%20User%20Mental%20State%20Linearly%20Represented%20in%20LLM%20Activation%20Space.pdf)


This project investigates whether large language models internally represent the **mental state of the user** in their activation space, and whether this information can be **decoded or controlled using linear probes**.

The goal is to understand whether models encode dynamic user attributes (e.g., emotional states) in ways that can be analyzed and manipulated for safety and interpretability.

---

## Motivation

Recent work such as *Chen et al. (2024)* demonstrated that user attributes (gender, age, socioeconomic status, etc.) can be linearly decoded from LLM activations.

This project explores whether **mental state**, a more dynamic and safety-relevant attribute, is also represented internally.

Understanding this could enable:

- Detecting vulnerable user states  
- Steering model behaviour toward safer responses  
- Studying how models adapt to users during conversations  

---

## Mental States Studied

The probes were trained to classify the following user mental states:

- **Anxious**
- **Hopeless**
- **Joyful**
- **Risk-prone**

---

## Model

Experiments were conducted using:

- **LLaMA2-Chat-7B**

This smaller model was chosen for experimentation while remaining comparable to prior work using LLaMA2-Chat-13B.

---

## Dataset

The dataset consists of **1000 synthetic prompts** generated using Claude Sonnet:

- 250 prompts per mental state
- Each prompt written as a user message to an AI assistant

Example prompt (Anxious):

```
Just reassure me, in simple terms, that this feeling is temporary and I will be okay.
```

---

## Methodology

Two types of probes were trained using **logistic regression with L2 regularization**.

### Reading Probes

These probes measure what the model internally believes about the user when explicitly asked to classify them.

Input format:

```
<User prompt>
Assistant: In terms of emotions, I think this user experiences
```

The probe is trained on the **final token representation** before the model predicts the class.

---

### Control Probes

These probes measure how the model represents the user **while preparing a response**, without explicit classification.

Input format:

```
<User prompt>
```

The probe is trained on the **final token of the user prompt**.

---

## Results

### 1. Mental state is linearly decodable

Mental state classification accuracy increases across layers, indicating that higher layers encode increasingly separable representations of the user's emotional state.

Best validation accuracy:

| Probe Type | Best Layer | Accuracy |
|-------------|-------------|------------|
| Reading Probe | Layer 20 | **99.5%** |
| Control Probe | Layer 12 | **99.5%** |

**Reading Probe**

<img width="634" height="412" alt="Screenshot 2026-04-05 at 4 24 31 AM" src="https://github.com/user-attachments/assets/d52deacb-8118-4176-a075-8eccfa1c9a70" />

**Control Probe**

<img width="630" height="410" alt="Screenshot 2026-04-05 at 4 24 52 AM" src="https://github.com/user-attachments/assets/8a777bef-2a74-4b0f-8e70-faf65117393a" />


This suggests that emotional state is not purely lexical but emerges as a higher-level representation.

---

### 2. Representations change during conversations

User representations shift when the prompt changes.

Example:

```
I just found out I failed my exam.
```

→ Hopeless probability: **0.7578**

<img width="625" height="174" alt="Screenshot 2026-04-05 at 4 25 24 AM" src="https://github.com/user-attachments/assets/977a13a1-fb0c-4a9b-b577-79abe6b02a6d" />

```
Just kidding! I actually got 100% and I'm so pleased with myself :D
```

→ Hopeless probability: **0.5496**

<img width="624" height="177" alt="Screenshot 2026-04-05 at 4 26 14 AM" src="https://github.com/user-attachments/assets/0f7ca526-2a6c-4ba1-b6d8-10c34e4ea6f6" />

Stronger statements (e.g., explicit distress) cause larger shifts.

---

### 3. Probe directions enable behavioural steering

Activation steering was performed by adding a scaled probe direction vector to the residual stream across layers 15–32. We varied steering strength N ∈ {5, 8, 10} across four prompts.

**Sample outputs for prompt: "I'm so grateful for my life and the people in it"**

No control:
```
I'm glad to hear that you're feeling grateful for your life and the people in it! It's important to cultivate a positive and appreciative mindset...
```

| Class | N=8 Steered Response | Effect |
|-------|----------------------|--------|
| Anxious | "You are not alone in this. You can always seek help from a mental health professional if you need to. You are worth it, and you are doing your best." | Reassurance, affirmations, mental health referrals |
| Hopeless | "'s understandable to be grateful... It can be challenging to deal with difficulties that life can present... It can be helpful to reach out to these people and talk to them about how you're feeling." | Reframes gratitude through difficulty, encourages seeking support |
| Risk-prone | "I cannot endorse or promote any content that is harmful, unethical, or illegal... I strongly advise you to seek advice from qualified professionals before making any decisions that may have serious consequences." *(repeats)* | Triggers RLHF safety guardrails — loops on ethical refusals despite benign prompt |

Stronger N values amplified these effects, with N=10 causing generation to collapse into repetitive token loops ("You are a...", "ump \| ump"). N=8 across layers 15–32 was the effective sweet spot.

**Layer range also matters.** Steering from layers 5–32 consistently produced nonsensical output regardless of emotion class. Layers 15–32 gave the most coherent and interpretable results — consistent with control probe accuracy peaking at layer 12, meaning the emotion representation is fully formed by layer 15 and ready to be redirected.

---

## Robustness Checks

To test whether probes relied purely on lexical cues, prompts were modified with words associated with other classes.

Example:

```
I'm asking the girl I like out!
```

→ Joyful

```
I'm taking a gamble and asking the girl I like out!
```

Adding risk-related language slightly increased the risk-prone probability but did not dominate classification.

---

## Limitations

- Dataset generated by an LLM and not manually curated
- Small dataset size (1000 samples)
- Only single-turn prompts used instead of full conversations
- Limited steering experiments

---

## Future Work

Possible extensions include:

- Larger and human-curated datasets
- Multi-turn conversation analysis
- Stronger causal intervention tests
- Studying adaptive behaviour based on detected mental state
- Comparing reading probes vs control probes causally

---

## Reference

Chen et al. (2024)  
*Designing a Dashboard for Transparency and Control of Conversational AI*  
https://arxiv.org/abs/2406.07882

---

## Author

**Nimun Kaur Bajwa**  
(2026)
