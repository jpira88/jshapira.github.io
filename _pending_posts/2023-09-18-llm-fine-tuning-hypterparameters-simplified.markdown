---
layout: post
title:  "LLM fine-tuning concepts and hyperparameters simplified" 
date:  2023-09-18 11:31:29 +0200
categories: jekyll update
author: j.shapira
tags: ["LLM/ML/AI"]
---

At the time of writing this entry, the tech world is caught up in what I like to call a LLM-BOOM.  
Everyone's implementing LLM based features, new models announced on an hourly basis and the LLM ecosystem, both open source and proprietary is exploding.  
One of the advantages of this high paced environment is that tools are getting better and easier to use.  
  

In this entry I want to focus on fine tuning.  
If fine-tuning an LLM model was not a very straightforward task a year ago, now you can run step-by-step tools such as
<a href="https://github.com/Lightning-AI/lit-gpt" target="_blank">lit-gpt</a>.  
But in order to use this tool (or any other for that matter) effectively, you need to properly configure it, and in order to properly configure it,
you need to understand what each configuration means and have sufficient general knowledge about what you're doing.

The purpose of this entry is to simplify explanation about concepts and hyperparameters related to fine tuning in terms anyone can understand.  
The explanations might be slightly over-simplified for experienced data scientist, but the target audience of this entry are professionals (such as software architects and engineers etc') which
aren't specialized in the field, but required to have some solid understanding for their daily work.  

I will use lit-gpt as an example tool due to Lightning AI's decision to choose simplicity over clean code, which makes the code easy to understand
and explain. But the concepts and parameters this entry will cover are generic to many fine tuning tools out there.

### Data sets: Training, Validation, and Test
The first thing you'll be required to do is to split your data set into training and test data set.
Many times you will also have a validation data set.
The optimal train/validation/test split depends on your data, but standard splits can be 80/10/10, 70/15/15 or 60/20/20.  

*Training data* is pretty self-explanatory, this is the data you'll be using to train your model.

*Validation data* used to automatically or manually optimize hyperparameters during training phase.
For example, every N intervals, we can validate the model performance with a loss function.
A loss function quantifies the difference between the predicated output as stated in the validation samples and the actual output produced by the model.
If we (manually or automatically) notice that the loss function stops decreasing (or stops increasing), we might want to adjust the hyperparameters.

*Test data* is the final data on which we will test the model performance.

### Tensors and PyTorch models
As you'll see once you run the preparation script, you'll get `train.pt` and `test.pt` file in your `/data` folder.
`.pt` files are saved PyTorch models. PyTorch models can store various data, such as weights, tokens, simple python objects and tensors.
A `PyTorch Tensor` is basically a generic multidimensional array containing numerical values on which we can perform computational actions.




These `.pt` files are pytorch models representing 

{% highlight python %}
{
'instruction': "My instruction...",
'input': 'My input...',
'output': "My output...",
'input_ids': tensor([ 19028, 304, 267, 10522, 325, 11117, 241, 4957, 25, 14687, , ... ], dtype=torch.int32),
'input_ids_no_response': tensor([ 19028, 304, 267, 10522, 325, 248, 2726, ... ], dtype=torch.int32),
'labels': tensor([ 19028, 304, 267, 10522, 325, 11117, 241, 4957, 25, 14687, , ... ], dtype=torch.int32)
}
{% endhighlight %}

These files contain pytorch objects with textual and tokenized representation of each instruction in your data set.
Later, these files will loaded and used to fine tune and test the model.

### Evaluation intervals and iterations