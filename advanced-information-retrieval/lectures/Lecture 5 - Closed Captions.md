# Lecture 5 - Sequence modelling in NLP

*Automatic closed captions generated with the Azure Speech API*

### **1** 
Welcome back everyone. Today we're going to talk about sequence modeling, so how can we actually look at words in their context they appear in and what techniques are available in the natural language processing field?  

*21.48 seconds*

### **2** Today 
So we're going to look at two different aspects here. So how you can actually do sequence modeling? The first is via convolutional neural networks, which I think might be a little bit little bit surprising because CNN's are normally associated with computer vision models. But you can also and in my opinion, very cleverly use them in text processing to model N grams or create character embeddings. And then the second more common method, at least until last year also state of the art in most cases, is using recurrence, namely recurrent neural networks that as the name says, recurrent in a loop, go over the sequence and operate on a sequence of word.s And here we're going to look at a simple RNN and a flavor of an RNN that is a little bit more complicated, but also more powerful - the LSTM - as well as the encoder - decoder architecture for general text processing tasks, and we're going to start looking at attention. So this lecture is based on the neural network methods for natural language processing book by Yoav Goldberg. Follow him on Twitter. It's always nice to get some insights from him. 

*108.32 seconds*

### **3** Neural Networks are like LEGO blocks  
As I said before in the previous lectures, neural networks are like Lego blocks. And around 2014 they, like the community, switched to nonlinear neural networks with dense input for machine learning tasks. And as soon as tensor shapes match up, you can combine modules in Neural Network and then these modules are trained together end to end. And here you can see like a very simplified example for text processing when we have an input sequence which gets encoded. Then we do some sort of feature extraction and a classifier matching or decoding module at the end to produce some sort of output. 

*58.44 seconds*

### **4** 
And to do that, we're now going to revisit representation learning. From last the last lecture.  

*13.0 seconds*

### **5** Representation Learning: Word N-Grams 
Right, so this slide was shown also in the last lecture as an imitation of simple word embeddings. And simple word embeddings cannot distinguish between different orderings of words, or respectively cannot distinguish between different n-grams. So meaning when words Co occur with each other, they are very likely to hold some meaning because of the position in the sentence they actually hold. So this sentence contains the same words but has a totally different meaning.  

*42.49 seconds*

### **6** 1D CNN 
And to solve that problem we can use one dimensional CNN's, so 2 dimensional CNN's are ubiquitous in computer vision, right? So you have an image and you apply CNN filters on top of the RGB values. If you kind of step back and look at that little bit more abstract. What are CNNs doing? So they apply a filter with a sliding window over the input data. The values in the filter region are merged into an output value guided by the filter, so they are multiplied with weights. And those filter weights or parameters are learned during training. So you know which parts of that filter you want to emphasize. And typically, in CNN's using GPUs we apply multiple filters at a time in parallel. So that multiple filters can learn different things. And 1D CNN's just means that you have a one dimensional filter and operate on one dimensional input. 

*83.08 seconds*

### **7** Modelling Word N-Grams with 1D CNNs 
And to actually model word n-grams with 1D CNN's we do the following. We take word embeddings, so here we have input sequence like "do you like a good cappuccino?" And each word gets a word embedding via a look up. So we look up this vector in our world embedding matrix and then we apply a single CNN here and we only apply one CNN. And the reason we have multiple CNN's here is that this one CNN slides over the sequence with a step size of one. So each word is, here, in this example. we have two grams and each word appears in two different 2 grams, except for the first and the last. And what comes out of the CNN is the m-gram representation that might have a different dimensionality than the input. So we can also use the CNN kind of as a compression of our vector representations. And each representation here corresponds to a 2 gram. So "do you", "you like", "like a", "a good", etc. Right, and by itself we can't really do anything with that n-gram representation, so here we are still in the feature extraction and Encoding phase and we need to give those representation to another component that then scores, classifies, or does something else with them. And the cool thing here is that we can actually train both the CNN and the word embedding end-to-end.  

*136.58 seconds*

