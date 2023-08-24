# ChunkGPT: Summarization for Large Texts

![ChunkGPT Logo](https://raw.githubusercontent.com/dimays/chunkgpt/main/chunkGPT-logo.png)

ChunkGPT is a lightweight Python library designed to assist users in summarizing long texts that exceed the token limit of OpenAI's GPT-based models. It breaks down large texts into manageable chunks, generates summaries for each chunk, and then combines these summaries into a coherent final summary.

## Installation

You can install ChunkGPT using pip:

```bash
pip install chunkgpt
```

## Dependencies

ChunkGPT depends on the following Python libraries, which are both included in the install:

| Library | Version | Purpose |
| ------- | ------- | ------- |
| [`openai`](https://pypi.org/project/openai/) | 0.27.8 | Accessing OpenAI and managing OpenAI API calls. |
| [`tiktoken`](https://pypi.org/project/tiktoken/) | 0.4.0 | Estimating token counts for a given string in order to appropriately apply chunk limits and determine OpenAI cost. |

## Usage

It's easy to use ChunkGPT! Follow these general steps to generate summaries for your lengthy texts:

```python
from chunkgpt.chunkgpt import Chunker

# Initialize the Chunker object
chunker = Chunker()

# Load your text to be summarized
with open('myfile.txt', 'r') as infile:
    mytext = infile.read()

# Generate a summary for the text
summary = chunker.summarize(mytext)

print(summary['result'])
```

## Advanced Usage

The behavior of ChunkGPT can be modified to suit the needs of your project. Below are some examples of advanced ways of modifying ChunkGPT.

### Customize the system message

The default `SYSTEM_MSG` is the result of several rounds of trial and error during development of ChunkGPT, and has been shown to produce relatively reliable and detailed results.

```python
SYSTEM_MSG = """You are SUMMIFY, a specialized AI assistant \
purpose-built to read long pieces of text and produce a concise \
summary that greatly reduces the overall length of the text \
without leaving out any critical details.

Users will hand you an entire document, or just a portion of the \
document, and it is your job to summarize the content in as few \
words as possible, while conveying the essential meaning of the text.

Do NOT refer to "the text" as an object. Instead, paraphrase the document you are \
given so that your summary can essentially replace the document 1-for-1, \
but with a significantly shorter length.

The summary MUST be concise.
The summary MUST be comprehensive.
Include ALL KEY POINTS, and exclude any unnecessary excess.
"""
```

However, if you are interested in experimenting with different system messages in order to tweak the quality or format of the final summary, you can define a `custom_system_msg` when initializing the `Chunker` object.

```python
from chunkgpt.chunkgpt import Chunker

sys_msg = "Summarize the text you are given to the best of your ability."

chunker = Chunker(custom_system_msg=sys_msg)
```

### Explicitly define your OpenAI API Key

By default, ChunkGPT look for an environment variable called `OPENAI_API_KEY`. If you choose to manage your API keys in a different way, you may define your API Key when initializing the `Chunker` object.

```python
from chunkgpt.chunkgpt import Chunker

my_key = 'myApiKey1234'

chunker = Chunker(api_key=my_key)
```

### Use a different model

ChunkGPT supports all `gpt-3.5` and `gpt-4` models from OpenAI.

```python
from chunkgpt.chunkgpt import Chunker

chunker = Chunker(model='gpt-4')
```

### Tweak the temperature

By default, ChunkGPT usees a temperature of 0. Temperatures closer to 0 will be less variable, while temperatures closer to 2 can be so variable as to be incomprehensible.

```python
from chunkgpt.chunkgpt import Chunker

chunker = Chunker(temperature=0.5)
```

### Tweak the token limits for chunks, their overlap, and the resulting summaries

`max_chunk_length` represents the max number of tokens used for each text chunk. Higher values may be more performant, but lower values may be more precise.

`chunk_overlap` represents the number of tokens that overlap between consecutive segments. Lower values may be more performant, but higher values may smooth over transitions between chunks. This value **must be smaller** than `max_chunk_length`.

`summary_length` represents the max number of tokens enforced for the output of the model. If your final summary result is getting cut off, you may want to consider increasing this value.

```python
from chunkgpt.chunkgpt import Chunker

chunker = Chunker(max_chunk_length=4000,
                  summary_length=200,
                  chunk_overlap=0)
```

*Note:* `summary_length` + `max_chunk_length` + the base number of tokens used by system and user prompts **must be smaller** than the `token_limit` of the model being used.

### Access Intermediate Summaries

If you wish to access the intermediate summaries, you can reference the `'chunks'` dictionary included in the response of `Chunker.summarize()`.

```python
from chunkgpt.chunkgpt import Chunker

chunker = Chunker()

text = "..."

summary = chunker.summarize(text)

for chunk in summary['chunks']:
    print(f"CHUNK {chunk}:")
    print("Original Text:")
    print(summary['chunks']['input'])
    print("Summary:")
    print(summary['chunks']['output'])
```

### Skip Final Reduction & Summarization Step

By default, `Chunker.summarize()` will reduce the final output until its length, combined with the system prompt and `summary_length`, falls within the overall `token_limit`.

However, if you find that this final summary is too short, and you want a more detailed summary, you can change this behavior by changing the `final_step` parameter.

This paramter is set to `'summarize'` by default, but it also accepts `'combine'`, which instructs ChunkGPT to skip this final summary step and instead return a combination of all intermediate summaries.

```python
from chunkgpt.chunkgpt import Chunker

chunker = Chunker()

text = "..."

summary = chunker.summarize(text, final_step='combine')
```

*Note: Because this final step is a simple concatenation of separately summarized excerpts, this option may lead to a more detailed, although possibly less coherent, final output.*

## `Chunker` Object

The `Chunker` object serves as the core mechanism responsible for breaking down large texts into smaller, manageable chunks, generating summaries for each of these chunks, and ultimately combining these chunk summaries into a coherent final summary.

### Attributes

You can customize the `Chunker` object's behavior by including any of the following attributes as keyword arguments when initializing the object:

| Attribute | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `model` | `str` | 'gpt-3.5-turbo' | Name of the OpenAI GPT model |
| `api_key` | `str` | `OPENAI_API_KEY` environment variable | Your [OpenAI API key](https://platform.openai.com/account/api-keys) |
| `max_chunk_length` | `int` | 2048 | Maximum number of tokens for each text chunk (including any overlap from the previous chunk) |
| `chunk_overlap` | `int` | 50 | Number of tokens that overlap between consecutive chunks |
| `summary_length` | `int` | 1024 | Maximum number of tokens for each summary (both intermediate summaries and final summary) |
| `temperature` | `float` | 0.0 | Floating point number between 0 and 2, represents the [`temperature`](https://platform.openai.com/docs/api-reference/chat/create#chat/create-temperature) parameter used for each OpenAI completion. |
| `custom_system_msg` | `str` | `None` | Alternative system message to use in place of the default `SYSTEM_MSG`. |

### Methods

#### `Chunker.summarize(text)`

Generate and return a summary for a given text, along with additional data relevant to the summarizing process as a whole.

**Arguments**

| Argument | Type | Description |
| -------- | ---- | ----------- |
| `text`   | `str` | The text to be chunked and summarized. |

This method takes a string as input, breaks it into smaller chunks if necessary, generates summaries for each chunk, and then combines these summaries into one coherent final summary.

If the combined length of the input text, ChunkGPT prompt, and the expected completion length does not exceed the model's token limit, a single summary will be produced.

If the combined length of the intermediate summaries exceeds the model's token limit, the summaries will be chunked again, reducing the overall size of the summaries until the full summarized contents falls within the token limit.

This method returns a dictionary with the following keys:

| Key | Type | Description |
| --------- | ---- | ----------- |
| `'original'` | `str` | The original, unaltered text represented as a string |
| `'result'` | `str` | The final resulting summary, comprehensive of all intermediate summaries. |
| `'chunks'` | `dict` | A dictionary containing each chunk of the text, along with OpenAI's chat completion for that chunk. Each element in this dictionary is itself a dictionary containing the keys `'input'`, representing the chunk text, and `'output'`, representing the GPT-based Chat Completion for this chunk. |
| `'intermediate_steps'` | `list` | A list containing a description of each intermediate step taken in the summarization process. |
| `'total_tokens'` | `int` | The total number of tokens consumed to generate this summary, including prompts and completions. |
| `'cost'` | `float` | The estimated cost (in cents) for this completion, based on the model's price and the total tokens used |

### Helper Methods

These methods are each responsible for a minor task within the summarization process. They are included here for reference, but you should rarely, if ever, need to access them directly.

#### `Chunker._chunk(text)`

Returns list of strings, each string representing a chunk of the original text that needs to be summarized. If the original text plus the system message and expected output is within the token limit, returns a single-item list containing the input text.

#### `Chunker._tokenize(text)`

Returns an encoded list of tokens from the given text string.

#### `Chunker._decode(tokens)`

Returns a decoded string from a list of tokens.

#### `Chunker._get_token_limit()`

Returns the token limit for each gpt-3.5 and gpt-4 model, current as of August 13, 2023.

#### `Chunker._get_price_per_token()`

Returns the price per token in cents for each gpt-3.5 and gpt-4 model, current as of August 13, 2023.

#### `Chunker._get_complete_token_count(text)`

Returns total number of tokens for a given input text, including the system prompt and expected number of tokens needed for the completion output.

#### `Chunker._get_base_token_count(text)`

Returns total number of tokens used by the system message and user message (prior to formatting with the input text).

#### `Chunker._construct_messages(text)`

Constructs a list of messages to submit to OpenAI for chat completion.

#### `Chunker._get_completion(text, num_retries=3)`

Uses the designated system_msg and the default USER_MSG template to get an OpenAI chat completion for the provided text.

If the completion fails, ChunkGPT will retry up to 3 times, with a delay of 1 second between retries.

## Examples

The [Github repo](https://www.github.com/dimays/chunkgpt) for ChunkGPT inclues long sample .txt files to test out in the `examples/` directory. Each file, when added to the existing system and user message templates, exceeds the token limit for all available OpenAI GPT models (as of August 13, 2023).

You can test out the utility of this library against large texts by using these sample texts and employing any GPT models or Chunker object attributes you wish to experiment with.

For example, see this script summarizing the 2023 IPCC Summary for Policymakers:
```python
from chunkgpt.chunkgpt import Chunker

chunker = Chunker()

with open('./examples/ipcc_summary_2023.txt') as infile:
        text = infile.read()

summary = chunker.summarize(text)

output_str = f"""
FINAL SUMMARY
{summary['result']}

TOTAL TOKENS IN ORIGINAL DOC
{len(chunker._tokenize(summary['original']))}

TOTAL TOKENS USED
{summary['total_tokens']}

MODEL USED
{chunker.model}

ESTIMATED COST
${round(summary['cost'] / 100, 2)}
"""

print(output_str)
```

Running the script above, you should expect the output to look something like this:
```
Initialized Chunker.
Split text into 16 chunks.
Got completion for chunk 1.
Got completion for chunk 2.
Got completion for chunk 3.
Got completion for chunk 4.
Got completion for chunk 5.
Got completion for chunk 6.
Got completion for chunk 7.
Got completion for chunk 8.
Got completion for chunk 9.
Got completion for chunk 10.
Got completion for chunk 11.
Got completion for chunk 12.
Got completion for chunk 13.
Got completion for chunk 14.
Got completion for chunk 15.
Got completion for chunk 16.
Split text into 2 chunks.
Got completion for chunk 1.
Got completion for chunk 2.
Got completion for combined summaries.
Reduced summary from 4462 to 2052.


FINAL SUMMARY
The IPCC's Climate Change 2023 Synthesis Report Summary for Policymakers emphasizes the urgent need for action to address greenhouse gas emissions and adapt to climate change. Key points include the role of human activities in global warming, the increase in global surface temperatures, the impact of well-mixed greenhouse gases, and the highest levels of CO2, methane, and nitrous oxide in millions of years. The report highlights the adverse impacts of climate change on ecosystems, water availability, food production, health, and infrastructure, as well as the barriers to adaptation. It discusses the projected increase in CO2 emissions, the gaps in current mitigation efforts, the costs and deployment of renewable energy technologies, and the uneven policy coverage for climate change. The report emphasizes the escalating risks and impacts of climate change, the need for adaptation, and the consequences for various regions and ecosystems. It underscores the importance of limiting global warming, the effectiveness of adaptation options, and the necessity of reaching net zero CO2 emissions. Delayed climate action will lead to increased global warming and more losses and damages, while accelerated climate action can provide co-benefits such as improved health and agricultural productivity. Ambitious mitigation pathways will require significant changes in economic structures and can be moderated by fiscal and regulatory reforms. Feasible, effective, and low-cost options for mitigation and adaptation are available, with differences across systems and regions. Net zero CO2 energy systems require a substantial reduction in fossil fuel use and widespread electrification. Rapid and far-reaching transitions across all sectors and systems are necessary for deep and sustained emissions reductions. Maintaining biodiversity and ecosystem services globally depends on effective conservation of approximately 30% to 50% of Earth's land, freshwater, and ocean areas. Cooperation with Indigenous Peoples and local communities is integral to successful adaptation and mitigation. Policy support for climate action is influenced by various actors in civil society, including businesses, youth, women, labor, media, Indigenous Peoples, and local communities. Finance, technology, and international cooperation are critical enablers for accelerated climate action, and closing the gaps in financial support for adaptation and mitigation is crucial, especially in developing countries. International cooperation is essential for ambitious climate action, including mobilizing finance, aligning finance flows with ambition levels, and supporting Nationally Determined Contributions (NDCs) and technology development and deployment.

KEY POINTS:
- Urgent action is needed to mitigate greenhouse gas emissions and adapt to climate change.
- Human activities are causing global warming, leading to an increase in global surface temperatures.
- Well-mixed greenhouse gases, such as CO2, methane, and nitrous oxide, are at their highest levels in millions of years.
- Climate change has adverse impacts on ecosystems, water availability, food production, health, and infrastructure.
- There are barriers to adaptation and gaps in current mitigation efforts.
- CO2 emissions are projected to increase, and renewable energy technologies need to be deployed.
- Climate change poses escalating risks and impacts, with consequences for various regions and ecosystems.
- Limiting global warming, effective adaptation options, and reaching net zero CO2 emissions are crucial.
- Delayed climate action will lead to increased global warming and more losses and damages.
- Accelerated climate action can provide co-benefits, such as improved health and agricultural productivity.
- Ambitious mitigation pathways require significant changes in economic structures and can be moderated by fiscal and regulatory reforms.
- Feasible, effective, and low-cost options for mitigation and adaptation are available, with differences across systems and regions.
- Net zero CO2 energy systems require a substantial reduction in fossil fuel use and widespread electrification.
- Rapid and far-reaching transitions across all sectors and systems are necessary for deep and sustained emissions reductions.
- Maintaining biodiversity and ecosystem services globally depends on effective conservation of land, freshwater, and ocean areas.
- Cooperation with Indigenous Peoples and local communities is integral to successful adaptation and mitigation.
- Policy support for climate action is influenced by various actors in civil society.
- Finance, technology, and international cooperation are critical enablers for accelerated climate action.
- Closing the gaps in financial support for adaptation and mitigation is crucial, especially in developing countries.
- International cooperation is essential for ambitious climate action, including mobilizing finance, aligning finance flows, and supporting Nationally Determined Contributions (NDCs) and technology development and deployment.

TOTAL TOKENS IN ORIGINAL DOCUMENT
31849

TOTAL TOKENS USED
44216

MODEL USED
gpt-3.5-turbo

ESTIMATED COST
$0.07
```

## Contributing
Contributions to improve ChunkGPT are welcome! If you find any issues or want to suggest enhancements, please submit an issue or pull request in the [GitHub repository](https://www.github.com/dimays/chunkgpt).

## License
This project is licensed under the [MIT License](https://www.github.com/dimays/chunkgpt/LICENSE).

## Acknowledgements
ChunkGPT was inspired by the need to summarize large texts efficiently using limited-token models. We thank the open-source community for their valuable contributions.

---

### Happy summarizing with ChunkGPT!