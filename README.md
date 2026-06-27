<div align="center">

# Tradutor de linguagem de sinais ⠎⠇⠞

</div>

---

## Overview

A linguagem de sinais consiste em gestos e expressões usadas principalmente por pessoas com deficiência auditiva para lhes ajudarem a se comunicar. Esse projeto é um trabalho para ajudar a diminuir a distância entre a comunidade de deficiêntes auditivos e pessoas sem essa deficiência utilizando IA.

Isso é uma API e Framework desenvolvida em Python construida como um tradutor de linguagem de sinais...

<div></div>
Um grande obstáculo é a falta de conjuntos de dados (globais e regionais) e de estruturas que os engenheiros de aprendizagem profunda e os programadores de software possam utilizar para criar produtos úteis para a comunidade-alvo. Este projeto visa potenciar a tradução de língua gestual, fornecendo componentes, ferramentas, conjuntos de dados e modelos robustos, tanto para a conversão de língua gestual para texto como de texto para língua gestual. Tem como objetivo facilitar a criação de tradutores de língua gestual para qualquer região, abrindo simultaneamente o caminho para a normalização da língua gestual.

Diferentes de muitos projetos esse permite traduzir frases inteiras e não apenas o alfabeto.

### Solução

Esse pacote vem com um "_extensible rule-based_" sistema de tradução de texto para sinais que pode ser usado para treinar modelos de "_Deep Learning_" com seus dados para ambos sinais para texto como texto para sinais.

Para criar um sistema de tradução baseado em regras para a sua língua regional, pode herdar as classes `TextLanguage` e `SignLanguage` e passá-las como argumentos à classe `ConcatenativeSynthesis`. Para escrever textos de exemplo com palavras suportadas, pode utilizar os nossos modelos linguísticos. Em seguida, pode utilizar esse sistema para afinar os nossos modelos de aprendizagem profunda.

