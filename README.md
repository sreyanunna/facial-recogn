# Facial Recognition Bias in Criminal Justice Systems

> Investigating how historically biased training data propagates into facial recognition systems deployed in law enforcement — with a focus on benchmark dataset limitations, demographic disparity measurement, and the mathematical constraints that make bias impossible to fully mitigate within current system architectures.

---

## Motivation

This research began personally. A close childhood friend of mine — an Indian immigrant living in the UK — was called in for police questioning and spent six months in jail for a crime he did not commit. He was identified through a sketch-based facial recognition match. That experience transformed what had been an intellectual interest into something I felt a genuine responsibility to investigate rigorously.

The technical cause is traceable. Facial recognition systems deployed in criminal justice contexts are trained predominantly on proprietary datasets drawn from custody image databases — repositories that encode decades of racially disproportionate policing. The model does not introduce bias; it faithfully reproduces it from its ground truth. Meanwhile, the benchmark datasets used to evaluate these systems in academic research — built from celebrity photographs, news images, and web-scraped media — bear almost no resemblance to the degraded CCTV footage, custody photographs, and forensic sketches that operational systems actually process.

This repository documents the research journey from that personal starting point through to a technical analysis of where bias enters the pipeline, how it is measured, and why current mitigation strategies are mathematically constrained from resolving it at the source.

---

## Research Steps

### Step 1 — Benchmark Dataset Analysis
Conducted exploratory data analysis across the most widely cited facial recognition benchmark datasets (LFW, VGGFace2, MS-Celeb-1M, FairFace, RFW). Examined demographic composition, image quality characteristics, and source provenance against the conditions of real criminal justice deployment.

**Key finding:** Benchmark datasets are structurally misaligned with criminal justice deployment conditions on every measurable axis — source population, image quality, task type, and consent context. They systematically over-represent the demographic groups *least* likely to be targeted by these systems operationally.

### Step 2 — Surveillance Lineage Investigation
Traced the funding and application lineage of major benchmark datasets. Found that research infrastructure marketed as neutral academic benchmarking is structurally connected to surveillance and defence applications.

**Key finding:** The LFW dataset, originating from University of Massachusetts academics, later received funding from the CIA and the NSA. As one provenance analysis documents, the datasets built from *"media in the wild"* collectively align with the advancement of surveillance technologies in commercial and defence contexts. Benchmark datasets are not neutral — they are lineage points toward surveillance systems.

### Step 3 — Fairness Framework Review and the Impossibility Theorem
Reviewed the landscape of algorithmic fairness frameworks applied to criminal justice AI, with particular focus on the COMPAS recidivism risk scoring debate as the canonical case study.

**Key finding:** The ProPublica/Northpointe debate about COMPAS is not resolvable by better engineering. Both sides are simultaneously correct — they are measuring different, mutually exclusive definitions of fairness. Allen Downey's case study demonstrates empirically that when recidivism base rates differ between demographic groups (themselves a product of historically unequal policing), it is mathematically impossible to simultaneously satisfy equal false positive rates *and* equal predictive value. This is the **Impossibility Theorem of Fairness**, formalised by Chouldechova (2017).

### Step 4 — Mitigation Landscape Assessment
Surveyed pre-processing, in-processing, and post-processing bias mitigation techniques using IBM AIF360 and Microsoft Fairlearn, testing adversarial debiasing and threshold adjustment against the COMPAS dataset and FairFace-evaluated FR models.

**Key finding:** All three mitigation stages operate downstream of the fundamental problem. Reweighting a dataset where *"re-arrest"* is used as a proxy for *"criminality"* is reweighting a biased signal. The bias is upstream, in the data pipeline decisions that determine what gets measured, who gets counted, and whose experience becomes the training signal.

---

## Current Research Findings

### Finding 1: The Benchmark-to-Deployment Gap

Facial recognition benchmark datasets are built from a fundamentally different population than criminal justice systems are deployed against. LFW's most represented identities are politicians and world leaders — Colin Powell, George W. Bush, Tony Blair. Criminal justice systems process custody photographs of individuals who have entered the criminal justice system, which is itself a dataset shaped by decades of racially disproportionate policing. Every accuracy figure published against a celebrity benchmark is measuring performance on the wrong population.

