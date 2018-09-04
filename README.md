# Unrestricted Adversarial Examples Challenge

In the Unrestricted Adversarial Examples Challenge, attackers submit arbitrary adversarial inputs, and defenders are expected to assign low confidence to difficult inputs while retaining high confidence and accuracy on a clean, unambiguous test set.

You can learn more about the motivation and structure of the contest in our recent paper:

**Unrestricted Adversarial Examples**<br>
*Authors*<br>
[http://arxiv.org/](http://arxiv.org/)

This repository contains code for the warm-up to the challenge, as well as [the public proposal for the contest](contest_proposal.md).

![image](https://user-images.githubusercontent.com/306655/44686400-f0b74800-aa02-11e8-8967-fa354244813f.png)


## Warm-up phase
### <a name="leaderboard"></a>Leaderboard
We include three attacks in the warm-up phase of the challenge:

- 1000 Linfinity-ball adversarial examples generated by SPSA
- 1000 spatial adversarial examples (via grid search)
- 100 L2-ball adversarial examples generated by a decision-only attack

The top five distinct models for each dataset are shown below. You can see all submissions in [the full scoreboard](scoreboard.md). 

##### Two-Class MNIST
| Defense               | Submitted by  | Spatial acc.<br>at 80% cov. | SPSA acc.<br>at 80% cov. | L2-ball acc.<br>at 80% cov. |  Submission Date |
| --------------------- | ------------- | ------------ |--------------- |--------------- | --------------- |
| [MadryPGD LeNet Baseline](#)  |  ?? |    **??**    |     **??**   |     **??**     |  Aug 28th, 2018 |
| [Undefended LeNet Baseline](#)   |  Google Brain   |    0.0%    |     0.0%    |     0.0%     |  Aug 27th, 2018 |

##### Bird or Bicycle
| Defense               | Submitted by  | Spatial acc.<br>at 80% cov. | SPSA acc.<br>at 80% cov. | L2-ball acc.<br>at 80% cov. |  Submission Date |
| --------------------- | ------------- | ------------ |--------------- |--------------- | --------------- |
| [Worst-of-10-Spatial Baseline](#)  |  ?? |    **??**    |     **??**   |     **??**     |  Aug 28th, 2018 |
| [Undefended ResNet Baseline](unrestricted_advex/pytorch_resnet_baseline)   |  Google Brain   |    0.0%    |     0.0%    |     0.0%     |  Aug 27th, 2018 |




### Implementing a defense

First install the requirements (assuming you already have working installation
of Tensorflow or pytorch)
```bash
git clone git@github.com:google/unrestricted-adversarial-examples.git
cd unrestricted-adversarial-examples

pip install -e bird-or-bicycle
pip install -e unrestricted-advex
```

Confirm that your setup runs correctly by training and evaluating an MNIST model.
```bash
cd unrestricted-advex/unrestricted_advex/mnist_baselines
CUDA_VISIBLE_DEVICES=0 python train_two_class_mnist.py --total_batches 10000
# Outputs look like (specific numbers may vary)
# 0 Clean accuracy 0.046875 loss 2.3123064
# 100 Clean accuracy 0.9140625 loss 0.24851117
# 200 Clean accuracy 0.953125 loss 0.1622512
# ...
# 9800 Clean accuracy 1.0 loss 0.004472881
# 9900 Clean accuracy 1.0 loss 0.00033166306

CUDA_VISIBLE_DEVICES=0 python evaluate_two_class_mnist.py
# Outputs look like (specific numbers may vary)
# Executing attack: null_attack
# Fraction correct under null_attack: 1.000
# Executing attack: spsa_attack
# Fraction correct under spsa_attack: 0.016
# Executing attack: spatial
# Fraction correct under spatial: 0.117
```

#### To be evaluated against our fixed warm-up attacks, your defense must implement the following API

It must be a function that takes in batched images (as a numpy array), and returns two scalar (e.g. logits) between `(-inf, inf)`. These correspond to the likelihood the image corresponds to each of the two classes (e.g. the bird and bicycle class)

```python
import numpy as np

def my_very_robust_model(images_batch_nchw):
  """ This function implements a valid unrestricted advex defense that always returns higher
  logits for the second class """
  batch_size = len(images_batch_nchw)
  logits_np = np.array([[-5.0, 5.0]] * batch_size)
  return logits_np.astype(np.float32)

from unrestricted_advex import eval_kit
eval_kit.evaluate_bird_or_bicycle_model(my_very_robust_model)
```

For ease of evaluation, your model must also maintain a throughput of at least **100 images per second** when evaluated on a P100 GPU on the `bird-or-bicyle` dataset

##### Your defense will be evaluated with the following mechanism

- The test dataset is passed through the model and converted to logits.
- `confidence` is defined as `max(bird_logit, bicycle_logit)` for each image.
- The 20% of images that resulted in logits with the lowest `confidence` are abstained on by the model and are discarded.
- The model’s score is the **accuracy on points that were not abstained on**.

##### Submitting your defense to the leaderboard

After evaluating your defense you can submit it to [the leaderboard](#user-content-leaderboard) by [editing the table](https://github.com/google/unrestricted-adversarial-examples/edit/master/README.md) and creating a pull request that links to your defense. The challenge organizers will respond to the pull request within five business days.

##### A note on uninteresting defenses that break the default attacks
We expect there to be several uninteresting defenses that are “robust” against the fixed set of attacks we have developed. By “uninteresting” we mean defenses that were designed explicitly to stop the attacks we have developed, but not necessarily other attacks. 

For example, it would be possible to break the confidence-based SPSA attack through gradient masking and not returning the true models’ confidence, but either 100% confidence or 0% for each input.

We encourage defense creators to not design defenses that are intentionally uninteresting.
 


## Contest phase

The contest phase will begin after the warm-up attacks have been conclusively solved. We have published the [contest proposal](https://github.com/google/unrestricted-adversarial-examples/blob/master/contest_proposal.md) and are soliciting feedback from the community.

## Running tests

```bash
pytest
```

## Authors
