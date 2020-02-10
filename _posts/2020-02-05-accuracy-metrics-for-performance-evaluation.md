---
layout: post
title: "Accuracy Metrics for Performance Evaluation of Machine Learning Models."
date: 2020-02-05
excerpt: "Some of the common accuracy metrics for performance evaluation"
tags: [Machine Learning]
comments: true
---

### When is your prediction function good?

1. Compare to previous  models, if they exist.
2. Is it good enough for business purposes?
3. But also helpful to have simple baseline models for comparison.
	1. To make sure you are doing significantly better than trivial models.
	2. To make sure the problem you are working on even has a useful target function.

### Regularized Linear Model

* Whatever facny model you are using(gradient boosting, neural network)
	* always spend some time building a linear baseline model.
* Build a regularized linear model
* If your fancier model is not beating linear,
	* perhaps something is wrong with your fancier model like hyperparameter settings, model architecture,etc or
	* you do not have enough data to beat the simpler model.

### Confusion Matrix

* A confusion matrix summarizes results for a binary classification problem.

<figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/accuracy-metrics/confusion-matrix.png?raw=true">
	<figcaption><center>Confusion matrix for binary classification.</center></figcaption>
</figure>

* `Accuracy`is the fraction of correct predictions.

<a href="https://www.codecogs.com/eqnedit.php?latex=\frac{a&plus;d}{a&plus;b&plus;c&plus;d}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\frac{a&plus;d}{a&plus;b&plus;c&plus;d}" title="\frac{a+d}{a+b+c+d}" /></a>