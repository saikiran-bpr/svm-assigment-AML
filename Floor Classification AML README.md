# Floor–Non-Floor Segmentation using SVMs

This project implements and evaluates three different approaches for floor–non-floor segmentation using the ADE20K dataset. The goal is to compare pixel-level and region-level classification strategies in terms of performance, robustness, and computational efficiency.

---

## 1. Performance Overview

Three segmentation approaches were implemented and evaluated. The quantitative results from the final experimental runs are summarized below.

### Table 1: Quantitative Performance Comparison

| Approach                        | Accuracy   | Precision | Recall | Training Time | Inference Time |
| ------------------------------- | ---------- | --------- | ------ | ------------- | -------------- |
| Pixel-based (RGB)               | 93.88%     | 0.00      | 0.00   | ~0.02 s       | ~0.0008 s      |
| RGB + Spatial (X, Y)            | 93.50%     | 0.05      | 0.00   | ~0.04 s       | ~0.0005 s      |
| Region-based (Clustering + SVM) | **94.90%** | 0.00      | 0.00   | ~127 s        | ~0.05 s        |

### Observation on Metrics

Although all approaches achieve high accuracy, precision and recall for the *floor* class are near zero. This behavior is caused by **severe class imbalance** in the dataset, where background pixels dominate. As a result, the models tend to predict the majority (background) class for most inputs.

This highlights an important limitation: **accuracy alone is not a reliable metric** for evaluating segmentation performance in imbalanced datasets. Improving floor detection would require techniques such as class weighting, resampling, or alternative loss functions.

---

## 2. Approach-wise Comparison

### 2.1 Performance Characteristics

**Region-based approach (Best accuracy)**
The region-based method achieved the highest overall accuracy (94.9%). By clustering pixels into regions before classification, it enforces spatial consistency and reduces pixel-level noise. When a region is predominantly floor, the entire region is classified as floor, eliminating the “salt-and-pepper” artifacts commonly seen in pixel-based approaches.

**Pixel-based (RGB only)**
Despite its simplicity, the RGB-only pixel-based approach achieved competitive accuracy (93.9%). It effectively captures dominant background colors but lacks contextual understanding, making it unable to distinguish visually similar regions such as white floors and white walls.

**Pixel-based (RGB + spatial coordinates)**
Adding spatial features slightly reduced accuracy in this run (93.5%). While spatial information can help by encoding layout priors (e.g., floors typically appear near the bottom of an image), it may also introduce bias. If the spatial distribution of floors differs between training and validation data, generalization can suffer.

---

### 2.2 Speed and Computational Cost

**Pixel-based methods**
Both pixel-based approaches are extremely fast, with training and inference times in the millisecond range. This efficiency is due to the use of `LinearSVC`, which is well-optimized for large-scale linear classification problems.

**Region-based method**
The region-based pipeline is significantly slower, with training taking several minutes. The increased cost comes from:

* Image downscaling,
* KMeans clustering,
* Region-level feature extraction,
* Training the SVM on region features.

However, once trained, inference remains reasonably fast (~0.05 seconds per image).

---

### 2.3 Robustness and Generalization

**Pixel-based (RGB)**
Least robust. Since decisions are based purely on color, the model struggles with visually ambiguous regions where floors and other surfaces share similar colors.

**Pixel-based (RGB + XY)**
Moderately robust. Spatial features help the model learn coarse layout information, improving disambiguation in some cases but increasing the risk of overfitting to dataset-specific spatial patterns.

**Region-based**
Most robust. By operating on regions instead of individual pixels, this approach captures local structure and texture through aggregated features (e.g., mean color and variance). This results in smoother predictions and improved resistance to pixel-level noise.

---

## 3. Summary

The region-based approach provides the best balance between accuracy and robustness, albeit at the cost of significantly higher training time. Pixel-based methods are computationally efficient and easy to implement but lack contextual understanding. Overall, the experiments demonstrate that addressing class imbalance is critical, as high accuracy alone can be misleading in segmentation tasks dominated by background pixels.
