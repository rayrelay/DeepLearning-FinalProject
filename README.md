# Project Report
# Stable Diffusion Image-to-Prompt Prediction

---

## 1. Problem Statement and Data Overview

### Challenge Background

The goal of the Kaggle competition [Stable Diffusion Image to Prompts](https://www.kaggle.com/competitions/stable-diffusion-image-to-prompts) is to predict the corresponding text prompts for images generated by the generative deep learning model (Stable Diffusion 2.0).\
The essence of this task is **multi-modal semantic alignment**: learning the mapping relationship between image and text embeddings to improve the interpretability and controllability of generative models.

### Data Description

- **Training Data** (prompts.csv): Contains image IDs (imgId) and corresponding prompts.
  - Example Prompt: "hyper realistic photo of very friendly and dystopian crater"
- **Image Data**: Generated by Stable Diffusion 2.0, originally sized 768x768, downsampled to 512x512, in PNG format.
- **Test Data**: Approximately 16,000 hidden images requiring predicted prompt embeddings.

### Data Scale and Structure

| Data Component             | Quantity/Dimension             | Description                                          |
| -------------------------- | ------------------------------ | ---------------------------------------------------- |
| Training Images            | Matches entries in prompts.csv | Several thousand (exact number undisclosed)          |
| Prompt Embedding Dimension | 384 dimensions                 | Encoded using sentence-transformers/all-MiniLM-L6-v2 |
| Test Images                | \~16,000 images                | Final evaluation based on this set                   |

---

## 2. Exploratory Data Analysis (EDA)

### Data Investigation and Visualization

1. **Prompt Analysis**

   - **Word Frequency Statistics**: A word cloud visualizes high-frequency words (e.g., "donut," "rose," "hyper").
   - **Length Distribution**: The average prompt length is 14.43 words, with a maximum of 23 words and a minimum of 9 words.

2. **Image Analysis**

   - **Size Validation**: All images are 512x512, with no missing or corrupted files.
   - **Style Diversity**: Covers landscapes, portraits, abstract art, and more.

### Data Cleaning and Preprocessing

- **Missing Value Handling**: No missing images or prompts.
- **Text Cleaning**: Special character removal, lowercase normalization, retention of key modifiers (e.g., "4k," "octane render").
- **Embedding Alignment**: Using a pre-trained text encoder to generate fixed-dimension (384) embedding vectors.

### Analysis Plan

1. **Model Selection**: Compare ResNet (local feature extraction) with ViT (global attention).
2. **Hyperparameter Optimization**: Adjust learning rate, dropout rate, etc., via random search.
3. **Evaluation Metrics**: Validation loss (MSE) and cosine similarity (between predicted and actual embeddings).

---

## 3. Model Architecture and Training

### Model Comparison and Selection

| Model                  | Key Features                                                                                 | Suitable Scenario                                       |
| ---------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| **ResNet50**           | Residual connections mitigate gradient vanishing, excels in local feature extraction         | General image classification, moderate complexity tasks |
| **Vision Transformer** | Self-attention mechanism captures global dependencies, ideal for complex pattern alignment   | High-resolution images, multi-object scenes             |
| **Tuned ResNet**       | Optimized hyperparameters (learning rate, dropout) for balance between speed and performance | Fast iteration in resource-constrained environments     |

### Training Configuration

- **Loss Function**: Mean Squared Error (MSE), minimizing the distance between predicted and actual embeddings.
- **Optimizer**: Adam (ResNet/ViT) and SGD (for comparison in fine-tuned ResNet experiments).
- **Hyperparameter Tuning**:

```python
hyperparam_space = {
    'lr': [1e-5, 3e-5, 1e-4, 3e-4],          # Learning rate
    'batch_size': [16, 32, 64],              # Batch size
    'dropout_rate': [0.2, 0.3, 0.4],         # Dropout probability
    'weight_decay': [1e-5, 1e-4, 0.0],       # Weight decay
    'optimizer': ['adam', 'sgd']             # Optimizer type
}
```

### Training Results Visualization

- **Loss Curve Comparison**:
  - ViT converges faster with lower validation loss (final MSE=0.0030), indicating that the global attention mechanism is more suited for complex semantic alignment.
  - Tuned ResNet mitigates overfitting risks with an optimized learning rate (3e-5) and dropout (0.2).

---

## 4. Results and Analysis

### Performance Comparison

| Model              | Validation MSE | Training Epochs |
| ------------------ | -------------- | --------------- |
| ResNet50           | 0.003841       | 15              |
| Vision Transformer | 0.003017       | 15              |
| Tuned ResNet       | 0.006454       | 20              |

- ViT still achieves the lowest MSE, reinforcing its superiority in global feature extraction.
- Tuned ResNet performs worse than the baseline ResNet50, indicating potential overfitting or suboptimal hyperparameter selection.

### Failure Case Analysis

- Low similarity samples:
  - **Input Image**: "A surrealist painting of melting clocks in a desert"
  - **Predicted Output**: "A desert landscape with rocks and sunset" (Similarity 0.28)
  - **Reason**: Abstract art styles cause semantic ambiguity, making it difficult for the model to capture details.

### Hyperparameter Influence

- **Learning Rate (LR)**: ViT training was unstable at LR=1e-4; reducing to 3e-5 stabilized convergence.
- **Batch Size**: A batch size of 32 sped up training for ViT but increased loss fluctuations.
- **Dropout**: Dropout=0.3 in ResNet reduced validation loss by 12%, effectively mitigating overfitting.

---

## 5. Conclusion and Improvement Suggestions

### Conclusion

1. **ViT remains the best model**: It achieves the lowest validation MSE (0.003017), confirming its strong performance in capturing complex patterns.
2. **Tuned ResNet underperforms**: Unlike previous results, tuning led to an increase in MSE, possibly due to overfitting.
3. **Data Constraints**: The limited dataset size may limit generalization, requiring further augmentation strategies.

### Improvement Suggestions

- **Model-Level Enhancements**:
  - Experiment with larger ViT variants (e.g., ViT-Large) or CLIP pre-trained models.
  - Introduce contrastive learning to enhance embedding space alignment.
- **Data-Level Enhancements**:
  - Generate more diverse prompt-image pairs.
  - Use data augmentation (e.g., random cropping, color perturbation).
- **Deployment Optimization**:
  - Model lightweighting (knowledge distillation) for real-time inference.

### Failure Reflections

- **ResNet Limitations**: Local convolution operations struggle with long-range dependencies, leading to prediction deviations for complex prompts.
- **Hyperparameter Sensitivity**: Overfitting risks require more robust validation strategies.

---

### Project Code and Visualization Results:

- GitHub Repository: [https://github.com/rayrelay/DeepLearning-FinalProject](https://github.com/rayrelay/DeepLearning-FinalProject)

Through this project, we gained deep insights into the challenges of multi-modal alignment and validated the superiority of Transformer architectures in generative tasks. The next step will explore multi-task learning and integrating larger-scale pre-trained models.