### **8** 1D CNNs in PyTorch 
How does this look in pytorch? So now I'm going to directly jump into pytorch code, and if you're not familiar with it, I suggest you look at the PY torch getting started guide to see what's going on. But maybe just as a too long didn't read Pytorch Works by having a model definition that defines the parameters of your model and a forward function that defines the control flow of the input through the model forming the output. And this 1D CNN in Pytorch works the following. So you define a sequential section here, and it's just a pytorch helper that in the forward function calls every other module every other NN module in the right order as you define it. We start by padding our input, so that's one thing about PY torch and CNN's. You always have to do the petting yourself, but in this case it's not very complicated. We just had the right side to match our kernel size, and in this case we have a kernel size of two and we have 300 input channels and 200 output channels, so every dot in the figure represents 100 values. Then at the end we also. in this case, applied a ReLU activation, but we don't necessarily have to do that. We could also leave out the activation or choose a different activation depending on what works best in this context. And then in the forward function we have to be careful. So this is kind of a hint for the second exercise that in PyTorch the convolution work in a different parameter ordering than what you would use for your embeddings, so you actually have to transpose the embeddings first. Put it into the convolution and then transpose the output again to receive the same, more intuitive way of. Using the embeddings, or let's say using the dimensions in the tensor as you would think of it. So you start with the batch size. Then you have the sequence length and then you actually have the dimensionality of your representation. 

*174.01 seconds*

### **9** 1D CNNs in PyTorch 
So let's take a closer look at how this works. The convolution requires a tensor of the shape, batch size, embedding dimensionality, and then sequence length. Although if we use the pytorch embedding layer to get our word embeddings, the embedding shape has the shape  batch size, sequence length and then embedding dimension so it doesn't match up. So what are we doing is we transpose the embeddings, meaning we basically we switch the last two dimensions on the values of the last two dimensions. And then, after we receive our convolution output, we shape it back again and transpose it back again to the final result, which is the batch size. Then comes to sequence length and then comes the convolutional output channels. 

*74.0 seconds*

### **10** Pooling 
Once we have a CNN output, a very common operation is to use a pooling layer and a pooling layer means, well, a pooling layer can have two different variations, right? So we have common variations. We have Max pooling, which retains only the strongest feature per region, or we have average pooling, which retains the average over our features per region, and the pooling layer does not train additional weights. But it is also applied as a sliding window over the input sequence. And it can further reduce the sequence length, for example. 

*51.5 seconds*

### **11** Pooling 
Right, so how does this work? So we come back to our example of our word embeddings, followed by CNN's to get our n-gram representations, and then to further reduce the sequence length, because that's kind of the whole point of our model, we want to come from a long sequence length to basically a single value in case of classification or retrieval scoring. And to further reduce the sequence length, we apply pooling. And in this case here we have a pooling window size of two again, so basically every 2 grams get reduced to a single pooled output and Max and average pooling commonly work on an index level of our vectors. So we don't just take the one vector that has the maximum value but we take the maximum for each index and combine them. Or we take the average for each index and combine them that way. And for PY Torch as a tip, it's very important to add padding here as well. Otherwise py torch will just ignore the last element if it doesn't fit the window size of the pooling layer. And here the output size of course depends on the input size. 

*110.41 seconds*

### **12** Pooling - Variable input length 
And this variable length input is shown here in those two examples. So on the left we again have our same example as before. But now let's assume we have a second input sentence and the 2nd input sentence is a very long sequence. So because every module in our model here is based on the sequence length of the input, we get a much bigger output shape or a much bigger tensor out of those consecutive operations, which of course becomes a problem because at some point we have to get down to a single dimensional tensor. So how do we do that? 

*48.46 seconds*

### **13** Adaptive (Dynamic) Pooling 
One way to achieve that is adaptive or dynamic pooling. Here the kernel size is a function of the input size, so that the result is a fixed output size and which is a very practical thing. And here in Pytorch this is available again with maximum and average variance. 

*27.77 seconds*

### **14** Adaptive (Dynamic) Pooling 
And if we apply this dynamic or adaptive pooling, we can set our desired output shape, for example, to two by two. And the pooling the pooling layer size adapts for each individual sample that will look at. 

*27.63 seconds*

### **15** Hierarchical CNNs 
Then another way to reduce to reduce our sequence length further is to use hierarchical CNN's. So that's again similar to computer vision models where we stack multiple CNN layers on top of each other and the output of the previous layer is the input of the next layer. As simple as that. And with that we can cover a broad range very quickly because common implementations of CNN's allow you to set the sliding windows step size. And they also allow you to put spacing between the input points. And yeah, you should check out those cool animations at the bottom there that really showcase how this is done and how this works. 

