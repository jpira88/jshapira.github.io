---
layout: post
title:  "Effective LLM fine-tuning for dummies" 
date:  2023-09-18 11:31:29 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["LLM/ML/AI"]
---

* TOC
{:toc}

### Intro
At the time of writing this entry, the tech world is caught up in what I like to call a LLM-BOOM.  
Everyone's implementing LLM based features, new models announced on an hourly basis and the LLM ecosystem, both open source and proprietary is exploding.  
A lot of professionals (such as software engineers and architects) which aren't specialized in AI and data science domains required to gain
sufficient knowledge to enable and lead features and projects in these domains.  
Luckily, one of the advantages of this LLM-BOOM is that tools are getting better and easier to use.

One of the most frequent tasks is to create LLMs with domain specific knowledge,
and the purpose of this post is to explain LLM fine tuning process, terminology and hyperparameters in simple terms for professionals
which aren't extremely proficient the data science and AI domains.

Fine-tuning an LLM model was not a very straightforward task a year ago, but now we have variety of tools allowing step-by-step fine tuning of models.
One of these tools is <a href="https://github.com/Lightning-AI/lit-gpt" target="_blank">Lit-GPT</a>, which is defined as an `Hackable implementation of state-of-the-art open-source large language models released under the Apache 2.0 license`.
Which basically means it's main design goal is simplicity over SOLID principles.

With that being said, in order to use this tool (or any other for that matter) effectively, you need to properly configure it, and in order to properly configure it,
you need to understand what each configuration means and have sufficient general knowledge about what you're doing.

I will use Lit-GPT as an example tool due to creators decision to choose simplicity over other design properties,
but the concepts and parameters this entry will cover are generic to many fine tuning other tools out there.


### Base model, fine-tuned model and initial weight and biases
In most use cases, when training a model from scratch, weights and biases are initialized with either random or constant values
and does not truthfully represent the data.  


During the training process the weights and biases are adjusted and the `base model` gains understanding of natural language, grammar and the data it was trained on.  
When fine tuning, you'll (usually) use a `base model` with pre-existing weights and biases, and train it on your domain-specific datasets to adjust the (but not re-generate) the weights.  


The model with the updated weights and biases is the `fine tuned model`.  
As you probably understand, fine tuning a model is much simpler, quicker and cheaper than training a model from scratch.  
Of course there is an exception if you attempt to train a base model on a domain specific data, but this is a more complex
use case and out of scope for this post.

### Preparing the base model
Now that we understand the difference between base and fine-tuned models, we need to select the base model
that we will fine tune.  
Lit-GPT supports 16 open source models (you can see the full list in the Github repo),
but we'll run our example based on falcon model.

It's pretty straightforward to download base model checkpoints and convert it from Hugging Face to Lit-GPT.  
If you're not familiar with <a href="https://huggingface.co/" target="_blank">HF</a>, it's a collaboration platform for te
AI & LLM community, I like to think of it as Github for AI.

You'll need to run the following:

{% highlight python %}
# Download the falcon7b-instruct base model:
python scripts/download.py --repo_id tiiuae/falcon-7b-instruct

# Convert from HF to Lit-GPT format:  
python scripts/convert_hf_checkpoint.py --checkpoint_dir checkpoints/tiiuae/falcon-7b-instruct

# Run test inference  
python generate/base.py --prompt "Annie, are you ok?" --checkpoint_dir checkpoints/tiiuae/falcon-7b
{% endhighlight %}

Now that we have our base model up and running, we can move forward.