Beyond the source population gap, image quality creates a second, independent failure of generalisability. Benchmark datasets consist of high-resolution, professionally lit, near-frontal photographs. Operational criminal justice FR runs on degraded CCTV footage, aged custody photographs, and — in the case documented here — forensic sketches. Sketch-to-face matching introduces a compounding error: the cross-race effect degrades human witness memory before the algorithm ever runs, stacking two independent bias sources into a single identification pipeline.

### Finding 2: Benchmark Datasets as Surveillance Infrastructure

The research literature presents benchmark datasets as neutral scientific tools. The funding and application lineage suggests otherwise. The LFW dataset received CIA and NSA funding. MS-Celeb-1M was built by Microsoft to accelerate recognition of one million named individuals. The IARPA Janus Benchmarks were built explicitly for intelligence agency use cases. As provenance researcher Adam Harvey documents at `exposing.ai`, these datasets collectively form infrastructure for remote biometric analysis, with overlapping applications in surveillance, person re-identification, and defence. Researchers building on these datasets are not working in a vacuum — they are extending a lineage with specific application endpoints already built in.

### Finding 3: The Impossibility Theorem and the COMPAS Debate

The most widely studied case of algorithmic bias in criminal justice is the COMPAS recidivism risk scoring system. ProPublica found that Black defendants were flagged as high risk at nearly twice the rate of white defendants who did not reoffend — a false positive rate disparity that they characterised as evidence of racial bias. Northpointe, the system's developer, responded that COMPAS achieves equal predictive accuracy across racial groups — that a score of 7 means the same probability of reoffending regardless of race.

Allen Downey's empirical case study demonstrates that both claims are simultaneously correct. They are measuring different fairness criteria — **equalised odds** (equal error rates across groups) versus **predictive parity** (equal precision across groups) — and Chouldechova's 2017 impossibility theorem proves that when base rates differ between groups, these two criteria are mathematically incompatible. You cannot satisfy both at once.

The base rate difference is not random. Black defendants reoffend at higher rates in the COMPAS dataset because they have more prior arrests — and they have more prior arrests because historically they have been policed at higher rates, not because they commit more crime. Stanford Open Policing data shows that Black drivers are searched at higher rates than white drivers despite *lower* contraband find rates — direct evidence that the underlying policing data does not measure criminality, it measures enforcement intensity. COMPAS learns from that signal. The bias is not in the algorithm. It is in what the ground truth label was built to mean.

### Finding 4: Mitigation Operates Downstream of the Source

Three categories of bias mitigation exist in the literature: pre-processing (fixing the training data), in-processing (adding fairness penalties during training), and post-processing (adjusting outputs after prediction). IBM's AIF360 toolkit implements all three. Experiments with adversarial debiasing and threshold adjustment on the COMPAS dataset confirm the pattern the literature predicts: reducing demographic disparity on one fairness metric comes at the cost of increasing it on another, or at the cost of overall accuracy — because the impossibility theorem is a mathematical constraint, not an engineering problem.

The only intervention that addresses the source is changing the ground truth label. As long as criminal justice AI systems use arrest records, conviction rates, and custody database images as proxies for criminality, they will reproduce the enforcement patterns those records encode. This is not a solvable problem within the current architecture of these systems. It requires a prior question: should these systems be deployed at all in contexts where the training data is irreparably contaminated by the history it is supposed to be neutral about.

---

## References

### GitHub Repositories