*53.73 seconds*

### **16** Multiple CNNs in PyTorch (with a lot of flexibility) 
And if you if you want to do that, if you want to set multiple convolutions in PyTorch and make it all work, there is a very nice pytorch helper class called module list. And this module list. Allows you to very flexible first define different modules for your PY torch model, in a sequential fashion, for example. But if you then in the forward function want to do some logic between your convolutional layers, you can do that by using the module list, where you can actually iterate through every entry in the module list and still have everything properly registered in the PY torch API so that your model knows about your parameters so that the optimizer knows about the parameters and when you want to save something, your parameters are saved as well. 

*75.86 seconds*

### **17** Representation Learning: Character Embeddings 
Furthermore, we can use CNN's also for character embeddings. So beyond more Word2Vec and word embeddings, we can do a lot of other stuff. And word embeddings, as said in the previous lecture, have the problem of out of vocabulary words that are not inside the vocabulary. So if you want to have or if you have a task, for example, when you use a lot of online text that comes from social media for example, and you don't have well formed words most of the time, character embeddings can be really helpful to represent your words better. And here we index by character ID. And CNN's, then produce character n-gram representations that get buildup. And can we just use as a replacement of word embeddings in your model? They might be used with or without organization, so if you also don't know about word boundaries, you can just run your character embeddings over your untouched input. Yeah, and the actual network implementation in my opinion, looks surprisingly similar to word embeddings. 

*97.66 seconds*

### **18** 
This brings us to our second topic today - Sequence Modelling with recurrent neural networks. 

*8.2 seconds*

### **19** Recurrent Neural Networks 
Recurrent Neural Networks model global patterns in sequence. And the nice thing about them is they allow for arbitrarily sized inputs. And they're trainable with backpropagation gradient descent, just as all the other techniques we looked at today. And the recursion actually gets unrolled to form a standard computation graph. There is a lot of stuff happening in the background in Neural Network libraries that you luckily don't have to implement yourself. And they have, RNN's have the ability to condition the next output on the entire sentence history, which allows for conditional generational models that more closely matched the language modeling task we looked at in the last lecture. And you could say more intuitively, follow that pattern to read a sequence in steps of 1, build up your knowledge about the sequence and then predict the next word, then the next word to talk about. Of course you can use RNNs with any sequence data, not just text, but because we are text course, we are only looking at text. 

*100.54 seconds*

### **20** Simple RNN 
Let's take a look at a very simple implementation of the recurrent neural network and the basically simplest RNN takes the ordering of elements into account. So here we have the formula which gives us an output Si at the position I, so as I is the state of the RNN at position I, and it depends on the previous state Si - 1. So you can see here is the recursion coming in an it not only depends on the previous state, but also on the input value at position I. And in the simple form we have the previous state and the current input value each multiplied with weight matrices, added a bias vector and activated with a nonlinear activation function, so G stands here for something like tanh or Relu or any other nonlinear activation function, so there are coming more and more activation functions up in the community and you can choose which works best for your problem. And the interesting thing here is that the weight matrices and the bias, those are the trainable parameters. So you train those to produce a better output. 

*108.83 seconds*

### **21** RNN as Encoder 
Visually this looks the following, so again we have our input sequence and for example we take word representations again out of a word embedding. Those are the vectors in yellow here. And then we start with the first word X1 and put it into the recurrence sequence and now we can see that each call, each recurrent call of the RNN both depends on the sequence of the last state as well as the input of the current state. And at the end we receive a vector Sn which, optimally, would represent the full sequence on the single output of our RNN. And yeah, well, then would be too easy, wouldn't it? If that's just the end of our lecture now and the answer to all our questions, no. So we have to

*80.43 seconds*

### **22** RNN flavor: LSTM 
take into account, for example the vanishing gradient problem. So the simple RNN suffers from the so called vanishing gradient problem, meaning that it forgets, although forget is a word that would kind of tell you that if it thinks but yeah, it doesn't think it's just, it's just a linear algebra behind it. But let's just say it forgets, it forgets information from a few 1000 steps ago and each step, after computation, we read and write the entire memory state Si. The long short term memory flavor of the RNN mitigates this vanishing gradient problem by introducing a gated memory access and it splits the state vector Si in two. So we have a memory cell and working memory. And this whole thing still is differentiable, which is pretty cool and you can see here in the citation LSTM has been proposed along time ago. 

