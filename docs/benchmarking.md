# Benchmarking

Although the evaluation of style transfer quality is inherently subjective, we introduced a partially objective assessment framework for our style transfer pipeline.

We compute the model's win rate using a standard benchmark dataset introduced by Kitov et al.,

"STYLE TRANSFER DATASET: WHAT MAKES A GOOD STYLIZATION?"

This dataset contains:

* 50 content images

* 50 style images

* Style-transferred outputs for all possible content–style combinations (Provided in four resolutions, though size variations are not critical for our purposes)

We refer to a combination of a content image, a style image, and the resulting stylization as a triplet.

To estimate win rate across triplets, we use Gemini 2.0 Flash as an automated evaluator due to its API accessibility and ease of integration.

## Evaluation Method

The generated image is compared against the "Ground Truth" stylization.

*   **Judge**: Gemini 2.0 Flash
*   **Criteria**: Which candidate better applies the style of the reference image to the content image?
*   **Metric**: Win rate of our pipeline against the ground truth.

## Results

We ran the evaluation on a subset of 200 triplets and found that our pipeline outperformed the ground truth in ~61.5% of cases.

---

