I recently gave a talk to engineering colleagues on current best practices in testing machine learning systems. I've typed up a version of the talk I gave below, which provides details of the current advice as far as I can tell, some of the currenlty available tools that can help with this task, as well as some of my own thoughts around the best way to prioritise the various advice.

The talk was heavily influenced by the list of references that I have given at the end. To be clear, this talk was derived in large part from the readings that I did. The originality here is 1) applying my own prioritisation based on my experience of building machine learning systems and 2) Linking up theoretical discussions with the practical steps of how these can actually be implemented. 

Hopefully it can be a helpful resource for anyone looking to add some tests into their machine learning software.

## Testing Mindset

#### Why ML systems are different

Testing ML systems is different from testing regular software. In the case of non-ML software, source code (input) is compiled into programs (output). For ML systems source code, data and models are all inputs to the system. This means that not only do we need to care about all the things normal software projects should care about (as these things can and [do go wrong in ML systems too](https://www.usenix.org/conference/opml20/presentation/papasian)), but also a bunch of things on top of that. This adds additional complexity that is harder to test than standard software: data is large and hard to interrogate, training is complex and it is hard to define what a "correct" outcome looks like and since ML Engineering is a nascent space, there is less well established tooling compared to regular software.

In addition, ML systems are more likely to fail silently than standard software systems.  In regular software you will normally get some sort of error if something isn't working correctly. In machine learning this is not guaranteed. ML systems can fail in many silent ways: for example, deployed model performance may degrade over time, production data may drift from training data, hyperparameters may have been chosen poorly or there may be upstream data pipepline changes that affect model inference. I'm sure you can think of many more cases like this.

The different component parts of ML systems from regular software systems (code as well as data and models) and the different failure states (silent not loud) mean that a different approach is required to testing ML systems than normal software systems. What is the approach then that we should take?

#### Recommendations for testing ML systems

The approach I recommend can be roughly broken down into three parts. First, for bits of your system that look like regular software code, it should be tested in the standard way. For example, logic that computes an input feature to your model should be tested for correctness and pipelines should be integration tested.

Second, data and model training code should be tested fairly loosely, with smoke tests that show you if something very wrong is happening for model training and expectation tests for your data.  Going much further than this will take up a lot of time and won't necessarily yield much reward. This may change as infrastructure and tooling is developed to help with this task.

Third, testing in production is a necessary reality for machine learning systems. Deploying a model to production is the only way to know if a model performs well on real world data or not. Monitoring your ML system is also vital in order to understand if your system performance is degrading, or something has broken silently.

So what are the tests that you should implement for ML systems? As far as I can tell, there is no consensus around a standard testing suite for ML software. I have read a bunch of resources linked at the end of this talk which cover this topic, and have pulled out commonalities and tried to make a set of recommendations that are acheivable for most ML Engineering teams that will help to identify most of the most common issues with ML software, whilst avoiding writing tests for the sake of it.

The ML Test score is one of the key resources here, and it breaks ML tests down into 4 sections: tests for features and data, tests for model development, tests for ML infrastructure and monitoring tests. We will look at each of these now and I will make some recommendations for tests that should be prioritised, as well as exisiting tools that are available to make this process easier.

## Tests for Features and Data

The input data to ML systems is one of the core differences from regular software, and one of the most common ways for ML systems to go wrong. There are 3 tests of features and data that should be implemented in ML systems. These are:

1. **Write a schema for feature expectations**

Domain knowlege and introspection of your data will lead you to have expectations of how the input data to your ML system should look. For example if you have a model that takes user's heights as input you might want to encode that these values should be between 50cm and 4m. This encoding will enable alerts to fire if there are any deviations from this (we will come back to this in the monitoring section). The standard tool for doing this in Python is a package called [great_expectations](https://greatexpectations.io/). It is a python library that allows you to validate, document and profile your data to catch any issues as soon as they emerge. 

You should not overuse these tools though. They can quickly add a lot of compute requirements to your system, especially if you have a lot of data and/or features. Alerts that fire will not necessarily translate into poor model performance, and data are likely to change over time, so creating too many alerts here could lead to [alert fatigue](https://www.atlassian.com/incident-management/on-call/alert-fatigue****), desensitizing alert receivers.

2. **Adhere to any meta-level requirements**

Where there are requirements for your system like the use of PII data or any other privacy concerns, these must be implemented. If possible you should programmatically enforce this so that it is impossible to use any inappropriate data as features to ML models. If that is not possible, implementation of this relies on good project governance and communication across teams. 

Data pipelines should also have appropriate privacy controls in place to ensure that PII data does not leak. Restrictions should be in place for raw user data such that it cannot be used inappropriately. Privacy transformations can be applied where appropriate, after which data can be made available to ML systems. These transformations could include [pseudonymisation](https://en.wikipedia.org/wiki/Pseudonymization), [tokenisation](https://en.wikipedia.org/wiki/Tokenization_(data_security)) or [generalisation](https://www.privitar.com/blog/data-generalization-advanced-de-identification/), depending on the use case.

3. **Write unit tests for feature calculations**

Unit tests should be written for all feature creation code to ensure it is implemented correctly. Common feature engineering techniques you may need to pay particular attention to here include: handling of missing values, encoding of categorical features and feature embeddings. 

[Pytest](https://docs.pytest.org/en/7.2.x/) is your friend for this task and should be able to handle most tests at this stage, especially if used in combination with the various testing functionality of commonly used data handling libraries like numpy, pandas and Pytorch. In general you should not chase overall code coverage in a machine learning system due to their complexity, but for this code you should aim for 100% coverage.

## Tests for Model Development

When developing your model there are a number of tests you should consider implementing. The focus for these tests is on making sure that model development is reproducible, the extra engineering effort of implementing an ML system are actually worth it, and that the system will work well for different slices of data and in an unbiased way.

1. **Version control your data and code** 

Whenever you develop a new model specification you should at least commit the code to a repo. This will help to ensure that you can reproduce training later avoiding headaches that can occur when you can't replicate your best training run, and can save time if you need to revisit different specifications, for example because the input data changed.

It is generally considered good practice to start simple with model specifications and slowly add more complexity. Simpler models should in general be preferred to more complex ones for a given level of performance, and engineering in this way makes it much easier to debug models that if you start at the deep end.

2. **Test your model for potential bias**

If there are any protected user categories that you are dealing with in your data, you should ensure that your model is not biased against them. For example if you are building a credit score model and your data includes the race or gender of the applicant, these should obviously not be included as features in your model, but you should also check that there are no strong correlations between input features and these protected categories. Also check how sensitive model outcomes are to changes in any of these sensitive fields.

[Equality of Opportunity in Supervised Learning](https://arxiv.org/abs/1610.02413) and [Man is to Computer Programmer as Woman is to Homemaker? Debiasing Word Embeddings](https://arxiv.org/abs/1607.06520) are further readings recommended by the ML test score paper for further details.

3. **Test your model against a simple baseline**

ML systems are more complex than non-ML systems and complex ML systems are more complex than simple ML systems. Additional complexity leads to greater costs both in terms of time (development and maintenance) and compute. To justify the additional expenditure you need to be sure that additional complexity is worth it. The simple way to do this is to add a simple baseline to test your models against. This will both enable cost-benefit conversations, and can help to debug issues in your training pipeline.

Here are a few baseline models that you could consider, the best choice will depend on the particular use case:

- Random: e.g. a classifier that predicts a class at random
- A simple heuristic: e.g. show a news feed in chronological order
- Zero rule baseline: e.g. predict the most frequent class
- Human baseline: e.g. self-driving vs. human drivers
- Exisiting solutions: does your model perform better than what is already out there?

4. **Expand your evaluation metrics to cover more than just performance**

The performance of your model is important, but when building an ML system it is not the only important evaluation metric. One especially important test here is to check your model performance on different slices of data. For example if you have a model that predicts taxi fares based on journey routes, you should check the performance in all of the geographies the model is likely to be deployed in. This will ensure that your model reaches minimum performance thresholds for all the major subsets of users of the system, and can also reveal misleading top-level performance (see [Simpsons Paradox](https://en.wikipedia.org/wiki/Simpson%27s_paradox)).

Other evaluation metrics that could also become part of your evaluation suite include:

1. Peturbation tests. What is the performance impact of adding noise to the test data set? How sensitive is your model?
2. Invariance tests. Does making changes to sensitive fields have a signficant impact on outcomes?
3. Directional expectation tests. Does changing features lead to expected change of direction in the output.
4. Model calibration. Do things that are predicted to happen 70% of the time actually happen 70% of the time? See `sklearn.calibration.CalibratedClassifierCV` for implementation of Platt scaling. See also [this implementation](https://github.com/gpleiss/temperature_scaling****) of temperature scaling and this[ blog post](https://www.unofficialgoogledatascience.com/2021/04/why-model-calibration-matters-and-how.html)
5. Confidence measurement. How confident are your model predictions? What is the threshold for certainty at which predictions should be shown?

## Tests for ML Infrastructure

The objectives for testing ML infrastructure are to try to make sure that the model and pipeline is correctly specified and that there are well established processes in place to update, and if necessary, roll back, model deployments. 

1. **Model pipeline and specification are "smoke tested"**

Testing model code is currently a really difficult thing to do, model APIs are complex and there can be subtle errors that can be hard to detect. Expecting thorough, fully covered tests here is likely asking too much of the ML engineering team. As such most recommendations are for smoke tests rather than in depth unit and integration tests. There are three recommendations for tests here.

First, testing model specification (you are using the model API correctly and the algorithm is doing what you want it to do) is difficult. The recommended way to test that at least nothing is going horrendously wrong is to ensure that your model can memorise a batch of training data. To implement this, train your model on a single batch of data and ensure that it's loss can get close to zero. You may need to turn off any regularisation your model has to do this. If you're using PyTorch Lightning there is a `overfit_batches` feature that can help. You can test not just the loss but also the wall time as a way to test for more subtle bugs that may affect your models ability to learn. This will be far from foolproof though.

Second, integration test the whole ML pipeline. This means having a test that assembles training data, generates features, trains a model, validates the output and deploys the model. The test should validate that data and code can move through the full pipeline and that the resultant model performs well. 

Both of these tests can be set up using pytest and should be part of continuous integration pipelines.

Third, there should be some process available to pass a single data example through the model to understand why it is interpreted the way it is. Questions often come up where a data point has led to some seemingly odd prediction, and as an ML Engineer you will need to be able to understand why. The [TensorFlow debugger](https://www.tensorflow.org/tensorboard/debugger_v2) can be used for this if you are using Tensorflow. 

2. **Deployment and rollback process is agreed and practiced**

Offline testing on its own, however good it is, can never guarantee that a model will work well on real data or your live production environment. You should therefore have well defined and practiced processes in place to deploy new models to (at least some) live data, and to roll back to previous model versions/ processes if the model fails to reach minimum performance thresholds.

You can use canary deployments to test your new model on some live data. Canary deployments work by diverting a small % of traffic to the new model. This will test that your model works on the production environment (it is not uncommon for there to be dependency mismatches as modeling code tends to change more quickly). As you gain confidence that the model inference works, you can slowly increase the proportion of traffic that the new model receives, until eventually it is fully deployed. Shadow deployment or A/B testing are alternative methods to achieve the same thing, but come with additional labour and computation costs.

Much like a firefighter, you need to be ready for when things go wrong, and be well placed to intervene in the system to roll back to a previous version where needed. Document and practice rolling back a deployed model to the last known good state, when you're not in an emergency scenario.

## Test in Production (Monitoring Tests for ML)

To ensure that your system is still performing to the standards expected of it, it is vital to monitor basic information about your system. Monitoring is the act of tracking and logging metrics that can help to determine when something goes wrong.

There are three sets of ML specific metrics that you should try to measure. 

1. **Performance**

The most important thing to track is how well your model is actually performing in the wild. This is not a trivial task, especially if the data you receive is unlabelled. There are a few different strategies you can use here. 

First you can consider tracking user feedback of your model. This is likely the most effective way to understand if your model is performing to the level that is expected, since loss metrics do not necessarily correlate well with user experience. To track this you would need to work closely with product managers to ensure that user feedback can be measured and tracked.

Second, tracking the accuracy of your model against loss metrics can't hurt either. It may be difficult to do this especially if you need to generate labels for the data seen in production, but there isn't really much excuse for not doing so, even if it ends up being a fairly hacky approach. You can label a sample of data every day/week to check in on model performance and accuracy over time.

You could also think about tracking any domain specific proxy metrics, for example edge cases that  you know your model doesn't perform well on. This approach is likely to be less useful than the two previously mentioned.

2. **Input data / features**

For your model to perform well in the wild, the data it sees in production shouldn't be too different from the data that it was trained on. So it makes good sense to check that this is the case. 

To do this, you can track whether or not the schema that you have defined previously (e.g. with `great_expectations`) is being met. Again though, this can get quite expensive logging and compute wise and won't necessarily highlight any performance degradations.

Another thing you can track is whether or not your data is drifting. How to do this is a whole topic in itself, so I won't go into too much detail here, but the basic idea is that you can compare the distribution of your data to distribution it was trained on. Data can drift suddenly or gradually and can indicate lots of things: upstream bugs, changes in user behaviour, seasonality effects, malicious actors, churned users. To read more on this see [this Gantry blog post](https://gantry.io/blog/youre-probably-monitoring-your-models-wrong/) for a discussion of recommended metrics to track drift.

3. **Predictions**

Finally it can be a good idea to check the predictions themselves for any distributional shifts. This can give you a quicker insight into something going wrong than relying on delayed performance based metrics. If you have a regression or classification task it is also relatively easy to detect any drift in these predictions, that could potentially indicate some issue that is worth investigating.

In addition to these ML-specific metrics, it also makes sense to track general performance metrics too, since lots of the issues with ML systems in practice have nothing to do with ML. These can include CPU/GPU utilisation, prediction latency, throughput, % 2xx API responses and many more. There are already good tools in place with most cloud providers to track this e.g. AWS Cloudwatch  or GCP Cloud Monitoring.

Finally, a very common cause of issues is upstream dependency changes. You should make sure to set up notifications to and actually read about any upstream dependency changes. 

## How do you monitor?

There are three main tools at your disposal to monitor your ML system.

1. Logs: you can get your system to produce logs of key information listed above. You can then process these either periodically or streaming.
2. Dashboards: you can track ML-specific mointoring metrics in a dashboard. These tend to be quite hard to integrate with exisiting monitoring tools (see for example this post). Likely the best approach to this is to use some of the open source or SaaS options that are designed specifically for ML use cases. For open source tools you can look at [Evidently AI](https://github.com/evidentlyai/evidently) or [whylogs](https://github.com/whylabs/whylogs). And for SaaS [Gantry](https://gantry.io/), [Aporia](https://www.aporia.com/), [Superwise](https://superwise.ai/), [Arize](https://arize.com/), [Fiddler](https://www.fiddler.ai/) and [Arthur](https://arthur.ai/.) are all options.
3. You should alert the right people about issues. An alert has three components: a policy (condition for an alert), notification channels (who should be notified), description of the alert (what has gone wrong). Try to avoid alert fatigue so don't set up too many.

## References

[Designing Machine Learning Systems](https://smile.amazon.co.uk/Designing-Machine-Learning-Systems-Production-Ready/dp/1098107969)

Full Stack Deep Learning [Lectures](https://www.youtube.com/@FullStackDeepLearning) and [Lecture Notes](https://fullstackdeeplearning.com/course/2022/)

[The ML Test Score](https://research.google/pubs/pub46555/)

[Reliable ML Systems Talk](https://www.usenix.org/conference/opml20/presentation/papasian)