*79.34 seconds*

### **23** LSTM - Gating Mechanism 
Let's take a closer look at the gating mechanism, often STM. So what does? What does actually element-wise control of writing new values mean? We have a Binary Gate G and this Binary Gate G is a Hadamard-product multiplied with X our input and 1 - g is also multiplied with Hadamard-product with our state vector. So what does Hadamard-product mean? Well, it's very fancy term of saying elementwise multiplication and you can see here under the formula there is an example of this element wise multiplication. So when the gate is 0, we don't take that entry and when the gate is one, we just take this entry as a whole. The problem with that is that a Binary Gate is not differentiable. So if we work in the integer domain, it's not differentiable. We need continuous floating point values in the range of one to zero to make this whole thing work so. What can we do? We can apply a sigmoid activation that gives us a range of one to zero and in the sigmoid activation most values are pushed to the border of the range to one or to zero. And here in the corner you can see how the sigmoid activation function looks like. 

*119.14 seconds*

### **24** LSTM (As defined in Neural Network Methods in NLP) 
And with that differentiable gate, we can see this LSTM structure now. And here I'm showing you the LSTM as it is defined in the neural network book, but of course there are a couple of different flavors, but the general idea stays the same. So again, we start off with the recursion, so now we have our state vector SG as the result of the LSTM of the current input, as well as the previous state vector. And this Sj is a concatenation of Cj and Hj. And here in the brackets and the brackets here just mean vector concatenation. Cj is our memory and Hj represents our hidden state. Again, this dot with the circle around means the Hadamard-product. And we can see split up the different components of the LSTM here. So basically we have what we did before in the simple RNN, we now do three times with our input state as well as the hidden state and different weight matrices for i, f and o. So the input gate forget gate and the output gate. And the result of the memory also gets a tanh activation applied to it. 

*132.63 seconds*

### **25** LSTM Citations per Year 
Now to something a little bit lighter, so I just told you that the LSTM has been proposed in 1997. And never really took off. So if you look at the Google Scholar Citation count you can see that around the year 2014 something happened and people suddenly started using LSTMs a lot. 

*31.97 seconds*

### **26** Recurrent Neural Networks 
Other recurrent neural network architectures of flavors are the bidirectional RNN. So here what we mean is that we duplicate the sequence, reverse the 2nd and compute the recurrence twice, basically in each direction, we start from the beginning and we start from the end. And then we concatenate the output per word, which is usable with all RNN flavors, including the LSTM. And in Pytorch, it's a simple setting, a parameter to true. The problem of course is that you also have to take into account that you now do twice the work, but it works pretty well. And then, the next thing you can do is just as the CNN's, you can also stack RNNs on top of each other, which means that you use the output of the first as the input for the next on a word level. Which can again be combined with the bidirectional RNN. But of course, this increases the computational complexity again and again. In practical terms, there are still a lot of problems and boilerplate code associated with recurrent neural networks, but luckily there have been a lot of libraries proposed lately and implemented that help you, and that abstract a lot of those things, such as packed sequences, when you want to not waste computation on your padding etc,.And one solution is the AllenNLP Library for PY torch which reps alot of components in Pytorch and adds an LP specific things on top. 

*137.46 seconds*

### **27** Sequence In + Out Tasks 
Well, so where can we actually apply RNNs? In every sequence in and sequence out tasks. So that means in translation in question, answering in summarization, in email, auto response, chat bots that describe how much do like cappuccino for some reason. Well, and one thing all those tasks having common instead solutions for each of the tasks influences the other tasks, and so the field can advance as a whole. 

*43.96 seconds*

### **28** Encoder - Decoder Architecture 
And how we actually output a sequence is with the so called encoder-decoder architecture. It's a very versatile architecture that supports input sequences and output sequences. And based on the training data, you can basically take the same model and use it for different tasks.  

*32.93 seconds*

