Focus on the post-training pipeline and SFT data curation. 
### R1-zero
R1-zero is built by conducting RL finetuning on a base model directly, without SFT.
#### RL finetuning
Use GRPO algorithm.

The reward consists two parts:
* reward from a "real environment" like a Math question verifier, or a code compiler
* reward from a trained format reward model that enforces the output in pre-defined tags, \<think\>, \<answer\>.

My understandings here: 🤔
* Because the reward is from a "real environment", that's why the paper says it conducts RL training without any supervised data.
* Only works for math questions, code, etc. that have clear solutions to get a reward score.
#### training template
A training template is designed to guide the model to first produce a reasoning process, followed by the final answer.
The template contains instructions **only on structural format**, avoiding any content-specific prompts, like reflective reasoning or any particular strategies.
#### Drawbacks
R1-zero has several issues, for instance readability and language mixing. 
### R1
The paper says it employs a 4 stage pipeline. 

I understand there are two iterations: 🤔
* First iteration is reasoning oriented, SFT + RL to train a model to generate reasoning training data
* Second, SFT + RL to re-train a base model on a wide range of data distribution

<img src="assets/deepseek-r1-pt.jpg" alt="deepseek r1 post training pipeline" width="500"/>

#### First iteration
The goal of this iteration is to generate reasoning data for the second iteration.
##### Collect human-friendly cold start data and do SFT
First step, collect a cold start dataset consisting thousands of **long CoT of high quality**:
* using few-shot prompting with a long CoT as an example
* directly prompting models to generate detailed answers with reflection and verification
* gather R1-zero outputs in a readable format
finally refining results through post-processing by human labellers.
<br>(🤔 which model is prompted to get the samples?)

Second step, Use the cold start data to do SFT on a base model.

The advantages of cold start data:
* better readability
* better performance by carefully designing the pattern for cold start data with human priors.

##### Reasoning-oriented RL like R1-zero
Same large-scale RL training as employed in R1-zero, focus on reasoning capabilities.

In addition, a language consistency reward is introduced, calculating the proportion of target language words in the CoT.

##### Collect data for next iteration SFT
For reasoning data:
* rejection sampling the currently RL trained model
* expand the dataset distribution by incorporate more data, some uses **a generative reward model** by feeding the ground-truth and model predictions into Deepseek-v3 for judgement. (🤔 why feed into V3 model if the ground-truth is known? )
* filter out samples with mixed languages, long paragraphs and code blocks.
* Multiple responses are sampled for each question, and only the correct ones are kept.

Non-reasoning data
* adopt DeepSeek-v3 pipeline and reuse portions of the SFT dataset
* call DeepSeek-v3 to generate CoT before answering the question by prompting. (🤔 sounds like the currently trained RL model is prompted to generate response, but provide V3 CoT in the prompt.)

#### Second iteration
First, SFT finetune a base model for two epochs using the data collected in the last iteration.

Second, RL finetune again. Reward signals include:
* For reasoning data, same as R1-zero
* For non-reasoning data, use a trained reward model aligned with human preference

