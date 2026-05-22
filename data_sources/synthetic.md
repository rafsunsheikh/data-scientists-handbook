# Synthetic Data

> **TL;DR** — Synthetic data is artificially generated data that mimics the statistical properties of real data without containing any actual records. It serves four primary purposes: privacy-preserving data sharing, testing and CI pipelines, augmenting rare classes, and stress-testing models. The quality spectrum ranges from simple rule-based generators (random sampling from distributions) to complex generative models (GANs, VAEs, diffusion models, LLMs). The key challenge is ensuring that synthetic data preserves the joint distributions and relationships that matter for your use case — superficially similar distributions can hide critical structural differences. Always evaluate synthetic data on utility (does training on it produce a good model?), fidelity (how close are the distributions?), and privacy (can you reverse-engineer the original data?).

## 1. When synthetic data is appropriate

| Use case | Why synthetic? | Risk |
|---|---|---|
| **Privacy-preserving sharing** | Share data without exposing PII | Distributional gap between synthetic and real |
| **Testing / CI pipelines** | Deterministic, repeatable test data | Real-world data may have patterns not in synthetic |
| **Augmenting rare classes** | Generate more examples of under-represented classes | May not capture full diversity of rare class |
| **Model stress-testing** | Generate edge cases and adversarial examples | Edge cases may be unrealistic |
| **Benchmarking** | Known ground truth for evaluation | Synthetic benchmarks may not reflect real performance |
| **Pre-training** | Large-scale pre-training before fine-tuning on real data | Pre-training on synthetic may hurt downstream performance |
| **When *not* to use** | — | Synthetic data can never capture unknown unknowns |

### When NOT to use synthetic data

- When real data is readily available and privacy is not a concern.
- When the downstream task depends on rare, complex interactions not captured by the generator.
- When regulatory requirements demand real data (e.g., clinical trials, financial audits).
- When the cost of generating high-quality synthetic data exceeds the cost of obtaining real data.

## 2. Rule-based generators

Generate data from hand-crafted rules and distributions.

### 2.1 Faker

`Faker` generates realistic-looking fake data: names, addresses, phone numbers, emails, company names, etc.

```python
from faker import Faker

fake = Faker()
fake.name()        # "John Doe"
fake.email()       # "john.doe@example.com"
fake.address()     # "123 Main St, Anytown, USA"
fake.text()        # "Lorem ipsum..."
fake.date()        # "2024-01-15"
fake.random_int(1, 100)  # 42
```

**Characteristics:**

- Fast, deterministic (seed-based).
- No relationships between fields (name is independent of address).
- Good for UI testing, seeding databases.
- Not suitable for ML training (no joint distributions).

### 2.2 Mimesis

Similar to Faker, with support for multiple locales and domain-specific data (medical, financial, legal).

### 2.3 Mockaroo

Web-based tool for generating CSV/JSON/SQL data with custom schemas.

### 2.4 Sampling from fitted distributions

Fit a distribution to real data and sample from it.

```python
import numpy as np
from scipy import stats

# Fit a distribution to real data
data = np.array([1.2, 2.3, 2.1, 3.5, 4.2, ...])
dist = stats.gamma
params = dist.fit(data)

# Sample from the fitted distribution
synthetic = dist.rvs(*params, size=1000, random_state=42)
```

**For multivariate data:**

```python
import numpy as np

# Fit a multivariate Gaussian
mean = np.mean(real_data, axis=0)
cov = np.cov(real_data, rowvar=False)

# Sample from multivariate Gaussian
synthetic = np.random.multivariate_normal(mean, cov, size=1000)
```

**Limitation:** Multivariate Gaussian assumes linear correlations and normality. Real data often has non-linear relationships and non-Gaussian marginals.

## 3. Statistical methods

### 3.1 Copulas

Copulas model the dependence structure separately from marginal distributions.

**How it works:**

1. Transform each marginal to uniform [0, 1] using the empirical CDF.
2. Fit a copula (Gaussian, t, Clayton, Gumbel, Frank) to the uniform data.
3. Sample from the copula.
4. Transform back to original scale using inverse CDF.

```python
from copulas.univariate import GaussianUnivariate
from copulas.multivariate import GaussianMultivariate

# Fit marginals
univariates = [GaussianUnivariate() for _ in range(num_columns)]
for col in range(num_columns):
    univariates[col].fit(data[:, col])

# Fit copula
copula = GaussianMultivariate(univariates)
copula.fit(data)

# Generate synthetic data
synthetic = copula.sample(1000)
```

**Advantages:**

- Captures non-Gaussian marginals.
- Models dependence structure.
- Computationally efficient.

**Disadvantages:**

- Assumes a specific copula family (may not fit the true dependence).
- Struggles with high-dimensional data (curse of dimensionality).
- Gaussian copula assumes symmetric dependence.