| Repository | Purpose |
|---|---|
| [`propublica/compas-analysis`](https://github.com/propublica/compas-analysis) | Original COMPAS bias analysis data and notebook |
| [`AllenDowney/RecidivismCaseStudy`](https://github.com/AllenDowney/RecidivismCaseStudy) | Empirical walkthrough of the impossibility theorem via COMPAS |
| [`mbarenstein/ProPublica_COMPAS_Data_Revisited`](https://github.com/mbarenstein/ProPublica_COMPAS_Data_Revisited) | FTC economist's data processing critique of ProPublica analysis |
| [`visionjo/facerec-bias-bfw`](https://github.com/visionjo/facerec-bias-bfw) | Balanced Faces in the Wild — demographic disparity in FR systems |
| [`joojs/fairface`](https://github.com/joojs/fairface) | FairFace dataset — 108K images balanced across 7 racial groups |
| [`seymayucer/FacialPhenotypes`](https://github.com/seymayucer/FacialPhenotypes) | Phenotype-based bias evaluation on RFW and VGGFace2 |
| [`dooleys/FR-NAS`](https://github.com/dooleys/FR-NAS) | Neural architecture search for fairness in face recognition |
| [`Trusted-AI/AIF360`](https://github.com/Trusted-AI/AIF360) | IBM AI Fairness 360 — 70+ fairness metrics, 10+ mitigation algorithms |
| [`dssg/aequitas`](https://github.com/dssg/aequitas) | Bias audit toolkit built for criminal justice applications |
| [`Call-for-Code-for-Racial-Justice/bias-detection-engine`](https://github.com/Call-for-Code-for-Racial-Justice/bias-detection-engine) | Sentencing disparity detection engine for public defenders |
| [`caltechvisionlab/frt-rapid-test`](https://github.com/caltechvisionlab/frt-rapid-test) | Rapid external audit tool for commercial FR APIs |
| [`Trueface-ai/awesome-bias-mitigation`](https://github.com/Trueface-ai/awesome-bias-mitigation) | Curated list of FR bias mitigation papers and implementations |
| [`mbilalzafar/fair-classification`](https://github.com/mbilalzafar/fair-classification) | Fair logistic regression with disparate mistreatment constraints |

### Datasets

| Dataset | Access | Key Characteristic |
|---|---|---|
| [Labeled Faces in the Wild (LFW)](http://vis-www.cs.umass.edu/lfw/) | Public | 13,233 images; 83.5% white; news/celebrity source |
| [FairFace](https://github.com/joojs/fairface) | Public | 108K images; balanced across 7 race groups, age, gender |
| [Racial Faces in the Wild (RFW)](http://www.whdeng.cn/RFW/) | Request | 40K images across Caucasian, Asian, Indian, African groups |
| [Balanced Faces in the Wild (BFW)](https://github.com/visionjo/facerec-bias-bfw) | Public | Equal face count across demographic subgroups |
| [ProPublica COMPAS Dataset](https://github.com/propublica/compas-analysis) | Public | 7K Broward County defendants; COMPAS scores + actual outcomes |
| [Stanford Open Policing Project](https://openpolicing.stanford.edu/) | Public | 100M+ traffic stops; documents racial disparity in enforcement rates |
| [VGGFace2](https://github.com/ox-vgg/vgg_face2) | Request | 3.3M images; entertainment industry skew |

### Key Papers

| Paper | Why It Matters |
|---|---|
| Angwin et al. (2016) — *Machine Bias*, ProPublica | Primary investigation establishing COMPAS racial disparity |
| Chouldechova (2017) — *Fair prediction with disparate impact* | Formal proof of the fairness impossibility theorem |
| Buolamwini & Gebru (2018) — *Gender Shades* | Foundational FR demographic audit; 43x error rate gap |
| Dressel & Farid (2018) — *Science Advances* | COMPAS no more accurate than untrained humans using 2 features |
| NIST FRVT Part 3 (2019) | Most comprehensive demographic bias evaluation across 189 algorithms |
| Barocas & Selbst (2016) — *Big Data's Disparate Impact* | Legal and technical framing of proxy variable bias |
| Harvey (2021) — *Origins and Endpoints*, exposing.ai | Provenance analysis of surveillance lineage in benchmark datasets |

### Tools Used

- [IBM AIF360](https://aif360.mybluemix.net/) — bias measurement and mitigation
- [Microsoft Fairlearn](https://fairlearn.org/) — disaggregated metric evaluation
- [DeepFace](https://github.com/serengil/deepface) — FR model evaluation across demographic groups
- [Aequitas](http://www.datasciencepublicpolicy.org/projects/aequitas/) — criminal justice bias audit reports

---

## Installation

```bash
git clone https://github.com/your-username/facial-recognition-bias.git
cd facial-recognition-bias
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Create a `.env` file from the template:
```bash
cp .env.example .env
# Add your OpenAI API key if using LLM-assisted analysis steps
```

---

## A Note on Scope

This repository does not claim to have solved the bias problem in facial recognition. It claims to have located where that problem actually lives — upstream, in the data pipeline decisions that precede any algorithm. The technical literature on mitigation is extensive. The harder, and less researched, question is what it means to deploy systems trained on historically contaminated data in contexts where the consequences of a false positive include imprisonment.

---

*Research in progress. Contributions, critiques, and dataset suggestions welcome via Issues.*
