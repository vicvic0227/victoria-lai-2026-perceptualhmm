---
title: Time-dependent strategy dynamics in a perceptual decision-making task

abstract: |
    Human perceptual estimation balances internal expectations with incoming sensory evidence. While static switching models demonstrate that observers alternate between discrete prior-dependent and evidence-tracking states, whether these transitions are time-independent or governed by dynamic, past-dependent cognitive momentum remains unclear. Using an input-driven Hidden Markov Model (HMM) fitted via the Expectation-Maximisation algorithm, we reanalysed human motion-direction estimates. Our dynamic model outperformed static variants by a large margin, revealing strong temporal inertia in strategy selection. Furthermore, transitions were actively modulated by trial-by-trial sensory coherence and spatial surprise, providing evidence for a boundedly rational brain that dynamically regulates state-dependent processing precision.

acknowledgments: |
    This work was supported by the Impact Scholars Program. We acknowledge the contributions of [former team members, teaching assistants, or mentors whose involvement does not meet the criteria of any authorship role].
---

# Description

The current project used the NMA dataset for the perceptual dot motion decision-making task {cite:p}`LaquitaineSwitchingObserverHuman2018a`. In the original paper, the author proposed that the process of estimating the motion direction, the switching observer model was a better fit than the Bayesian observer model (the posterior distribution resulting from the observed percept in a particular trial by the prior), as the former provide more explanatory power to the phenomenon of the bimodality distribution of individuals’ estimates when the presented motion was very different from the constant prior experimental mean of  225˚. The experiment dissociates the effect of sensory evidence and priors by varying the  1) motion coherence–percentage of dots moving in the same direction, and 2) prior standard deviation – the strength of the presented dot distribution, and suggested that switching observers will switch between prior and evidence that depends on the two factors. For example, when motion coherence is low, subjects tend to choose estimates from the prior distribution, leading to a bias toward estimates near the prior, whereas an increase in prior standard deviation introduces greater variability in subjects’ estimates. 

However, they did not pursue the question further. Since the switching observer model assumes subjects switch between prior mean and sensory evidence based on their relative strength independently over time, we aim to examine whether switching behaviour is time-dependent and whether factors such as motion coherence can influence individuals' internal states during decision-making. Recent reviews have challenged the assumption that decision strategies are stable across trials and have shown that models that deploy non-stationary dynamics, such as discrete-state switches, provide greater explanatory power for human behaviour than some standard static models {cite:p}`GunawanTimeevolvingPsychologicalProcesses2022;uraiStructureUncoveredUnderstanding2026a`. To determine whether observers continuously integrate expectations and sensory evidence into a single percept while perceptual noise remains constant across trials, or whether they discretely switch between independent cognitive states, whether the switching is dependent on previous states and precision of sensory processing, we compared 3 models, Probabilistic Mixture model (PM), Weighted Mean model (WM) and the Hidden Markov Model.

## 1. Weighted Mean model with a global variance (WM) 
 Unlike a discrete-switching framework, this model assumes that on every trial t, the brain multiplicatively combines its internal structural expectation (the prior) with the incoming sensory evidence to form a unified, blended posterior mean. The relative contribution of sensory evidence versus prior is determined by a trial-by-trial linear predictor $w_{\text{evidence,t}}$ and $w_{\text{prior,t}}$, which passes through a standard logistic function:
  $$w_{\text{evidence,t}} = \frac{1}{1 + \exp\left(-(b_0 + b_{\text{coh}} \cdot \text{coherence} + b_{\alpha} \cdot \alpha)\right)}$$ 

  $$w_{\text{prior,t}} = {1 - w_{\text{evidence,t}}}$$


i.e. the weight of selecting the evidence component on any given trial, $w_{\text{evidence,t}}$, is determined by coherence and $b_{\text{coh}}$ , the sensitivity to sensory clarity/ contrast, together with $\alpha$, the absolute magnitude of the displayed angle location relative to the prior.

$$\mu_t = \text{atan2} \left( w_{\text{evidence}, t} \cdot \sin(\theta_{\text{evidence}, t}), w_{\text{prior}, t} + w_{\text{evidence}, t} \cdot \cos(\theta_{\text{evidence}, t}) \right)$$


The likelihood was modelled as a single von Mises distribution centred on the circular weighted mean ($\mu_t$) of the prior $\theta_{\text{prior}, t}$ and evidence direction angle $\theta_{\text{evidence}, t}$, with a single shared concentration parameter kappa . The von Mises distribution is a circular equivalent of the Gaussian distribution:
$$p(y_t \mid \mu_t, \kappa) = \frac{\exp(\kappa \cdot \cos(y_t - \mu_t))}{2\pi \cdot I_0(\kappa)}$$

## 2. Probabilistic Mixture model (PM)
Given evidence from the original paper that the switching prior sampling model provides a better fit, as some subjects were not certain about the prior mean, it would be better to model observations generated from two different von Mises distributions, with the likelihood of internal perceptual readout represented by two distributions centred on either displayed angle or the prior mean of 0. PM has the same model structure as WM, except that it assumes that the subject responds based on either the prior distribution with concentration prior or the evidence distribution with ev . Here, the weights wact as a coin flip to determine which distribution drives which response during that trial. We set the bounds of bcoh to be highly positive (10, 50), as we assume higher coherence drives the model to weight the physical evidence over the prior.

Evidence Likelihood:  $$p_{\text{evidence}}(y_t) \sim \text{von Mises}(\alpha, \,  \kappa_{\text{ev}})$$
Prior Likelihood: $$p_{\text{prior}}(y_t) \sim \text{von Mises}( 0.0, \, \kappa_{\text{prior}})$$