### 3.2 Bayesian networks

Model variables as a directed acyclic graph (DAG) with conditional probability distributions.

**Advantages:**

- Captures causal structure (if the DAG is correct).
- Handles mixed data types (categorical + continuous).
- Interpretable structure.

**Disadvantages:**

- Structure learning is NP-hard.
- Requires domain knowledge for correct DAG.
- Computationally intensive for large graphs.

## 4. Generative models

### 4.1 GANs (Generative Adversarial Networks)

Two networks compete: a generator creates fake data, a discriminator tries to distinguish real from fake.

**Architecture:**

```
Generator: noise → neural network → synthetic data
Discriminator: real data / synthetic data → real / fake
```

**Training:**

```
for each epoch:
    for each batch:
        # Train discriminator
        fake_data = generator(noise_batch)
        d_loss = -mean(log(discriminator(real_data))) - mean(log(1 - discriminator(fake_data)))
        discriminator.update(d_loss)

        # Train generator
        fake_data = generator(noise_batch)
        g_loss = -mean(log(discriminator(fake_data)))
        generator.update(g_loss)
```

**Variants:**

| Variant | Use case |
|---|---|
| **GAN** | Basic image generation |
| **DCGAN** | Deep CNN-based GAN (images) |
| **WGAN** | Wasserstein loss (more stable training) |
| **WGAN-GP** | WGAN with gradient penalty |
| **StyleGAN** | High-quality image generation |
| **CycleGAN** | Image-to-image translation |
| **Pix2Pix** | Conditional image generation |
| **CTGAN** | Tabular data (Conditional GAN) |
| **TableGAN** | Tabular data (GAN) |
| **TVAE** | Tabular data (VAE-based) |
| **CTAB-GAN** | Tabular data with conditional bias correction |

### 4.2 VAEs (Variational Autoencoders)

Encoder maps data to latent space (mean + variance). Decoder maps latent back to data. Trained with reconstruction loss + KL divergence.

**Advantages over GANs:**

- More stable training (no adversarial game).
- Continuous latent space (interpolation works).
- Deterministic encoding (same input → same latent).

**Disadvantages:**

- Blurry outputs (for images).
- May not capture sharp modes in the distribution.

### 4.3 Diffusion models

Iteratively add noise to data, then learn to reverse the process.

**How it works:**

1. **Forward process:** Gradually add Gaussian noise over T timesteps until data is pure noise.
2. **Reverse process:** Train a neural network to predict and remove the noise at each step.

**Advantages:**

