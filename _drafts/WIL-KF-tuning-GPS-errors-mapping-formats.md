---
layout: post
title: Tuning Extended Kalman Filter Process Noise
mathjax: true
---

*TL;DR: Using [Discriminative Training of Kalman Filters (2005)](http://www.roboticsproceedings.org/rss01/p38.pdf) to your tune filter's process noise.*

I recently was asked about how you tune the noise covariance matrices for (Extended) Kalman Filters.  It was a pointed question, where I felt obliged to have an answer, and while I had a good answer for the measurement noise $R$, I was at a loss for a general way of tuning the process noise matrix $Q$.  All I could remember was hand tuning $Q$ based on some educated guesses.  I was embarrassed, so I set out to find an answer.

![Right: Rudolf Kalman, inventor of the Kalman Filter, receives the National Medal of Science from President Obama. Left: Margaret Hamilton, one of the leading minds who developed the Apollo on-board flight software, sits in the Apollo module.  The navigation system interface is above her head, below the spherical attitude indicator, running Kalman's filter.](/images/rudolf_kalman_margaret_hamilton2.png)

*Right: [Rudolf Kalman](https://en.wikipedia.org/wiki/Rudolf_E._K%C3%A1lm%C3%A1n), inventor of the Kalman Filter, receives the National Medal of Science from President Obama. Left: [Margaret Hamilton](https://en.wikipedia.org/wiki/Margaret_Hamilton_(scientist)), one of the leading minds who developed the Apollo on-board flight software, sits in the Apollo module.  The navigation system interface is above her head, below the spherical attitude indicator, running Kalman's filter.*

<!--excerpt-->

In short, tuning $Q$ is hard, most do it by hand, there's a lot of literature on automated tuning, and often these methods are better than hand tuning.

# The problem with process noise

It's good that I was embarrassed, because it was motivating, but it's funny to find out that I had no reason to be.  After some googling I kept running into answers like this [^Satish]:

> Dear Satish, the answer cannot be given immediately, since noise analysis is typically a matter of special investigations, often expensive. To specify Q, you need to know the motor model and connect it to possible physical disturbances. Then approximate the noise sources with white Gaussian distribution. Matrix R is much easy to ascertain, because the measurement equipment often has some error characteristics. 

Sorry, Satish!  Here's another[^not-straightforward]:

> R can be found by processing the measurements while the output of the system is held constant. In this case, only noise remains in the data after its mean is removed.
> 
> The covariance can be calculated easily from the remaining portion of the data.
> 
> Q is not that straightforward. It's generally constructed intuitively but there are some points that need to be regarded choosing it. Unmodeled dynamics and parameter uncertainities are modeled as process noise generally. Checking some papers and book of Dan Simon, you can determine a proper Q for your application.

If you dig around, the answer to tuning $R$ is consistent, either:

1. Consult the sensor specifications and create a diagonal matrix with the sensor variance along the diagonal, or
2. Compare your sensor measurements against a strong ground truth and derive $R$, or
3. A special case of #2 is: hold the system in a constant state and measure the sensor noise.

These are straightforward and often easy solutions.  But what can we do for $Q$?  Is hand tuning $Q$ the best we can do?

*A side note: if you are trying to measure GPS noise, doing #3 could amount to setting your phone outside on a bench.  However, there's a problem here I plan to touch on in a later post: GPS noise consists of multiple random components, some of which are correlated in time, so that you have an error term that is partly [stationary noise](https://en.wikipedia.org/wiki/Stationary_process) (roughly, "time-independent"), and partly slow moving bias.  You also see this problem in other sensors, such as bias, moving or stable, in gyroscopes.  I may address this in another post, as **Discriminant Training** provides a nice solution: you can appropriately model the slow moving bias with your motion and measurement models.*

# The literature

The Kalman filter, and its variants, are old.  Kalman filters were used in the Apollo navigation computer[^ApolloWow]:

> The Apollo computer used 2k of magnetic core RAM and 36k wire rope [...]. The CPU was built from ICs [...]. Clock speed was under 100 kHz [...].

I'm looking forward to the new MacBook Pro, I hear it has 24 terabyte wire rope!

Since Kalman filters have been around for a while, others have worked on this problem.  A lot.  For example, [Raman K. Mehra](https://www.researchgate.net/profile/Raman_Mehra) has two [\(1\)](https://www.researchgate.net/publication/3025930_On_the_identification_of_variances_and_adaptive_Kalman_filtering) papers [\(2\)](https://www.researchgate.net/publication/3026625_Approaches_to_Adaptive_Filtering) on the topic from 1970 and 1972 with a combined 1398 citations between them.

Two modern papers popped up consistently on this topic:

1. [Discriminative Training of Kalman Filters, 2005, by Pieter Abbeel, Adam Coates, Michael Montemarlo, Andrew Ng, and Sebastian Thrun.](http://www.roboticsproceedings.org/rss01/p38.pdf)[^discriminative-training]
2. [How To NOT Make the Extended Kalman Filter Fail, 2013, by René Schneider and Christos Georgakis.](https://www.researchgate.net/publication/263942618_How_To_NOT_Make_the_Extended_Kalman_Filter_Fail)[^how-not-to-fail]

*How To NOT Fail* is worth looking at for insights.  However, let's focus on the methods in *Discriminative Training*.

# Discriminative Training of Kalman Filters

Abbeel et al provide a really nice analysis of why it's difficult to fit the process noise.  Firstly, because the noise is not coming from a single source of error, but rather several sources:

>1. Mis-modeled system and measurement dynamics.
>2. The existence of hidden state in the environment not
modeled by the EKF.
>3. The discretization of time, which introduces additional
error.
>4. The algorithmic approximations of the EKF itself, such as
the Taylor approximation commonly used for linearization.

Secondly, "the noise is assumed to be independent
over time".  And thirdly, though I don't know that they explicitly state this, it is very hard to observe process noise, and often expensive if feasible.

What they propose to do is fairly simple:

1. Consider a range of objective functions for an EKF.
2. Establish a train-test data split, paired with ground truth robot path given by $y_{0:T}$ .
3. Train process noise matrices from a common initial estimate by picking a loss function, and using a coordinate ascent algorithm, iterating until the algorithm converges.
4. Test these process noise matrices against the test set using two evaluation metrics: RMSE and log-loss vs a ground truth dataset.

Standard ML, right?  Let's survey the steps:

## Loss functions:

They propose six objective functions, two of which we can disregard because they are slightly modified versions of the others, and less effective.

### Maximizing the joint (JOINT) likelihood:

This approach solves for $\langle R_\text{joint}, Q_\text{joint} \rangle$ that maximize the likelihood:

$$\text{log} \ p(x_{0:T}, y_{0:T}, z_{0:T} \mid u_{1:T})$$

where $x_{0:T}$ denotes the full robot state over time (including hidden states that $y_{0:T}$ cannot observe), $y_{0:T}$ are the ground truth observations of the path of the robot, and $z_{0:T}$ are the sensor observations we expect to have in production.

It turns out that the above probability decomposes conveniently, and we can compute $\langle R_{\text{joint}}, Q_{\text{joint}} \rangle$ without even needing to run the filter:

$$\begin{align*}
R_\text{joint} &= \frac{1}{T} \sum_1^T \left\Vert x_t - f(x_{t-1}, u_t) \right\Vert^2 \\
Q_\text{joint} &= \frac{1}{T+1} \sum_0^T \left\Vert z_t - g(x_t) \right\Vert^2
\end{align*}$$

Here $f$ and $g$ denote the motion model and the observation model respectively.  It's more conventional to use $g$ and $h$ but we'll stick to the paper's notation.

### Minimizing the RMS of the prediction (RES)

This is just good ol' compare your prediction against your ground truth and compute RMSE.  We're seeking:

$$\require{AMSmath} \DeclareMathOperator*{\argmin}{argmin}
\langle R_\text{res}, Q_\text{res} \rangle = \argmin_{R, Q} \sum_0^T \left\Vert y_t - h(\mu_t) \right\Vert^2$$

Where $h(\mu_t) = (\mu_{x,t}, \mu_{y,t})$, the projection of the state vector into $(x,y)$ coordinates.

Note that to compute this, we need to generate the sequence of state estimates $\mu_{0:T}$, so this requires that we run the filter.  Since that does take time (although if they can run it on Apollo's navigation computer, it shouldn't take much), this is where it becomes important that we have a good training algorithm in step 3.

### Maximizing the ground truth likelihood (PRED)

Here we are interested in the maximizing the likelihood of our ground truth measurements, in essence demanding that our model believe the ground truth to be pretty likely:

$$\require{AMSmath} \DeclareMathOperator*{\argmax}{argmax}
\langle R_\text{pred}, Q_\text{pred}\rangle = \argmax_{R,Q} \sum_0^T \text{log} \ p(y_t \mid z_{0:t}, u_{1:t})$$

Set $\Omega = H_t \Sigma_t H_t^\intercal + P$, where $H_t$ is the Jacobian of $h(\cdot)$ (from above), and $P$ is the covariance matrix for the ground truth data $\{y_t\}$.  If $P$ is neglible we can drop it, and we get, with some work:

$$\require{AMSmath} \DeclareMathOperator*{\argmax}{argmax}
\langle R_\text{pred}, Q_\text{pred}\rangle = \argmax_{R,Q} \sum_0^T \left[ - \text{log} |2\pi\Omega_t| - (y_t - h(\mu_t))^\intercal \Omega_t^{-1} (y_t - h(\mu_t)) \right]$$

There's an interesting observation to be made here.  If the filter is less confident, and so $\Sigma_t$ is larger, then the first term will contribute more negatively, but the second term will be smaller ("$\Omega_t$ is the denominator" in essence).  The confidence and the quality of prediction are being placed in opposition, so that the filter does better the more appropriate it's confidence level is at each time step.

### Maximizing the measurements likelihood (MEAS)

From the paper, "We now apply the idea in the previous step to the measurement
data $z_{0:T}$."  Imagining you have no ground truth data to train against, you could try to optimize for the best likelihood for the measurements from your production sensors.

$$\begin{align*}
\require{AMSmath} \DeclareMathOperator*{\argmin}{argmin}
\langle R_\text{meas}, Q_\text{meas}\rangle &= \argmin_{R,Q} \ \text{log} \ p(z_{0:T} \mid u_{1:T}) \\
&= \argmin_{R,Q} \ \text{log} \ \prod_0^T p(z_t \mid z_{0:t-1}, u_{1:T}) \\
&= \argmin_{R,Q} \ \sum_0^T \text{log} \ p(z_t \mid z_{0:t-1}, u_{1:T})
\end{align*}$$

Those familiar with the derivation of the recursive Bayes filter equation will notice that $p(z_t \mid z_{0:t-1}, u_{1:T})$ is essentially the normalization term arising from Bayes rule, and is something we normally discard.  That term can be expanded to:

$$\begin{align*}
p(z_t \mid z_{0:t-1}, u_{1:T}) &= \int_{x_t} p(z_t \mid x_t) \ p(x_t \mid z_{0:t-1}, u_{1:t}) dx_t \\
&= \int_{x_t} \mathcal{N}_{g(x_t), Q}(z_t) \ \mathcal{N}_{\bar{\mu}_t, \bar{\Sigma}_t}(x_t) \ dx_t
\end{align*}$$

This requires running the filter to evaluate.

## Train-Test split

This is straightforward, nothing to see here.  Just make sure that you filter has time to intialize and stabilize, and you might want to chop out the initial bootstrapping when the filter is starting with largish covariance and reducing.

## Training algorithm for the noise matrices

Sebastian Thrun presents an intuitive approach to optimization in one of the Udacity Self Driving Car degree lessons: an algorithm called twiddle.  In that context you are optimizing the coefficients of a [PID Controller](https://en.wikipedia.org/wiki/PID_controller), to keep your a simulator car driving smoothly and in the center of the track.  The idea of twiddle is:

1. Select a parameter to adjust.
2. "Twiddle" the parameter, as if you were adjusting knobs up and down.  First trying larger values, then smaller values, gradually adjusting the parameter according to how the performance improves.
3. Move to the next parameter, and repeat, returning to previously adjusted parameters in rotation.
4. Stop when you can no longer significantly improve performance by adjusting any of the parameters.

As Thrun was a coauthor, it's not surprising to find this practical and intuitive approach used to optimize the noise matrices:

> The training algorithm used in all our experiments is a
coordinate ascent algorithm: Given initial estimates of $R$ and
$Q$, the algorithm repeatedly cycles through each of the entries
of $R$ and $Q$. For each entry $p$, the objective is evaluated
when decreasing and increasing the entry by $\alpha_p$ percent. If
the change results in a better objective, the change is accepted
and the parameter $\alpha_p$ is increased by ten percent, otherwise $\alpha_p$
is decreased by fifty percent. Initially we have $\alpha_p = 10$. We
find empirically that this algorithm converges reliably within
20-50 iterations.

Because this algorithm focuses on one parameter at a time, and focuses on exploitation over exploration, it would normally be vulnerable to find all kinds of sub-optimal local minima.  However, since we are creating a diagonal noise matrix, and we can be fairly sure that there are not to many of these sub-optimal local minima, we are probably safe.

## Evaluate the results on RMSE and log loss vs ground truth

The authors ran these procedures and compared them to a state-of-the-art EKF tuned by Carnegie Mellon researchers on the RMSE and log-loss metrics.  For convenience, here is the RMSE definition used

$$ \left( \frac{1}{T} \sum_1^T \Vert y_t - h(\mu_t) \Vert^2 \right)^{1/2} $$

And the log loss definition:

$$ - \frac{1}{T} \sum_1^T \text{log} \ p(y_t \mid z_{0:t}, u_{1:T}) $$

Here are the results:

|---
| **Objective** | **RMSE** (meters) | **log-loss**
| - | - | -
| JOINT | 0.287 | 23.6
| RES | 0.270 | 1.06
| PRED | 0.294 | 0.167
| MEAS | 0.294 | 60.2
| Carnegie Mellon | 0.39 | 0.75

As you can see, they all performed well on RMSE, all beating Carnegie Mellon's tuned filter by 25% or better.  Here's a zoomed in picture of the ground truth and the various filters:

![EKF performance by algo](/images/discriminant_training_opt_performance.png){:class="img-responsive"}

Here we see several paths from a robot moving in a rough loop from the paper's test set.  Ground truth $y_{0:t}$ is shown in black, $z_{0:t}$ shows as black triangles, RES is shown in blue dots and dashes, PRED in green dashes, and MEAS in red dots.

The obvious takeaway here is that the PRED (green dashes) method of optimizing your $Q$ and $R$ matrices for high confidence in the ground truth data is optimal.  It only lags the others minimally on RMSE, but outperforms dramatically on log loss.  Because the loss function formulation balances over confidence and under confidence, the filter tightly fits the ground truth data with an appropriate level of confidence.

JOINT and MEAS failed badly on log loss.  Both set ups seek high confidence on the current measurements, which will make them vulnerable to large log loss when they are wrong.  The authors attribute this in part on correlated noise.  By tracking the measurements too closely, when the noise correlation drops, the predicted state is out of position.  Perhaps this is why it's visually obvious that MEAS is prone to intermittent large corrections on new GPS readings.

# Conclusions

The major conclusions for me are:

1. You don't have to hand tune your noise matrices.  This is a relatively simple procedure to set up, and well worth the time if you are working on a production system.
2. If you had ground truth data, PRED is the best procedure, hands down.  Implementing that should get you pretty close to optimal performance for your motion model, measurement model, sensor suite and robot.
3. I recently was considering what to do in the case that you do not have a ground truth dataset.  I had considered implementing some kind of clumsy version of MEAS.  However, considering the over-confidence issues that arose, I would hesitate before doing so.  If I were to try it, I would do the following:
    - The motion model implemented in *Discriminant Training* included learning a bias term for the gyro.  If I was going to optimize without ground truth, I would want to model a slow moving bias term in the GPS measurements.
    - I would think about how I could add a regularization parameter to the MEAS loss function to discourage unreasonably low covariance.
    - I would monitor the model for large jumps in position and overall consistency of the filter.
    - I would be inclined to use a more complex 3D motion model including roll and pitch measurements to try to capture process noise that was from unmodeled physics.

I may, as time allows, try implementing this procedure.

What do you think?  Have you run into problems like these before?

[^Satish]: [ResearchGate: How to find Q and R in Kalman Filter?](https://www.researchgate.net/post/How_to_find_Q_and_R_in_Kalman_Filter2){:target="blank_"}
[^not-straightforward]: [ResearchGate: How do we determine noise covariance matrices Q & R?](https://www.researchgate.net/post/How_do_we_determine_noise_covariance_matrices_Q_R)
[^ApolloWow]: [TRS-80.org: An interview with Jack Crenshaw](http://www.trs-80.org/interview-jack-crenshaw/)
[^discriminative-training]: [Discriminative Training of Kalman Filters, 2005, by Pieter Abbeel, Adam Coates, Michael Montemarlo, Andrew Ng, and Sebastian Thrun.](http://www.roboticsproceedings.org/rss01/p38.pdf)
[^how-not-to-fail]: [How To NOT Make the Extended Kalman Filter Fail, 2013, by René Schneider and Christos Georgakis.](https://www.researchgate.net/publication/263942618_How_To_NOT_Make_the_Extended_Kalman_Filter_Fail)
