<div align="center">

# Tradutor de linguagem de sinais ⠎⠇⠞

</div>

---

## Overview

A linguagem de sinais consiste em gestos e expressões usadas principalmente por pessoas com deficiência auditiva para lhes ajudarem a se comunicar. Esse projeto é um trabalho para ajudar a diminuir a distância entre a comunidade de deficiêntes auditivos e pessoas sem essa deficiência utilizando IA.

Isso é uma API e Framework desenvolvida em Python construida como um tradutor de linguagem de sinais...

<div></div>
Um grande obstáculo é a falta de conjuntos de dados (globais e regionais) e de estruturas que os engenheiros de Deep learning e os programadores de software possam utilizar para criar produtos úteis para a comunidade-alvo. Este projeto visa potenciar a tradução de língua gestual, fornecendo componentes, ferramentas, conjuntos de dados e modelos robustos, tanto para a conversão de língua gestual para texto como de texto para língua gestual. Tem como objetivo facilitar a criação de tradutores de língua gestual para qualquer região, abrindo simultaneamente o caminho para a normalização da língua gestual.

Diferentes de muitos projetos esse permite traduzir frases inteiras e não apenas o alfabeto.

### Solução

Esse pacote vem com um "_extensible rule-based_" sistema de tradução de texto para sinais que pode ser usado para treinar modelos de "_Deep Learning_" com seus dados para ambos sinais para texto como texto para sinais.

Para criar um sistema de tradução baseado em regras para a sua língua regional, pode herdar as classes `TextLanguage` e `SignLanguage` e passá-las como argumentos à classe `ConcatenativeSynthesis`. Para escrever textos de exemplo com palavras suportadas, pode utilizar os nossos modelos linguísticos. Em seguida, pode utilizar esse sistema para afinar os nossos modelos de Deep learning.

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
<div>
Texto para Linguagem de sinais

Existem duas abordagens para este problema:

1. Concatenação baseada em regras

   1. Rotular um dicionário de linguagem gestual com todos os tokens de palavras que possam ser mapeados para esses gestos.
   2. Analise o texto de entrada e reproduza os videoclipes adequados para cada token.
      1. Crie um processador de texto herdando de `slt.languages.TextLanguage` (consulte o subpacote [`slt.languages.text`](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/tree/main/sign_language_translator/languages/text) para mais detalhes)
      2. Mapeie a gramática e as palavras do texto para a língua gestual, herdando de `slt.languages.SignLanguage` (consulte o subpacote [`slt.languages.sign`](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/tree/main/sign_language_translator/languages/sign) para mais detalhes)
      3. Utilize o nosso modelo baseado em regras [`slt.models.ConcatenativeSynthesis`](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/blob/main/sign_language_translator/models/text_to_sign/concatenative_synthesis.py) para a tradução.
   3. É mais rápido, mas o **significado das palavras tem de ser desambiguado** na entrada. Consulte a abordagem de Deep Learning para lidar automaticamente com palavras ambíguas e **palavras que não constam no dicionário**.

2. Deep Learning (seq2seq)
   1. Gerar a sequência de nomes de ficheiros que devem ser concatenados
   2. Ou sintetize os sinais diretamente utilizando um codificador de texto multilingue pré-treinado e 1. uma GAN, um modelo de difusão ou um descodificador para sintetizar uma sequência de vetores de pose (`shape = (time, num_landmarks * num_coordinates)`) 1. Mover um avatar com esses vetores de pose (Fácil) 2. Utilizar a transferência de movimento para gerar um vídeo (Médio) 3. Sintetizar um fotograma de vídeo para cada vetor (Difícil) 2. um modelo de síntese de vídeo (Muito difícil)
   </div>
   </li>

<li>
<div>
<summary><b>
Processamento de linguagem
</b></summary>

1. Processamento de sinais
   - Extração de pontos de referência do mundo 3D com o Mediapipe.
   - Visualização de poses com o matplotlib.
   - Transformações de poses (aumentação de dados) com o scipy.
