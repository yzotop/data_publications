---
link: https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f
tags:
  - data
data_type:
  - AB tests
---
<span style="background:#ff4d4f">[[assests.ipynb|скрипт]]</span>



Hi, we are [Nikita](https://www.linkedin.com/in/marnikitta) and [Daniel](https://www.linkedin.com/in/daniel-savenkov-098b41149) from the CoreML team at VK. It’s our job to design and improve recommender systems for friends, music, videos and the news feed. This involves tons of A/B tests, and our progress at the end of the day relies heavily on how accurate and efficient our A/B tests were.

The two most essential things in A/B tests are the design of the experiments and accurate analysis of the experiments’ results. In this article, we will stick to the most common design and compare various statistical analysis procedures, from the very standard t-test and Mann-Whitney test to state-of-the-art approaches like the reweighted bootstrap. The code to reproduce everything you will see in this post is available here: [https://github.com/marnikitta/stattests](https://github.com/marnikitta/stattests). The “sandbox” notebook you can use to play with the tests is here: [https://github.com/marnikitta/stattests/blob/master/notebooks/Sandbox.ipynb](https://github.com/marnikitta/stattests/blob/master/notebooks/Sandbox.ipynb).

After reading this article, you will learn how to choose the right statistical test from the many available and run it on your own data.

![](https://miro.medium.com/v2/resize:fit:700/1*7teeHm7SEKBT6rL93rwcew.png)

# Table of Contents

1. [Preliminaries](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#ae36)  
    [1.1. CTR Prediction Example  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#671b)[1.2. Hypothesis Testing  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#7ed5)[1.3. Data Generation  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#609f)[1.4. Sensitivity and p-value CDF](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#304c)  
    [1.5. P-value sanity check](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#1e3b)
2. [Tests on the number of clicks  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#9e58)[2.1. T-test on the number of clicks  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#17eb)[2.2. Mann-Whitney on number of successes](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#44d2)
3. [Tests on global CTRs  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#2dc9)[3.1. Assumptions  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#f325)[3.2. Binomial z-test: it fails  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#e032)[3.3. Bootstrap for global CTR  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#3854)[3.4. Delta method for global CTR  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#6f38)[3.5. Bucketization  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#d2d3)[3.6. Linearization](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#5eb5)
4. [Tests on user CTRs  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#a429)[4.1. Assumptions  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#2099)[4.2. T-test on user CTRs  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#9100)[4.3. T-test on smoothed user CTRs  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#df9c)[4.4. Intra-user correlation aware weighting](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#ca28)
5. [So what? Which test should I use?](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#1d70)
6. [Methods not in the list  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#1132)[6.1. Transformation of clicks  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#a1a0)[6.2. CUPED  
    ](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#5c90)[6.3. Using future metric prediction](https://vkteam.medium.com/practitioners-guide-to-statistical-tests-ed2d580ef04f#e8b7)

# 1. Preliminaries

## 1.1. CTR Prediction Example

Let’s assume that we want to assess the quality of our new CTR prediction model for ad targeting. Controlled experiments (A/B tests) are the most robust way to measure it.

The first step is to define the key metric that we want to monitor. In our example, we use the number of _clicks_ on ads: if our new model is better, it will select and display more relevant ads, and, as a result, the average number of _clicks_ will increase.

![](https://miro.medium.com/v2/resize:fit:700/1*hSCGdc11HCDQFNE25WsIBw.png)

In A/B tests, we split users into two random groups: control and treatment. For users in the control group, we will use our current production model, and for users in the treatment group, we will deploy the new CTR prediction model.

When the experiment is finished, we will compare the average values of our metric (average number of _clicks_ per user) in both the treatment and control groups using one of the available statistical hypothesis tests.

## 1.2. Hypothesis Testing

To run a statistical test, we start by formulating null and alternative hypotheses in terms of random variables. Usually, the null hypothesis states that nothing has changed. In our example, it would be “the average number of _clicks_ in the control and treatment groups are the same”. In contrast, the alternative hypothesis says that there is a notable difference between groups. In statistics, they are usually denoted as H0 and H1 respectively:

![](https://miro.medium.com/v2/resize:fit:700/1*gQZJTaP6QCFm3ZWb9s-93g.png)

After hypotheses are formulated, we can employ one of the dozens of statistical tests to conduct an analysis. Each of them can be characterised by two numbers that reflect the “quality” of the test: false-positive rate (FRP) and sensitivity.

FPR is the probability of rejecting H0 while it is actually true. If a statistical procedure is correct (all assumptions are met and the statistical test works fine with the given data), this rate is controlled by p-value.

Sensitivity is the probability of rejecting H0 while it is actually not true.

==It is easy to imagine a statistical test with zero FPR. We would just have to accept H0 regardless of the results of the A/B test. By doing so, we would never have a false-positive because we would never reject H0. But this statistical test would obviously have zero sensitivity because we would never reject H0 even if it were actually false.==

On the other hand, it is easy to imagine a statistical test with 100% sensitivity. We would just have to reject H0 regardless of the results of the A/B test. By doing so, we would have a 100% probability of rejecting H0 while it was actually not true. Clearly, this statistical test would have 100% FPR because we would never accept H0 even if it were actually true.

As you might guess, in practice there is always a trade-off. Higher FPR gives you higher sensitivity and vice versa. Let’s plot FPR versus sensitivity for a very standard Mann-Whitney test (the details on how to obtain a plot like this will be discussed below).

![](https://miro.medium.com/v2/resize:fit:700/1*h8WmDXaZ_QmCEzi0k0eMvg.png)

If you are a machine learning specialist, you should recognise this as an ROC curve. We always want this curve to go as close to the top-left corner as possible.

The vertical grey line corresponds to 5% FPR, the standard p-value threshold. The intersection of this line with the curve gives you the sensitivity of your test that corresponds to 5% FPR. We will call it the _power of the statistical test at 5% FPR_ (or just power). In this example, the power of the standard Mann-Whitney test at 5% FPR is 65%.

From this point, we will fix FPR to 5% and compare the powers of a vast variety of tests given this level. For the sake of brevity, we will refer to the power of a statistical test at 5% FRP as _power_.

Power of the statistical test depends on four things:

1. The test itself
2. Distribution of the experimental data
3. Effect size
4. Size of the test groups

We’re going to simulate some reasonable data for our CTR-prediction task to plot ROC curves and powers for a vast variety of tests and to understand which test to use depending on the data distribution.

## 1.3. Data Generation

![](https://miro.medium.com/v2/resize:fit:700/1*XcNUMWgoZi1DnRgjY0NbUg.png)

Let’s consider the most general CTR prediction task. Control and treatment groups have an equal number of users. Each user has seen some number of ads and has clicked on some (or none) of them. So each user is characterized by the number of _views_ and the number of _clicks_.

We assume that the distribution of views across users is log-normal: most of the users saw a small number of ads, and few users — a lot. In experiments, we would control the skewness of this distribution.

![](https://miro.medium.com/v2/resize:fit:700/1*MVZoZocuEiOZLOeylKdmew.png)

Each user is characterised by their own average conversion rate (_ground truth user CTR_). We assume that the _ground truth user CTR_ follows beta-distribution. It’s a common choice for rate distribution, as it’s bounded by zero and one. In our experiments, we will fix the mean of the distribution (mean = _success rate)_ and vary the variance. Zero variance would mean that every user has the same average CTR, and as we increase it there would be more diverse users. The variance of beta-distribution with fixed mean is controlled by the beta parameter.

![](https://miro.medium.com/v2/resize:fit:700/1*dvV4nwrmmSC3yVbyiywUvQ.png)

Each impression can be converted into click with probability=_ground truth user CTR_, so we sample the number of clicks for each user from a binomial distribution with the number of trials equal to _views_ and success probability equal to _ground truth user CTR_.

![](https://miro.medium.com/v2/resize:fit:700/1*m2O4ruIlvBvVkox8cAtW5g.png)

Success rate (mean of the beta distribution) in the treatment and control groups will differ by some uplift.

![](https://miro.medium.com/v2/resize:fit:700/1*y_YxHTstwjFzmVmnc27FIg.png)

To recap, our data generation looks like this:

1. Initialize N users for the control group, N for the treatment group.
2. Generate the number of ad views for each user in treatment and control groups from the same log-normal distribution.
3. Generate the _ground truth user CTR_ for each user from beta-distribution with mean _success_rate_control_ for the control group and _success_rate_control * (1 + uplift)_ for the treatment group.
4. Generate the number of ad clicks for each user from a binomial distribution with the number of trials equal to the number of views and the success rate equal to the _ground truth user CTR._

The whole pipeline repeated once gives us the results of one “synthetic A/B test”. We will repeat it _NN_ times to get results of _NN_ synthetic A/B tests. For each A/B test, we will run a number of statistical tests and compare performance by plotting a ROC curve.

## 1.4. Sensitivity and p-value CDF

From our data generation pipeline, we obtain NN artificial experiments where H0 is actually false and should be rejected. Let’s use a statistical test to obtain NN p-values, with one p-value per experiment. In an ideal case, we want to have all p-values equal to zero, since we know that actual treatment and control groups differ by uplift. In a real case, the p-values would be distributed somehow. Let’s take a look at the cumulative density function (CDF) of p-values.

![](https://miro.medium.com/v2/resize:fit:700/1*dWUPDPM2_HzEidRlzmHCuQ.png)

Note that the density of p-values is bigger when the p-value is small. P-values are mostly concentrated in a small value region, which is good if we know that H0 is wrong (we have non-zero uplift at the treatment group).

And there’s something even trickier. Note how the p-value CDF here is actually equal to the statistical test sensitivity. If we were to take the CDF’s value at 0.05 p-value, this would actually be the fraction of experiments when p-value <= 0.05. CDF(0.05) is the fraction of experiments when we reject H0. And as we know that H0 is not true in all experiments, CDF(0.05) is the fraction of experiments in which we reject H0 when H0 is actually not true (sensitivity). _So the plot we see here is exactly the ROC curve for the statistical test (sensitivity vs FPR)_. We will use this approach to plot ROC curves for a wide variety of statistical tests and to compare them.

## 1.5. P-value sanity check

While conducting a statistical test, we always claim that False Positive Rate is controlled by a p-value threshold. If this is true, then everything is fine. But unfortunately, it could also be wrong. The most common case when it could be wrong is when the statistical test assumptions are violated (e.g., a violation of normality for the t-test). An important thing to note is that violation of the assumptions is necessary, but not sufficient to break the statistical test. If p-value controls FPR reasonably well, the test will give us the correct results despite the violation of the assumptions. So for every statistical test, we should be able to check whether p-value controls FPR for our data.

In order to do so, we need to generate A/A tests. Let’s use the same data generation pipeline, but with zero uplift. By doing so, we will sample treatment and control groups from exactly the same distribution, so H0 will be true here. The p-value correctly controls FPR if and only if the p-value is distributed uniformly on [0, 1] for A/A tests.

To understand this fact, let’s consider the p-value threshold of 0.05. When the actual p-value falls below this threshold, we reject H0 and get a false-positive (as stated previously, H0 is true as long as we’re working with A/A tests). We claim that in this case FPR is equal to our p-value threshold, e.g. for A/A tests, p-value falls below the 0.05 threshold in only 5% of cases. This is exactly the uniformness requirement. We can use a statistical test with the given data only if the p-value distribution on A/A tests is uniform. We will check it for every statistical test in our simulations and most of the time we will see this beautiful diagonal CDF which means that everything is fine.

![](https://miro.medium.com/v2/resize:fit:700/1*f4RxFSV531MtIcl3CXmwvQ.png)

In the following sections, we will introduce a dozen statistical tests with different performance, data distribution assumptions, and computational efficiency. Each test is presented in a similar way: we give a brief intuition, data assumptions, source reference, and analysis of its performance on different data distributions.

# 2. Tests on the number of clicks

## 2.1. T-test on the number of clicks

[https://en.wikipedia.org/wiki/Student%27s_t-test#Independent_two-sample_t-test](https://en.wikipedia.org/wiki/Student%27s_t-test#Independent_two-sample_t-test)

**Additional** **assumptions:** no assumptions (normality, equal variances among groups)

Let’s start with the most basic approaches. The first one is a t-test on the average number of _clicks_. We want the average number of _clicks_ in the treatment group to be higher than in the control group, and we will test the significance of this difference.

T-tests involve strong assumptions on data, such as normality and equal variances among test groups. However, empirically, t-tests are robust to violations of the assumptions. We will see that the CDF of p-value under H0 is uniform for quite a wide range of data distribution parameters. This is the reason why assumptions are given in brackets.

Let’s set uplift to 20% and plot the results for our t-test.

![](https://miro.medium.com/v2/resize:fit:700/1*p3OIOnUk-3zWJ6vZh0Ek-g.png)

We use the same dashboard to compare the performance of all tests. The large plot in the top left is a sensitivity curve. In the bottom left corner, there is a bar that shows the power given a 0.05 p-value threshold. You can find the same value in the sensitivity plot where the blue curve crosses the grey vertical line. The most powerful test would have the longest bar.

In the top right corner, there is an FPR curve which we will use to monitor test health. The curve should be as close as possible to being diagonal. And finally, there are two histograms that reflect the data generation process: _views_ and _ground truth user CTR_ distributions. They would change as we change the skewness of _views_ and the variance of _ground truth user CTR_.

As we can see, the standard t-test works reasonably well. The test power given 0.05 p-value threshold is almost 0.7 and the CDF under H0 is pretty uniform.

But this might be considered to be a form of cheating. As you might have noticed, the distributions of _views_ and _ground truth user CTRs_ in this example are very “convenient” for the t-test. _Views_ are not heavily skewed and a standard deviation of the _ground truth user CTR_ is small. Let’s change the skewness of the _views_ distribution.

![](https://miro.medium.com/v2/resize:fit:700/1*yO40hpSLl5P7ghX-Ms7GdQ.gif)

As you can see, higher skewness drops the power of the t-test significantly. Furthermore, for very skewed distributions, t-test p-values clearly don’t follow uniform distribution under H0. In the region of small values, p-values overestimate FPR (the plot lies beyond diagonal in this region). It becomes a “conservative” estimate of FPR. It is fine to use this test, as we can still be sure that FPR <= p-value, but the power of the test is low.

The main reason for this is the strong violation of the normality assumption for highly skewed distribution. If the _views_ distribution is heavily skewed, _clicks_ distribution will be skewed as well. The more it is skewed, the more samples (users) are required to restore the sample’s mean normality.

## 2.2. Mann-Whitney on number of successes

[https://en.wikipedia.org/wiki/Mann%E2%80%93Whitney_U_test](https://en.wikipedia.org/wiki/Mann%E2%80%93Whitney_U_test)

**Additional** **assumptions:** no assumptions (a single value of metric must not account for the majority of measurements)  
**Problems:** complicated hypothesis, computational complexity

Mann-Whitney is another basic approach for comparing the average number of clicks. This test is known to work well with skewed distributions because of the following:

1. It does not require any additional assumptions of data distribution
2. Since it works with rank (clicks) rather than clicks itself, it is more robust to outliers

The assumption given in brackets is a mild condition on ties (equal metric values) in the data. The MW test won’t work properly if 99.9% of users have, for example, 2 _clicks_. In our simulation, we don’t have cases like this, which is why the MW test works fine for us. But you should be careful if the vast majority of users in your dataset made 0 _clicks_, as the MW test could fail on your data. Further discussion on this assumption can be found here: [https://www.tandfonline.com/doi/full/10.1080/00031305.2017.1305291](https://www.tandfonline.com/doi/full/10.1080/00031305.2017.1305291)

The main caveat of this test is that it does not compare sample means of test groups. This test is sometimes said to compare sample medians, but it doesn’t. Despite its H0 being quite intuitive (refer to Wikipedia), it is equivalent to neither median nor mean comparison. But empirically it works fine and is widely used.

Another limitation of the MW test is its computation complexity. In t-tests, we only need sample means, variances, and counts, which can be efficiently computed in a distributed environment. In contrast, the MW test requires global sorting of metrics, which can be a deal-breaker in certain scenarios.

Let’s compare the MW test to the t-test.

![](https://miro.medium.com/v2/resize:fit:700/1*dveuJa-igTF0cjNWjrhDEw.gif)

As you can see, when the skewness is low, the power of the MW test is roughly the same as the power of the t-test (the t-test is even somewhat better, since the rank operation in the MW test leads to some information loss). But when the distribution becomes heavily skewed and the power of the t-test drops, the power of the MW test drops much slower.

Moreover, MW test p-values under H0 follow uniform distribution even for very skewed _views_, which means that p-value always gives the correct FPR. But the MW test starts to outperform the t-test even when both tests follow uniform distribution under H0. As you can see, the t-test does not like skewness in data.

Besides the skewness of _views,_ another source of _skewness_ in _clicks_ is the distribution of _ground truth user CTRs_. If the CTR for all users is roughly the same, the skewness of _clicks_ will be roughly the same as the skewness of _views_. If the CTR changes a lot from user to user, the skewness of _clicks_ can be even higher than skewness of _views_. Let’s change the distribution of _ground truth user CTRs_ and observe what happens.

![](https://miro.medium.com/v2/resize:fit:700/1*3lX5hFChW3RLyP6PoitSLA.gif)

As you can see, the MW test starts to outperform the t-test when the variance of the _ground truth user CTR_ increases. Distribution under H0 is fine for both tests, but the t-test suffers from skewness once again.

Therefore, if your data is heavily skewed, consider using the MW test instead of a t-test. But when the skewness is low, a t-test might be a better alternative. If you’re working with big data, the computational complexity of the MW test might be challenging to deal with.

# 3. Tests on global CTRs

The most common case in A/B testing is to compare CTRs rather than _clicks_. The intuition behind it is very clear: typically the variance of _clicks_ is high. We always have users with a lot of _views_ and a lot of _clicks_ as well as users with very few _views_ and _clicks_. The variance of the CTR is much smaller. Smaller variance usually makes statistical tests more powerful.

## 3.1. Assumptions

We usually assume that the direction of change in the CTR and _clicks_ is always the same. The latter is, in fact, a strong assumption, so let’s discuss it.

![](https://miro.medium.com/v2/resize:fit:700/1*sWoFJNmJMnyxfF-APRS8hA.png)

The directionality of the global CTR is the same as the directionality of the sum of clicks _if we don’t change views in our experiment._ It can be a reasonable assumption if working with, for example, targeting an ad banner on a frontpage. It is extremely unlikely that by changing the banner we will change the average number of times a user sees it, since it depends on the number of frontpage visits. It may not be a valid assumption for news feed ranking because better ranking increases users’ engagement and they tend to visit the news feed more often.

Let’s discuss common approaches to comparing global CTRs.

## 3.2. Binomial z-test: it fails

[https://en.wikipedia.org/wiki/Binomial_test](https://en.wikipedia.org/wiki/Binomial_test)

**Additional** **assumptions:**

- we don’t influence _views_;
- each _click_ and each _view_ is an i.i.d sample.

One may think that our data is actually a binary sample of _total clicks_ (ones) and _total views_ minus _total clicks_ (zeros). Unfortunately, these samples are not i.i.d. because a single user’s clicks are not dependent. Imagine a user with a very high CTR. All his samples tend to be ones (clicks) and are clearly dependent. It’s standard in the industry to not use this test for global CTR, but let’s give it a try.

![](https://miro.medium.com/v2/resize:fit:700/1*Noi7_H24NId-mFIodRs1wQ.gif)

As you can see, when all users have the same _ground truth user CTR_, the binomial test performs really well. Theoretically, in this case it is the most powerful test. Unfortunately, when we increase the variance of the _ground truth user CTR_ and test assumptions are violated, this test underestimates FPR and CDF under H0 lies far above the diagonal. That means that we can’t be sure that FPR <= p-value and we should stay away from this test.

## 3.3. Bootstrap for global CTR

[https://en.wikipedia.org/wiki/Bootstrapping_(statistics)#Bootstrap_hypothesis_testing](https://en.wikipedia.org/wiki/Bootstrapping_(statistics)#Bootstrap_hypothesis_testing)

**Additional** **assumptions:** we don’t influence _views_

As you can see from the previous section, we can’t use the vanilla z-test for global CTR. A common choice for global CTR estimation is bootstrap. In the bootstrap, we resample users and calculate the global CTR for each sample. Then we can use this bootstrap distribution to build confidence intervals and run statistical tests.

The bootstrap is considered hard to compute when working with big data. This is often the case, but for the global CTR, we can employ the Poisson bootstrap ([http://www.unofficialgoogledatascience.com/2015/08/an-introduction-to-poisson-bootstrap26.html](http://www.unofficialgoogledatascience.com/2015/08/an-introduction-to-poisson-bootstrap26.html)). The idea is simple: instead of resampling users, we draw the number of times each user appears in a sample from the Poisson distribution. Then we weigh _views_ and _clicks_ with this number. In our code, we use this technique to speed up computations.

The global CTR can be expressed as a weighted average of user CTRs, with each user CTR weighted by the number of views.

![](https://miro.medium.com/v2/resize:fit:700/1*Uu8m09UpHL3-8RKBdqRv5g.png)

In global CTR, the contribution of each user is proportional to the number of views. We use this expression in code and run a _weighted user CTR bootstrap_ to obtain the bootstrapped global CTR distribution_._

Let’s compare the bootstrap to the MW test.

![](https://miro.medium.com/v2/resize:fit:700/1*tmwXVPvh5zoxwozyZNuIfw.gif)

When the skew of _views_ is small, the power of the bootstrap is roughly equal to the power of the MW test. But when the skew increases, the bootstrap performs much better than the MW test. The main reason is that the bootstrap explicitly takes into account _views_ for each user. Also, note that when the _views_ skew is really high, the p-value of the bootstrap starts to fail to assess the FPR (CDF under H0 goes above the diagonal). Good practice in any A/B testing is to run an A/A test sanity check on the p-value of the selected statistical test.

Let’s compare the MW test and the bootstrap while changing the variance of the _ground truth user CTR_.

![](https://miro.medium.com/v2/resize:fit:700/1*QcGhOI5Mp5kUztLzNEnvAw.gif)

As you see, when the variance of the ground truth user CTR is high enough, the MW test starts to outperform the bootstrap (the average number of clicks becomes less noisy than the global CTR). The reason why the global CTR fails would be covered in the “Intra-user correlation aware weighting” section (hint: _views_ is not an optimal weight for a user’s CTR)

Therefore, if your data is highly skewed but the _ground truth user CTR_ variance is small, consider using the bootstrap. But don’t forget to check the uniformness of p-value under H0. Also note that even with the Poisson bootstrap, the calculation of this test is computationally demanding. We will discuss three efficient alternatives to the bootstrap below.

## 3.4. Delta method for global CTR

[https://dl.acm.org/doi/10.1145/3219819.3219919](https://dl.acm.org/doi/10.1145/3219819.3219919)

**Additional** **assumptions:** we don’t influence _views_

The binomial z-test fails for the global CTR because the i.i.d. assumption does not hold. It makes a biased estimate of the variance of the global CTR. The delta method is an efficient way to get an unbiased estimate of the variance of the global CTR.

Let’s compare the delta method to the bootstrap and MW test with varying _views_ skewness.

![](https://miro.medium.com/v2/resize:fit:700/1*6hW_Nq49HWQkkieRpqR2Vg.gif)

As you can see, the power of the delta method is roughly equal to the power of the bootstrap, but when the skewness is reasonably high, p-value underestimates the FPR. You need to be really careful when using this test, and running an A/A test sanity check is a must here.

Let’s vary the _ground truth user CTR_ variance.

![](https://miro.medium.com/v2/resize:fit:700/1*h45W27PyMNpYf5n-kCT6KA.gif)

In this plot, the behavior of the delta method is pretty much the same as the behavior of the bootstrap. All in all, you should always choose the bootstrap if you are able to compute it. If the computation of the bootstrap is too hard on your data, you can use the delta method, but make sure to check the distribution of p-value under H0.

## 3.5. Bucketization

**Additional** **assumptions:** we don’t influence _views_

Another simple and efficient alternative to the bootstrap is bucketization. Instead of drawing independent samples, we split all users into a small number of buckets and calculate the global CTR for each bucket. Each group can be viewed as a ‘meta-user’ and the bucket’s global CTR can be viewed as the meta-user CTR. Then we can run a standard t-test on top of meta-users since buckets are independent.

This test has a hyperparameter for the number of buckets. The common choice is 200, but you may want to tune it for your task. In our simulation, we fix it to 50.

Let’s compare bucketization to the bootstrap and the MW test varying the skewness of views.

![](https://miro.medium.com/v2/resize:fit:700/1*omnRydOCYFoT-zvtXZICww.gif)

When _views_ distribution is skewed, bucketization has a similar performance to the bootstrap or even better, as it takes into account the number of views for each user, like the bootstrap. Note that when the skewness is increasing, p-value under H0 starts to deviate from the uniformness substantially. An A/A test is a must for bucketization.

Let’s vary the _ground truth user CTR_ variance.

![](https://miro.medium.com/v2/resize:fit:700/1*asDioWduZQ1hfoTn-uxjwQ.gif)

When _ground truth user CTR_ variance is high, bucketization fails like the bootstrap does because they estimate the same metric.

If you are working with large amounts of data, you should definitely consider using bucketization for estimating _global CTRs_ and other kinds of metrics that can’t be addressed by a classic t-test.

## 3.6. Linearization

[https://dl.acm.org/doi/10.1145/3159652.3159699](https://dl.acm.org/doi/10.1145/3159652.3159699)

**Additional** **assumptions:** we don’t influence _views_

This is yet another computationally efficient alternative to bootstrap.

It is quite clear that _clicks_ and _views_ are correlated. The more _views_ a user has, the more _clicks_ a user will make. Let’s use control group data to fit a regression of clicks on views:

![](https://miro.medium.com/v2/resize:fit:700/1*Et10QraHbTTQ2q72ERs0mA.png)

And then let’s calculate the new metric for each user:

![](https://miro.medium.com/v2/resize:fit:700/1*6hC4OcpfHvobNSz_oKUTQA.png)

Then we can run a t-test on _linearized clicks_ just like we would on ordinary _clicks_. If our additional assumption holds, i.e., we don’t influence views in our experiment, we have a theoretical guarantee that the direction of change of _linearized сlicks_ is the same as the direction of change of the _global CTR_ and _clicks_.

Let’s compare this approach to the bootstrap and the MW test varying the skewness of _views_.

![](https://miro.medium.com/v2/resize:fit:700/1*MbhJuDWpHtBYCp6I6ftB2A.gif)

For reasonably high skewness, linearization performs roughly the same as the bootstrap, but for very high skewness, performance of linearization drops. Note that p-values in this approach start to overestimate the FPR when given very high skewness.

Let’s vary the _ground truth user CTR_ variance.

![](https://miro.medium.com/v2/resize:fit:700/1*P-LPf0POjXlLB-11liYMAg.gif)

Here, linearization behaves pretty similar to the bootstrap.

Overall, linearization performance is comparable to the bootstrap, the delta method, and bucketization. They all take users’ _views_ into account, which boosts performance when the _ground truth user CTR_ variance is moderate. Regardless, you should always check their performance on your data.

At VK, we commonly use bucketization for ad-hoc metrics analysis, as it can be quickly computed across multiple days with a simple SQL query. The delta method is used for confidence intervals of percentage uplifts of metrics (it is very similar to the global CTR. See the paper for details). And finally, the bootstrap is used as a bulletproof double-check for critical decisions.

# 4. Tests on user CTRs

## 4.1. Assumptions

Global CTRs are challenging to work with. We can’t do a z-test for them directly since i.i.d. does not hold in this case. That’s why we had to come up with some tricks and hacks, such as using the bootstrap, delta method, etc. But what about working with user CTRs (not to be confused with the _ground truth user CTR,_ which we only have in the simulation and don’t observe in real-world scenarios) It may seem that if the average user CTR increases, the total number of _clicks_ will increase as well. But in reality, _this is only true given one important assumption: user CTR (and uplift in user CTR) does not depend on views_.

Here is an example when this assumption does not hold. Imagine you have 100 users in your test group: 99 of them have 100 _views_ and one of them has 100000. The initial CTR of every user is 2%. Let’s increase the CTR of 99 users with 100 _views_ by a factor of 2 and decrease CTR of the user with 100000 _views_ by a factor of 2. The average user CTR will increase approximately by a factor of 2 (recall: each user contributes equally to the average user’s CTR), but the total number of _clicks_ will drop, because the majority of _clicks_ came from the user with 100000 _views_. If you believe that there is a dependency of user CTR on _views_ in your data (e.g., users who are more active tend to have higher CTRs), you probably can skip this section because these methods will fail on your data.

In all subsequent statistical tests, we will assume that _user CTR_ (and uplift in user CTR) and _views_ are independent.

## 4.2. T-test on user CTRs

**Additional** **assumptions:**

- we don’t influence _views_
- user CTRs (and uplift in user CTR) and views are independent

For user CTRs, i.i.d. assumption holds because users are independent, which is why we can simply use the a t-test on user CTRs. We expect the power of this test to be low because users with few _views_ (and, as a result, noisy user CTR estimate) will contribute as much as users with a lot of _views_ do.

Let’s see how it actually works.

![](https://miro.medium.com/v2/resize:fit:700/1*HTLvDz8M7QzTuLAUA8cDqA.gif)

![](https://miro.medium.com/v2/resize:fit:700/1*4GPTm45mz0Fe343FR3yr_A.gif)

As expected, the test’s performance is very poor. But there is one weird trick to deal with that.

## 4.3. T-test on smoothed user CTRs

**Additional** **assumptions:**

- we don’t influence _views_
- user CTR (and uplift in user CTR) does not depend on _views_

**Problems:** no theoretical guarantees

The problem with the t-test on user CTRs is that we have users with few _views_ and their user CTRs are noisy. We know almost nothing about the CTR of users with only 3 views, regardless of the number of their _clicks_. When a user has 20 _views,_ we can make an acceptable estimate of this user’s CTR. And if a user has 200 _views,_ we can make a really good estimate of this user’s CTR.

In the bootstrap, we introduced weights to account for the number of _views_ in a statistical test and actually ended up with the global CTR. Here we will do something else. Let’s use Laplace smoothing for CTR estimation:

![](https://miro.medium.com/v2/resize:fit:700/1*B9qrJG4g_keGAqa80ystYA.png)

Alpha is some hyperparameter. Intuition is the following: when _views_ are high enough, _smoothed CTR_ is almost equal to user CTR. When _views_ are low, the smoothed CTR is almost equal to the global CTR. In other words, if a user has a lot of _views_, we can be sure that _clicks_/_views_ is a good estimate of user CTR for this user, and when a user has few _views,_ we set our estimate to the global CTR.

Although the _smoothed CTR_ is quite intuitive, _there is no theoretical guarantee that its directionality is the same as the directionality of the total number of clicks_. That is probably the main reason not to use it.

Let’s check how it performs.

![](https://miro.medium.com/v2/resize:fit:700/1*UUmjySX0PfR6glcBRTKr8w.gif)

![](https://miro.medium.com/v2/resize:fit:700/1*PWrDDuGkeOooqCM_RJD8vA.gif)

This test does a really good job when skewness is high and works roughly the same as the bootstrap when varying the _ground truth user CTR_ variance.

It is a good option for skewed _views_ distribution, but remember that _there is no theoretical guarantee that the direction of the smoothed CTR is the same as the direction of the total number of clicks._

## 4.4. Intra-user correlation aware weighting

[https://arxiv.org/abs/1911.03553](https://arxiv.org/abs/1911.03553)

**Additional assumptions:**

- we don’t influence _views_
- user CTR (and uplift in user CTR) does not depend on _views_

The direction of change in the global CTR is the same as the direction of change in total _clicks,_ given that our experiment doesn’t change _views_ (see additional assumptions). Given that the user CTR and uplift in it do not depend on _views_ (see additional assumptions), the direction of the average user CTR is the same as well. As we could see, the tests working with the global CTR are much more powerful than a t-test on user CTR. Recall that the global CTR is a weighted average of user CTRs with weight equal to _views_. Thus by weighting user CTRs with _views,_ we get substantial gain in the power of statistical tests.

But is weighting by _views_ the optimal choice for a statistical test? The important thing to understand here is that _we can use any weights as long as they do not depend on user CTRs._ Thus we can weight user CTRs with any positive function of views (recall additional assumptions: a user’s CTR does not depend on _views_)_._ Researchers from eBay found the most efficient theoretically justified weights. These weights are actually equal to the effective number of views of each user ([https://en.wikipedia.org/wiki/Effective_sample_size](https://en.wikipedia.org/wiki/Effective_sample_size)):

![](https://miro.medium.com/v2/resize:fit:700/1*R0dfDkpi8-1NCGvcpswaiQ.png)

where _views_ is the number of views of each user and 𝝆 is the intra-user correlation. For more details on the intra-user correlation and on its calculation, please refer to the paper or our code.

When intra-user correlation is zero, weight is equal to _views,_ like in the case with the _global CTR_. When intra-user correlation is one, all users would have the same weight of one, like in the case with the _average user CTR._ In the real world, this correlation is somewhere between zero and one and should be estimated for each metric separately.

Let’s apply those weights to _user CTR_s and use them in bootstrap (note that it is possible to use these weights in bucketization and linearization as well).

![](https://miro.medium.com/v2/resize:fit:700/1*F3Q0lONJKI4__6q9GL4zDg.gif)

![](https://miro.medium.com/v2/resize:fit:700/1*5QFben5W6p_kxKj3f9dDrg.gif)

As you can see, weighing gives us a very good boost in statistical test power. But it doesn’t come free. Compared to standard bootstrap and bucketization, here we have an additional assumption: user CTR (and uplift in user CTR) does not depend on _views_.

# 5. So what? Which test should I use?

That’s a tricky question to answer.

Of course, you can tweak our code, generate data that is pretty much like yours, and check the performance of all of the tests we discussed. Doing so, don’t forget about the additional assumptions, as they are crucially important.

For those who don’t feel like tweaking our code, we provide some general information below, which should be enough to choose a test for your case.

Regardless of what test you choose, always check its false-positive rate under H0. The simplest way to do so is to run a single A/A test and collect metrics from it. Then you can generate an infinite number of A/A tests from it by randomly resplitting users into two groups. Aim for a diagonal p-value CDF.

When choosing a test, you should first check the additional assumptions:

1. Your experiment does not influence _views._
2. User CTRs in your data do not depend on _views,_ and uplift in user CTRs (the treatment effect of your experiment) does not depend on _views._

If neither first nor second holds in your case, you should consider only _clicks_. You should choose between the t-test on _clicks_ and the Mann-Whitney test on _clicks_. In our simulation, the latter was, in general, more powerful and more stable, but if your data is not skewed (or the skewness is low), a t-test may be a better option. Also keep in mind that the Mann-Whitney test is much more computationally intensive, which can be crucial when working with big data.

If the first holds and the second does not, you can use _global CTR methods_. In our simulations, bootstrap seems to be the best choice. If it is computationally infeasible in your case, you should consider alternatives such as the delta method, bucketization, or linearization.

If both the first and second hold, you can use weighting hacks. In our simulation, a weighted bootstrap performs the best. A t-test on a smoothed CTR seems to work very well, but keep in mind that it has no theoretical guarantees.

When using a fancy statistical test, remember that it is very important to compare it with baselines. As you might notice, in some cases, the classical Mann-Whitney test outperforms fancy weighted bootstraps. But in other cases, those fancy methods provided to be a substantial boost in power.

All in all, the best option for you depends mostly on the data distribution, and it is worth trying to reproduce the data distribution in simulation and choose the option that performs the best.

# 6. Methods not in the list

## 6.1. Transformation of _clicks_

_Clicks_ distribution is skewed. One can try to fix it by replacing _clicks_ with log(1 + _clicks_) or by capping _clicks_ at some number (e.g. 100 or 1000). Usually, it increases the power of a t-test somehow, but it introduces additional assumptions. It’s easy to construct an example where avg(x) increases while avg(log(x)) decreases and vice versa. When thinking about how to accurately analyze A/B test results, it is worth considering cappings, logarithms, and the assumptions they require. In some cases, it provides additional benefits in the power of tests, but it really depends too much on the specific task. This is the reason why we have not covered it.

## 6.2. CUPED

[https://www.researchgate.net/publication/237838291_Improving_the_Sensitivity_of_Online_Controlled_Experiments_by_Utilizing_Pre-Experiment_Data](https://www.researchgate.net/publication/237838291_Improving_the_Sensitivity_of_Online_Controlled_Experiments_by_Utilizing_Pre-Experiment_Data)

Using pre-experimental data to reduce the variance of the metric is a brilliant idea. Roughly speaking, the authors substitute _clicks_ with (_clicks_ — alpha * _clicks_before_experiment_), where alpha is a regression coefficient of _clicks_before_experiment_ on _clicks_. Rather than comparing clicks, the authors compare how _clicks_ for a particular user differ from their pre-experiment clicks. This trick reduces variance a lot and can be applied to any of the statistical tests we’ve discussed.

## 6.3. Using future metric prediction

[https://research.yandex.com/publications/108](https://research.yandex.com/publications/108)

Further development of the CUPED idea. Rather than using a linear model to predict _clicks_ from _clicks_before_experiment,_ the authors use an arbitrarily ML model to predict _clicks_ using all pre-experimental data. It can be applied to any of the statistical tests we’ve discussed.