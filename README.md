# Adversarial Attacks and Defenses of Image Classifiers, NIPS 2017 competition track

*From [NIPS 2017 competition track website](https://nips.cc/Conferences/2017/CompetitionTrack)*
> Most existing machine learning classifiers are highly vulnerable to adversarial examples. 
> An adversarial example is a sample of input data which has been modified very slightly in 
> a way that is intended to cause a machine learning classifier to misclassify it. In many 
> cases, these modifications can be so subtle that a human observer does not even notice the 
> modification at all, yet the classifier still makes a mistake. Adversarial examples pose 
> security concerns because they could be used to perform an attack on machine learning systems, 
> even if the adversary has no access to the underlying model.
> 
> To accelerate research on adversarial examples and robustness of machine learning classifiers 
> we organize a challenge that encourages researchers to develop new methods to generate 
> adversarial examples as well as to develop new ways to defend against them. As a part of the 
> challenge participants are invited to develop methods to craft adversarial examples as well 
> as models which are robust to adversarial examples.

There are three sub-competitions

* [Non-targeted Attack](https://www.kaggle.com/c/nips-2017-non-targeted-adversarial-attack)
* [Targeted Attack](https://www.kaggle.com/c/nips-2017-targeted-adversarial-attack)
* [Defense](https://www.kaggle.com/c/nips-2017-defense-against-adversarial-attack)

Dev dataset and toolkit available 
[here](https://github.com/tensorflow/cleverhans/tree/master/examples/nips17_adversarial_competition).

Code tested on Python 3, TensorFlow 1.3, and Keras 2.0.

## Approach for Non-Targeted Attack

My approach is based on the FGSM with Random Perturbation method described in 
[this paper](https://arxiv.org/abs/1705.07204) by Florian Tramèr, Alexey Kurakin, Nicolas Papernot, 
Dan Boneh, and Patrick McDaniel. On a high level, it is an iterative FGSM on an ensemble of models,
with random perturbation added before computing gradients. In addition to random perturbation,
I also added some small geometric transformation to increase transferrability.

One observation is that although it is important to attack based on an ensemble of models, it is
not always productive to use all the models in all the steps. Therefore, each iterative step is
broken down into two parts:

1. Compute the gradients on a set of models and add them to a sum.
2. Take the sign of the sum of the above gradients and apply the changes to the input images.

## Approach for Targeted Attack

Similar to that for non-targeted attack. The main difference here is that targeted attack seems
to be much more delicate than non-targeted attack. I was only able to successfully hit target
if I switch off random perturbation and geometric transformation. The step sizes also have to
be much smaller than that in non-targeted attack, and finding an attack schedule that works for
an ensemble of models is also more challenging.

## Approach for Defense

The basic idea is to manipulate the pixel values of the input *adversarial* images to reduce
the total variation distance among the predictions of an ensemble of models. The idea mostly
comes from the observation that targeted attacks are hard, and the models typically do not agree
on a label on the adversarial images generated by my own attacks. The approach itself does not
seem to be particularly strong against my own attacks.

## Submission source code

My submission to the three sub-competitions are in the `submissions` directory. 

Files containing pretrained weights need to be placed in the working directory. All `.h5` weights
are from [Keras](https://keras.io/). All `.ckpt` weights are from 
[TensorFlow-Slim](https://github.com/tensorflow/models/tree/master/research/slim).
Some of the models share the same architecture. The easiest way to load those models at the same
time is to change the variable names. This is done using the `rename_vars.py` script,
which is a lightly modified version of the one found 
[here](https://gist.github.com/batzner/7c24802dd9c5e15870b4b56e22135c96).

In the actual submission, the final graphs are frozen, serialized and saved to a `.pb` file for 
fast loading. This is done with the `prepare_models.py` script.

Note that the submission code as is has some known bugs as commented in the code.

