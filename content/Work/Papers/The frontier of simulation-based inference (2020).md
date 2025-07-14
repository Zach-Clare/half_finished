Kyle Cranmera, Johann Brehmera, and Gilles Louppec
https://www.pnas.org/doi/10.1073/pnas.1912789117

The introduction argues that [[Mechanistic models||mechanistic models]] are poorly suited for [[Statistical Inference (SI)||statistical inference (SI)]] practises because of the probability density function being inaccessible from a single observation (i.e. from a single simulation run). 

The models that create a single observation are supposedly called implicit models. Simulations that simulate a distribution over some sample are supposedly described as prescribed models. In this sense, my [[A tool to help fitting for Magnetopause standoff distance||many-observation ML assessment tool]] turns CMEM as an implicit model into a prescribed model? I'm not sure if that's true.

[[Simulation Based Inference (SBI)]] is essentially the idea that, using many simulations of implicit models, we can extract usually intractable [[Statistical Inference (SI)|SI]] values, just through volume.

It then goes on to share two traditional methods to overcome this pre-existing barrier using domain knowledge-driven insights, given they are experts in their field. This isn't so relevant here.

At this point, the paper splits into various headings:
- [[Simulation Based Inference (SBI)||Simulation-Based Inference]]
	- Simulators
	- Inference
	- Traditional Methods
	- Frontiers of [[Simulation Based Inference (SBI)||Simulation-based Inference]]
	- A Revolution in machine Learning
	- Active learning
	- Integration and Augmentation
- Workflows for [[Simulation Based Inference (SBI)||Simulation-Based Inference]]
	- Introduction
	- Using the Simulator Directly during Inference
	- Surrogate Models
	- Pre-processing and Post-processing
	- Recommendations
- Discussion (conclusion)
## Simulation-Based Inference
### Simulators
It describes various stages of simulators, namely three (given without identifying names):
- The parameters describe the underlying [[Mechanistic models||mechanistic model]] and therefore affect the results in expected ways. Think coefficients, pathogen incubation rates, or fundamental constants in physics. When Sam Wharton fits CMEM to PPMLR cubes, the bulk of that operation (or of the model comparison) would fit primarily here. Simulators that use mainly these mechanisms are the easiest to fit because the errors and distributions are clear and would follow distributions that someone with decent domain knowledge could reasonably expect.
- The interactions of the model may contain latent (hidden, or with unobvious effects) variables or parameters (labelled in this paper as `z`) that are unobservable in practise. These are perhaps variables that change with time or affect the base variables and are likely unextractable during a single observation. The effects of parameters could be differentiable or discontinuous. This area, to me, seems to describe the core component of complex simulators/models.
- The third stage is where model outputs correspond to observations in the form of anything from unstructured numbers to highly structured images or high-dimensionality data. For example, the SXI simulator (using CMEM as the above two preliminary stages) introduces the instrument response to place simulator outputs in the format of expected observations, with specific focus on aspect ratio, resolution, Poisson noise, and sensor characteristics (vignette, background noise).

The fact that these many different steps lead to such varied outcomes means that there is no one-size-fits-all inference method that _just works_. This paper aims to tell us what we need to think about when choosing inference methods.

### Inference