- State-of-the-art image generation (DALL-E, Stable Diffusion, Midjourney).
- Better mode coverage than GANs (doesn't collapse to single mode).
- High-quality outputs.

**Disadvantages:**

- Slow generation (requires many reverse steps).
- Computationally expensive to train.
- Overkill for tabular data (GANs/VAEs usually sufficient).

**Tabular diffusion:**

- **Diffusion-CTAB** — Diffusion models for tabular data.
- **TDF** (Tabular Diffusion Model) — Conditional diffusion for tabular data.

### 4.4 LLMs for text

Large language models can generate synthetic text data.

**Methods:**

- **Prompt-based generation:** "Generate 100 customer service dialogues about billing issues."
- **Fine-tuned generation:** Fine-tune an LLM on domain-specific text.
- **Back-translation:** Translate text to another language and back (for NLP data augmentation).
- **Paraphrasing:** Use an LLM to paraphrase existing text.

**Python:**

```python
from transformers import pipeline

generator = pipeline("text-generation", model="gpt2")
synthetic = generator("Customer service: I have a problem with my order",
                       max_length=200, num_return_sequences=5,
                       do_sample=True, temperature=0.7)
```

**Caveats:**

- LLMs may reproduce training data (privacy risk).
- Generated text may contain biases from the training data.
- Quality depends on the prompt and model.

### 4.5 Tabular-specific generators

| Tool | Method | Best for |
|---|---|---|
| **CTGAN** (SDV) | Conditional GAN with mode-specific normalization | General tabular |
| **TVAE** (SDV) | Tabular VAE | General tabular |
| **CTAB-GAN** | GAN with conditional bias correction | Tabular with complex constraints |
| **GCT** | Graph-based conditional tabular | Tabular with categorical constraints |
| **TDVAE** | Tabular diffusion model | High-fidelity tabular |
| **RFN** | Realistic Functional Noise | Tabular with complex distributions |

## 5. Simulation

### 5.1 Agent-based modeling

Simulate individual agents with rules, observe emergent behavior.

**Python:** `Mesa`

```python
from mesa import Agent, Model
from mesa.time import RandomActivation
from mesa.datacollection import DataCollector

class Customer(Agent):
    def __init__(self, unique_id, model, budget):
        super().__init__(unique_id, model)
        self.budget = budget

    def step(self):
        # Simple purchasing decision
        if self.budget > 0 and random.random() < 0.1:
            self.budget -= random.uniform(1, 10)

class StoreModel(Model):
    def __init__(self, n_customers=100):
        self.schedule = RandomActivation(self)
        for i in range(n_customers):
            agent = Customer(i, self, budget=random.uniform(10, 100))
            self.schedule.add(agent)
        self.datacollector = DataCollector({"total_budget": lambda m: sum(a.budget for a in m.schedule.agents)})

    def step(self):
        self.schedule.step()
        self.datacollector.collect(self)

model = StoreModel()
for _ in range(100):
    model.step()
```

### 5.2 Discrete-event simulation

Simulate events occurring at specific points in time.

**Python:** `SimPy`

```python
import simpy
import random

def customer(env, name, service_time):
    with env.request() as req:
        yield req
        yield env.timeout(random.expovariate(1.0 / service_time))

env = simpy.Environment()
for i in range(10):
    env.process(customer(env, f"Customer {i}", service_time=5.0))
env.run(until=100)
```

### 5.3 Physics-based simulation

**Tools:**

| Tool | Use |
|---|---|
| **Gazebo** | Robotics simulation |
| **NVIDIA Isaac Sim** | Robotics, autonomous vehicles |
| **Blender** | 3D rendering, synthetic images |
| **Unreal Engine** | High-fidelity synthetic environments |
| **CARLA** | Autonomous vehicle simulation |
| **AirSim** | Autonomous vehicles (built on Unreal) |

## 6. Augmentation as synthetic data

Augmentation transforms existing data to create new training examples.

### 6.1 Image augmentation

| Transform | Description | Library |
|---|---|---|
| Rotation | Random rotation | Albumentations, torchvision |
| Flip | Horizontal / vertical flip | Albumentations |
| Crop | Random / center crop | Albumentations |
| Color jitter | Brightness, contrast, saturation | torchvision |
| CutOut / MixUp | Mask random regions / blend images | Albumentations |
| RandomErasing | Random rectangular erasing | torchvision |
| Affine | Random affine transform | Albumentations |
| Elastic transform | Elastic deformation | Albumentations |

```python
import albumentations as A

transform = A.Compose([
    A.HorizontalFlip(p=0.5),
    A.Rotate(limit=15, p=0.5),
    A.ColorJitter(brightness=0.2, contrast=0.2, p=0.3),
    A.GaussianBlur(blur_limit=3, p=0.3),
    A.CoarseDropout(max_holes=3, max_height=32, max_width=32, p=0.3),
])

augmented = transform(image=original_image)
```

### 6.2 Text augmentation

| Method | Description |
|---|---|
| **Synonym replacement** | Replace words with synonyms (WordNet) |
| **Random insertion** | Insert synonyms at random positions |
| **Random swap** | Swap two words |
| **Random deletion** | Remove words randomly |
| **Back-translation** | Translate to another language and back |
| **LLM paraphrasing** | "Rewrite this sentence" |
| **EDA (Easy Data Augmentation)** | Combination of above |

### 6.3 Audio augmentation

| Transform | Description |
|---|---|
| Add noise | Add Gaussian or real noise |
| Pitch shift | Change pitch (preserve duration) |
| Time stretch | Change speed (preserve pitch) |
| SpecAugment | Mask time/frequency bins in spectrogram |
| MixUp | Mix two audio clips |
| RIR simulation | Simulate room impulse response |

## 7. Differential privacy

Formal privacy guarantees: synthetic data is generated with calibrated noise so that no individual's data can be identified.

**Key concept:** Adding Laplace or Gaussian noise to query results, with noise magnitude proportional to the *sensitivity* of the query and inversely proportional to the privacy parameter ε (epsilon).

```python
from diffprivlib.tools import histograms

# Differentially private histogram
# ε controls privacy: smaller = more private, noisier
hist, bin_edges = histograms(data, epsilon=1.0, bins=20)
```

**Frameworks:**

| Framework | Language | Use |
|---|---|---|
| **TensorFlow Privacy** | Python | DP-SGD for deep learning |
| **PyTorch Opacus** | Python | DP-SGD for PyTorch |
| **Diffprivlib** | Python | Utility functions (histograms, mean, PCA) |
| **Opacus** | Python | Differential privacy for PyTorch |
| **Apple DP** | Swift / Python | Apple's internal DP framework |
| **Google DP** | Python | Google's internal DP framework |

**Privacy accounting:**

- **Rényi DP:** Tracks cumulative privacy loss across multiple operations.
- **δ:** Probability of complete privacy breach (usually 10⁻⁵).
- **(ε, δ)-DP:** The formal guarantee. ε = 1.0, δ = 10⁻⁵ is typical.

## 8. Evaluation

### 8.1 Utility evaluation

Does training on synthetic data produce a good model on real data?

```python
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import RandomForestClassifier

# Train on synthetic, test on real
model = RandomForestClassifier()
scores = cross_val_score(model, synthetic_X, synthetic_y, cv=5)
print(f"Synthetic training accuracy: {scores.mean():.3f} (+/- {scores.std() * 2:.3f})")

# Train on real, test on real (upper bound)
model.fit(real_X, real_y)
real_scores = cross_val_score(model, real_X, real_y, cv=5)
print(f"Real training accuracy: {real_scores.mean():.3f} (+/- {real_scores.std() * 2:.3f})")
```

**Metrics:**

| Metric | What it measures |
|---|---|
| **Downstream accuracy** | Accuracy of model trained on synthetic, tested on real |
| **AUC-ROC** | Discrimination ability |
| **F1 score** | Balance of precision and recall |
| **Calibration error** | How well predicted probabilities match actual frequencies |

### 8.2 Fidelity evaluation

How close are synthetic distributions to real distributions?

| Metric | What it measures |
|---|---|
| **KL divergence** | Divergence between two distributions (lower = better) |
| **Wasserstein distance** | Earth-mover distance between distributions |
| **PSI (Population Stability Index)** | Stability of distribution over time |
| **Correlation difference** | |Δcorrelation(real) − Δcorrelation(synthetic)| |
| **JS divergence** | Jensen-Shannon divergence (symmetric KL) |
| **MMD** | Maximum Mean Discrepancy (kernel-based) |

```python
from scipy.stats import kldiv, wasserstein
import numpy as np

# KL divergence (marginal)
real_hist, bin_edges = np.histogram(real_data, bins=20, density=True)
syn_hist, _ = np.histogram(synthetic_data, bins=bin_edges, density=True)
kl = kldiv(real_hist + 1e-10, syn_hist + 1e-10)  # add epsilon for zero bins

# Wasserstein distance
wasserstein_dist = wasserstein(real_data, synthetic_data)
```

### 8.3 Privacy evaluation

Can an attacker reverse-engineer the original data?

| Attack | Method |
|---|---|
| **Membership inference** | Train a classifier to determine if a record was in the training data |
| **Attribute inference** | Predict a missing attribute from the synthetic data |
| **Reconstruction attack** | Reconstruct original records from synthetic samples |
| **Similarity attack** | Find synthetic records that exactly match real records |

```python
fromMembershipInferenceAPI import MembershipInferenceAPI

# Membership inference attack
attack = MembershipInferenceAPI(model)
success_rate = attack.attack(real_data, synthetic_data)
print(f"Membership inference success rate: {success_rate:.3f} (random = 0.5)")
```

**Acceptable thresholds:**

- Membership inference success rate < 0.55 (random = 0.5).
- Zero exact matches between synthetic and real records.
- No sensitive attributes perfectly reconstructible.

## 9. Tools and Python ecosystem

| Tool | Use |
|---|---|
| `Faker` | Rule-based fake data generation |
| `sdv` (Synthetic Data Vault) | CTGAN, TVAE, CTAB-GAN for tabular |
| `copulas` | Statistical copula modeling |
| `torchgan` | GANs with PyTorch |
| `diffusers` | Hugging Face diffusion models |
| `albumentations` | Image augmentation |
| `torchvision.transforms` | Image augmentation |
| `nlpaug` | Text augmentation |
| `torchaudio` | Audio augmentation |
| `Mesa` | Agent-based simulation |
| `SimPy` | Discrete-event simulation |
| `diffprivlib` | Differential privacy utilities |
| `Opacus` | DP-SGD for PyTorch |
| `TensorFlow Privacy` | DP-SGD for TensorFlow |
| `GRETNA` | Graph-based synthetic data |
| `YData Profiling` + `YData Synthetic` | Data profiling + synthetic generation |

## 10. References

- Dwork, C. *Differential Privacy*. ICALP (2006).
- Goodfellow, I. et al. *Generative Adversarial Nets*. NeurIPS (2014).
- Radford, A. et al. *Improving Language Understanding by Generative Pre-Training* (GPT). OpenAI.
- Kingma, D. & Welling, M. *Auto-Encoding Variational Bayes*. ICLR (2014).
- Song, J. et al. *Denoising Diffusion Probabilistic Models*. NeurIPS (2020).
- Xu, L. et al. *Modeling Tabular Data using Conditional GAN* (CTGAN). NeurIPS (2019).
- Morariu, C. I. & Sattigeri, P. et al. *Synthetic Data Vault* (SDV) documentation.
- Abadi, M. et al. *Deep Learning with Differential Privacy*. CCS (2016).
