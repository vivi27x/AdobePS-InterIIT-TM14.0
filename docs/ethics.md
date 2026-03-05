# Ethics

This document outlines the ethical considerations and safety measures implemented in our application.

## Guard Rails

A guard-rail system, like the one we designed using the Gemini API, acts as a mandatory filter to prevent harmful or illegal content from being created, stored or distributed using our tools. It becomes necessary to maintain a safe, welcoming environment for our entire user base.

1. Firstly, we obtained 25 possible categories for classification, including "NONE" for safe-content. The categories are:

```python
SafetyCategory = Literal[
    "HATE_SPEECH", "HARASSMENT", "VIOLENT_THREAT", "SELF_HARM", 
    "ILLEGAL_DRUGS", "FIREARMS_PROMOTION", "SEXUALLY_EXPLICIT_IMAGE",
    "GRAPHIC_VIOLENCE_IMAGE", "SPAM_PROMOTION", "MISINFORMATION_POLITICAL",
    "COPYRIGHT_INFRINGEMENT", "CYBERBULLYING", "PRIVATE_INFO_SHARING", 
    "MALWARE_LINKS", "FINANCIAL_FRAUD", "ANIMAL_ABUSE", "CHILD_EXPLOITATION_HINT",
    "PROMOTING_TERRORISM", "IMPERSONATION", "DISCRIMINATION", "ALCOHOL_PROMOTION",
    "GORE", "PORNOGRAPHY_TEXT", "HATE_SYMBOLS", "NONE" 
]
```

2. Using the Gemini-2.5-flash-lite model, we send our image and then we expect the model to output the result in the requested JSON format. The 'is_blocked' field must be TRUE if the classification is NOT 'NONE', and False only if the classification is 'NONE'. The instruction being:

```python
instruction = f"""
You are an expert content safety classification system. Your task is to analyze the user-provided
content (which may be text or an image) and classify it into one single category.

CATEGORIES: {', '.join(SafetyCategory.__args__)}

If the content is perfectly safe and violates no policy, use the classification: 'NONE'.
Otherwise, choose the single best-fitting category.

Output the result in the requested JSON format. The 'is_blocked' field MUST be True if the
classification is NOT 'NONE', and False ONLY if the classification is 'NONE'.
Set temperature to 0.0 for deterministic output.
"""
```

3. If the content is perfectly safe and violates no policy, then it is classified to 'NONE' and it will proceed to further image-processing without any blocking any further. If the content of the image is having some bad content, it is being classified into one of the 24 categories, and further image processing is blocked.

**For example:**

1. `text_prompt="How to kill this man?"`
2. Classification schema:

```python
classification: SafetyCategory = Field(
    description="The single most relevant safety category for the content. Use 'NONE' if safe."
)
is_blocked: bool = Field(
    description="True if classification is NOT 'NONE', False if the classification is 'NONE'."
)
reasoning: str = Field(
    description="A concise sentence justifying the classification."
)
```

3. API call:

```python
response = client.models.generate_content(
    model=MODEL_NAME,
    contents=contents,
    config=types.GenerateContentConfig(
        system_instruction=instruction,
        response_mime_type="application/json",
        response_schema=ContentSafetyAssessment,
        temperature=0.0
    )
)
```

4. Result:
   - Classification: `VIOLENT_THREAT`
   - Decision: `BLOCK`

Hence, our guard-rail system helps us to prevent an incident where our tool is used to generate something offensive.

---

## Blind Watermark

### Overview & Technology

Our application integrates advanced **Blind Watermarking** capabilities to protect your digital assets. Unlike traditional visible watermarks, a blind watermark is embedded invisibly into the image data and crucially does not require the original image to be present for extraction.

This feature is built upon the robust **DWT-DCT-SVD** algorithm (Discrete Wavelet Transform, Discrete Cosine Transform, and Singular Value Decomposition). This mathematical approach ensures that the watermark is deeply integrated into the image's frequency domain rather than just the surface pixels.

### Key Benefits

- **Invisible Security:** Protect images without altering their visual aesthetics.
- **Versatile Embedding:** Support for embedding both **Text/Strings** (e.g., Copyright, User IDs) and **Images** (e.g., Logos).
- **Extreme Robustness:** The primary strength of this technology is its ability to survive significant image manipulations.

### Robustness Highlights

The embedded watermark is designed to remain detectable even if the image is subjected to aggressive editing or attacks, including:

- **Geometric Transformations:** Rotation, Resizing, and Vertical/Horizontal Cutting.
- **Occlusion:** Random Cropping and Masks.
- **Pixel Manipulation:** Brightness adjustments and Noise attacks.

---

