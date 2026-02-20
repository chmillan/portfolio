  
# Robust Indicator Design for Noisy Performance Measurements


**Simulation-based evaluation of robust estimators under high variability**

> Public technical report · Full document available  
> 📄 Full report: https://medicoes.nic.br/media/Publicacao-internet-escolas-2024.pdf

---

## Context

Continuous Internet speed measurements in Brazilian public schools exhibit substantial variability. Observed download speeds frequently fluctuate due to network oscillations, infrastructure differences, and usage peaks.

The analytical challenge was to define a **robust and comparable indicator** capable of representing the speed typically available to students — without being distorted by extreme values.

---

## Statistical Problem

Empirical distributions across schools show:

- Strong right skewness  
- Long tails  
- Occasional bimodality  
- Heterogeneous variance  

Naïve summaries such as maximum or mean systematically overestimate performance.

The goal was to identify a reference metric that balances:

- Sensitivity to real performance changes  
- Resistance to extreme spikes  
- Cross-school comparability  

---

## Methodological Approach

The evaluation combined empirical analysis and controlled simulations.

### 1. Empirical characterization  
Millions of real measurements were analyzed to understand distributional patterns across schools.

### 2. Simulation-based comparison  
Synthetic datasets were generated to replicate observed behaviors (symmetric, skewed, bimodal).  
Artificial outliers were injected in controlled proportions (1–5%).

Four candidate strategies were evaluated:

- P95  
- P99  
- IQR-based rule  
- Adaptive distribution-sensitive rule  

Performance was assessed using precision, recall, and F1-score under bootstrap resampling.

---

## Key Findings

**P99**  
Highly selective but misses relevant upper-tail behavior.

**IQR rule**  
Too conservative in long-tailed distributions; increases false positives.

**Adaptive rule**  
Flexible but reduces standardization and comparability.

**P95**  
Consistently achieved the best balance between robustness and representativeness across distribution types.

---

## Empirical Application

When applied to real school data:

- Maximum values were frequently inflated by isolated peaks  
- P95 reduced artificial inflation  
- Stable schools remained largely unaffected  
- Highly variable schools showed meaningful correction  

P95 preserved typical performance while mitigating distortion.

---

## Conclusion

P95 offers the most stable and interpretable compromise between robustness and sensitivity.

It captures the upper practical performance region of the distribution while avoiding systematic overestimation — making it suitable for large-scale monitoring and policy benchmarking.

---

## Transferable Expertise

- Simulation-based methodological validation  
- Robust statistics under heavy-tailed distributions  
- Outlier detection framed as a classification problem  
- Bootstrap-based performance evaluation  
- Bias–variance trade-off analysis  
- Indicator design for monitoring systems  
  