### **29** Encoder - Decoder Architecture 
So how does this look like? So we're going again with our initial example of a simple RNN. And this simple RNN now is located in the encoder here in orange. And in our simplest form we just take the last state, if we assume that this is a good representation of our input. And we take this last state SN, we rebranded as C for context, and now in the decoder we again use an RNN, but a different one. And starting from this context vector, we do the same thing as Encoding. But now we decode, which means we call an RNN recurrently. RNN in this case gets as input the context as well as the terms that it already outputed. And the recurrent state as well. So now the RNN produces one output vector and this output vector, here in this case for the word cappuccino, does not automatically tell you which word you actually should output. So what we do is we basically classify which word we should output based on our vocabulary. Very similar to the way Word2Vec train we compute a softmax probability over our vocabulary and the word with the highest probability gets outputed And we can, if we have training examples of input and output sequences, we know how much error we get in the softmax probability based on the gold standard data. 

*140.16 seconds*

### **30** Encoder - Decoder 
To formalize our encoder decoder, we take an RNN and this time we abstracted a little bit more and just say OK X1:N is our input sequence. And RNN Encoder encapsulates all the states in between. This RNN encoder produces C, the context vector. And this context, vector is fed into the decoder RNN together with the output of the decoder. So of course if we start for the first word, we don't have that and we start with a 0 or random vector. But after our second word, we take the previous one. And, yeah, here P is the probability of the term position V given the previous sequence of the output T1:V-1. And the probability as I said before, is computed over a softmax, that's a normalized exponential function. And our goal is to produce the best output sequence that maximizes the probability over our output sequence as a whole. 

*99.33 seconds*

### **31** Problems with the Plain Encoder - Decoder  
And of course, as with the simple RNN, there are a couple of problems with this plane encoder-decoder architecture. So, for example, the original authors of the encoder-decoder architecture Sutskever et al. found in the context of translation, that reversing the input sequence is best. And they wrote a very nice sentence in their paper that says basically we don't know why, but it's probably better to have short-term dependencies in the data set. Right, so what are things we can do to improve our plain encoder-decoder architecture? There are a couple of things. First, if we do a simple greedy search for the best output word where we individually maximize the probability, we might not maximize the output sequence as a whole and therefore we can use beam search to look at multiple paths at the same time. Then, the fixed length context vector definitely is a bottleneck, especially for longer sequences. And here we can use something called attention mechanism, which I will talk about in a minute. The next problem is if we have a lot of out of vocabulary words and we actually want to use our system so that the user sees output from our machine learning system. It's not nice when every sentence contains a lot of unknown tokens, so for example, if names are not known or so. So for that there is something called a pointer generator network, that also just selects words from the input.  

*143.55 seconds*

### **32** Generating the Output: Beam Search 
Let's talk about beam search. The way that the decoder is set up is that each output word depends on the previous one. And if we want to take more than one word into consideration, we actually have to build up a tree of possibilities. And Beam search is just a general purpose heuristic graph search algorithm, that can be applied to many problems. But here we can expand the most probable paths. So we don't actually have to follow paths that are not very probable, not very likely. But we only followed the best options and we can set a threshold for that. And then we can decode with multiple runs of the next word. So our decoding actually becomes quite interactive. And the Beam Search, of course, increases our computational complexity. But we always have to be aware of that and a very interesting fact from the original paper again is that an ensemble of five LSTMs with the beam size of two is cheaper than a single LSTM with the beam size of 12. 

*110.11 seconds*

### **33** Attention 
The next improvement we're looking at is attention and attention is a very general concept, which is here applied in the context of encoder-decoder. And so we'll look at it in this context, but just so you know that the attention mechanism can be applied to many more occurrences of different types of models. So the attention mechanism allows you to search for relevant parts of the input sequence by creating a weighted average context vector. So we're not taking the last vector as our context vector, but we actually learn to weight all possible context vectors. And the weights are based on the Softmax again, so they sum up to one. Then the attention is, in general, parameterized and trained end to end with the model. The attention does not require special training data or special training regime to work. In general, attention is very effective and versatile, and of course there is a jungle of different versions and purposes. So, one thing that attention provides is some sort of interpretability. So it is not a perfect solution, but it provides a window into the model, and we can easily showcase the different salience or important factors of individual words.  

*119.44 seconds*

