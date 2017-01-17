# PORTFOLIO
#### A place to show off personal projects, both ML and not
The titles are links to the repositories themselves, and are followed by a summary of each project's design. For those interested, the projects' READMEs have more detailed overviews.
___

## Machine Learning
### [OmegaGo](https://github.com/samlobel/OmegaGo)
Definitely the biggest ML-undertaking I've done alone, although not the most unique. Written in TensorFlow, my program trains a convolutional value network completely through self-play on a 5x5 board. I also made a Tkinter GUI so you can play any of the trained models. Shows steady improvement with training, and can beat me every time.
#### Design overview
* The beginning of the game and the end of the game clearly have different input distributions, which neural networks have trouble coping with. It puts a lot of strain on a network to intelligently assign a value to an empty board as well as to a nearly full one. To narrow this distribution, I train three separate value networks, each of which is used for and trained on only boards from oen stage of the game (beginning, middle, and end).
* Unlike AlphaGo, my model only has a value function. Since I was not training my model with labelled data, a policy function would be __VERY__ slow to converge. Moreover, since my model does not perform a monte carlo search, a policy function was not necessary for efficient move-picking.
* To simulate a policy function where needed, I look ahead for each valid move, calculate the values for these moves, create a 5x5 input corresponding to these values, and pass it through a softmax of varying temperatures. The temperature starts high (closer to random move-choice), and slowly anneals to lower temperatures as training progresses (smarter move-choice).
* I generate boards to train in the usual fashion, by playing a game and randomly saving a board from that game, as well as the game's result.
* The endgame is easiest to train, because the its inputs are closer in time to the training signal. Thus, for each training round, I train the models in reverse order (end to beginning). This makes the earlier models more closely connected to the true values, because once the game transitions to a later, trained model, it makes smarter decisions.
* The training pipeline consists of alternatively generating boards and training the models, in descending order, all while slowly lowering the softmax temperature.

___

### [Edgewise Scaling](https://github.com/samlobel/EDGEWISE_SCALING)
This idea came about while training generative models, when I noticed that, at first, the images they outputted were dimmer around the edges than they were near the center. When using zero-padding, a filter near the edge has much less signal coming into it when compared to a filter near the center, leading to less variance in the signal. The achievement in this project was developing a scaling operation that corrected this variance, making truer initializations, and leaving the network with one less thing to learn.
#### Design overview
* The trickiest part of this was correctly calculating the amount of scaling needed given the various options to zero-padded convolution. With a stride of one, and a filter size of 5, the filter covers 25 input pixels in the center, but only 9 at the corner, and 15 somewhere along the edge. But this becomes much more complicated with other strides, because the top-left corner and the bottom-right corner may be different. Even filter-sizes are similarly difficult, because they don't have a true center.
* To get around this, I calculate a scaling-map by performing the convolution on an all-ones input through an all-ones filter. Since every multiplicitive interaction equals one, this outputs the number of pairwise multiplications that are summed together for each output point. You can input different image-sizes, different filter-sizes, and different strides. Take the square root of this, and you have the proper variance-scaling operation.
* You use this scaling-map to scale the output of a true convolutional operation. This leaves your ouput with uniform variance
* Once you get this scaling-map, actually take the convolution of input with filters, scale the result by the scaling-map, and add your bias on (if desired). This is the output of a conv_with_scaling layer! To be used wherever you would otherwise use convolutional layers, but especially inside of generative models.

___

### [Remove-zero Optimizer](https://github.com/samlobel/REMOVE_ZEROS_OPTIMIZER)
This is an optimizer I'm particularly proud of, made to deal with sparse gradients especially well. Sparsity can make important-but-infrequent parameters slow to update. For example, in MNIST, if there is a pixel that is only non-zero for 60 images, is only active for pictures of 8s, its training signal will be incredibly weak, even though the feature has high classification importance. This algorithm uses minibatch statistics to provide stronger updates to infrequent parameters.

#### Design overview
* Unlike most gradient descent algorithms, where you take the gradient with respect to the average minibatch error, in this algorithm we start with the gradients with respect to __each__ sample in a minibatch.
* To get the true average of the gradients, for each parameter you would take the sum of the gradients, and divide by the number of samples.
* For this procedure, instead you divide by the number of __non-zero__ samples. Therefore, all parameters will get updated as if every sample had an update for them.
* You then update in this direction, scaled by whatever learning rate you choose. 
* This procedure's strength over something like ADAGRAD is that it treats sparse gradients characteristically differently than it does something with conflicting gradients (equal pull in the positive and negative direction). 


___

### [Direct second-derivative estimation for Optimization](https://github.com/samlobel/DIRECT_CURVATURE_ESTIMATION)
This project experiments with a more direct computation of the second-derivative than is used in algorithms like ADAM. Instead of maintaining a moving average, this method uses more of a taylor-expansion approach, and chooses a step size based on a direct compuation of the second-derivative.
#### Design overview
* The algorithm begins like most optimization algorithms, by taking the gradient of the parameters with respect to some minibatch error.
* You then clone your model, and updates the models parameters using the gradient and a __VERRRRY__ small step size. You then re-calculate the error using this slightly updated model.
* These three values (the error at two points, and the slope along their connecting line at one of the points) is enough information to estimate the second derivative using a taylor-expansion.
* This value can be used in a number of ways. Intuitively, you want to make bigger updates to the model in places with a negative second derivative, and smaller ones in places with a positive second derivative. No matter how you do it, you use this value to decide on a true step size by which to update the parameters of your model.
* I wanted an upper bound to my estimate, so I passed the second derivative through a sigmoid, and choose a step size by multiplying some maximum step size (0.1 for example) by this value.
* Another way I experimented with is using the predicted parabola to see how far you would have to go for it to begin increasing again, and going something like one hundredth of this distance.

___


## Non-Machine Learning
### [IDEAA](https://github.com/samlobel/IDEAA)
A server, written with Flask, which provides a web interface to a series of mass spectrometry scripts. Much easier than delving into the command prompt, especially if computers aren't what you do for a living.

Full disclosure: I wrote this as a helpful tool for my father's laboratoy (he's a biochemist). It's now used in his lab, as well as a few others which do similar work. We're planning to release it on a broader scale soon.

## [Infinify](http://ec2-54-191-228-243.us-west-2.compute.amazonaws.com/)
##### Also, might be [here](http://infinify.us)
They say a picture within a picture is worth a million words, so I'll let this one speak for itself. It's a webapp I built with a friend, hosted on AWS.

## [Autorecorder](https://github.com/samlobel/AutoRecorder)
I wrote this after playing music alone, and one too many times wishing I had pressed record before I started playing. This program opens when I start my computer, and stores a queue of the last 5 minutes of sound going in to the microphone. You can save the 5 minutes at any time. I use it almost every day. Mac toolbar app, written in Swift.




