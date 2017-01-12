# PORTFOLIO
##### A place to show off personal projects, both ML and not
___

## Machine Learning
### [OmegaGo](https://github.com/samlobel/OmegaGo)
Definitely the biggest ML-undertaking I've done alone, although not the most unique. Written in TensorFlow, my program trains a convolutional value network completely through self-play on a 5x5 board. I also made a Tkinter GUI so you can play any of the trained models. Shows steady improvement with training, and can beat me every time.
#### Design overview
* The beginning of the game and the end of the game clearly have different input distributions, which neural networks have trouble coping with. It puts a lot of strain on a network to intelligently assign a value to an empty board as well as to a nearly full one. To narrow this distribution, I train three separate value, each which is used a different stage of the game (beginning, middle, and end), and is only trained on boards from that stage.
* Unlike AlphaGo, my model only has a value function. Since I was not training my model with labelled data, a policy function would be slow to converge. Moreover, since my model does not perform a monte carlo search, a policy function was not necessary for efficient move-picking.
* To simulate a policy function, I look ahead for each valid move, calculate the values for these moves, create a 5x5 input corresponding to these values, and pass it through a softmax of varying temperatures. The temperature starts high (closer to random move-choice), and slowly anneals to lower temperatures as training progresses (smarter move-choice).
* I generate boards to train in the usual fashion, by playing a game and randomly saving a board from that game, as well as the game's result.
* The endgame is easiest to train, because the its inputs are closer in time to the training signal. Thus, for each training round, I train the models in reverse order (end to beginning). This makes the earlier models more closely connected to the true values, because once the game transitions to a later, trained model, it makes smarter decisions.
* The training pipeline consists of alternatively generating boards and training the models, in descending order, all while slowly lowering the softmax temperature.

___

### [Edgewise Scaling](https://github.com/samlobel/EDGEWISE_SCALING)
This came about while training generative models, when I noticed that, at first, the images they outputted were dimmer around the edges than they were near the center. When using zero-padding, a filter near the edge has much less signal coming into it when compared to a filter near the center, leading to less variance in the signal. The achievement in this project was developing a scaling operation that corrected this variance, making truer initializations, and leaving the network with one less thing to learn.
#### Design overview
* The trickiest part of this was correctly calculating the amount of scaling needed given the various options to zero-padded convolution. With a stride of one, and a filter size of 5, the filter covers 25 input pixels in the center, but only 9 at the corner, and 15 somewhere along the edge. But this becomes much more complicated with other strides, because the top-left corner and the bottom-right corner may be different.
* To get around this, I calculate a scaling-map by performing the convolution on an all-ones input through an all-ones filter. Since every multiplicitive interaction equals one, this outputs the number of pairwise multiplications that are summed together for each output point. You can input different image-sizes, different filter-sizes, and different strides. Take the square root of this, and you have the proper variance-scaling operation.
* You use this scaling-map to scale the output of a true convolutional operation. This leaves your ouput with uniform variance
* Once you get this scaling-map, actually take the convolution of input with filters, scale the result by the scaling-map, and add your bias on (if desired). This is the output of a conv_with_scaling layer! To be used wherever you would otherwise use convolutional layers, but especially inside of generative models.

___

### [Direct second-derivative estimation for Optimization](https://github.com/samlobel/DIRECT_CURVATURE_ESTIMATION)
This experiments with a more direct computation of the second-derivative than is used in algorithms like ADAM. Instead of maintaining a moving average, this method uses more of a taylor-expansion approach, and chooses a learning rate based on a direct compuation of the second-derivative.
#### Design overview
* The algorithm begins like most optimization algorithms, by taking the gradient of the parameters with respect to some minibatch error.
* The algorithm then clones the model, and updates the models parameters using the gradient, and a VERRRRY small step size. You then re-calculate the error using this slightly updated model.
* These three values (the error at two points, and the gradient along their connecting line at one of the points) is enough information to estimate the second derivative using a taylor-expansion.
* This value can be used in a number of ways. Intuitively, you want to make bigger updates to the model in places with a negative second derivative, and smaller ones in places with a positive second derivative. No matter how you do it, you use this value to decide on a true step size by which to update the parameters of your model.
* I wanted an upper bound, so I passed the second derivative through a sigmoid, and choose a step size by multiplying some maximum step size (0.1 for example) by this value.

___

