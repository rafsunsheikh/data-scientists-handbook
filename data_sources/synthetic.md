# Synthetic Data

> **Stub — contributions welcome.**

## What to cover

- When synthetic data is appropriate: testing pipelines, privacy-preserving sharing, augmenting rare classes, stress-testing models.
- Rule-based generators: `Faker`, `Mimesis`, `mockaroo`.
- Statistical: sampling from a fitted distribution, copulas.
- Generative models: GANs, VAEs, diffusion, LLMs for text, tabular generators (CTGAN, TVAE).
- Simulation: agent-based (Mesa), discrete-event (SimPy), physics (Gazebo, Isaac Sim).
- Augmentation as synthetic data: image transforms (Albumentations), text paraphrasing, audio noise injection.
- Differential privacy: synthetic data with formal guarantees.
- Evaluation: utility (does training on synthetic preserve downstream accuracy?), privacy (membership inference resistance), fidelity (column-wise and joint distributions).
- Pitfalls: distributional gaps that look fine until production, leaked training-data memorization.
- When *not* to use synthetic data (and to gather real instead).
