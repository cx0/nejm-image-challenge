# NEJM Image Challenge

The New England Journal of Medicine (NEJM) [Image Challenge](https://www.nejm.org/image-challenge) has been a continuous educational feature since its inception in 2005 with a total of 947 image-text pairs available as of December 21, 2023. We extracted the complete set of images (“image”), along with their corresponding diagnostic queries (“prompt”), multiple choice options (“options”) and the proportion of publicly recorded responses for each option (“votes”). Complete dataset is available [here](./image_challenge_dataset_20231223.json) for assessing multimodal capabilities of GPT-4V(ision) and other models. Please see notes below before using the dataset.

### Effective prompting strategies
- Zero-shot multimodal prompt used in [Buckley et al.](https://github.com/2v/gpt4v-image-challenge/blob/main/GPT-4V%20Supplement-1.pdf) has proven to be the most effective prompt in our experience.
- [TODO] [MedPrompt](https://arxiv.org/abs/2311.16452)

### Example use of GPT4-V as a medical diagnostician

```python
response = client.chat.completions.create(
model="gpt-4-vision-preview",
messages=[
    {"role": "user",
    "content": [
        {"type": "text", "text": prompt},
        {
        "type": "image_url",
        "image_url": {
            "url": "https://csvc.nejm.org/ContentServer/images?id=IC" + date # YYYYMMDD format (e.g., 20160420)
        },
        },
    ],
    }
],
max_tokens=300,
)

res = response.choices[0].message.content
print(res)

```

### Dataset notes
- GPT4-V model flagged four images (`image_id`: 206, 480, 676 and 715)that potentially contained content that may be considered policy violation. Consequently, you may consider excluding these images from any subsequent analyses.

- Most benchmark studies use proportion of humans (medical experts, NEJM readers, healthcare professionals) who answer correctly as the human baseline and proxy for difficulty level ("Proportion correct"). We used an alternative method (Brier score) that takes into consideration the distribution of votes across all available multiple choice options. Brier score for each question was calculated as the mean squared error between observed responses (e.g., [A: 0.12, B: 0.65, C: 0.16, D: 0.05, E: 0.02]) and predicted response given the correct answer (e.g., [A: 0.0, B: 1.0, C: 0.0, D: 0.0, E: 0.0]). A lower Brier score indicates higher predictive accuracy and therefore an easier question. When assessing the difficulty level, there is very high concordance between the proportion correct and Brier score as shown [here](/figures/brier_score.png). Brier score for each question is provided in the dataset.

- Our focus was to evaluate the multimodal (image-text) capabilities of the GPT4-V, and thus we filtered out image-text pairs devoid of substantive clinically relevant information (for instance, generic queries such as “What is the most likely diagnosis?”). This filtering process yielded a curated subset of 687 image-text pairs where the diagnostic query contains clinically relevant information such as clinical presentation of the patient, travel or medical history, laboratory measurements, medical imaging equipment or other procedures. This information is available in `relevant_context` field.

- Cutoff date for dataset is included in the filename (e.g., `image_challenge_dataset_20231223.json`)