On any single trial, the overall probability density of observing response $y_t$ is a weighted linear combination of the two components.
$$p(y_t) = w_e \cdot p_{\text{evidence}}(y_t) + (1 - w_e) \cdot p_{\text{prior}}(y_t)$$


Model comparisons yield $\Delta$ AIC = 21,417.9 and $\Delta$ BIC = 21,306.00 in favour of Reparameter 2, indicating that it is a better fit.

## 3. Input-driven Hidden Markov Model
Under reparameter 2 mode, the choice of switching on trial t  has zero historical memory and it is entirely unaffected by what the participant did on trial t-1. A Hidden Markov model enables the capture of internal mental states, such as mind-wandering, by assigning the most probable state to each trial in a session {cite:p}`AshwoodMiceAlternateDiscrete2022`. In our model, the two hidden states correspond to prior and evidence modes, which correspond to distinct decision-making strategies, parametrised by a set of GLM weights that describe how subjects weigh different task covariates to make a decision in each state. 
 
### 3.1 Latent State Transitions
The transition is modelled using multinomial logistic regression. The discrete hidden states at time t are denoted as  $z_t \in \{1, \dots, K\}$ .  $z_t$ represents the discrete latent state at trial t, where $z_t$ =0  corresponds to the Prior state and $z_t$  = 1 represents the Evidence state.


Given you are in the previous state $j$ with external input $u_t$, the exogenous input feature vector at trial $t$. In this paradigm, $u_t = [\theta_{\text{stim}}, \text{coherence}]^\top$, incorporating both structural sensory deviation and evidence strength. The probability of jumping from the current state $j$ to the next state $k$ is a softmax over all possible next states. Here, $\log P_{j,k}$ represents the baseline, unnormalized transition bias independent of external task inputs. The vector $w_{j,k}$ represents the transition weights (input coefficients) corresponding to target state $k$ when originating from state $j$. These weights determine how strongly motion coherence and stimulus orientation modulate and shift behavioral strategies.

$$ \operatorname{Pr}(z_t = k \mid z_{t-1} = j, u_t) = \frac{\exp\left\{ \log P_{j,k} + w_{j,k}^\top u_t \right\}}{\sum_{k'=1}^K \exp\left\{ \log P_{j,k'} + w_{j,k'}^\top u_t \right\}} $$

### 3.2 Emission Likelihoods

The model maps the continuous probability with observed behaviours during model estimation via the EM algorithm. The overall likelihood of response $y_t$ is evaluated as a mixture density. Specifically, the emission likelihoods are weighted by their posterior probabilities and marginalised across all latent states:
$$p(y_t \mid u_{1:t}, y_{1:t-1}) = \sum_{k=0}^{1} p(y_t \mid z_t = k) \Pr(z_t = k \mid u_{1:t}, y_{1:t-1})$$


Where the conditional state emissions are defined as:
$$p(y_t \mid z_t = 0) = \text{von Mises}(y_t \mid 0, \kappa_{\text{prior}}) = \frac{\exp(\kappa_{\text{prior}} \cdot \cos(y_t))}{2\pi \cdot I_0(\kappa_{\text{prior}})}$$

$$p(y_t \mid z_t = 1) = \text{von Mises}(y_t \mid \alpha, \kappa_{\text{ev}}) = \frac{\exp(\kappa_{\text{ev}} \cdot \cos(y_t - \ (\alpha))}{2\pi \cdot I_0(\kappa_{\text{ev}})}$$



State 0 (Prior state) represents observations that cluster around 0, which is the prior mean and State 1(Evidence state) represents those that cluster around the displayed stimulus angle.

Results demonstrated a significant improvement of model fit of the input HMM compared with PM, AIC= 1996.22, BIC= 1772.32. While previous static models correctly identified that observers switch between discrete prior-dependent and evidence-dependent states, our work demonstrates that this switching is a dynamic, memory-retaining process driven by temporal inertia and actively updated by shifting covariates, such as motion coherence and large stimulus-angle deviations(@figure-param ). Our study extends the switching observer phenomenon by providing a temporal account for individual differences in strategy switching that could account for the accuracy difference between the two, where subject 1 has a mean error rate of 29.38% and subject 10 has 67.17% error in the high difficulty block, respectively(@figure-main), implying these fluctuations discrepancies could reflect attentional allocation and that relying on priors with changing coherence might not be optimal.

One of our limitations is that we assumed that the behavioral variabilities were explained by kprior and kevidence that motor noises and random lapses are not modelled. Future studies could disentangle the psychological meaning behind a “prior state”: whether it involves a strategic exploration or merely disengagement.

```{figure} figure1.png
:name: figure-param
:alt: Figure 1.

Group-level parameter distributions from the HMM model: compared with coherence, alpha() has less weight on driving state transitions. Baseline transition probabilities P(0->1) indicated that, without strong coherence, subjects tend to stay at the prior state once they start relying on it to make decisions.
```



```{figure} figure2.png
:name: figure-main
:alt: Figure 2.

The plots show modelled assigned/learned latent-state fluctuations across different runs and subjects, denoted by the orange curve, P(evidence), and how these fluctuations are driven by coherence input sensitivity to transition (the coherence component of $w_{j,k}^\top u_t$ at time $t$).Under the same prior std (i.e., 80 degrees), which reflects significant task difficulty, some subjects, like subject 1, show time-dependent persistence in the evidence mode even though the coherence input has remained low across consecutive trials. In contrast, subject 10 shows more frequent state fluctuations following the input, with transitions to the prior mode immediately upon low coherence, and only stays in the evidence mode when coherence is high. 