2. Processamento de texto
   - Normalizar a entrada de texto, substituindo caracteres ou grafias desconhecidos por palavras suportadas.
   - Desambiguar palavras dependentes do contexto para garantir uma tradução precisa.
     «spring» -> [«spring (manancial)», «spring (mola metálica)»]
   - Tokenizar o texto (ao nível da palavra e da frase).
   - Classificar os tokens e marcá-los com etiquetas.

</div>
</li>

<li>
<div>
<summary><b>
Datasets
</b></summary>

Para ver o dataset utilizado: [_sign-language-datasets_ repo](https://github.com/sign-language-translator/sign-language-datasets).
Veja essa [documentaçao](https://slt.readthedocs.io/en/latest/datasets.html) Para saber mais sobre como criar um conjunto de dados de vídeos de língua gestual (ou sobre as características dos dados obtidos através de luvas de captura de movimento).

**Seus dados deve ser incluidos**:

1. Um dicionário ao nível da palavra (vídeos de sinais individuais e tokens de texto correspondentes (palavras e frases))
2. Reproduções do dicionário. (Instalar várias câmaras sincronizadas e gravar pessoas aleatórias a reproduzir os vídeos do dicionário. ([notebook](https://colab.research.google.com/github/sign-language-translator/notebooks/blob/main/data_collection/clip_extractor.ipynb)))
3. Frases paralelas
   1. Frases em linguagem escrita normal comparadas com vídeos em língua gestual. (Utilizar os nossos modelos linguísticos para gerar frases compostas por palavras do dicionário.)
   2. Frases em linguagem escrita normal comparadas com a [explicação textual](https://github.com/sign-language-translator/sign-language-datasets#glossary) da frase correspondente em língua gestual.
   3. Frases em língua gestual comparadas com a sua explicação textual
   4. Frases em língua gestual comparadas com traduções em várias línguas escritas
4. Regras gramaticais da língua gestual
   1. Ordem das palavras (por exemplo, SUJEITO OBJETO VERBO TEMPO)
   2. Palavras sem significado (por exemplo, «o», «sou», «são»)
   3. Palavras ambíguas (por exemplo, «mola» (mola de aço) e «mola» (fonte de água))

**Tente incorporar**:

1. Vários ângulos de câmara
2. Intérpretes diversos para captar todos os _sotaques_ dos sinais
3. Originalidade na identificação dos tokens de palavras
4. Variações nos sinais para o mesmo conceito

Procure captar as variações nos sinais de uma forma escalável e que acomode a diversidade, contribuindo assim para o avanço dos esforços de normalização da língua gestual.

</div>
</li>
</ol>

### Objetivos

1. Permitir a **integração** da língua gestual nas aplicações existentes.
2. Apoiar a criação de soluções **personalizadas** para línguas gestuais com poucos recursos.
3. Melhorar a qualidade da **educação** para as pessoas surdas e aumentar as taxas de alfabetização.
4. Promover a **inclusão** na comunicação das pessoas com deficiência auditiva.
5. Estabelecer um quadro para a **normalização** da língua gestual.

## Uso

Clique e seja direcionado para o [slt.**readthedocs**.io](https://slt.readthedocs.io) para ver mais detalhes de uso em Python, CLI and gradio GUI.
veja o case de teste em [_test cases_](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/tree/main/tests).

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

**Gerar exemplos de treino**: escrever uma frase com um modelo de linguagem e, a partir dela, sintetizar um vídeo de língua gestual com um único comando:

```bash
slt translate --model-code rule-based --text-lang urdu --sign-lang pk-sl --sign-format video \
"$(slt complete '<' --model-code urdu-mixed-ngram --join '')"
```

## Languages

<div>
<summary><b>Linguagem de texto</b></summary>

Funções disponível:

- Normalização de texto
- "Tokenização" (palavras, frases & sentenças)
- Classificação de tokens (Marcação)
- Desambiguação do significado de palavras

| Nome                                                                                                                                      | Vocabulario          | Token ambíguo | Sinais |
| ----------------------------------------------------------------------------------------------------------------------------------------- | -------------------- | ------------- | ------ |
| [English](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/blob/main/sign_language_translator/languages/text/english.py) | 1591 palavras+frases | 167           | 776    |
| [Urdu](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/blob/main/sign_language_translator/languages/text/urdu.py)       | 2080 palavras+frases | 227           | 776    |
| [Hindi](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/blob/main/sign_language_translator/languages/text/hindi.py)     | 137 palavras+frases  | 5             | 84     |

</div>

<div>
<summary><b>Linguagem de sinais</b></summary>

Funções disponível:

- Correspondência entre palavras e expressões e sinais
- Reestruturação de frases de acordo com as regras gramaticais
- Simplificação de frases (eliminação de palavras vazias)

| Name                                                                                                                                                                       | Vocabulario | Dataset  |                                     Parallel Corpus                                     |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | -------- | :-------------------------------------------------------------------------------------: |
| [Pakistan Sign Language](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/languages/sign/pakistan_sign_language.py) | 776         | 23 hours | [detalhes](https://github.com/sign-language-translator/sign-language-datasets#datasets) |

</div>

## Modelos

<div>
<summary><b>Traduções</b>: Linguagem texto para sinais </summary>

| Nome                                                                                                                                                                              | Arquitetura           | Descrição                                                                                                                                                                                         | Input  | Output                     |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ | -------------------------- |
| [Concatenative Synthesis](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/text_to_sign/concatenative_synthesis.py) | Regras + Tabelas Hash | O tradutor baseado em regras principal, utilizado principalmente para sintetizar o conjunto de dados de tradução.<br/>Inicialize-o utilizando os objetos TextLanguage, SignLanguage e SignFormat. | string | slt.Video \| slt.Landmarks |

</div>

<div>
<summary><b>Incorporação de sinais/Extração de características</b>:</summary>

| Nome                                                                                                                                                                                                 | Arquitetura                                                                                                      | Descrição                                                                                               | Formato do Input                                               | Formato do Output                             |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- | --------------------------------------------- |
| [MediaPipe Landmarks<br>(Pose + Hands)](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/video_embedding/mediapipe_landmarks_model.py) | CNN based pipelines: [Pose](https://arxiv.org/pdf/2006.10204.pdf), [Hands](https://arxiv.org/pdf/2006.10214.pdf) | Codifica vídeos em vetores de pose (mundo 3D ou imagem 2D) que representam os movimentos do intérprete. | Lista de imagens numpy<br/>(n_frames, altura, largura, canais) | torch.Tensor<br/>(n_frames, n_landmarks \* 5) |

</div>

<div>
<summary><b>Geração de dados</b>: Modelos de linguagens</summary>

[Modelos disponível](https://github.com/sign-language-translator/sign-language-datasets/releases/tag/v0.0.1)

| Nome                                                                                                                                                                                                  | Arquitetura                                  | Descrição                                                                                                            | Formato do Input                                             | Formato do Output                                                               |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| [Modelo de linguagem N-Gram](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/ngram_language_model.py)                  | Tabelas de hash                              | Prevê o próximo token com base em estatísticas aprendidas sobre os N tokens anteriores.                              | Lista de tokens                                              | (token, probabilidade)                                                          |
| [Modelo de linguagem Transformer](https://github.com/sign-language-translator/sign-language-translator/blob/main/sign_language_translator/models/language_models/transformer_language_model/model.py) | Transformers apenas com descodificador (GPT) | Prevê o próximo token utilizando atenção do tipo «query-key-value», transformações lineares e probabilidades suaves. | torch.Tensor<br/>(batch, token_ids)<br/><br/>Lista de tokens | torch.Tensor<br/>(batch, token_ids, vocab_size)<br/><br/>(token, probabilidade) |

</div>

<div>
<summary><b>Incorporação de texto</b>:</summary>

[Modelos disponível](https://github.com/sign-language-translator/sign-language-datasets/releases/)

| Nome                                                                                                                                                               | Arquitetura | Descrição                                                                                                                   | Formato do Input | Formato do Output         |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------- | --------------------------------------------------------------------------------------------------------------------------- | ---------------- | ------------------------- |
| [Vector Lookup](https://github.com/Brenokluck/projeto-univille-tradutor-de-sinais/blob/main/sign_language_translator/models/text_embedding/vector_lookup_model.py) | HashTable   | Encontra o índice do token e devolve o vetor correspondente. Tokeniza frases e calcula o vetor médio dos tokens conhecidos. | string           | torch.Tensor<br/>(n_dim,) |

</div>

## Como construir o seu próprio tradutor de linguagem de sinais

Para criar o seu próprio tradutor de língua gestual, vai precisar destes componentes essenciais:

<ol>
<li>
<div>
<summary>Recolha de dados</summary>

1.  Reúna uma coleção de [vídeos de dicionário](https://github.com/sign-language-translator/sign-language-datasets/releases/tag/v0.0.2) (ao nível da palavra) que mostrem pessoas a realizar gestos de língua gestual. Estes podem ser obtidos junto de escolas e organizações para surdos. Deve gravar várias pessoas a realizar o mesmo gesto para captar vários _acentos_ do mesmo. Instale várias câmaras em locais diferentes, em paralelo, para aumentar ainda mais a quantidade de dados.
2.  Prepare um [ficheiro JSON](https://github.com/sign-language-translator/sign-language-datasets/blob/main/parallel_texts/pk-dictionary-mapping.json) que associe os nomes dos ficheiros de vídeo do dicionário às palavras e frases correspondentes na língua falada, que sejam sinónimas dos gestos.
3.  Prepare um [corpus paralelo](https://github.com/sign-language-translator/sign-language-datasets/blob/main/parallel_texts/pk-synthetic-sentence-mapping.json) de dados sintéticos contendo frases em linguagem escrita e sequências de nomes de ficheiros de vídeo de linguagem gestual. Pode utilizar modelos linguísticos para gerar estas frases e sequências.
4.  Prepare um conjunto de dados de [vídeos de frases](https://github.com/sign-language-translator/sign-language-datasets/releases/tag/v0.0.3) em língua gestual, rotulados com [traduções e glossários] (https://github.com/sign-language-translator/sign-language-datasets/blob/main/parallel_texts/pk-sentence-mapping.json) em vários idiomas escritos.

</div>
</li>

<li>
<div>
<summary>Processador de linguagem</summary>

1.  Implemente uma subclasse de `slt.languages.TextLanguage`:
    - Divida o seu texto em tokens e atribua as etiquetas adequadas aos tokens para um processamento mais eficiente.
2.  Crie uma subclasse de `slt.languages.SignLanguage`:
    - Associe os tokens de texto aos nomes dos ficheiros de vídeo utilizando os dados JSON fornecidos.
    - Reorganize a sequência dos nomes dos ficheiros de vídeo para que se alinhem com a gramática e a estrutura da língua gestual.

</div>
</li>

<li>
<div>
<summary>Tradução baseada em regras</summary>

1.  Passe as instâncias das suas classes do passo anterior para a classe `slt.models.ConcatenativeSynthesis` para obter um objeto tradutor baseado em regras.
2.  Construa frases na sua língua de texto e utilize o tradutor baseado em regras para gerar traduções para a língua gestual. (Pode utilizar os nossos modelos de linguagem para gerar esses textos.)

</div>
</li>

<li>
<div>
<summary>Ajuste fino de modelos Deep Learning</summary>

1.  Utilize os vídeos de linguagem gestual (sintética e real) e as frases de texto correspondentes da etapa anterior.
2.  Aplique o nosso fluxo de treino para aperfeiçoar um modelo selecionado, com vista a melhorar a precisão e a qualidade da tradução.

</div>
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
