
### Readme on Thoracic Modelling Overview

This project builds a **multi-label chest X-ray classifier** that predicts the presence of several thoracic findings (e.g., effusion, atelectasis, pneumothorax) from frontal radiographs. The model is explicitly positioned as a **clinical decision-support tool**, not an autonomous diagnostic system.

#### Why this modeling approach?

**Business problem.** Radiology departments are under significant pressure: rising imaging volumes, workforce shortages, and turnaround-time targets. Many chest X-rays are normal or show common, low-risk patterns, while a minority contain critical findings that require urgent attention. A model that can **triage and prioritize** studies has clear operational and patient-safety value:

- Surface potentially critical cases earlier in the worklist.
- Reduce time spent on clearly normal or low-risk studies.
- Provide a consistent, reproducible “second pair of eyes” for subtle findings.

**Technical choice.** We use **DenseNet121** with ImageNet pretraining as the backbone because:

- It is a **well-established convolutional architecture** with strong performance on medical imaging benchmarks.
- Pretraining on ImageNet provides a strong starting point, allowing the model to learn useful features even with noisy labels.
- Dense connections improve gradient flow and parameter efficiency versus simpler CNNs, which is valuable on large, high-resolution image data.

The head on top of DenseNet121 is intentionally simple (global pooling → small dense layer → sigmoid outputs). This keeps the system **interpretable and maintainable**, and reduces overfitting risk on noisy labels.

#### How the model works (at a high level)

1. **Input.** A frontal chest X-ray is resized to 224×224 and normalized using DenseNet’s `preprocess_input` (aligned with ImageNet pretraining).  
2. **Feature extraction.** DenseNet121 transforms the image into a compact feature representation capturing textures, shapes, and patterns relevant to lung fields, mediastinum, and pleural spaces.  
3. **Classification head.** A lightweight dense layer plus dropout converts these features into **per-disease probabilities** (one sigmoid neuron per label). The output is a vector like:

   \\[P(\\text{Effusion}), P(\\text{Atelectasis}), \\dots, P(\\text{No Finding})\\]

4. **Training signal.** The model is optimized with **binary cross-entropy** for each label, using noisy, report-derived ground truth. We train two variants:

   - **Model A (frozen DenseNet):** Only the top classification layers are trained, using DenseNet as a fixed feature extractor.  
   - **Model B (fine-tuned DenseNet):** The entire backbone is updated with a smaller learning rate.

   A **ReduceLROnPlateau** scheduler monitors validation AUC and automatically decreases the learning rate when improvement stalls, providing a more stable convergence without manual retuning.

5. **Selection & evaluation.** On a **patient-level** held-out test set, we compare the two models by validation AUC, then evaluate the best model using:

   - Global metrics (loss, binary accuracy, AUC)  
   - **Per-label AUROC**  
   - **Macro-average AUROC** across labels  

The current baseline achieves a **macro-average AUROC of ~0.72**, meaning it consistently ranks positive cases above negative ones substantially better than random (0.5), especially for common findings. This is a **strong baseline** given noisy labels and limited tuning.

#### Business impact scenarios

With appropriate clinical controls, this model can support several high-value workflows:

- **Worklist prioritization.** Flag studies likely to contain critical findings (e.g., effusion, pneumothorax) so radiologists see them earlier in their queue.  
- **Quality and safety net.** Provide a secondary signal that can be used for “double-check” rules (e.g., re-review if model score is high but report is normal).  
- **Volume management.** Help staff triage high volumes of mostly normal exams, focusing human attention where it’s most needed.  
- **Population-level analytics.** Aggregate predictions longitudinally to identify trends in disease prevalence across sites, patient cohorts, or time periods.

These are **supportive** use cases: they enhance human performance and operational efficiency without replacing clinical judgment.

#### Controls, limitations, and risk management

Because this is a medical-imaging model, we assume and communicate clear boundaries:

- **Weak labels.** Training labels come from radiology reports and may contain omissions or inaccuracies. The model is calibrated as a **signal**, not an oracle.  
- **Class imbalance & rare findings.** Some diseases are under-represented, so their individual AUROC may be lower or unstable. We report **per-label AUROC** and macro-average AUROC rather than a single headline number.  
- **Bias and confounding.** Demographic skews (age, sex) and acquisition differences (PA vs AP views) can introduce bias. We partially mitigate this with patient-level splits and careful EDA, but further fairness and subgroup analysis is recommended before deployment.  
- **Human-in-the-loop.** Any production usage requires:
  - Radiologist oversight and final decision-making.  
  - Clear UI that presents model predictions as **advisory**, not definitive.  
  - Ongoing monitoring of model performance and drift, with periodic recalibration.

#### Why this matters

From an executive perspective, this project demonstrates that:

- A **modern CNN backbone (DenseNet121)** can be wrapped in a pragmatic pipeline (EDA → preprocessing → patient-level splitting → modeling → evaluation) that produces **clinically relevant signal** from chest X-rays.  
- Even with noisy labels and conservative training, the model reaches **meaningful discrimination (~0.72 macro-AUROC)**, providing a strong starting point for triage and decision-support applications.  
- The design intentionally balances **performance, interpretability, and risk**, making it suitable as a pilot for further experimentation, stakeholder feedback, and eventual integration into radiology workflows under proper governance.