### **34** Attention mechanism 
OK, so now let's take a look at the attention mechanism. We basically zoom into our into our last picture where we have a decoder state T at a certain time stamp T of the decoder and we have all our available encoder states S1 to SN. And now each of those is concatenated with the decoder state. And applied through a fully connected linear layer. Then this fully connected linear layer gives us the raw attention weights, so to speak. Those raw attention weights can be any number, and to move them into the range of 0 to one, make them some up to one, we apply a softmax normalization. And now we have a distribution over our encoder state vectors. And distribution is basically a single vector of filled with values from one to zero. And now we can elementwise, multiply the attention distribution with the encoder vectors and sum them up to form a single context vector that is attended by the decoder state. Yes, and so, the fully connected layer actually contains the learned parameters, so the softmax attention and the multiplications and summations, there aren't any parameters in there. So the fully connected layer is what gets learned to guide the attention. And in this case also what we should note is that the encoder states at that stage are read only memory, so the encoder states are not updated, but we only create our context vector. 

*149.44 seconds*

### **35** Additive Attention 
Right, and now let's have a look at how we can formalize one of the attention possibilities, the so called additive attention mechanism. And it is defined as follows. So we have our encoded input, S1:n, then one decoder state and we want to get out a context vector. And this context vector is made up of the individual attention weights multiplied with each encoded state. And then the Softmax, as said before, sums up to one. And the the last line here in this formula is basically the formal description of this single dense or fully connected, feed forward, MLP, that there are many ways to name that layer. But it's basically a single weight matrix, in this case, U. We have a single bias vector B. We apply a tanh and then again a scaling Factor V. Of course, many other attention varieties exist, and you can have a look at them in the book. 

*95.88 seconds*

### **36** Recall the Encoder - Decoder Architecture 
Alright, so how does the attention mechanism now fit into the big picture? Let's recall our encoder-decoder architecture as shown a couple of minutes ago, and now let's. 

*16.0 seconds*

### **37** Encoder - Decoder & Attention 
put in the attention. So the attention mechanism sits in between the two. And is called every time the decoder needs a new context state. And yeah, the attention is computed a new for each new RNN state of the decoder. And so here, in this example, we can see that when we output the word love in this toy example, we say that the decoder attends very strongly to the word like 'cause they are connected. Of course that's extremely oversimplification, but if you need to visualize that, you can do so.  

*58.07 seconds*

### **38** Encoder - Decoder & Attention 
And here again, we formalize the encoder-decoder architecture with attention and what we did differently this time is basically we added our attend module before the decoder on every state change. The rest of the architecture stayed the same. 

*28.77 seconds*

### **39** Pointer Generator (Summarization) 
Then the last improvement we're going to look at today is the so-called pointed generator. And in this example on the left here, you can see that, in a practical scenario, when we, for example, want to generate summarization of text, ee encounter a lot of words that are not in the vocabulary because they are just so infrequent, such as the name of someone. And if we don't do anything about it, our model just has to output the unknown token because it doesn't know better. Or it makes mistakes by taking wrong information. So the point of generator network actually learns to differentiate between the vocabulary we know about and vocabulary from our input that we just want to copy at certain positions into the output.  

*84.57 seconds*

### **40** Pointer Generator  
And works as follows. So it again we get this context vector based on our attention distribution over our encoder hidden states. But, in addition to the context vector, we also get a gating parameter Pgen in this case. That combines the vocabulary distribution with the input distribution of the words. And so basically can learn to point at a word, to just copy it to the output and not generate a new word out of our known vocabulary. Again, the pointer generation as well as the attention can be trained end to end, which I think is pretty cool. 

*54.4 seconds*

### **41** Interested? Here is more  
All right, uhm? So before we finish with this lecture, I just wanted to say, of course, there are many more things out there, and if you're interested in natural language processing, here are some pointers that to get you started. So the Stanford lecture and a website called distill.pub, which produces very high quality articles, interactive articles that explain different machine learning and natural language processing things. Then we have the awesome book by Yoav Goldberg and also a list of cool and interesting NLP people to follow on Twitter which I like very much.  

*59.47 seconds*

### **42** Summary: Neural Networks for NLP 
So what I want you to take away from today's lecture is the following. Neural Networks are versatile, the techniques are shared between tasks. So even though people work on different tasks a lot of ideas are shared between the tasks and the community moves forward as a whole. Then CNNs can be utilized for n-gram representation learning, in a way taht you wouldn't think if you just look at computer vision models. And finally recurrent neural networks model sequences and are very useful if you want to work on input and output of sequences via the encoder-decoder model.  

*61.79 seconds*

### **43** Thank You  
Thank you for your attention and I hope you tune in next time see you. 

*7.76 seconds*

### Stats
Average talking time: 76.9444578488372
