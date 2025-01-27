---
description: Determine the behaviour of your LLM by selecting a suitable task.
---

# Tasks

In this section we will see what's a `Task` and the list of tasks available in `distilabel`.

## Task

The `Task` class takes charge of setting how the LLM behaves, deciding whether it acts as a _generator_ or a _labeller_. To accomplish this, the `Task` class creates a prompt using a template that will be sent to the [`LLM`](../technical-reference/llms.md). It specifies the necessary input arguments for generating the prompt and identifies the output arguments to be extracted from the `LLM` response. The `Task` class yields a `Prompt` that can generate a string with the format needed, depending on the specific `LLM` used.

All the `Task`s defines a `system_prompt` which serves as the initial instruction given to the LLM, guiding it on what kind of information or output is expected, and the following methods:

- `generate_prompt`: This method will be used by the `LLM` to create the prompts that will be fed to the model.
- `parse_output`: After the `LLM` has generated the content, this method will be called on the raw outputs of the model to extract the relevant content (scores, rationales, etc).
- `input_args_names` and `output_args_names`: These methods are used in the [`Pipeline`](../technical-reference/pipeline.md) to process the datasets. The first one defines the columns that will be extracted from the dataset to build the prompt in case of a `LLM` that acts as a generator or labeller alone, or the columns that should be placed in the dataset to be processed by the _labeller_ `LLM`, in the case of a `Pipeline` that has both a _generator_ and a _labeller_. The second one is in charge of inserting the defined fields as columns of the dataset generated dataset.

After defining a task, the only action required is to pass it to the corresponding `LLM`. All the intricate processes are then handled internally:

```python
--8<-- "docs/snippets/technical-reference/tasks/generic_transformersllm.py"
```

Given this explanation, `distilabel` distinguishes between two primary categories of tasks: those focused on text generation and those centered around labelling. These `Task` classes delineate the LLM's conduct, be it the creation of textual content or the assignment of labels to text, each with precise guidelines tailored to their respective functionalities. Users can seamlessly leverage these distinct task types to tailor the LLM's behavior according to their specific application needs.

## Text Generation

These set of classes are designed to steer a `LLM` in generating text with specific guidelines. They provide a structured approach to instruct the LLM on generating content in a manner tailored to predefined criteria.

### TextGenerationTask

This is the base class for _text generation_, and includes the following fields for guiding the generation process:

- `system_prompt`, which serves as the initial instruction or query given to the LLM, guiding it on what kind of information or output is expected.
- A list of `principles` to inject on the `system_prompt`, which by default correspond to those defined in the UltraFeedback paper[^1],
- and lastly a distribution for these principles so the `LLM` can be directed towards the different principles with a more customized behaviour.