### Creating domain specific dataset
When fine tuning general purpose models with domain specific knowledge, most of the time (but not always) you'll need to create some sort of input-output datasets.  
It can be instructions with inputs and outputs, questions and answers or other formats.  
In our case the fine tuning will be done on an instruction data set (which means the more specific term in our case is instruction tuning, but let's ignore that for simplicity purposes)
based on the <a href="https://github.com/tatsu-lab/stanford_alpaca" target="_blank">Alpaca</a> dataset, which looks like this:  

{% highlight json %}
[
.....
    {
        "instruction": "What are the three primary colors?",
        "input": "",
        "output": "The three primary colors are red, blue, and yellow."
    },
    {
        "instruction": "Evaluate this sentence for spelling and grammar mistakes",
        "input": "He finnished his meal and left the resturant",
        "output": "He finished his meal and left the restaurant."
    },
.....
]
{% endhighlight %}

To proceed with our use case, you should create a domain specific dataset in a similar format:

{% highlight json %}
[
.....
    {
        "instruction": "My instruction...",
        "input": "My input...",
        "output": "My output...",
    }
.....
]
{% endhighlight %}



### Preparing data sets
After creating the data set, we can run the preparation process.  
If you examine the `scripts/prepare_alpaca.py` file, you'll be able to see that you can set various parameters.  
While most of them are pretty self-explanatory (destination path, checkpoint dir etc'), some of them deserve further explanation.

#### Data split with a seed - train, test and validation sets
One of the things you'll need to define is the `test_split_size` which indicates the size of the data you'll use to test the performance.
In this case we have only training and test sets, but it is a common practice to use also validation data sets.  
The optimal train/validation/test split depends on your data, but standard splits can be 80/10/10, 70/15/15 or 60/20/20.  

`Training data` is pretty self-explanatory, this is the data you'll be using to train your model.

`Validation data` used to automatically or manually optimize hyperparameters during training phase.
For example, every N intervals, we can validate the model performance with a loss function.  

A `loss function` quantifies the difference between the predicated output as stated in the validation samples and the actual output produced by the model.  
If we (manually or automatically) notice that the loss function stops decreasing (or stops increasing), we might want to adjust the hyperparameters.

`Test data` is the final data on which we will test the model performance.

You'll also be required to set a `seed`.  
A seed usually used in order to reproduce random generation, so for a specific seed, the random sequence will always be the same.  
In our case, it is used to take the same random order of `test_split_size` samples from the data set.
Although the test samples are randomized, it is important to use seed for reproducibility of experiments.
The actual number of the seed is not important as long as it is consistent per experiment. 


#### Tensors and PyTorch models
As you'll see once you run the preparation script, you'll get `train.pt` and `test.pt` file in your `/data` folder.  
- `.pt` files are saved PyTorch models.   

- PyTorch models can store various data, such as weights, tokens, simple python objects and tensors.  

- A `PyTorch Tensor` is basically a generic multidimensional array containing numerical values on which we can perform computational actions.

In our case, these files contain pytorch objects with textual and tokenized representation of each instruction in your data set:

{% highlight python %}
{
'instruction': "My instruction...",
'input': 'My input...',
'output': "My output...",
'input_ids': tensor([ 19028, 304, 267 , ... ], dtype=torch.int32),
'input_ids_no_response': tensor([ 19028, 304, 267, ... ], dtype=torch.int32),
'labels': tensor([ 19028, 304, 267 , ... ], dtype=torch.int32)
}
{% endhighlight %}

You can see that `input_ids` are stored as a tensor, which means we can perform computational actions on it, for example, get the MIN and MAX values of the tensor:

{% highlight python %}
x = tensor([ 19028, 304, 267])
torch.aminmax(x)
>> torch.return_types.aminmax(min=tensor(304), max=tensor(19028))
{% endhighlight %}

There are a lot of out of the box operations we can perform on a tensor, all of them documented in <a href="https://github.com/tatsu-lab/stanford_alpaca" target="_blank">PyTorch's API</a>.  


#### Max sequence length
Depending on the model capabilities and the hardware you're running (specifically GPU), you'll want to set
the max length sequence which will limit the number of tokens produced by the tokenizer to avoid out of memory errors.  
As you can see from the example above, the training prompts are tokenized into sequence of numbers representing the text.  
The max sequence length parameter limiting the number of produced tokens.


### Starting the actual fine tuning
Once we got the data ready, we can start with the actual fine tuning process by running:

{% highlight python %}
python finetune/adapter_v2.py
{% endhighlight %}

But before doing so, let's check out this file and understand how fine tuning works and the configuration we need to apply for our use case.

#### Fine-tuning flow
In our case (and many others), in a high level, fine tuning flow is as follows:

- Taking data from the training data set.
- Running X training iterations adjusting the weights and biases.
- Saving checkpoints with weights and biases every Y iterations.
- Evaluating every Z times the performance with a `loss function` (explained above).
- If pleased with the results, saves final version of the weights and biases.


As you can see from examining the file, we have bunch parameters related to actual flow.

- `eval_iters` - controls how many times time we will adjust the weights and biases.
- `eval_interval` - controls how often we will run a loss function to evaluate the performance.
- `save_interval` - controls how ofter we will save checkpoints during the training.
- `devices` - determines on how many GPUs we will be running.

Now that we got the gist of the fine tuning flow, let's go over the more significant hyperparameters that 
will determine the quality of the training.

#### Learning rate and catastrophic forgetting
Learning rate is a hyperparameter which dictates how drastically the model weights will be modified during fine tuning.
This should be a relatively a low number (hence the default value is 0.009) so the model will not forget it's initial training (a process called catastrophic forgetting),
but high enough for the weights to represent the newly learned data.

#### Epochs and epoch sizes
An `epoch` is a term that refers to passing an entire training data set 1 time during fine tuning.  
An `epoch_size` is the number of total samples to train on.

#### Gradient accumulation, batches and micro-batches
A `batch` is the number of samples to use at a time before updating the weights.  
For example, if your entire training dataset consist of 1000 samples, and you define a batch as 100,
it will take 10 iterations to complete 1 epoch.

Finding the optimal batch size to achieve good results with minimum iterations and weights updates is no easy task.
Too big of a batch - you'll be out of memory,
too small - training will be slow and weight updates will be noisy.

This is where the gradient accumulation technique is used.  
Perhaps I'm over simplifying the explanation since this is not a post for data scientists, but in simple terms -
when we're using gradient accumulation, we are processing `micro_batch` size of samples at a given time, which prevents out of memory errors,
accumulating the required parameter changes to minimize the loss (aka make the prediction better), and once the micro batches reached
the batch size, we're actually updating the parameters (weights and biases).

That way, we're not constantly updating the parameters which reduces noice and improves performance, nor we're risking running out of memory.

For clarity, I've extracted the relevant code from the fine tuning script into a simple implementation example:

{% highlight python %}

# setting parameters for batch and micro batch
batch_size = 128 / devices
micro_batch_size = 2
gradient_accumulation_iters = batch_size // micro_batch_size

# iter_num is the number of iteration, omitted from the example
is_accumulating = iter_num % gradient_accumulation_iters != 0

# accumulate the changes to the parameters
with fabric.no_backward_sync(model, enabled=is_accumulating):
    logits = model(input_ids, lm_head_chunk_size=128)
    # shift the targets such that output n predicts token n+1
    logits[-1] = logits[-1][..., :-1, :]
    loss = chunked_cross_entropy(logits, targets[..., 1:])
    fabric.backward(loss / gradient_accumulation_iters)

# if done accumulating, save
if not is_accumulating:
    optimizer.step()
    optimizer.zero_grad()
    step_count += 1

{% endhighlight %}

#### Overfitting, underfitting and weight decay
In basic terms,
`overfitting` occurs when a model performs very good on data it has seen but very poorly on new unseen data.  
`Underfitting` occurs when a model performs poorly on both seen and unseen data.  
`Weight decay` is a technique used to prevent overfitting of a model.
This parameter basically reduce weight adjustment during an update and needs to be carefully adjusted since a number too small
will cause overfitting while a number too big will cause underfitting.

#### Warmup iterations
`Warmup iterations` is the number of epoch iterations where the defined `learning rate` (aka how drastically weights are changed) is lower than the default learning rate defined for the fine tuning process.  
Each warmup iteration the learning rate is increasing up until it reaches the defined hyperparameter value.  
It is used to prevent the model's parameters drastically change upon exposure to new data, which might cause early over fitting and other undesired results.

### Running inference
Now that we have finished fine tuning the model we can run an inference.

{% highlight python %}
## [adapter_path] - Path to the checkpoint with trained adapter weights, which are the output of `adapter_v2.py`.
## [checkpoint_dir] - The path to the checkpoint folder of the base model.
python generate/adapter_v2.py --adapter_path [adapter_path] --checkpoint_dir [checkpoint_dir] --prompt "Annie, are you ok?"
{% endhighlight %}


