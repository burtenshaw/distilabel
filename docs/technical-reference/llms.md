---
description: Take a look at the different classes that allow interacting with the Large Language Models.
---
# LLMs

In this section we will see what's an `LLM` and the different `LLM`s implementations available in `distilabel`.

## LLM

The [`LLM`][distilabel.llm.base.LLM] class encapsulates the functionality for interacting with a large language model.

It distinguishes between *task* specifications and configurable parameters that influence the LLM behavior.

For illustration purposes, we employ the [`TextGenerationTask`][distilabel.tasks.text_generation.base.TextGenerationTask] in this section and guide you to the dedicated [`Tasks`](../technical-reference/tasks.md) section for comprehensive details.

LLM classes share several general parameters and define implementation-specific ones. Let's explain the general parameters first and the generate method, and then the specifics for each class.

### General parameters

Let's briefly introduce the general parameters we may find[^1]:

[^1]:
    You can take a look at this blog post from [Cohere](https://txt.cohere.com/llm-parameters-best-outputs-language-ai/) for a thorough explanation of the different parameters.

- `max_new_tokens`: this parameter controls the maximum number of tokens the LLM is allowed to use.

- `temperature`: parameter associated to the creativity of the model, a value close to 0 makes the model more deterministic, while higher values make the model more "creative".

- `top_k` and `top_p`: `top_k` limits the number of tokens the model is allowed to use to generate the following token sorted by probability, while `top_p` limits the number of tokens the model can use for the next token, but in terms of the sum of their probabilities.

- `frequency_penalty` and `presence_penalty`: the frequency penalty penalizes tokens that have already appeared in the generated text, limiting the possibility of those appearing again, and the `presence_penalty` penalizes regardless of the frequency.

- `num_threads`: some LLMs work better when using several threads to generate text, this parameter allows to specify the number of threads to use.

- `prompt_format` and `prompt_formatting_fn`: these two parameters allow to tweak the prompt of our models, for example we can direct the `LLM` to format the prompt according to one of the defined formats, while `prompt_formatting_fn` allows to pass a function that will be applied to the prompt before the generation, for extra control of what we ingest to the model.

Besides the general parameters, some `LLM` subclasses also have some implementation specific parameters to control the text generation, but those will be explained in the corresponding section.

### `generate` method

Once you create an `LLM`, you use the `generate` method to interact with it. This method accepts two parameters:

- `inputs`: which is a list of dictionaries containing the inputs for the `LLM` and the `Task`. Each dictionary must have all the keys required by the `Task`.

    ```python
    inputs = [
        {"input": "Write a letter for my friend Bob..."},
        {"input": "Give me a summary of the following text:..."},
        ...
    ]
    ```

- `num_generations`: which is an integer used to specify how many text generations we want to obtain for each element in `inputs`.

The output of the method will be a list containing lists of `LLMOutput`. Each inner list is associated to the corresponding input in `inputs`, and each `LLMOutput` is associated to one of the `num_generations` for each input.

  ```python
  >>> llm.generate(inputs=[...], num_generations=2)
  [ # (1)
      [ # (2)
          { # (3)
              "model_name": "notus-7b-v1",
              "prompt_used": "Write a letter for my friend Bob...",
              "raw_output": "Dear Bob, ...",
              "parsed_output": {
                  "generations":  "Dear Bob, ...",
              }
          },
          {
              "model_name": "notus-7b-v1",
              "prompt_used": "Write a letter for my friend Bob...",
              "raw_output": "Dear Bob, ...",
              "parsed_output": {
                  "generations":  "Dear Bob, ...",
              }
          },
      ],
      [...],
  ]
  ```

  1. The outer list will contain as many lists as elements in `inputs`.
  2. The inner lists will contain as many `LLMOutput`s as specified in `num_generations`.
  3. Each `LLMOutput` is a dictionary

The `LLMOutput` is a `TypedDict` containing the keys `model_name`, `prompt_used`, `raw_output` and `parsed_output`. The `parsed_output` key is a dictionary that will contain all the `Task` outputs.

  ```python
  {
      "model_name": "notus-7b-v1",
      "prompt_used": "Write a letter for my friend Bob...",
      "raw_output": "Dear Bob, ...",
      "parsed_output": { # (1)
          "generations":  "Dear Bob, ...",
      }
  },
  ```

  1. The keys contained in `parsed_output` will depend on the `Task` used. In this case, we used `TextGenerationTask`, so the key `generations` is present.

If the `LLM` uses a thread pool, then the output of the `generate` method will be a [Future](https://docs.python.org/3/library/concurrent.futures.html#concurrent.futures.Future) having as result a list of lists of `LLMOutput` as described above.

### Validate prompts

Before calling the LLM with your dataset we can take a look at the prompts that will be sent to the engine without actually making the call, to check the data is as expected. The following examples show two different `LLM` cases, but just take into account the input will have to be in the format expected from the `Task`:

!!! Validation

    === "Generator"
        
        ```python
        --8<-- "docs/snippets/technical-reference/llm/validate_prompts_1.py"
        ```

    === "Labeller"

        ```python
        --8<-- "docs/snippets/technical-reference/llm/validate_prompts_2.py"
        ```


## Integrations

### OpenAI

These may be the default choice for your ambitious tasks.

For the API reference visit [OpenAILLM][distilabel.llm.openai.OpenAILLM].

```python
--8<-- "docs/snippets/technical-reference/llm/openai_generate.py"
```

### Llama.cpp

Applicable for local execution of Language Models (LLMs). Use this LLM when you have access to the quantized weights of your selected model for interaction.

Let's see an example using [notus-7b-v1](https://huggingface.co/argilla/notus-7b-v1). First, you can download the weights from the following [link](https://huggingface.co/TheBloke/notus-7B-v1-GGUF):

```python
--8<-- "docs/snippets/technical-reference/llm/llamacpp_generate.py"
```

For the API reference visit [LlammaCppLLM][distilabel.llm.llama_cpp.LlamaCppLLM].

### vLLM

Highly recommended to use if you have a GPU available, as it is the fastest solution out
there for batch generation. Find more information about it in [vLLM docs](https://docs.vllm.ai/en/latest/).

```python
--8<-- "docs/snippets/technical-reference/llm/vllm_generate.py"
```

For the API reference visit [vLLM][distilabel.llm.vllm.vLLM].

### Ollama

Highly recommended to use if you have a GPU available, as it is one of the fastest solutions out and also has metal support for the MacOS M1 chip and its follow-ups.
Find more information about it in the [Ollama GitHub](https://github.com/jmorganca/ollama/).

Before being able to use Ollama you first need to install it. After that, you can select [one of the models from their model library](https://ollama.ai/library) and use it as follows:

```bash
ollama serve
ollama run notus # or other model name
```

!!! note
    The `ollama run <model_name>` command will also set pre-defined generation parameters for the model. These can be found in their library and overridden by passing them as arguments to the command as shown [here](https://github.com/jmorganca/ollama/blob/main/docs/modelfile.md#valid-parameters-and-values).

We can then re-use this model name as a reference within `distilabel` through our `OllamaLLM` implementation:

```python
--8<-- "docs/snippets/technical-reference/llm/ollama_generate.py"
```

### HuggingFace LLMs

This section explains two different ways to use HuggingFace models:

#### Transformers

This is the option to use a model hosted on the HuggingFace Hub. Load the model and tokenizer in the standard manner as done locally, and proceed to instantiate your class.

For the API reference visit [TransformersLLM][distilabel.llm.huggingface.transformers.TransformersLLM].

Let's see an example using [notus-7b-v1](https://huggingface.co/argilla/notus-7b-v1):

```python
--8<-- "docs/snippets/technical-reference/llm/transformers_generate.py"
```

#### Inference Endpoints

HuggingFace provides a streamlined approach for deploying models through [Inference Endpoints](https://huggingface.co/inference-endpoints) on their infrastructure. Opt for this solution if your model is hosted on the HuggingFace Hub.

For the API reference visit [InferenceEndpointsLLM][distilabel.llm.huggingface.inference_endpoints.InferenceEndpointsLLM].

Let's see how to interact with these LLMs:

```python
--8<-- "docs/snippets/technical-reference/llm/inference_endpoint_generate.py"
```

### Together Inference

Together offers a product named Together Inference, which exposes some models for diverse tasks such as chat, text generation, code, or image; exposing those via an endpoint within their API either as serverless endpoints or as dedicated instances.

See their release post with more details at [Announcing Together Inference Engine – the fastest inference available](https://www.together.ai/blog/together-inference-engine-v1).


```python
--8<-- "docs/snippets/technical-reference/llm/together_inference_generate.py"
```

### Vertex AI LLMs

Google Cloud Vertex AI platform allows to use Google proprietary models and deploy other models for online predictions. `distilabel` integrates with Vertex AI trough `VertexAILLM` and `VertexAIEndpointLLM` classes.

To use one of these classes you will need to have configured the Google Cloud authentication using one of these methods:

- Settings `GOOGLE_CLOUD_CREDENTIALS` environment variable
- Using `gcloud auth application-default login` command
- Using `vertexai.init` Python SDK function from the `google-cloud-aiplatform` library before instantiating the `LLM`.


#### Proprietary models (Gemini and PaLM)

`VertexAILLM` allows to use Google proprietary models such as Gemini and PaLM. These models are served trough Vertex AI and its different APIs:

- **Gemini API**: which offers models from the Gemini family such as `gemini-pro` and `gemini-pro-vision` models. More information: [Vertex AI - Gemini API](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/gemini).
- **Text Generation API**: which offers models from the PaLM family such as `text-bison`. More information: [Vertex AI - PaLM 2 for text](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/text#model_versions).
- **Code Generation API**: which offers models from the PaLM family for code-generation such as `code-bison`. More information: [Vertex AI - Codey for code generation](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/code-generation).


```python
--8<-- "docs/snippets/technical-reference/llm/vertexaillm_generate.py"
```

#### Endpoints for online prediction

`VertexAIEndpointLLM` class allows to use a model deployed in a Vertex AI Endpoint for online prediction to generate text. Unlike the rest of `LLM`s classes which comes with a set of pre-defined arguments in its `__init__` method, `VertexAIEndpointLLM` requires to provide the generation arguments to be used in a dictionary that will pased to the `generation_kwargs` argument. This is because the generation parameters will be different and have different names depending on the Docker image deployed on the Vertex AI Endpoint.

```python
--8<-- "docs/snippets/technical-reference/llm/vertexaiendpointllm_generate.py"
```
### Anyscale

Anyscale Endpoints offers open source large language models (LLMs) as fully managed API endpoints. Interoperate with open source models as you would do it with OpenAI:

```python
--8<-- "docs/snippets/technical-reference/llm/anyscale_generate.py"
```

For the API reference visit [AnyscaleLLM][distilabel.llm.anyscale.AnyscaleLLM].

### MistralAI

[Mistral.ai](https://mistral.ai/), the company behind awesome Open Source models like [Mixtral 8x7B](https://mistral.ai/news/mixtral-of-experts/), offers their models in their AI platform. Visit their [available models](https://docs.mistral.ai/platform/endpoints/#generative-endpoints) and start creating `distilabel` datasets with them.

```python
--8<-- "docs/snippets/technical-reference/llm/mistralai_generate.py"
```

For the API reference visit [MistralAILLM][distilabel.llm.mistralai.MistralAILLM].

## `ProcessLLM` and `LLMPool`

By default, `distilabel` uses a single process, so the generation loop is usually bottlenecked by the model inference time and Python GIL. To overcome this limitation, we provide the `ProcessLLM` class that allows to load an `LLM` in a different process, avoiding the GIL and allowing to parallelize the generation loop. Creating a `ProcessLLM` is easy as:

```python
--8<-- "docs/snippets/technical-reference/llm/processllm.py"
```

1. The `ProcessLLM` returns a `Future` containing a list of lists of `LLMOutput`s.
2. The `ProcessLLM` needs to be terminated after usage. If the `ProcessLLM` is used by a `Pipeline`, it will be terminated automatically.

You can directly use a `ProcessLLM` as the `generator` or `labeller` in a `Pipeline`. Apart from that, there would be situations in which you would like to generate texts using several `LLM`s in parallel. For this purpose, we provide the `LLMPool` class:

```python
--8<-- "docs/snippets/technical-reference/llm/llmpool.py"
```
