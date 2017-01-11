# PORTFOLIO
##### A place to show off personal projects, both ML and not
___

## Machine Learning
### [AlphaGo](https://github.com/samlobel/OmegaGo)
Definitely the biggest ML-undertaking I've done alone, although not the most unique. Written in TensorFlow, my program trains a convolutional value network completely through self-play on a 5x5 board. I also made a Tkinter GUI so you can play any of the trained models. Shows steady improvement with training, and can beat me every time.
#### Design overview
* The beginning of the game and the end of the game clearly have different input distributions, which neural networks have trouble coping with. It puts a lot of strain on a network to intelligently assign a value to an empty board as well as to a nearly full one. To narrow this distribution, I train three separate value, each which is used a different stage of the game (beginning, middle, and end), and is only trained on boards from that stage.
* Unlike AlphaGo, my model only has a value function. Since I was not training my model with labelled data, a policy function would be slow to converge. Moreover, since my model does not perform a monte carlo search, a policy function was not necessary for efficient move-picking.
* To simulate a policy function, I look ahead for each valid move, calculate the values for these moves, create a 5x5 input corresponding to these values, and pass it through a softmax of varying temperatures. The temperature starts high (closer to random move-choice), and slowly anneals to lower temperatures as training progresses (smarter move-choice).
* I generate boards to train in the usual fashion, by playing a game and randomly saving a board from that game, as well as the game's result.
* The endgame is easiest to train, because the its inputs are closer in time to the training signal. Thus, for each training round, I train the models in reverse order (end to beginning). This makes the earlier models more closely connected to the true values, because once the game transitions to a later, trained model, it makes smarter decisions.
* The training pipeline consists of alternatively generating boards and training the models, in descending order, all while slowly lowering the softmax temperature. 