Esse projeto foi baseado em um já veemente documentado, veja a documentação aqui:
<kbd>[documentation](https://sign-language-translator.readthedocs.io)</kbd>

### Componentes maiores

<ol>
<li>
<div>
Linguagem de sinais para texto

1. Extrai detalhes de video
   1. veja o [`slt.models.video_embedding`](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/tree/main/sign_language_translator/models/video_embedding) sub pacote e p [`$ slt embed`](https://slt.readthedocs.io/en/latest/#embed-videos) comando.
   2. Atualmente o Mediapipe 3D landmarks está sendo usado para deep learning.
2. Transcreve e traduz sinais em multiplas linguagens de texto.
3. Para treinar palavra por palavra em tarefas de redação sobre glossário, utilize também um conjunto de dados sintético criado através da concatenação de sinais para cada palavra de um texto. (veja [`slt.models.ConcatenativeSynthesis`](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/blob/main/sign_language_translator/models/text_to_sign/concatenative_synthesis.py))
4. Ajustar uma rede neural, como, por exemplo, uma proveniente de [`slt.models.sign_to_text`](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/tree/main/sign_language_translator/models/sign_to_text) ou o codificador de qualquer modelo seq2seq multilingue, no seu conjunto de dados.

</div>
</li>

<li>
<details>
<summary>
Text to Sign Language
</summary>

There are two approaches to this problem:

1. Rule Based Concatenation

   1. Label a Sign Language Dictionary with all word tokens that can be mapped to those signs. See our mapping format [here](https://github.com/sign-language-translator/sign-language-datasets/blob/main/parallel_texts/pk-dictionary-mapping.json).
   2. Parse the input text and play appropriate video clips for each token.
      1. Build a text processor by inheriting `slt.languages.TextLanguage` (see [`slt.languages.text`](https://github.com/sign-language-translator/sign-language-translator/tree/main/sign_language_translator/languages/text) sub-package for details)
      2. Map the text grammar & words to sign language by inheriting `slt.languages.SignLanguage` (see [`slt.languages.sign`](https://github.com/sign-language-translator/sign-language-translator/tree/main/sign_language_translator/languages/sign) sub-package for details)
      3. Use our rule-based model [`slt.models.ConcatenativeSynthesis`](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/text_to_sign/concatenative_synthesis.py) for translation.
   3. It is faster but the **word sense has to be disambiguated** in the input. See the deep learning approach to automatically handle ambiguous words & **words not in dictionary**.

2. Deep learning (seq2seq)
   1. Either generate the sequence of filenames that should be concatenated <!-- TODO: `slt.models.mGlossBART` -->
      1. you will need a [parallel corpus](https://github.com/sign-language-translator/sign-language-datasets/blob/main/parallel_texts/pk-sentence-mapping.json) of normal text sentences against sign language gloss (sign sequence written word-for-word)
   2. Or synthesize the signs directly by using a pre-trained multilingual text encoder and
      1. a GAN or diffusion model or decoder to synthesize a sequence of pose vectors (`shape = (time, num_landmarks * num_coordinates)`) <!-- TODO: `slt.models.SignPoseGAN` -->
         1. Move an Avatar with those pose vectors (Easy) <!-- TODO: `slt.models.Avatar` -->
         2. Use motion transfer to generate a video (Medium) <!-- TODO: `slt.models.SignMotionTransfer` -->
         3. Synthesize a video frame for each vector (Difficult) <!-- TODO: `slt.models.DeepPoseToImage` -->
      2. a video synthesis model (Very Difficult) <!-- TODO: `slt.models.DeepSignVideoGAN` -->

</details>
</li>

<li>
<details>
<summary><b>
Language Processing
</b></summary>

1. Sign Processing
   - 3D world landmarks extraction with Mediapipe.
   - Pose Visualization with matplotlib.
   - Pose transformations (data augmentation) with scipy.
2. Text Processing
   - Normalize text input by substituting unknown characters/spellings with supported words.
   - Disambiguate context-dependent words to ensure accurate translation.
     "spring" -> ["spring(water-spring)", "spring(metal-coil)"]
   - Tokenize text (word & sentence level).
   - Classify tokens and mark them with Tags.

</details>
</li>

<li>
<details>
<summary><b>
Datasets
</b></summary>

For our datasets & conventions, see the [_sign-language-datasets_ repo](https://github.com/sign-language-translator/sign-language-datasets) and its [releases](https://github.com/sign-language-translator/sign-language-datasets/releases).
See this [documentation](https://slt.readthedocs.io/en/latest/datasets.html) for more on building a dataset of Sign Language videos (or motion capture gloves' output features).

**Your data should include**:

1. A word level Dictionary (Videos of individual signs & corresponding Text tokens (words & phrases))
2. Replications of the dictionary. (Set up multiple syncronized cameras and record random people performing the dictionary videos. ([notebook](https://colab.research.google.com/github/sign-language-translator/notebooks/blob/main/data_collection/clip_extractor.ipynb)))
3. Parallel sentences
   1. Normal text language sentences against sign language videos. (Use our Language Models to generate sentences composed of dictionary words.)
   2. Normal text language sentences against the [text gloss](https://github.com/sign-language-translator/sign-language-datasets#glossary) of the corresponding sign language sentence.
   3. Sign language sentences against their text gloss
   4. Sign language sentences against translations in multiple text languages
4. Grammatical rules of the sign language
   1. Word order (e.g. SUBJECT OBJECT VERB TIME)
   2. Meaningless words (e.g. "the", "am", "are")
   3. Ambiguous words (e.g. spring(coil) & spring(water-fountain))

**Try to incorporate**:

1. Multiple camera angles
2. Diverse performers to capture all _accents_ of the signs
3. Uniqueness in labeling of word tokens
4. Variations in signs for the same concept

Try to capture variations in signs in a scalable and diversity accommodating way and enable advancing sign language standardization efforts.

</details>
</li>
</ol>

### Goals

1. Enable **integration** of sign language into existing applications.
2. Assist construction of **custom** solutions for resource poor sign langauges.
3. Improve **education** quality for the deaf and elevate literacy rates.
4. Promote communication **inclusivity** of the hearing impaired.
5. Establish a framework for sign language **standardization**.

## How to install the package

```bash
pip install sign-language-translator
```

<details>
<summary>Editable mode (<code>git clone</code>):</summary>

The package ships with some optional dependencies as well (e.g. deep_translator for synonym finding and mediapipe for a pretrained pose extraction model). Install them by appending `[all]`, `[full]`, `[mediapipe]` or `[synonyms]` to the project name in the command (e.g `pip install sign-langauge-translator[full]`).

```bash
git clone https://github.com/sign-language-translator/sign-language-translator.git
cd sign-language-translator
pip install -e ".[all]"
```

```bash
pip install -e git+https://github.com/sign-language-translator/sign-language-translator.git#egg=sign_language_translator
```

</details>

## Usage

Head over to [slt.**readthedocs**.io](https://slt.readthedocs.io) to see the detailed usage in Python, CLI and gradio GUI.
See the [_test cases_](https://github.com/sign-language-translator/sign-language-translator/blob/main/tests) or the [_notebooks_ repo](https://github.com/sign-language-translator/notebooks) to see the internal code in action.

### Web GUI

Individual models deployed on HuggingFace Spaces:

[![HuggingFace Spaces](https://img.shields.io/badge/text%20to%20sign-ConcatenativeSynthesis%2BLM-mediumslateblue)](https://huggingface.co/spaces/sltAI/ConcatenativeSynthesis)

<!-- #FF3270 #FFD702 #861FFF #097EFF #10B981 #3B82F6 #6366F1 #F59E0B #EF4444 -->

### Python

```python
import sign_language_translator as slt

model = slt.models.ConcatenativeSynthesis(
   text_language="urdu", sign_language="pk-sl", sign_format="video" )

text = "this-very-good-is"
sign = model.translate(text)
sign.show()

model.sign_format = slt.SignFormatCodes.LANDMARKS
model.sign_embedding_model = "mediapipe-world"

# ==== English ==== #
model.text_language = slt.languages.text.English()
sign_2 = model.translate("This is an apple.")
sign_2.save("this-is-an-apple.csv", overwrite=True)

# ==== Hindi ==== #
model.text_language = slt.TextLanguageCodes.HINDI
sign_3 = model.translate("this-very-good-is")
```

</br>

```python
import sign_language_translator as slt

sign = slt.Video.load_asset("your name what is?")
sign.show_frames_grid()

embedding_model = slt.models.MediaPipeLandmarksModel()
embedding = embedding_model.embed(sign.iter_frames())

slt.Landmarks(embedding.reshape((-1, 75, 5)),
              connections="mediapipe-world"  ).show()
```

```python
help(slt.languages.SignLanguage)
help(slt.languages.text.Urdu)
help(slt.models.ConcatenativeSynthesis)
```

### Linhas de Comando

```bash
$ slt

Usage: slt [OPTIONS] COMMAND [ARGS]...
   Sign Language Translator (SLT) command line interface.
   Documentation: https://sign-language-translator.readthedocs.io
Options:
  --version  Show the version and exit.
  --help     Show this message and exit.
Commands:
  assets     Assets manager to download & display Datasets & Models.
  complete   Complete a sequence using Language Models.
  embed      Embed Videos Using Selected Model.
  translate  Translate text into sign language or vice versa.
```

**Generate training examples**: write a sentence with a language model and synthesize a sign language video from it with a single command:

```bash
slt translate --model-code rule-based --text-lang urdu --sign-lang pk-sl --sign-format video \
"$(slt complete '<' --model-code urdu-mixed-ngram --join '')"
```

## Languages

<details>
<summary><b>Text Languages</b></summary>

Available Functions:

- Text Normalization
- Tokenization (word, phrase & sentence)
- Token Classification (Tagging)
- Word Sense Disambiguation

| Name                                                                                                                                         | Vocabulary         | Ambiguous tokens | Signs |
| -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ | ---------------- | ----- |
| [English](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/text/english.py) | 1591 words+phrases | 167              | 776   |
| [Urdu](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/text/urdu.py)       | 2080 words+phrases | 227              | 776   |
| [Hindi](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/text/hindi.py)     | 137 words+phrases  | 5                | 84    |

</details>

<details>
<summary><b>Sign Languages</b></summary>

Available Functions:

- Word & phrase mapping to signs
- Sentence restructuring according to grammar
- Sentence simplification (drop stopwords)

| Name                                                                                                                                                                       | Vocabulary | Dataset  |                                    Parallel Corpus                                     |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | -------- | :------------------------------------------------------------------------------------: |
| [Pakistan Sign Language](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/sign/pakistan_sign_language.py) | 776        | 23 hours | [details](https://github.com/sign-language-translator/sign-language-datasets#datasets) |

</details>

## Models

<details>
<summary><b>Translation</b>: Text to sign Language</summary>

<!-- [Available Trained models]() -->

| Name                                                                                                                                                                              | Architecture        | Description                                                                                                                                            | Input  | Output                     | Web Demo                                                                                                                                                                  |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ------ | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Concatenative Synthesis](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/text_to_sign/concatenative_synthesis.py) | Rules + Hash Tables | The Core Rule-Based translator mainly used to synthesize translation dataset.<br/>Initialize it using TextLanguage, SignLanguage & SignFormat objects. | string | slt.Video \| slt.Landmarks | [![Open in Spaces](https://huggingface.co/datasets/huggingface/badges/resolve/main/open-in-hf-spaces-sm.svg)](https://huggingface.co/spaces/sltAI/ConcatenativeSynthesis) |

<!--                                                                                                                                                                              | [pose-gen]()        | Encoder-Decoder Transformers (Seq2Seq)                                                                                                              | Generates a sequence of pose vectors conditioned on input text. | torch.Tensor<br/>(batch, token_ids) | torch.Tensor<br/>(batch, n_frames, n_landmarks*3) |  | -->

</details>

<!-- <details>
<summary>Translation: Sign Language to Text</summary>

[Available Trained models]()

| Name        | Architecture                                | Description                                                                                                  | Input format                                             | Output format                       |
| ----------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------- | ----------------------------------- |
| [gesture]() | CNN+Encoder - Decoder Transformer (seq2seq) | Encodes the pose vectors depicting sign language sentence and generates text conditioned on those encodings. | torch.Tensor<br/>(batch, n_frames=1000, n_landmarks * 3) | torch.Tensor<br/>(batch, token_ids) |
</details> -->

<!--
<details>
<summary>Video: Synthesis/Generation</summary>

[Available Trained models]()

| Name        | Architecture                                | Description                                                                                                  | Input format                                             | Output format                       |
| ----------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------- | ----------------------------------- |
| [gesture]() | CNN+Encoder - Decoder Transformer (seq2seq) | Encodes the pose vectors depicting sign language sentence and generates text conditioned on those encodings. | torch.Tensor<br/>(batch, n_frames=1000, n_landmarks * 3) | torch.Tensor<br/>(batch, token_ids) |
</details>
-->

<details>
<summary><b>Sign Embedding/Feature extraction</b>:</summary>

<!-- [Available Trained models]() -->

| Name                                                                                                                                                                                                 | Architecture                                                                                                               | Description                                                                                       | Input format                                                 | Output format                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ | --------------------------------------------- |
| [MediaPipe Landmarks<br>(Pose + Hands)](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/video_embedding/mediapipe_landmarks_model.py) | CNN based pipelines. See Here: [Pose](https://arxiv.org/pdf/2006.10204.pdf), [Hands](https://arxiv.org/pdf/2006.10214.pdf) | Encodes videos into pose vectors (3D world or 2D image) depicting the movements of the performer. | List of numpy images<br/>(n_frames, height, width, channels) | torch.Tensor<br/>(n_frames, n_landmarks \* 5) |

</details>

<details>
<summary><b>Data generation</b>: Language Models</summary>

[Available Trained models](https://github.com/sign-language-translator/sign-language-datasets/releases/tag/v0.0.1)

| Name                                                                                                                                                                                             | Architecture                    | Description                                                                                         | Input format                                                | Output format                                                                 |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------- | --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------------------------- |
| [N-Gram Langauge Model](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/ngram_language_model.py)                  | Hash Tables                     | Predicts the next token based on learned statistics about previous N tokens.                        | List of tokens                                              | (token, probability)                                                          |
| [Transformer Language Model](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/transformer_language_model/model.py) | Decoder-only Transformers (GPT) | Predicts next token using query-key-value attention, linear transformations and soft probabilities. | torch.Tensor<br/>(batch, token_ids)<br/><br/>List of tokens | torch.Tensor<br/>(batch, token_ids, vocab_size)<br/><br/>(token, probability) |

</details>

<details>
<summary><b>Text Embedding</b>:</summary>

[Available Trained models](https://github.com/sign-language-translator/sign-language-datasets/releases/)

| Name                                                                                                                                                                  | Architecture | Description                                                                                                             | Input format | Output format             |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------- | ------------ | ------------------------- |
| [Vector Lookup](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/text_embedding/vector_lookup_model.py) | HashTable    | Finds token index and returns the coresponding vector. Tokenizes sentences and computes average vector of known tokens. | string       | torch.Tensor<br/>(n_dim,) |

</details>

<!--
## Servers

| Name (repository) | Framework | Docker  | Status       |
| ----------------- | --------- | ------- | ------------ |
| [slt-models]()    | FastAPI   | [url]() | Coming Soon! |
| [slt-backend]()   | Django    | [url]() | Coming Soon! |
| [slt-frontend]()  | React     | [url]() | Coming Soon! |

You can interact with the live version of the above servers at [something.com](https://www.something.com)
-->

## How to Build a Translator for Sign Language

To create your own sign language translator, you'll need these essential components:

<ol>
<li>
<details>
<summary>Data Collection</summary>

1.  Gather a collection of [dictionary videos](https://github.com/sign-language-translator/sign-language-datasets/releases/tag/v0.0.2) (word level) featuring individuals performing sign language gestures. These can be obtained from schools & organizations for the deaf. You should record multiple people perform the same sign to capture various _accents_ of the sign. Set up multiple cameras in different locations in parallel to further augment the data.
2.  Prepare a [JSON file](https://github.com/sign-language-translator/sign-language-datasets/blob/main/parallel_texts/pk-dictionary-mapping.json) that maps dictionary video file names to corresponding text language words & phrases that are synonymous with the gestures.
3.  Prepare a synthetic data [parallel corpus](https://github.com/sign-language-translator/sign-language-datasets/blob/main/parallel_texts/pk-synthetic-sentence-mapping.json) containing text language sentences and sequences of sign language video filenames. You can use langauge models to generate these sentences & sequences.
4.  Prepare a dataset of sign language [sentence videos](https://github.com/sign-language-translator/sign-language-datasets/releases/tag/v0.0.3) that are labeled with [translations & glosses](https://github.com/sign-language-translator/sign-language-datasets/blob/main/parallel_texts/pk-sentence-mapping.json) in multiple text languages.

</details>
</li>

<li>
<details>
<summary>Language Processing</summary>

1.  Implement a subclass of `slt.languages.TextLanguage`:
    - Tokenize your text language and assign appropriate tags to the tokens for streamlined processing.
2.  Create a subclass of `slt.languages.SignLanguage`:
    - Map text tokens to video filenames using the provided JSON data.
    - Rearrange the sequence of video filenames to align with the grammar and structure of sign language.

</details>
</li>

<li>
<details>
<summary>Rule-Based Translation</summary>

1.  Pass instances of your classes from the previous step to `slt.models.ConcatenativeSynthesis` class to obtain a rule-based translator object.
2.  Construct sentences in your text language and use the rule-based translator to generate sign language translations. (You can use our language models to generate such texts.)

</details>
</li>

<li>
<details>
<summary>Deep Learning Model Fine-Tuning</summary>

1.  Utilize the (synthetic & real) sign language videos and corresponding text sentences from the previous step.
2.  Apply our training pipeline to fine-tune a chosen model for improved accuracy and translation quality.

</details>
</li>
</ol>

Veja o `código` em [Build Custom Translator section in ReadTheDocs](https://sign-language-translator.readthedocs.io/en/latest/#building-custom-translators).

## Hierarquia do projeto

<details>
<summary><b style="font-size:large;"><code>sign-language-translator</code></b></summary>

<pre>
├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/README.md">README.md</a>
├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/pyproject.toml">pyproject.toml</a>
├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/requirements.txt">requirements.txt</a>
├── <b>docs</b>
│   └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/docs">*</a>
├── <b>tests</b>
│   └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/tests">*</a>
│
└── <b style="font-size:large;">sign_language_translator</b>
    ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/cli.py">cli.py</a> <sub><sup>`> slt` command line interface</sup></sub>
    ├── <i><b>assets</b></i> <sub><sup>(auto-downloaded)</sup></sub>
    │   └── <a href="https://github.com/sign-language-translator/sign-language-datasets">*</a>
    │
    ├── <b>config</b>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/config/assets.py">assets.py</a> <sub><sup>download, extract and remove models & datasets</sup></sub>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/config/colors.py">colors.py</a> <sub><sup>named RGB tuples for visualization</sup></sub>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/config/enums.py">enums.py</a> <sub><sup>string short codes to identify models & classes</sup></sub>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/config/settings.py">settings.py</a> <sub><sup>global variables in repository design-pattern</sup></sub>
    │   ├── <i><a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/config/urls.json">urls.json</a></i>
    │   └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/config/utils.py">utils.py</a>
    │
    ├── <b>languages</b>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/utils.py">utils.py</a>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/vocab.p">vocab.py</a> <sub><sup>reads word mapping <a href="https://github.com/sign-language-translator/sign-language-datasets/tree/main/parallel_texts">datasets</a></sup></sub>
    │   ├── sign
    │   │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/sign/mapping_rules.py">mapping_rules.py</a> <sub><sup>strategy design-pattern for word to sign mapping</sup></sub>
    │   │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/sign/pakistan_sign_language.py">pakistan_sign_language.py</a>
    │   │   └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/sign/sign_language.py">sign_language.py</a> <sub><sup>Base class for text to sign mapping and sentence restructuring</sup></sub>
    │   │
    │   └── text
    │       ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/text/english.py">english.py</a>
    │       ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/text/hindi.py">hindi.py</a>
    │       ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/text/text_language.py">text_language.py</a> <sub><sup>Base class for text normalization, tokenization & tagging</sup></sub>
    │       └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/text/urdu.py">urdu.py</a>
    │
    ├── <b>models</b>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/_utils.py">_utils.py</a>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/utils.py">utils.py</a>
    │   ├── language_models
    │   │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/abstract_language_model.py">abstract_language_model.py</a>
    │   │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/beam_sampling.py">beam_sampling.py</a>
    │   │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/mixer.py">mixer.py</a> <sub><sup>wrap multiple language models into a single object</sup></sub>
    │   │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/ngram_language_model.py">ngram_language_model.py</a> <sub><sup>uses hash-tables & frequency to predict next token</sup></sub>
    │   │   └── transformer_language_model
    │   │       ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/transformer_language_model/layers.py">layers.py</a>
    │   │       ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/transformer_language_model/model.py">model.py</a> <sub><sup>decoder-only transformer with controllable vocabulary</sup></sub>
    │   │       └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/transformer_language_model/train.py">train.py</a>
    │   │
    │   ├── sign_to_text
    │   ├── text_to_sign
    │   │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/text_to_sign/concatenative_synthesis.py">concatenative_synthesis.py</a> <sub><sup>join sign clip of each word in text using rules</sup></sub>
    │   │   └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/text_to_sign/t2s_model.py">t2s_model.py</a> <sub><sup>Base class</sup></sub>
    │   │
    │   ├── text_embedding
    │   │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/text_embedding/text_embedding_model.py">text_embedding_model.py</a> <sub><sup>Base class</sup></sub>
    │   │   └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/text_embedding/vector_lookup_model.py">vector_lookup_model.py</a> <sub><sup>retrieves word embedding from a vector database</sup></sub>
    │   │
    │   └── video_embedding
    │       ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/video_embedding/mediapipe_landmarks_model.py">mediapipe_landmarks_model.py</a> <sub><sup>2D & 3D coordinates of points on body</sup></sub>
    │       └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/video_embedding/video_embedding_model.py">video_embedding_model.py</a> <sub><sup>Base class</sup></sub>
    │
    ├── <b>text</b>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/text/metrics.py">metrics.py</a> <sub><sup>numeric score techniques</sup></sub>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/text/preprocess.py">preprocess.py</a>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/text/subtitles.py">subtitles.py</a> <sub><sup>WebVTT</sup></sub>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/text/synonyms.py">synonyms.py</a>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/text/tagger.py">tagger.py</a> <sub><sup>classify tokens to assist in mapping</sup></sub>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/text/tokenizer.py">tokenizer.py</a> <sub><sup>break text into words, phrases, sentences etc</sup></sub>
    │   └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/text/utils.py">utils.py</a>
    │
    ├── <b>utils</b>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/utils/archive.py">archive.py</a> <sub><sup>zip datasets</sup></sub>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/utils/arrays.py">arrays.py</a> <sub><sup>common interface & operations for numpy.ndarray and torch.Tensor</sup></sub>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/utils/download.py">download.py</a>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/utils/parallel.py">parallel.py</a> <sub><sup>multi-threading</sup></sub>
    │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/utils/tree.py">tree.py</a> <sub><sup>print file hierarchy</sup></sub>
    │   └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/utils/utils.py">utils.py</a>
    │
    └── <b>vision</b>
        ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/vision/_utils.py">_utils.py</a>
        ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/vision/utils.py">utils.py</a>
        ├── landmarks
        │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/vision/landmarks/connections.py">connections.py</a> <sub><sup>drawing configurations for different landmarks models</sup></sub>
        │   ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/vision/landmarks/display.py">display.py</a> <sub><sup>visualize points & lines on 3D plot</sup></sub>
        │   └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/vision/landmarks/landmarks.py">landmarks.py</a> <sub><sup>wrapper for sequence of collection of points on body</sup></sub>
        │
        ├── sign
        │   └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/vision/sign/sign.py">sign.py</a> <sub><sup>Base class to wrap around sign clips</sup></sub>
        │
        └── video
            ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/vision/video/display.py">display.py</a> <sub><sup>jupyter notebooks inline video & pop-up in CLI</sup></sub>
            ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/vision/video/transformations.py">transformations.py</a> <sub><sup>strategy design-pattern for image augmentation</sup></sub>
            ├── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/vision/video/video_iterators.py">video_iterators.py</a> <sub><sup>adapter design-pattern for video reading</sup></sub>
            └── <a href="https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/vision/video/video.py">video.py</a>
</pre>

</details>
## licença

Este projeto está licenciado ao abrigo da [Licença Apache 2.0](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/blob/main/LICENSE). Está autorizado a utilizar a biblioteca, criar versões modificadas ou incorporar partes do código no seu próprio trabalho. O seu produto ou investigação, seja comercial ou não comercial, deve atribuir o devido crédito ao(s) autor(es) original(is), citando este repositório.

Stay Tuned for research Papers!

## Crédito e gratitude

Esse projeto usa como base o projeto desenvolvido e documentado por [Mudasar](https://github.com/mdsrqbl);
