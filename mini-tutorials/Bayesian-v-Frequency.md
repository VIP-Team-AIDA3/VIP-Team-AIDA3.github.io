# Frequentist vs. Bayesian Statistics: A Tutorial on AI for Autonomous Aviation

*A hands-on guide to the two dominant philosophies of statistical inference, illustrated with UAV/aircraft sensor and flight-test examples.*

You don't need any stats background for this. Fractions are enough.

## Two ways of thinking about an unknown number

Most of what we care about in this program is something we can't measure directly. How often does the aircraft land inside the box. How trustworthy is the runway detector. Whether the model we fit to yesterday's flight data will hold up tomorrow. In each case we have some data and we want a number that the data doesn't hand us outright.

Statisticians have two traditions for doing this, and they've been arguing about it for roughly a century.

The frequentist tradition says the true value is a fixed number that we happen not to know. Your job is to estimate it from the data you collected, and only from that data.

The Bayesian tradition says you should treat your uncertainty about the number as itself a kind of probability. You start with whatever you believed beforehand, the data comes in, and you update.

The rest of this is working out what that difference does in practice.

---

## Table of Contents

1. [Why This Matters for Autonomous Aviation](#1-why-this-matters-for-autonomous-aviation)
2. [The Core Philosophical Split](#2-the-core-philosophical-split)
3. [Frequentist Statistics](#3-frequentist-statistics)
4. [Bayesian Statistics](#4-bayesian-statistics)
5. [Side-by-Side: The Same Airspeed-Sensor Problem, Two Ways](#5-side-by-side-the-same-airspeed-sensor-problem-two-ways)
6. [Sequential Updating: Why Bayesian Thinking Feels Natural in Flight Computers](#6-sequential-updating-why-bayesian-thinking-feels-natural-in-flight-computers)
7. [Hypothesis Testing: Frequentist p-values vs. Bayesian Model Comparison](#7-hypothesis-testing-frequentist-p-values-vs-bayesian-model-comparison)
8. [When to Use Which](#8-when-to-use-which)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Exercises](#10-exercises)
11. [Summary Table](#11-summary-table)
12. [Further Reading](#12-further-reading)

---

## 1. Why This Matters for Autonomous Aviation

Every autonomous aircraft — a fixed-wing UAV, a multirotor, or a full-scale aircraft with an autopilot — is a statistics problem wearing a flight-control jacket. Almost every subsystem you'll touch in this VIP eventually runs into the same core question: **"My sensor (or my model) gives me a noisy number — what do I actually believe the true state of the aircraft is?"**

- Your barometric altimeter and GPS altitude disagree by a few meters. How much should you trust each, and how do you fuse them into one altitude estimate the autopilot can act on?
- Your perception stack (vision-based obstacle detection, ADS-B traffic classification, runway detection) reports a detection with some confidence. Turning that confidence into a decision — evade or don't, land or go around — is a statistical decision problem.

Both frequentist and Bayesian statistics give you rigorous tools for these questions. They differ in what "probability" *means*, and therefore in how they turn noisy flight-test data into conclusions you can certify, publish, or fly on. Understanding both makes you a sharper flight-test engineer and a better reader of the state-estimation and sensor-fusion algorithms that sit at the heart of every autonomy stack. Let us work through this example:

Say we flew ten autolands and eight of them landed in the box.

A frequentist estimate is eight out of ten, so 80%. You used the data, that's the answer, and there isn't much more to say about it.

For a Bayesian estimate you have to ask what you believed before the test. Suppose you genuinely had no idea. A way to represent that is to pretend you'd already seen one success and one failure before the test started, which is a mild way of saying "could go either way." Add the real results to those two pretend ones and you get nine successes out of twelve attempts, or 75%.

The Bayesian number came out lower because we admitted up front that we weren't sure. That matters more than it might look. Eight out of ten is a fragile number. One more failed landing and it drops to 73%, which is a large move for a single flight, and it should make you suspicious of how much the original 80% was really telling you.

Run the same comparison with more data and the disagreement evaporates. If we'd flown 200 times with 160 successes, the frequentist answer is 80% and the Bayesian answer is 161 out of 202, which is 79.7%. For practical purposes those are the same number. This is the general pattern: the two traditions converge when data is plentiful, and the Bayesian approach is the more cautious of the two when it isn't.

---

## 2. The Core Philosophical Split

| | Frequentist | Bayesian |
|---|---|---|
| What is probability? | The long-run frequency of an event over many repeated trials | A degree of belief, which can apply to a single event or a fixed unknown parameter |
| What is a parameter (e.g., true airspeed bias, true sensor noise)? | A fixed, unknown constant. It doesn't have a "distribution" — *your estimate* of it does. | A random variable with its own probability distribution, representing your uncertainty about it |
| Where does prior knowledge go? | Nowhere formal — only the current flight-test data counts | Explicitly encoded as a **prior distribution**, updated by data via Bayes' theorem |
| Typical output | A point estimate + confidence interval, or a p-value | A full **posterior distribution** over the parameter |

A useful mental picture: imagine estimating the true bias $b$ of a pitot-static airspeed sensor from repeated readings taken while the aircraft is held at a known reference airspeed (e.g., in a wind tunnel, on a tow test, or cross-checked against GPS ground speed in still air).

- A **frequentist** says: "$b$ is some fixed number. I don't know it, but it doesn't have a probability distribution — my *measurements* do. I'll compute an estimate $\hat{b}$ and describe how that estimate would vary if I repeated this whole calibration run many times."
- A **Bayesian** says: "I have some belief about $b$ before I run the calibration — pitot tubes of this model and installation typically show a bias within about ±1 kt, per the manufacturer's datasheet and prior flight tests. I'll combine that prior belief with today's data to get an updated, more precise belief — a full probability distribution over $b$ itself."

Neither is "wrong." They are different, self-consistent frameworks, and both show up constantly in flight-test and avionics engineering — often on the very same problem, sometimes in the same certification document.

---

## 3. Frequentist Statistics

### 3.1 The Core Idea: Maximum Likelihood

Suppose you take $n$ independent, noisy airspeed readings $x_1, x_2, \dots, x_n$ from a pitot-static sensor while the aircraft is held at a known, fixed true airspeed $\mu$ (e.g., 50.0 kt, verified against a calibrated reference such as GPS ground speed in zero-wind conditions), and you model the sensor noise as Gaussian:

$$x_i \sim \mathcal{N}(\mu, \sigma^2)$$

The frequentist approach asks: **"What value of $\mu$ makes the data I actually observed most probable?"** This is Maximum Likelihood Estimation (MLE). For Gaussian noise, the answer is simply the sample mean:

$$\hat{\mu}_{MLE} = \bar{x} = \frac{1}{n}\sum_{i=1}^{n} x_i$$

and the uncertainty of that estimate is captured by the **standard error**:

$$SE = \frac{s}{\sqrt{n}}, \qquad s = \sqrt{\frac{1}{n-1}\sum_{i=1}^n (x_i - \bar{x})^2}$$

### 3.2 Confidence Intervals

A 95% confidence interval is:

$$\hat{\mu} \pm t_{0.975, \, n-1} \cdot SE$$

**The correct interpretation (this trips almost everyone up):** a 95% CI does *not* mean "there's a 95% chance the true value is in this interval." $\mu$ is fixed — it's either in the interval or it isn't. The correct statement is: *if you repeated this entire calibration run (collect $n$ new readings, compute a new interval) many times, about 95% of the resulting intervals would contain the true reference airspeed.* The randomness lives in the *procedure*, not in $\mu$.

### 3.3 Worked Example: Calibrating a UAV Airspeed Sensor

Your UAV's pitot-static airspeed sensor is being calibrated on a tow test at a known reference airspeed of exactly 50.0 kt (verified via GPS ground speed on a still-air day). You log 12 readings.

```python
import numpy as np
from scipy import stats

# 12 readings from the pitot airspeed sensor (kt), true reference airspeed = 50.0 kt
readings = np.array([50.8, 49.6, 51.2, 50.1, 49.4, 50.9,
                      50.3, 49.8, 51.0, 50.5, 49.7, 50.6])

n = len(readings)
mean_hat = np.mean(readings)                 # MLE point estimate
s = np.std(readings, ddof=1)                 # sample standard deviation
se = s / np.sqrt(n)                          # standard error of the mean

# 95% confidence interval using the t-distribution (small sample)
t_crit = stats.t.ppf(0.975, df=n - 1)
ci_low, ci_high = mean_hat - t_crit * se, mean_hat + t_crit * se

print(f"Point estimate (MLE):      {mean_hat:.3f} kt")
print(f"Sample std dev:            {s:.3f} kt")
print(f"Standard error:            {se:.3f} kt")
print(f"95% confidence interval:   [{ci_low:.3f}, {ci_high:.3f}] kt")
```

**Verified output:**
```
Point estimate (MLE):      50.325 kt
Sample std dev:            0.602 kt
Standard error:            0.174 kt
95% confidence interval:   [49.943, 50.707] kt
```

Since the known reference airspeed (50.0 kt) falls inside the interval, you'd conclude the sensor shows no statistically detectable bias at this sample size — though the interval is wide enough that a bias of a few tenths of a knot could easily be hiding, which matters when this reading feeds a stall-margin protection law.

### 3.4 Hypothesis Testing: Is the Airspeed Sensor Biased?

Formally: $H_0$: $\mu = 50.0$ kt (no bias) vs. $H_1$: $\mu \neq 50.0$ kt (biased).

```python
t_stat, p_value = stats.ttest_1samp(readings, popmean=50.0)
print(f"t-statistic: {t_stat:.3f}")
print(f"p-value:     {p_value:.4f}")
```

```
t-statistic: 1.871
p-value:     0.0882
```

**Correct interpretation of the p-value:** *if the sensor truly had zero bias*, there would be an 8.8% chance of seeing a sample mean at least this far from 50.0 kt just from random noise. Since $p = 0.088 > 0.05$, we **fail to reject** $H_0$ — we don't have strong enough evidence to call the sensor biased at this sample size. Note this is *not* "there's an 8.8% chance the sensor is unbiased" — that backwards reading is the single most common misinterpretation of a p-value, and one you do not want propagating into a flight-test report.

---

## 4. Bayesian Statistics

### 4.1 Bayes' Theorem

Everything Bayesian flows from one identity:

$$P(\theta \mid D) = \frac{P(D \mid \theta)\, P(\theta)}{P(D)}$$

In words:

$$\text{posterior} = \frac{\text{likelihood} \times \text{prior}}{\text{evidence}}$$

- **Prior** $P(\theta)$ — what you believed about the parameter $\theta$ before seeing data $D$ (e.g., what the pitot tube manufacturer's calibration spec, or last season's flight-test campaign, told you).
- **Likelihood** $P(D\mid\theta)$ — how probable the observed data is, for each candidate value of $\theta$ (same likelihood function frequentists use for MLE!).
- **Posterior** $P(\theta \mid D)$ — your updated belief about $\theta$ after seeing today's calibration data. This is a full probability *distribution*, not a single number.
- **Evidence** $P(D)$ — a normalizing constant so the posterior integrates to 1.

### 4.2 Worked Example: The Same Airspeed Sensor, Bayesian Style

Let's redo the airspeed-sensor calibration Bayesian-style, using the classic **Normal-Normal conjugate model**, which has a clean closed-form solution — no numerical integration required.

**Setup.** Assume the sensor noise has a known standard deviation $\sigma = 0.6$ kt (from the manufacturer's datasheet or a prior characterization campaign). We want to estimate the true bias-adjusted mean reading $\mu$.

**Prior.** Before today's tow test, you trust the datasheet enough to believe $\mu \sim \mathcal{N}(\mu_0, \sigma_0^2)$ with $\mu_0 = 50.0$ kt (the known reference airspeed — you expect the sensor to be roughly unbiased) and $\sigma_0 = 1.0$ kt (fairly loose — this particular installation hasn't been characterized yet).

**Update rule.** For $n$ Gaussian observations with known noise $\sigma$, the posterior is also Gaussian, with a beautifully interpretable closed form:

$$\sigma_{post}^2 = \left(\frac{1}{\sigma_0^2} + \frac{n}{\sigma^2}\right)^{-1}, \qquad
\mu_{post} = \sigma_{post}^2 \left(\frac{\mu_0}{\sigma_0^2} + \frac{n\bar{x}}{\sigma^2}\right)$$

This is literally a **precision-weighted average** of the prior mean and the data mean — precision meaning $1/\text{variance}$. More data (larger $n$) or a more confident prior (smaller $\sigma_0$) pulls the posterior mean toward that source more strongly. This is exactly the math your flight computer's sensor-fusion filter runs every time it blends a new measurement with its current state estimate.

```python
import numpy as np

readings = np.array([50.8, 49.6, 51.2, 50.1, 49.4, 50.9,
                      50.3, 49.8, 51.0, 50.5, 49.7, 50.6])
n = len(readings)
xbar = np.mean(readings)
sigma = 0.6          # known measurement noise (datasheet)

mu0, sigma0 = 50.0, 1.0   # prior belief

# Conjugate Normal-Normal update
posterior_var = 1.0 / (1.0/sigma0**2 + n/sigma**2)
posterior_mean = posterior_var * (mu0/sigma0**2 + n*xbar/sigma**2)
posterior_sd = np.sqrt(posterior_var)

print(f"Prior:      mean = {mu0:.3f} kt, sd = {sigma0:.3f} kt")
print(f"Data mean:  {xbar:.3f} kt  (n = {n})")
print(f"Posterior:  mean = {posterior_mean:.3f} kt, sd = {posterior_sd:.3f} kt")

# 95% credible interval (symmetric, since posterior is Gaussian)
from scipy import stats
ci_low, ci_high = stats.norm.ppf([0.025, 0.975], loc=posterior_mean, scale=posterior_sd)
print(f"95% credible interval: [{ci_low:.3f}, {ci_high:.3f}] kt")
```

**Verified output:**
```
Prior:      mean = 50.000 kt, sd = 1.000 kt
Data mean:  50.325 kt  (n = 12)
Posterior:  mean = 50.316 kt, sd = 0.171 kt
95% credible interval: [49.981, 50.650] kt
```

**Interpretation — and here is the payoff:** the 95% **credible** interval means exactly what people *wish* a confidence interval meant: *given today's data and the prior, there is a 95% probability that the true bias-adjusted airspeed $\mu$ lies in [49.981, 50.650] kt.* That's a direct probability statement about the parameter itself — legitimate in the Bayesian framework because $\mu$ is treated as a random variable with a distribution, not a fixed unknown constant.

Notice also that the posterior mean (50.316) is pulled very slightly toward the prior mean (50.000) compared to the pure data mean (50.325) — that's the prior doing its job. With only 12 data points and a fairly loose prior, the pull is small; with fewer flight-test points or a tighter, better-characterized prior, it would be larger.

---

## 5. Side-by-Side: The Same Airspeed-Sensor Problem, Two Ways

| Quantity | Frequentist | Bayesian |
|---|---|---|
| Point estimate | 50.325 kt (sample mean / MLE) | 50.316 kt (posterior mean) |
| Interval | [49.943, 50.707] kt (95% CI) | [49.981, 50.650] kt (95% credible interval) |
| What the interval means | 95% of intervals from repeated calibration runs would contain the true $\mu$ | 95% probability the true $\mu$ is in *this* interval, given data + prior |
| Uses prior info? | No | Yes (datasheet belief pulled the estimate slightly toward 50.0 kt) |
| What if $n$ were huge (e.g., a full flight-test campaign)? | Estimate converges to true value; interval shrinks | Posterior converges to the same place — the prior's influence vanishes as data dominates |

This last row matters: with enough data, **Bayesian and frequentist point estimates converge**. The philosophical disagreement is sharpest in small-data, high-prior-knowledge regimes — exactly the regime flight test lives in (you rarely get thousands of tow-test runs or wind-tunnel points with a real airframe; every flight hour is expensive).

---

## 6. Sequential Updating: Why Bayesian Thinking Feels Natural in Flight Computers

A key practical advantage of the Bayesian framework: **the posterior from the last timestep becomes the prior for the next one.** You don't need to store all your raw data and recompute from scratch — you can update belief incrementally as each new measurement streams in from the IMU, GPS, barometer, or pitot tube at 50–200 Hz. This is *exactly* what a Kalman filter does at every timestep, and it's why the EKF/UKF is the standard state estimator in essentially every autopilot (PX4, ArduPilot, and every certified flight management system alike).

```python
import numpy as np

# Simulate sequential Bayesian updating as pitot readings stream in one at a time
true_airspeed = 50.0
sigma = 0.6                 # known sensor noise
mu, sigma0 = 50.0, 1.0      # start with the same prior as before

rng = np.random.default_rng(42)
stream = rng.normal(true_airspeed, sigma, size=20)

print(f"{'step':>4} {'reading':>9} {'posterior mean':>15} {'posterior sd':>13}")
for t, x in enumerate(stream, start=1):
    # precision-weighted update using ONE new point at a time
    precision_prior = 1.0 / sigma0**2
    precision_obs = 1.0 / sigma**2
    sigma0_new_sq = 1.0 / (precision_prior + precision_obs)
    mu_new = sigma0_new_sq * (mu * precision_prior + x * precision_obs)
    mu, sigma0 = mu_new, np.sqrt(sigma0_new_sq)
    print(f"{t:4d} {x:9.3f} {mu:15.3f} {sigma0:13.3f}")
```

**Verified output (first and last few steps):**
```
step   reading  posterior mean  posterior sd
   1    50.183          50.134         0.514
   2    49.376          49.813         0.391
   3    50.450          50.003         0.327
   ...
  18    49.425          49.951         0.140
  19    50.527          49.981         0.136
  20    49.970          49.981         0.133
```

Watch two things happen as the loop runs: the posterior mean drifts toward the true airspeed (50.0 kt), and the posterior standard deviation *monotonically shrinks* — each new reading makes the filter more certain. This is the beating heart of a 1-D Kalman filter with no process noise (a "static" Kalman filter). Add a motion/process model — e.g., aircraft dynamics predicting how airspeed, attitude, or position evolve between measurements — and you have the general Kalman filter that fuses IMU + GPS + barometer + pitot into the single state estimate your autopilot's control laws actually fly on.

---

## 7. Hypothesis Testing: Frequentist p-values vs. Bayesian Model Comparison

Suppose your team changes an autoland control law and wants to know whether touchdown accuracy really improved. You run a batch of simulated (or flight-test) approaches and record touchdown deviation from the runway centerline, in meters (lower is better).

**Frequentist approach — two-sample t-test:**

```python
import numpy as np
from scipy import stats

old_law = np.array([12.4, 12.6, 12.1, 12.9, 12.3, 12.7, 12.5, 12.2])   # touchdown deviation, m
new_law = np.array([11.9, 12.1, 11.7, 12.3, 11.8, 12.0, 11.6, 12.2])   # touchdown deviation, m

t_stat, p_value = stats.ttest_ind(old_law, new_law)
print(f"Mean old law: {old_law.mean():.3f} m, Mean new law: {new_law.mean():.3f} m")
print(f"t = {t_stat:.3f}, p = {p_value:.4f}")
```

```
Mean old law: 12.463 m, Mean new law: 11.950 m
t = 4.001, p = 0.0013
```

With $p = 0.0013 < 0.05$, you reject $H_0$ (no difference) and conclude the reduction in touchdown deviation is statistically significant — very unlikely to be simulation/flight-test noise alone.

**Bayesian approach — posterior over the *difference*:**

Rather than a single p-value, a Bayesian analysis (e.g., using a Bayesian t-test or a simple Monte Carlo simulation from each group's posterior) produces a full posterior distribution for $\Delta = \mu_{old} - \mu_{new}$, from which you can directly state things like "there is a >99.9% posterior probability that the new control law reduces touchdown deviation" and "the improvement is likely between 0.26 m and 0.76 m." This also naturally answers *operational* significance ("is this improvement big enough to matter for runway excursion risk?"), not just statistical significance — something a p-value alone cannot tell you.

```python
import numpy as np

rng = np.random.default_rng(0)
n_sims = 200_000

# Draw posterior samples for each group's mean using flat (uninformative) priors,
# which for Gaussian data reduces to sampling from a t-distribution centered at
# the sample mean -- here approximated with a normal for simplicity.
old_post = rng.normal(old_law.mean(), old_law.std(ddof=1)/np.sqrt(len(old_law)), n_sims)
new_post = rng.normal(new_law.mean(), new_law.std(ddof=1)/np.sqrt(len(new_law)), n_sims)

diff = old_post - new_post
print(f"P(new control law reduces touchdown deviation) = {(diff > 0).mean():.4f}")
print(f"95% credible interval for improvement: "
      f"[{np.percentile(diff, 2.5):.3f}, {np.percentile(diff, 97.5):.3f}] m")
```

```
P(new control law reduces touchdown deviation) = 1.0000
95% credible interval for improvement: [0.260, 0.764] m
```

---

## 8. When to Use Which

| Situation | Favor | Why |
|---|---|---|
| You need a quick, standardized test for a flight-test report or DER/DAR review | Frequentist | p-values and CIs are the established convention most aviation test and certification documentation expects |
| You have real prior knowledge (sensor datasheets, prior flight-test campaigns, aerodynamic models) you want to formally include | Bayesian | Priors let you encode that knowledge instead of ignoring it |
| You have very little data (a handful of flight-test points — every hour of flight time is expensive) | Bayesian | A sensible prior regularizes noisy small-sample estimates |
| You're building a real-time state estimator (attitude/position/velocity fusion, EKF/UKF, GPS-denied navigation) | Bayesian | Kalman/particle filters are sequential Bayesian updates by construction |
| You need a simple, uncontroversial answer to "is control law A different from B?" for a design review | Frequentist | t-tests/ANOVA are fast, well-understood, and computationally cheap |
| You want the probability of a hypothesis being true (e.g., "this contact is a real obstacle"), not just data-given-hypothesis | Bayesian | Only Bayesian posteriors directly give $P(\theta \mid D)$; p-values give $P(D \mid H_0)$-flavored quantities |
| You're fusing detections from a perception stack (vision, LiDAR, ADS-B) with prior traffic/terrain models for detect-and-avoid | Bayesian | Bayesian filtering (e.g., Bayesian occupancy grids, multi-hypothesis tracking) is the standard approach |
| You have a LOT of flight-test or simulation data | Either | With enough data both approaches converge to essentially the same answer |

In practice, most autonomous-aviation teams use **both**: frequentist tools (t-tests, ANOVA, regression CIs) for offline flight-test analysis and comparing control-law or perception-model designs, and Bayesian tools (EKF/UKF sensor fusion, Bayesian occupancy grids, particle filters for GPS-denied nav) for real-time state estimation and perception onboard the aircraft. Knowing both lets you pick the right tool instead of forcing every problem into one framework — and lets you speak the language reviewers on both sides of the aisle expect.

---

## 9. Common Pitfalls

1. **Misreading a p-value as "probability the null hypothesis is true."** It isn't. $p = P(\text{data this extreme or more} \mid H_0)$, not $P(H_0 \mid \text{data})$.
2. **Misreading a confidence interval as a probability statement about the parameter.** A 95% CI is about the *procedure*, not this one interval. (This is precisely the gap a Bayesian credible interval closes.)
3. **"p-hacking" — testing until $p < 0.05$.** Running many tests on the same flight-test dataset and reporting only the significant one invalidates the 5% error guarantee. Pre-register your hypothesis (and your test plan) before collecting data when possible — standard practice in flight test for exactly this reason.
4. **Choosing an overconfident Bayesian prior and never checking it.** A prior that's too tight can swamp real flight-test data with only a handful of samples, potentially masking a real sensor fault. Always sanity-check sensitivity to the prior (try a looser one and see if conclusions change) — especially before that prior informs a safety-critical fusion filter.
5. **Confusing statistical significance with operational significance.** A tiny, real effect (say, a 0.05 m touchdown improvement) can still be statistically significant with enough data while being operationally meaningless. Conversely, an underpowered flight-test campaign can fail to detect a real, safety-relevant bias just because $n$ was too small.
6. **Ignoring that MLE and Bayesian MAP (maximum a posteriori) coincide when the prior is "flat".** If you use an uninformative (flat) prior, the Bayesian posterior mode equals the frequentist MLE — the two frameworks aren't always as far apart as they seem.
7. **Treating a Kalman filter's output as ground truth without checking filter consistency.** A Kalman filter is only a valid Bayesian estimator if its noise models ($Q$, $R$) are correct. Overconfident noise covariances (too small $R$/$Q$) produce a posterior that's confidently wrong — a classic cause of state-estimation divergence in flight.

---

## 10. Exercises

Try these with your own team's UAV/aircraft sensor or flight-test data — replace the numbers below with real measurements when you can.

1. A fault detector catches 98% of real faults and falsely alarms on 2% of healthy flights. Faults occur on one flight in 500. Take 10,000 flights, count how many alarms are real, and decide whether an alarm should worry you.
2. Five tests were run and four passed. Give the frequentist estimate and the Bayesian estimate using the counting method from the ten-landings section, and explain what's shaky about the frequentist one here.

---

## 11. Summary Table

| Concept | Frequentist Term | Bayesian Term |
|---|---|---|
| Best single estimate | Maximum Likelihood Estimate (MLE) | Posterior mean / MAP |
| Uncertainty range | Confidence interval | Credible interval |
| Comparing two conditions (e.g., control laws) | p-value, t-test | Posterior probability of a difference |
| Prior knowledge | Not formally included | Encoded as a prior distribution |
| Parameter treated as | Fixed unknown constant | Random variable with a distribution |
| Sequential/streaming data | Recompute from scratch (or use running statistics) | Natural: posterior becomes next prior |
| Classic autonomous-aviation use | Flight-test reports, control-law A/B testing, sensor calibration | EKF/UKF state estimation, sensor fusion, GPS-denied nav, detect-and-avoid |

---

## 12. Further Reading and Tutorials

For a first pass, the 3Blue1Brown video on Bayes' theorem is about as clear as this material gets, and StatQuest on YouTube covers every term in this lesson at a slower pace. Additionally, Steven Brunton has very helpful videos on YouTube where he walks through examples.

If you want a book, Richard McElreath's *Statistical Rethinking* is the most approachable serious treatment of Bayesian thinking, and his lecture series is free online. Allen Downey's *Think Bayes* is also free, is built around Python, and stays light on the math. On the frequentist side, Wasserman's *All of Statistics* is a compact reference for looking up what a given test actually does.

Before you write up any results, read Greenland et al., "Statistical tests, P values, confidence intervals, and power: a guide to misinterpretations" (2016). It's a numbered list of twenty-five specific mistakes and it will save you from most of them.

For our side of things, Thrun, Burgard and Fox's *Probabilistic Robotics* derives Kalman filters and particle filters as repeated applications of Bayes' rule, which connects this lesson directly to the GNC work. This also shows exactly how Bayesian inference underlies localization, mapping, and sensor fusion; the math transfers directly to UAV state estimation. Additionally, Allen Downey's *Think Bayes*, a free, code-first introduction to Bayesian statistics in Python, is very close in spirit to the worked examples above. 

Additional resources are below:

- Wasserman, L. *All of Statistics* — a rigorous but readable bridge between the two frameworks.
- Gelman, A. et al. *Bayesian Data Analysis* — the standard Bayesian reference, freely available online.
- Groves, P. *Principles of GNSS, Inertial, and Multisensor Integrated Navigation Systems* — the standard reference connecting Bayesian/Kalman filtering theory directly to aircraft navigation and sensor fusion.
- SciPy `scipy.stats` documentation, and `PyMC` documentation for going beyond conjugate models into general-purpose Bayesian computation (MCMC) when closed-form posteriors aren't available (e.g., for complex perception-confidence models).

---





