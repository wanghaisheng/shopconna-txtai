# Workflow

![workflow](../images/workflow.png#only-light)
![workflow](../images/workflow-dark.png#only-dark)

Workflows are a simple yet powerful construct that takes a callable and returns elements. Workflows operate well with pipelines but can work with any callable object. Workflows are streaming and work on data in batches, allowing large volumes of data to be processed efficiently.

Given that pipelines are callable objects, workflows enable efficient processing of pipeline data. Transformers models typically work with smaller batches of data, workflows are well suited to feed a series of transformers pipelines. 

An example of the most basic workflow:

```python
workflow = Workflow([Task(lambda x: [y * 2 for y in x])])
list(workflow([1, 2, 3]))
```

This example simply multiplies each input value and returns a outputs via a generator. 

Workflows are run with Python or configuration. Examples of both methods are shown below.

## Example

A full-featured example is shown below in Python. This workflow transcribes a set of audio files, translates the text into French and indexes the data.

```python
from txtai.embeddings import Embeddings
from txtai.pipeline import Transcription, Translation
from txtai.workflow import FileTask, Task, Workflow

# Embeddings instance
embeddings = Embeddings({
    "path": "sentence-transformers/paraphrase-MiniLM-L3-v2",
    "content": True
})

# Transcription instance
transcribe = Transcription()

# Translation instance
translate = Translation()

tasks = [
    FileTask(transcribe, r"\.wav$"),
    Task(lambda x: translate(x, "fr"))
]

# List of files to process
data = [
  "US_tops_5_million.wav",
  "Canadas_last_fully.wav",
  "Beijing_mobilises.wav",
  "The_National_Park.wav",
  "Maine_man_wins_1_mil.wav",
  "Make_huge_profits.wav"
]

# Workflow that translate text to French
workflow = Workflow(tasks)

# Index data
embeddings.index((uid, text, None) for uid, text in enumerate(workflow(data)))

# Search
embeddings.search("wildlife", 1)
```

## Configuration-driven example

Workflows can be defined using Python as shown above but they can also run with YAML configuration.

```yaml
writable: true
embeddings:
  path: sentence-transformers/paraphrase-MiniLM-L3-v2
  content: true

# Transcribe audio to text
transcription:

# Translate text between languages
translation:

workflow:
  index:
    tasks:
      - action: transcription
        select: "\\.wav$"
        task: file
      - action: translation
        args: ["fr"]
      - action: index
```

```python
# Create and run the workflow
from txtai.app import Application

# Create and run the workflow
app = Application("workflow.yml")
list(app.workflow("index", [
  "US_tops_5_million.wav",
  "Canadas_last_fully.wav",
  "Beijing_mobilises.wav",
  "The_National_Park.wav",
  "Maine_man_wins_1_mil.wav",
  "Make_huge_profits.wav"
]))

# Search
app.search("wildlife")
```

The code above executes a workflow defined in the file `workflow.yml`. The API is used to run the workflow locally, there is minimal overhead running workflows in this manner. It's a matter of preference.

See the following links for more information.

- [Workflow Demo](https://huggingface.co/spaces/NeuML/txtai)
- [Workflow YAML Examples](https://huggingface.co/spaces/NeuML/txtai/tree/main/workflows)
- [Workflow YAML Guide](../api/configuration/#workflow)

## Methods

Workflows are callable objects. Workflows take an input of iterable data elements and output iterable data elements. 

### ::: txtai.workflow.Workflow.__init__
### ::: txtai.workflow.Workflow.__call__
### ::: txtai.workflow.Workflow.schedule

## More examples

See [this link](../examples/#workflows) for a full list of workflow examples.
