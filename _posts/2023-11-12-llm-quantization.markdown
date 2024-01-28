---
layout: post
title:  "Reducing resource requirements: Hands-on LLM quantization and post-quantization performance assessment for dummies" 
date:  2023-09-18 11:31:29 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["LLM/ML/AI"]
---

* TOC
{:toc}

### Intro
In LLM context, `quantization` is a technique used to reduce the precision of the models parameters (weights and biases) in order to reduce
the model's memory footprint.  
While it has a lot of advantages, the main ones are that it allows us to reduce hardware requirements for the model we use (or use a bigger model with the same hardware),
and reduce inference cost (or increase the speed).  

Similar to <a href="/jekyll/update/2023/09/18/llm-fine-tuning-hypterparameters-simplified.html" target="_blank">Effective LLM fine-tuning for dummies</a>,
the content of this post is for professionals (such as software engineers and architects) which aren't specialized in AI and data science domains but required to gain
sufficient knowledge to enable and lead features and projects in these domains, so the content is simplified and covers the main subjects without getting into too many details while maintaining the hands-on approach.

### Post training quantization VS quantization aware training
Typically, quantization process can be implemented in two different phases, during training (or fine-tuning), or during inference.  
Quantization during training usually referred to as `QAT` (quantization aware training), while quantization during inference referred to as `PTQ` (post training quantization).  
`QAT` is a computationally expensive process but might result in a better model accuracy than the cheaper, simpler `PTQ`.  
With that being said, depending on the data and the precision after the quantization, the reduction in the model accuracy is often negligible.
In this post, we'll do some hands-on `PTQ` quantization.

### Precision to memory usage prediction
As a generic rule of thumb, to predict *approximate* memory requirement of a specific model, you should multiply the number of parameters
by the precision of the weights.  
Let's take `falcon-7b` with a `single precision float 32bit` weights as an example.  
`Float 32bit` takes up 4 bytes, so if we multiply the number of the bytes by the number of parameters, we'll get:  
`4 × 7e9 = 2.8e10 = 28,000,000,000 bytes = 28gb`.  
Based on the same logic, weight reduction to a `half precision float 16bit` which takes 2 bytes will require approximately 14gb memory.

### Rule of thumb: accuracy VS resource requirements balance
The optimal weight precision to accuracy balance, like a lot of things in the LLM ecosystem - depends on the use case and the data.  
As a rule of thumb, quantizing to `int 8bit` offers hardly noticed accuracy degradation while reducing memory requirements by 50% or 75%, depending on the original
parameter precision.  
With that being said, more aggressive precision reduction such as `int4`, `float4`, or even `double quantization` does exist and definitely worth testing whether they are good enough
for your data and use case.

### Running quantized model inference
Similar to the <a href="/jekyll/update/2023/09/18/llm-fine-tuning-hypterparameters-simplified.html" target="_blank">Effective LLM fine-tuning for dummies</a> post,
I will use `LIT-GPT` as the tool due to its simplicity.  
Running quantized inference with LIT-GPT is pretty straightforward, for example:  

{% highlight python %}
python generate/base.py --prompt "Annie, are you ok?" --checkpoint_dir checkpoints/tiiuae/falcon-7b --precision bf16-true
{% endhighlight %}

This command will run an inference with a 16bit precision parameters.  
As explained above, there additional options such as `int4`, `float4` etc.  
For the full list of options check <a href="https://github.com/Lightning-AI/lit-gpt/blob/main/tutorials/quantize.md" target="_blank">LIT-GPT quantization Github page</a>.

### Automatic post-quantization accuracy evaluation
It's a good idea to manually evaluate handpicked set of examples before and after quantization to assess the model's performance after quantization.  
But if we want to perform a more thorough assessment on a larger test set, or to run it as a step in one of our CI/CD pipelines, an automatic tool will be required.  
One of these tools is `evalute`, which can be found in <a href="https://huggingface.co/evaluate-metric" target="_blank">hugging face</a> (or <a href="https://github.com/huggingface/evaluate" target="_blank">Github</a>).  
`Evaluate` provides 53 easy tools evaluation tools which we can leverage to test post quantization model performance.  

#### Pre and post quantization summarization quality example using evaluate
For the sake of the example, let's assume that our use case is to test text summarization capabilities of a language model pre- and post-quantization.  
*One of the multiple* metrics we can use to compare the quality of the summary is ROUGE (Recall-Oriented Understudy for Gisting Evaluation).  
We can run the evaluation module twice, first time with the pre-quantized inference output, second time with post-quantized - and compare the scores.  

{% highlight python %}
import evaluate

rouge = evaluate.load('rouge')

predictions = ["Star Trek and Star Wars, while both set in space, differ significantly in their themes and focus. Star Trek, primarily a TV franchise, explores scientific and philosophical issues, emphasizing diplomacy and exploration. Its stories are set in a universe where Earth is part of a vast, multi-species United Federation of Planets. The narrative often revolves around the crew of starships, like the USS Enterprise, as they embark on peaceful explorations and diplomatic missions. In contrast, Star Wars is a space opera centered around the epic battle between good (the Jedi) and evil (the Sith). It's a film series set in a galaxy far, far away, where mystical Force users and grand space battles take center stage. Star Wars is known for its iconic characters like Darth Vader and its focus on destiny and heroism, differing from Star Trek's more grounded and intellectual approach."]
references = ["Star Trek and Star Wars are both popular space franchises, but they have distinct differences. Star Trek, which originated as a television series, is known for its focus on exploration, science, and philosophy. It is set in a future where Earth is part of an interstellar federation, and the narrative usually involves the crew of spacecraft, such as the Enterprise, engaging in exploration and diplomatic efforts. Star Wars, on the other hand, is a cinematic saga that leans more towards the space opera genre, with a clear delineation of good (represented by the Jedi) and evil (embodied by the Sith). It takes place in a distant galaxy and is more centered on action, adventure, and the mystical Force, with legendary figures like Luke Skywalker and Darth Vader. Unlike Star Trek's emphasis on rationality and discovery, Star Wars focuses on the theme of heroism and the classic battle between light and dark."]

results = rouge.compute(predictions=predictions,references=references)
print(results)

>>> {'rouge1': 0.5570469798657718, 'rouge2': 0.23648648648648649, 'rougeL': 0.3691275167785235, 'rougeLsum': 0.3691275167785235}
{% endhighlight %}

As explained above, similar ROUGE score does not guarantee a good post quantization accuracy since it just measures similar terms and phrases, but it's enough
to demonstrate the ease of use of the `evaluate` package.


