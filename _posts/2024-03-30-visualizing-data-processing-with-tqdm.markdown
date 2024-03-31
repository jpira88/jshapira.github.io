---
layout: post
title:  "Visualizing data processing progress with tqdm" 
date:  2024-03-30 11:31:29 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["BackEnd","DataEngineering","LLM/ML/AI"]
---

* TOC
{:toc}

### Intro
This entry would be just a quick tip for those of you who are working with data processing in Python and would benefit from a visualizing the progress.
There's a simple library called `tqdm` which allow you to do that with a minimal overhead.  

Let's check the usage with a model training dummy code:  

{% highlight python %}

from tqdm import tqdm
from time import sleep

train = lambda data: [sleep(0.01) for _ in tqdm(data, desc="Training", leave=False)]
validate = lambda data: [sleep(0.01) for _ in tqdm(data, desc="Validating", leave=False)]
load_data = lambda: [x for x in range(1000)]

all_data = load_data()

batches = [all_data[i:i + 100] for i in range(0, len(all_data), 100)]
for batch in tqdm(batches, desc="Processing batch", leave=False):
    train(batch)
    validate(batch)

{% endhighlight %}


As you can see, we have 3 dummy functions for loading data, training and validating, each of them invoking `sleep` function to simulate data processing.  
The main code consists of batch processing of the training data.

If we run the code, we'll be able to see the progress of the batches, the training and the validation of each batch, separately.

{% highlight shell %}

Processing batch:   0%|          | 0/10 [00:00<?, ?it/s]

Training:   0%|          | 0/100 [00:00<?, ?it/s]
Training:  10%|█         | 10/100 [00:00<00:00, 95.57it/s]
Training:  20%|██        | 20/100 [00:00<00:00, 96.27it/s]
Training:  30%|███       | 30/100 [00:00<00:00, 95.79it/s]
Training:  40%|████      | 40/100 [00:00<00:00, 95.70it/s]
Training:  50%|█████     | 50/100 [00:00<00:00, 95.16it/s]
Training:  60%|██████    | 60/100 [00:00<00:00, 95.29it/s]
Training:  70%|███████   | 70/100 [00:00<00:00, 94.88it/s]
Training:  80%|████████  | 80/100 [00:00<00:00, 94.61it/s]
Training:  90%|█████████ | 90/100 [00:00<00:00, 94.76it/s]
Training: 100%|██████████| 100/100 [00:01<00:00, 94.72it/s]
                                                           
Validating:   0%|          | 0/100 [00:00<?, ?it/s]
Validating:  10%|█         | 10/100 [00:00<00:00, 95.87it/s]
Validating:  20%|██        | 20/100 [00:00<00:00, 95.28it/s]
Validating:  30%|███       | 30/100 [00:00<00:00, 95.82it/s]
Validating:  40%|████      | 40/100 [00:00<00:00, 95.41it/s]
Validating:  50%|█████     | 50/100 [00:00<00:00, 95.66it/s]
Validating:  60%|██████    | 60/100 [00:00<00:00, 95.02it/s]
Validating:  70%|███████   | 70/100 [00:00<00:00, 95.23it/s]
Validating:  80%|████████  | 80/100 [00:00<00:00, 94.96it/s]
Validating:  90%|█████████ | 90/100 [00:00<00:00, 94.84it/s]
Validating: 100%|██████████| 100/100 [00:01<00:00, 94.92it/s]

Processing batch:  10%|█         | 1/10 [00:02<00:18,  2.10s/it]

Training:   0%|          | 0/100 [00:00<?, ?it/s]
Training:  10%|█         | 10/100 [00:00<00:00, 94.69it/s]
Training:  20%|██        | 20/100 [00:00<00:00, 95.85it/s]
Training:  30%|███       | 30/100 [00:00<00:00, 95.53it/s]
Training:  40%|████      | 40/100 [00:00<00:00, 94.67it/s]
Training:  50%|█████     | 50/100 [00:00<00:00, 95.10it/s]
Training:  60%|██████    | 60/100 [00:00<00:00, 95.35it/s]
Training:  70%|███████   | 70/100 [00:00<00:00, 95.70it/s]
Training:  80%|████████  | 80/100 [00:00<00:00, 96.15it/s]
Training:  90%|█████████ | 90/100 [00:00<00:00, 96.05it/s]
Training: 100%|██████████| 100/100 [00:01<00:00, 95.77it/s]
                                                           
Validating:   0%|          | 0/100 [00:00<?, ?it/s]
Validating:  10%|█         | 10/100 [00:00<00:00, 96.14it/s]
Validating:  20%|██        | 20/100 [00:00<00:00, 96.19it/s]
Validating:  30%|███       | 30/100 [00:00<00:00, 96.02it/s]
Validating:  40%|████      | 40/100 [00:00<00:00, 96.20it/s]
Validating:  50%|█████     | 50/100 [00:00<00:00, 95.70it/s]
Validating:  60%|██████    | 60/100 [00:00<00:00, 95.70it/s]
Validating:  70%|███████   | 70/100 [00:00<00:00, 95.40it/s]
Validating:  80%|████████  | 80/100 [00:00<00:00, 94.90it/s]
Validating:  90%|█████████ | 90/100 [00:00<00:00, 95.30it/s]
Validating: 100%|██████████| 100/100 [00:01<00:00, 95.33it/s]

........................

{% endhighlight %}

### Integrations, advanced features and performance
tqdm supports Pandas, Keras, Dask and IPython/Jupyter integrations and provides advances features such as manual updates of the progress,
callbacks support, custom messages, formatting and more.  
On top of that, even though tqdm claims to add only 60ns per iteration (and 80ns for gui), one of the more important features is the control over performance related properties such as intervals and miniters.

Refer to <a href="https://github.com/tqdm/tqdm" target="_blank">its Github page</a> for the full docs.