[^1]: The principles can be found [here][distilabel.tasks.text_generation.principles] in the codebase. More information on the _Principle Sampling_ can be found in the [UltraFeedfack repository](https://github.com/OpenBMB/UltraFeedback#principle-sampling).

For the API reference visit [TextGenerationTask][distilabel.tasks.text_generation.base.TextGenerationTask].

### SelfInstructTask

The task specially designed to build the prompts following the Self-Instruct paper: [SELF-INSTRUCT: Aligning Language Models
with Self-Generated Instructions](https://arxiv.org/abs/2212.10560).

From the original [repository](https://github.com/yizhongw/self-instruct/tree/main#how-self-instruct-works):

_The Self-Instruct process is an iterative bootstrapping algorithm that starts with a seed set of manually-written instructions and uses them to prompt the language model to generate new instructions and corresponding input-output instances_, so this `Task` is specially interesting for generating new datasets from a set of predefined topics.

```python
--8<-- "docs/snippets/technical-reference/tasks/generic_openai_self_instruct.py"
```

For the API reference visit [SelfInstructTask][distilabel.tasks.text_generation.self_instruct.SelfInstructTask].

You can personalize the way in which your SelfInstructTask behaves by changing the default values of the parameters to something that suits your use case. Let's go through them:

- **System Prompt**: you can control the overall behaviour and expectations of your model.
- **Application Description**: a description of the AI application. By default, we use "AI Assistant".
- **Number of instructions**: number of instructions in the prompt.
- **Criteria for Query Generation**: the criteria for query generation that we want our model to have. The default value covers default behaviour for SelfInstructTask. This value is passed to the .jinja template, where extra instructions are added to ensure correct output format.

You can see an example of how to customise a SelfInstructTask to create Haikus in the snippet in the subsection Custom TextGenerationTask.



### EvolInstructTask

The task is specially designed to build the prompts following the Evol-Instruct strategy proposed in: [WizardLM: Empowering Large Language Models to
Follow Complex Instructions](https://arxiv.org/abs/2304.12244).

From the original [repository](https://github.com/nlpxucan/WizardLM/tree/main?tab=readme-ov-file#overview-of-evol-instruct):

_Evol-Instruct is a novel method using LLMs instead of humans to automatically mass-produce open-domain instructions of various difficulty levels and skill range, to improve the performance of LLMs_.

Use this `Task` to build more complete and complex datasets starting from simple ones.

```python
--8<-- "docs/snippets/technical-reference/tasks/generic_openai_evol_instruct.py"
```

You can take a look at a [sample dataset](https://huggingface.co/datasets/argilla/distilabel-sample-evol-instruct) generated using `EvolInstructTask`.

!!! note
    The original definition of `EvolInstruct` considers an elimination evolving step with different
    situations to remove the responses considered failures. Section 3.2, _Elimination Evolving_ in [WizardLM paper](https://arxiv.org/abs/2304.12244) shows these steps. We have implemented steps 2-4 as part of this task, but not step one. Step 1 of the elimination process can be implemented using labellers in `distilabel`.

For the API reference visit [EvolInstructTask][distilabel.tasks.text_generation.evol_instruct.EvolInstructTask].

#### EvolComplexity

The [Deita framework](https://arxiv.org/abs/2312.15685) presents a data selection framework composed of two initial steps that consist on adopting an evolution-based approach as defined in [WizardLM](https://arxiv.org/abs/2304.12244). The first of the evolution steps, related to the *complexity* is the same `EvolInstruct` task, exposed with the same name given in the paper:

```python
--8<-- "docs/snippets/technical-reference/tasks/generic_openai_evol_complexity.py"
```

For the API reference visit [EvolComplexityTask][distilabel.tasks.text_generation.evol_complexity.EvolComplexityTask].

#### EvolQuality

The second step from the [Deita framework](https://arxiv.org/abs/2312.15685) consists on enhancing the *quality* of the instructions, in the same spirit from `EvolComplexityTask`. The `EvolQualityTask` can be used to augment the *quality* of the instructions by *enhancing helpfulness, augmenting relevance, enriching depth, fostering creativity, and supplying additional details*, following the *Deita* implementation:

```python
--8<-- "docs/snippets/technical-reference/tasks/generic_openai_evol_quality.py"
```

For the API reference visit [EvolQualityTask][distilabel.tasks.text_generation.evol_quality.EvolQualityTask].


### Custom TextGenerationTask

The following examples show different cases of creating your custom `TextGenerationTask`. Inherit from `TextGenerationTask` and implement the `generate_prompt` and `parse_output` to customise the behavior of the `LLM`.

=== "OSS Instruct Task"

    This task implements the OSS Instruct from [Magicoder: Source Code Is All You Need](https://arxiv.org/abs/2312.02120). Generate problems and solutions from seed problems following the paper implementation:

    ```python
    --8<-- "docs/snippets/technical-reference/tasks/custom_task_text_generation_oss.py"
    ```

=== "Haiku SelfInstructTask"

    Here you cn see an example of how to customise a `SelfInstructTask` to create Haikus. The following [Haiku DPO dataset](https://huggingface.co/datasets/davanstrien/haiku_dpo) contains more information on how this dataset was created.

    ```python
    --8<-- "docs/snippets/technical-reference/tasks/custom_task_selfinstruct_haikus_docs.py"
    ```

=== "WizardLM Equal Prompts Task"

    The following task, obtained from [WizardLM: Empowering Large Language Models to Follow Complex Instructions](https://arxiv.org/abs/2304.12244), can be used to check whether two instructions are equal or different to decide whether to keep in your dataset or remove redundant instructions:

    ```python
    --8<-- "docs/snippets/technical-reference/tasks/custom_task_text_generation_wizardlm.py"
    ```


## Labelling

Instead of generating text, you can instruct the `LLM` to label datasets. The existing tasks are designed specifically for creating both `PreferenceTask` and `CritiqueTask` datasets.

### Preference

Preference datasets for Language Models (LLMs) are sets of information that show how people rank or prefer one thing over another in a straightforward and clear manner. These datasets help train language models to understand and generate content that aligns with user preferences, enhancing the model's ability to generate contextually relevant and preferred outputs.

Contrary to the `TextGenerationTask`, the `PreferenceTask` is not intended for direct use. It implements the default methods `input_args_names` and `output_args_names`, but `generate_prompt` and `parse_output` are specific to each `PreferenceTask`. Examining the `output_args_names` reveals that the generation will encompass both the rating and the rationale that influenced that rating.

#### UltraFeedbackTask

This task is specifically designed to build the prompts following the format defined in the ["UltraFeedback: Boosting Language Models With High Quality Feedback"](https://arxiv.org/abs/2310.01377) paper.

From the original [repository](https://github.com/OpenBMB/UltraFeedback): _To collect high-quality preference and textual feedback, we design a fine-grained annotation instruction, which contains 4 different aspects, namely instruction-following, truthfulness, honesty and helpfulness_. This `Task` is designed to label datasets following the different aspects defined for the UltraFeedback dataset creation.

The following snippet can be used as a simplified UltraFeedback Task, for which we define 3 different ratings, but take into account the predefined versions are intended to be used out of the box:

```python
--8<-- "docs/snippets/technical-reference/tasks/ultrafeedback.py"
```

=== "Helpfulness"

    The following example creates a UltraFeedback task to emphasize helpfulness, that is overall quality and correctness of the output:

    ```python
    --8<-- "docs/snippets/technical-reference/tasks/openai_for_helpfulness.py"
    ```

=== "Truthfulness"

    The following example creates a UltraFeedback task to emphasize truthfulness and hallucination assessment:

    ```python
    --8<-- "docs/snippets/technical-reference/tasks/openai_for_truthfulness.py"
    ```

=== "Honesty"

    The following example creates a UltraFeedback task to emphasize honesty and uncertainty expression assessment:

    ```python
    --8<-- "docs/snippets/technical-reference/tasks/openai_for_honesty.py"
    ```

=== "Instruction Following"

    The following example creates a UltraFeedback task to emphasize the evaluation of alignment between output and intent:

    ```python
    --8<-- "docs/snippets/technical-reference/tasks/openai_for_instruction_following.py"
    ```

Additionally, we at Argilla created a custom subtask for UltraFeedback, that generates an overall score evaluating all the aspects mentioned above but within a single subtask. Otherwise, in order to get an overall score, all the subtasks above should be run and the average of those scores to be calculated.

=== "Overall Quality"

    The following example uses a `LLM` to examinate the data for our custom overall quality criteria, which includes the different criteria from UltraFeedback (Correctness & Informativeness, Honesty & Uncertainty, Truthfulness & Hallucination and Instruction Following):

    ```python
    --8<-- "docs/snippets/technical-reference/tasks/openai_for_overall_quality.py"
    ```

For the API reference visit [UltraFeedbackTask][distilabel.tasks.preference.ultrafeedback.UltraFeedbackTask].

#### JudgeLMTask

The task specially designed to build the prompts following the UltraFeedback paper: [JudgeLM: Fine-tuned Large Language Models Are Scalable Judges](https://arxiv.org/abs/2310.17631). This task is designed to evaluate the performance of AI assistants.

```python
--8<-- "docs/snippets/technical-reference/tasks/openai_judgelm.py"
```

For the API reference visit [JudgeLMTask][distilabel.tasks.preference.judgelm.JudgeLMTask].

#### UltraJudgeTask

This class implements a `PreferenceTask` specifically for a better evaluation using AI Feedback. The task is defined based on both UltraFeedback and JudgeLM, but with several improvements / modifications.

It introduces an additional argument to differentiate various areas for processing. While these areas can be customized, the default values are as follows:

```python
--8<-- "docs/snippets/technical-reference/tasks/ultrajudge.py"
```

Which can be directly used in the following way:

```python
--8<-- "docs/snippets/technical-reference/tasks/openai_ultrajudge.py"
```

For the API reference visit [UltraJudgeTask][distilabel.tasks.preference.ultrajudge.UltraJudgeTask].

#### ComplexityScorerTask

This class implements a `PreferenceTask` to rate a of instructions according to its complexity difficulty. Defined in [Deita framework](https://arxiv.org/abs/2312.15685), it's intended use is the scoring of instructions whose complexity has been enhanced by means of the `EvolComplexity` method defined, inspired on the `EvolInstruct` method from [WizardLM](https://arxiv.org/abs/2304.12244).  

```python
--8<-- "docs/snippets/technical-reference/tasks/generic_openai_complexity_scorer.py"
```

For the API reference visit [ComplexityScorerTask][distilabel.tasks.preference.complexity_scorer.ComplexityScorerTask].

#### QualityScorerTask

This class implements a `PreferenceTask` to rate a list of instructions according to its quality. Follows the same idea defined in the `ComplexityScorerTask` from the [Deita framework](https://arxiv.org/abs/2312.15685), but in this case it rates the instructions in terms of concepts like helpfulness, relevance, accuracy, depth, creativity, and level of detail of the response.

```python
--8<-- "docs/snippets/technical-reference/tasks/generic_openai_quality_scorer.py"
```

By default, the quality is defined as the following the paper prompt, but can be modified updating the `task_description` as in the following example (keep in mind the default `task_description` corresponds to the `EvolQuality` criteria defined to evolve the initial instructions, so this should be taken into account):

```python
--8<-- "docs/snippets/technical-reference/tasks/generic_openai_quality_scorer_custom.py"
```

For the API reference visit [QualityScorerTask][distilabel.tasks.preference.quality_scorer.QualityScorerTask].

### Critique

The `CritiqueTask` is designed to be a labeller for generated text, while not only adding scores based on a rubric, but also critiques explaining the reasons why those scores have been provided. The critique can either be using a reference answer (gold answer) as e.g. Prometheus does, or just by generating the critique per each of the N provided generations.

The resulting datasets after running a pipeline with the `CritiqueTask` are useful towards either training a model to generate critiques based on the critiques generated by a more powerful model as e.g. GPT-4 from OpenAI, or to be used directly for DPO fine-tuning. The fact that the critique is generated per each pair, a balanced dataset could be generated with individual critiques and their scores, so that then we can e.g. define a threshold on what's considered chosen and rejected, to then run DPO fine-tunes.

While the `CritiqueTask` may seem fairly similar to the `PreferenceTask`, there is a core difference, which is the fact that the critiques are provided per each response or even to a single response, with no need to compare or rate them against each other.

#### UltraCMTask

This task is specifically designed to build the prompts following the format defined in the ["UltraFeedback: Boosting Language Models With High Quality Feedback"](https://arxiv.org/abs/2310.01377) paper.

UltraCM is a model that has been fine-tuned using the UltraFeedback dataset, so as to produce critiques for the generated content, as the authors claim in their paper: "Moreover, since ULTRAFEEDBACK provides detailed textual feedback, we also fine-tune a model that could critique model responses automatically. Our critique model, UltraCM, generates reasonable and detailed comments on various tasks.".

Ideally, the `UltraCMTask` will be more consistent when used with either their fine-tuned model UltraCM or with OpenAI, as both have been proven to produce successfully the structured content following the prompt formatting, and not only structured, but also meaningful and reasonable.

See the following snippet, with an example on how to instantiate the `UltraCMTask` which only requires the system prompt, and it can be modified based on how is the critique intended to be formulated, while the system prompt shown below is the default one as of the UltraFeedback paper.

```python
--8<-- "docs/snippets/technical-reference/tasks/ultracm.py"
```

#### PrometheusTask

This task is specifically designed to build the prompts following the format defined in the ["Prometheus: Inducing Fine-grained Evaluation Capability in Language Models"](https://arxiv.org/abs/2310.08491) paper.

Ideally, the `PrometheusTask` should only be used to format the prompts for the Prometheus models as those are the ones that have been fine-tuned to follow the same formatting and will produce consistent results compared to other base models or fine-tuned with different formats. In this case, since the formatting used by Prometheus follows the Llama 2 format, those are recommended. Otherwise, OpenAI has also proved to produce consistent results.

The following snippet can be used out of the box to define a simple `PrometheusTask` with the system prompt, the scoring criteria and the score descriptions, but those can be modified while keeping in mind that Prometheus always expects 5 scores from 1-5 with a meaningful description, as well as with a criteria relevant to the scores defined.

```python
--8<-- "docs/snippets/technical-reference/tasks/prometheus.py"
```
