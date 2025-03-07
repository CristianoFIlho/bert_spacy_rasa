# Finetune BERT Embeddings with spaCy and Rasa

**For whom this repository might be of interest:**

This repository describes the process of finetuning the *german pretrained BERT* model of [deepset.ai](https://deepset.ai/german-bert)
on a domain-specific dataset, converting it into a [spaCy](https://spacy.io/) packaged model and loading it in [Rasa](https://rasa.com/) to evaluate its
performance on domain-specific **Conversational AI** tasks like *intent detection* and *NER*.
If there are questions though, feel free to ask.

This repository is meant for those who want to have a quick dive into the matter. 

I am going to use the [10kGNAD](https://tblock.github.io/10kGNAD/) dataset for this task but it should be easy to
modify the files for your specific use case.

**Short-term Roadmap**:

- [x] Add [DistilBERT](https://github.com/huggingface/transformers/tree/master/examples/distillation) support
- [ ] Add CUDA Installation Guide
- [x] Add [RoBERTa](https://arxiv.org/abs/1907.11692) support
- [x] Add NER support

___
## Updates

**Update 06.07.2020: Alternative ways of usage**

Hi everyone,

long time no see! 

Though the content of this repository should still do the job even with the newest Rasa version, a lot happened the past months.

* [Spacy updated to version 2.3](https://spacy.io/usage/v2-3#_title)
* HuggingFace released version 3 of their [transformers](https://github.com/huggingface/transformers) library
* Rasa released version [1.10.5](https://rasa.com/docs/rasa/changelog/#id1) of their library

Since there is a lot of change going on regarding Spacy version 3, the currently used spacy-transformers library most likely
won't get any more updates. I therefore strongly recommend to use the transformers library to achieve the same finetuning 
results as with Spacy. In order to do so, simply use the script "huggingface_finetune.py" alongside a [HFTransformersNLP](https://rasa.com/docs/rasa/nlu/components/#hftransformersnlp) component the following way:

```
pipeline:
 - name: HFTransformersNLP
   model_name: "bert"
   model_weights: "PATH_TO_YOUR_FINETUNED_MODEL_DIRECTORY"
   cache_dir: "PATH_TO_SOME_CACHE_FOLDER"
 - name: LanguageModelFeaturizer
 - name: DIETClassifier
   random_seed: 42
   intent_classification: True
   entity_recognition: False
   use_masked_language_model: True
   epochs: 80
   number_of_transformer_layers: 4
   transformer_size: 256
   drop_rate: 0.2
   weight_sparsity: 0.7
   batch_size: [64, 256]
   embedding_dimension: 50
   hidden_layer_sizes:
     text: [512, 128]
```

Please change the settings according to your situation, especially the Hyperparameters for DIET. The given ones prove to perform
good on the german language.

Tested with:

* python = 3.6.8
* transformers = 3.0.0
* rasa = 1.10.5


**Update 24.03.2020: Changes to Rasa and Spacy**

I verified that everything is still working with:

* python = 3.6.8
* spacy = 2.2.4
* spacy-transformers = 0.5.1
* rasa = 1.8.2

Besides, Rasa added the [HFTransformersNLP](https://rasa.com/docs/rasa/nlu/components/#hftransformersnlp) pipeline element to its core which enables the user to use every [pretrained model](https://huggingface.co/transformers/pretrained_models.html) accordingly. However, this currently doesn't replace the finetuning aspect which still significantly boosts the models performance. I am currently working on a finetuning CustomComponent for Rasa.

**Update 28.12.2019: DistilBERT**

I finally got the time to add a [DistilBERT](https://github.com/huggingface/transformers/tree/master/examples/distillation) version
that can be used for finetuning and as a Spacy model used in Rasa.

I order to use this one you need to follow these steps:

1. Modify the files changed in [this PR](https://github.com/explosion/spacy-transformers/pull/120) in your local spacy-transformers installation
2. Modify the files changed in [this PR](https://github.com/explosion/spacy-transformers/pull/121) in your local spacy-transformers installation
3. Download [this DistilBERT model](https://huggingface.co/distilbert-base-german-cased) from HuggingFace into your repository
4. In the downloaded directory make sure to have the following files present: `config.json`, `pytorch_model.bin`, `vocab.txt`
5. Use the command `python examples/init_model.py --lang de --name distilbert-base-german-cased /path/to/model` from the `spacy-transformers` repo
6. You should see a new folder *distilbert-base-german-cased* with the spacy-initiated model files. Use:

```
python -m spacy package distilbert-base-german-cased/ /packaged_model
cd /packaged_model/de_distilbert_base_german_cased-0.0.1
python setup.py sdist
pip install dist/de_distilbert_base_german_cased-0.0.1.tar.gz
```

* (Optional) If you want to **finetune** de_distilbert_base_german_cased, change the `trf_textcat` architecture to `softmax_last_hidden`
* (Optional) If you want to **create an Excel-file** for finetuning out of an existing **`nlu.md` from Rasa**, you can use `create_xlsx_dataset_from_rasa_nlu.py` to create one.

It is worth to mention, that every model that is supported by the `transformers` library can be converted and used
this way. If you want to do that, simply use the `init_model.py` of `spacy-transformers` this way:

```
python examples/init_model.py --lang xx --name TRANSFOMERS_MODEL_NAME /path/to/model
```
___

**Update 28.12.2019: NER finetuning**

I finally got the time to evaluate the NER support for training an already finetuned BERT/DistilBERT model on
a *Named Entity Recognition* task. 

In order to use this one, follow these steps:

1. Modify the files in [this PR](https://github.com/explosion/spacy-transformers/pull/95) in your current spacy-transformers installation
2. Modify the files changed in [this PR](https://github.com/explosion/spacy-transformers/pull/120) in your local spacy-transformers installation
3. Modify the files changed in [this PR](https://github.com/explosion/spacy-transformers/pull/121) in your local spacy-transformers installation
4. Use the added `bert_finetuner_ner.py` script from the spacy-transformers library on any pretrained BERT-architectured model

After the finetuning process finished, you can treat the resulting model as later explained in this guide by *packaging* it
for the usage in Rasa.

## Installation

### Requirements

Basically all you need to to is execute:

```
pip install -r requirements.txt
```

The scripts are tested using the following libraries:

* python = 3.6.8
* spacy = 2.2.3
* spacy-transformers = 0.5.1
* rasa = 1.6.0
* transformers 2.3.0

Please keep in mind that some of the dependencies are work in progress and there might be inter-incompatibilities. 
However, at the time of writing this, the libraries can simply be installed by using `pip`.

I strongly suggest do finetune and test BERT with GPU support since finetuning on even a good CPU
can last several hours per epoch. 
___
### Getting started

#### Preparing the dataset

The *split* is done by the finetuning script. If you want to have a different setting,
feel free to modify the script.

As suggested, we do a simple but stratified train-test split with 15% as the test subset and 85% as the training subset, which results in 8732 training
samples and 1541 evaluation samples. As there are many possibilities left, this is only one
possible approach. While converting the `articles.csv` into a pandas dataframe, there were some broken lines
which currently are omitted.
___
#### Loading the pretrained BERT

The script assumes the pretrained BERT to be installed with:

```
python -m spacy download de_trf_bertbasecased_lg
```

For the sake of interest, I have added the ``bert_config.json`` from Deepset's awesome work
if someone wonders how the ``de_trf_bertbasecased_lg`` was trained.
___
#### Finetune the pretrained BERT

You can start the finetuning process by using:

```
python bert_finetuner_splitset.py de_trf_bertbasecased_lg -o finetuning\output
```

Currently, I am using a ```softmax_pooler_ouput``` configuration for the ``trf_textcat``component.
I'd suggest a ``softmax_last_hidden`` as the next approach. The other parameters
were set based on several evaluations and might be modified for your specific use case.
___
#### Package the finetuned BERT with spaCy and install it

You can easily package your newly trained model by using:

```
python -m spacy package finetuning/output /packaged_model
cd /packaged_model/de_trf_bertbasecased_lg-1.0.0
python setup.py sdist
pip install dist/de_trf_bertbasecased_lg-1.0.0.tar.gz
```

I recommend **changing the model's name** to avoid unnecessary inconveniences
by editting the config file and modifying the ``name`` value of `/finetuning/output/meta.json`.

___
#### Load the spaCy model as part of your Rasa pipeline (optional)

At the time of writing this, BERT outperforms most of the recent state-of-the-art approaches
in NLP/NLU tasks, e.g. document classification. 
Since those techniques are used in several **conversational AI** tasks like **intent detection**, I thought it might be a good idea to evaluate its performance with **Rasa** - IMHO one of the
best open source CAI engines currently available.

If someone is interested in building a chatbot with Rasa, it might be a good idea to read the
[Getting started](https://rasa.com/docs/getting-started/) guide.

Assuming that someone is familiar with Rasa, here is one possible configuration proposal which
loads the newly added finetuned BERT model as a part of the training pipeline:

```
language: de
pipeline: 
 - name: SpacyNLP
   case_sensitive: 1
   model: de_trf_bertbasecased_lg_gnad
 - name: SpacyTokenizer
 - name: SpacyFeaturizer
 - name: SklearnIntentClassifier
```

As you can see, I just specified the model's name, using the spaCy architecture with
Rasa. This works, even if ``python -m spacy validate`` does **not** show your model.

Assuming that you might want to test the performance with Rasa, you can use the ``test_bot`` directory
which contains the skeletton for a Rasa bot to do so. In advance, use:

```
python rasa_bot_generator.py
cp test.md test_bot/test_data/
cp train.md test_bot/data/
cd test_bot
rasa train --data data/ -c config.yml -d domain.yml --out models/
rasa run -m models/ --enable-api
```

to create a valid ``stories.md`` and a valid ``domain.yml``. Please keep in mind that
this will be a minimal sample from which I don't recommend to use it productively.

If the bot is loaded, you can use the endpoint:

```
http://localhost:5005/model/parse

POST
{
	"text": "<any article you want to get its domain for>"
}

```
___
#### Evaluate different pipelines

To keep things simple, there are two scripts which will do the work for you.

**bert_classify** evaluates the finetuned BERT by training a logistic regression
and a simple SVM classifier.

```
python -m bert_classify.py 
```

**bert_rasa_classify** loads the trained Rasa model and uses the pretrained BERT features to evaluate the
model's performance on the test data. Keep in mind that Rasa *compresses* your model, so you simply
have to unzip/untar it and also modify the path to the NLU model in the script.

```
python -m bert_rasa_classify.py 
```

Please be aware of the fact that to evaluate the **generalization capabilities** of the model,
it would be better to split the original dataset into three parts such that there is a dataset
completely unknown by the model (i.e. train/validation/test split).
___
#### Productive usage of a large BERT model

TBD
___

#### A note on NER (Named Entity Recognition)

As soon as I realized that I won’t be able to use the finetuned BERT-spaCy model in rasa for e.g. extracting entities like PERSON (in fact, duckling is currently not able to do that), I thought about how this would be done in general:

1. Use the SpacyFeaturizer and SpacyEntityExtractor which currently would be recommended but which is not possible due to manual effort on the side of BERT (as mentioned, I am working on that).
2. Finetuning the pretrained BERT that afterwards is converted into a spaCy-compatible model on any NER dataset is absolutely possible and intended. We can finetune the BERT on both tasks alongside. If so, the model contains everything we are going to need to derive entities from it. Currently just not with spaCy directly. Instead we could use a CustomBERTEntityExtractor which loads the model that the pipeline already has loaded and do the work, that spaCy is currently not “able” to do.

3. Since 2 seems to be an overhead at least for the moment, why not do the following:
```
language: de
pipeline: 
 - name: SpacyNLP
   case_sensitive: 1
   model: de_trf_bertbasecased_lg_gnad
 - name: SpacyTokenizer
 - name: SpacyFeaturizer
 - name: SklearnIntentClassifier
 - name: SpacyNLP
   case_sensitive: 1
   model: de_core_news_md
 - name: RegexFeaturizer
 - name: CRFEntityExtractor
 - name: DucklingHTTPExtractor
   dimensions: ['time', 'duration', 'email']
   locale: de_DE
   timezone: Europe/Berlin
   url: http://localhost:8001
 - name: SpacyEntityExtractor
   dimensions: ['PER', 'LOC', 'CARDINAL']
 - name: rasa_mod_regex.RegexEntityExtractor
 - name: EntitySynonymMapper

```
This pipeline will then load and use the features of de_trf_bertbasecased_lg_gnad for SklearnIntentClassifier, and the features of de_core_news_md for SpacyEntityExtractor.

This is not a neat solution and it should only be used until there is a smarter way (1,2) but it works.

It should be mentioned, that of course you are able to even train your own with spaCy.


#### Troubleshooting


##### CUDA Out of Memory

As discussed in a [spacy-trf-issue](https://github.com/explosion/spacy-pytorch-transformers/issues/48) you may run into
memory problems. I have tested the finetuning script on a *GTX 1080 with 8GB VRAM* and even with a batch size of
2 (which is absolutely *not* recommended), I got memory problems.

One way to deal with it is to use the sentencizer which splits larger documents into sentences while keeping their original labels.
Another way is to reduce the batch size by half, to 12. BERT models usually need bigger batches but for the sake of functionality, I tried it.

Currently I am using a *T80 with 12 GB VRAM*, sentencizing and a lowered batch size and that setup worked fine.


##### AttributeError: module 'thinc_gpu_ops' has no attribute 'mean_pool'

As discussed [here](https://github.com/explosion/spacy-pytorch-transformers/issues/27) you might run into the mentioned
error. I was able to resolve it by manually cloning thinc-gpu-ops, running ``pip install -r requirements.txt`` (that actually installed cython) and then running ``pip install`` .

___




A *thank you* goes to all of the **amazing open source workers** out there:

* [Rasa](https://github.com/RasaHQ)
* [spaCy](https://github.com/explosion/spaCy)
* [Deepset](https://deepset.ai/german-bert)
* [HuggingFace](https://github.com/huggingface/pytorch-transformers)
* [MKaze](https://github.com/mkaze/)



