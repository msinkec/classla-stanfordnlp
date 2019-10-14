# StanfordNLP: A Python NLP Library for Many Human Languages
## A [CLASSLA](http://www.clarin.si/info/k-centre/) Fork For Processing South Slavic Languages 

This is a fork of the [Stanford NLP Group's official Python NLP library](https://github.com/stanfordnlp/stanfordnlp). The main changes to the official library are
- possibility of training the tagger and lemmatiser on datasets which are not UD-parsed
- using an external dictionary while performing lemmatization
- looking up (word, upos, feats) triples in dictionaries during lemmatization, fallbacking to (word, upos)
- speeding up the seq2seq lemmatization training procedure by training only on instances occurring less than 5 times in the training data
- speeding up the lemmatization by seq2seqing only forms not seen in the lookup lexicon (implemented in stanfordnlp only in the pipeline)
- encoding POS information at the beginning of the sequence, improving significantly seq2seq lemmatization results

## Running the tool

### Part-of-speech tagging

For now, there are available pre-trained models for part-of-speech tagging for 
- standard Slovenian http://hdl.handle.net/11356/1251 
- standard Croatian http://hdl.handle.net/11356/1252 
- standard Serbian http://hdl.handle.net/11356/1253

Once you placed the PoS-tagging model files into the ```models/pos/``` path, you can run the following commands (for (1) Slovenian, (2) Croatian, or (3) Serbian)

```
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name ssj500k --eval_file data/ssj500k.dev.conllu --output_file temp --gold_file data/ssj500k.dev.conllu --shorthand sl_ssj --mode predict
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name hr500k --eval_file data/hr500k.dev.conllu --output_file temp --gold_file data/hr500k.dev.conllu --shorthand hr_set --mode predict
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name SETimes.SR --eval_file data/SETimes.SR.dev.conllu --output_file temp --gold_file data/SETimes.SR.dev.conllu --shorthand sr_set --mode predict
```

If you do not want to evaluate the tagger, but just annotate a new file, you can leave out the ```--gold_file``` argument.

The input to part-of-speech tagging is a CONLLU-formated file.

### Lemmatisation

Similarly to PoS tagging, there are pre-trained models for lemmatisation available as well for
- standard Slovenian http://hdl.handle.net/11356/1254
- standard Croatian http://hdl.handle.net/11356/1255
- standard Serbian http://hdl.handle.net/11356/1256

Running the lemmatiser for Slovenian, if models are placed in the ```models/lemma/``` directory, can be performed as follows:
```
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file ssj500k+Sloleks --eval_file data/pos.ssj500k.dev.conllu --output_file temp --gold_file data/pos.ssj500k.dev.conllu --mode predict
```

Similarly, for Croatian or Serbian, these are the corresponding commands:

```
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file hr500k+hrLex --eval_file data/pos.hr500k.dev.conllu --output_file temp --gold_file data/pos.hr500k.dev.conllu --mode predict
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file SETimes.SR+srLex --eval_file data/pos.SETimes.SR.dev.conllu --output_file temp --gold_file data/pos.SETimes.SETimes.SR.dev.conllu --predict
``` 

Again, leaving out the ```-gold_file``` argument no evaluation will be performed.

The input to lemmatisation is a CONLLU-formated file which was previously part-of-speech tagged.

### Parsing

For UD depency parsing, the pre-trained models are also available for
- standard Slovenian http://hdl.handle.net/11356/1258
- standard Croatian http://hdl.handle.net/11356/1259
- standard Serbian http://hdl.handle.net/11356/1260

Parsing Slovenian data, once models are placed in the ```models/lemma/``` directory, can be performed as follows:

```
python -m stanfordnlp.models.parser --save_dir models/depparse/ --save_name ssj500k_ud --eval_file data/pos.lemma.ssj500k_ud.dev.conllu --gold_file data/pos.lemma.ssj500k_ud.dev.conllu --shorthand sl_ssj --output_file temp --mode predict
```
Similarly, for Croatian or Serbian, these are the corresponding commands:

```
python -m stanfordnlp.models.parser --save_dir models/depparse/ --save_name hr500k_ud --eval_file data/pos.lemma.hr500k_ud.dev.conllu --gold_file data/pos.lemma.hr500k_ud.dev.conllu --shorthand hr_set --output_file temp --mode predict
python -m stanfordnlp.models.parser --save_dir models/depparse/ --save_name SETimes.SR_ud --eval_file data/pos.lemma.SETimes.SR_ud.dev.conllu --gold_file data/pos.lemma.SETimes.SR_ud.dev.conllu --shorthand sr_set --output_file temp --mode predict
```

Again, leaving out the ```-gold_file``` argument no evaluation will be performed.

The input to lemmatisation is a CONLLU-formated file which was previously part-of-speech tagged and lemmatised.

## Training your own models

### Part-of-speech tagging

Below are examples for training models for standard Slovene, Croatian and Serbian, assuming (1) CLARIN.SI embeddings and (2) corresponding training datasets (ssj500k, hr500k, SETimes.SR) are in the specified locations. Training is performed on train+test data, while dev is used for evaluation.

```
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name ssj500k --wordvec_file ~/data/clarin.si-embed/embed.sl-token.ft.sg.vec.xz --train_file ../babushka-bench/datasets/sl/ssj500k/train+test.conllu --eval_file ../babushka-bench/datasets/sl/ssj500k/dev.conllu --gold_file ../babushka-bench/datasets/sl/ssj500k/dev.conllu --mode train --shorthand sl_ssj --output_file temp.sl
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name hr500k --wordvec_file ~/data/clarin.si-embed/embed.hr-token.ft.sg.vec.xz --train_file ../babushka-bench/datasets/hr/hr500k/train+test.conllu --eval_file ../babushka-bench/datasets/hr/hr500k/dev.conllu --gold_file ../babushka-bench/datasets/hr/hr500k/dev.conllu --mode train --shorthand hr_set --output_file temp.hr
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name SETimes.SR --wordvec_file ~/data/clarin.si-embed/embed.sr-token.ft.sg.vec.xz --train_file ../babushka-bench/datasets/sr/SETimes.SR/train+test.conllu --eval_file ../babushka-bench/datasets/sr/SETimes.SR/dev.conllu --gold_file ../babushka-bench/datasets/sr/SETimes.SR/dev.conllu --mode train --shorthand sr_set --output_file temp.sr
```

### Lemmatization

Given that the lemmatization process relies on XPOS, to make the training data as realistic as possible, we train the lemmatization process on automatically part-of-speech-tagged data. We also use external lexicons for lemmatization (Sloleks, hrLex and srLex).

The pre-tagging is performed as defined in the section of running part-of-speech tagging. The commands for training the lemmatizers are given below.

```
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file ssj500k+Sloleks --train_file pretagged/pos.ssj500k.train+test.conllu --eval_file pretagged/pos.ssj500k.dev.conllu --output_file temp.lemma.sl --gold_file pretagged/pos.ssj500k.dev.conllu --external_dict ~/data/morphlex/Sloleks --mode train --num_epoch 30 --decay_epoch 20 --pos
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file hr500k+hrLex --train_file pretagged/pos.hr500k.train+test.conllu --eval_file pretagged/pos.hr500k.dev.conllu --output_file temp.lemma.hr --gold_file pretagged/pos.hr500k.dev.conllu --external_dict ~/data/morphlex/hrLex --mode train --num_epoch 30 --decay_epoch 20 --pos
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file SETimes.SR+srLex --train_file pretagged/pos.SETimes.SR.train+test.conllu --eval_file pretagged/pos.SETimes.SR.dev.conllu --output_file temp.lemma.sr --gold_file pretagged/pos.SETimes.SR.dev.conllu --external_dict ~/data/morphlex/srLex --mode train --num_epoch 30 --decay_epoch 20 --pos
```

### Parsing

Preparing the data to be automatically pretagged on morphosyntactic and lemma level:

```
# Slovene tagging
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name ssj500k --eval_file ../babushka-bench/datasets/sl/ssj500k/dev_ud.conllu --output_file pretagged/pos.ssj500k_ud.dev.conllu --gold_file ../babushka-bench/datasets/sl/ssj500k/dev_ud.conllu --shorthand sl_ssj --mode predict
cat ../babushka-bench/datasets/sl/ssj500k/train_ud.conllu ../babushka-bench/datasets/sl/ssj500k/test_ud.conllu > pretagged/ssj500k_ud.train+test.conllu
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name ssj500k --eval_file pretagged/ssj500k_ud.train+test.conllu --output_file pretagged/pos.ssj500k_ud.train+test.conllu --gold_file pretagged/ssj500k_ud.train+test.conllu --shorthand sl_ssj --mode predict
# Croatian tagging
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name hr500k --eval_file ../babushka-bench/datasets/hr/hr500k/dev_ud.conllu --output_file pretagged/pos.hr500k_ud.dev.conllu --gold_file ../babushka-bench/datasets/hr/hr500k/dev_ud.conllu --shorthand hr_set --mode predict
cat ../babushka-bench/datasets/hr/hr500k/train_ud.conllu ../babushka-bench/datasets/hr/hr500k/test_ud.conllu > pretagged/hr500k_ud.train+test.conllu
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name hr500k --eval_file pretagged/hr500k_ud.train+test.conllu --output_file pretagged/pos.hr500k_ud.train+test.conllu --gold_file pretagged/hr500k_ud.train+test.conllu --shorthand hr_set --mode predict
# Serbian tagging
python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name SETimes.SR --eval_file ../babushka-bench/datasets/sr/SETimes.SR/dev_ud.conllu --output_file pretagged/pos.SETimes.SR_ud.dev.conllu --gold_file ../babushka-bench/datasets/sr/SETimes.SR/dev_ud.conllu --shorthand sr_set --mode predict
cat ../babushka-bench/datasets/sr/SETimes.SR/train_ud.conllu ../babushka-bench/datasets/sr/SETimes.SR/test_ud.conllu > pretagged/SETimes.SR_ud.train+test.conllu
(pytorch) nikolal@kt-gpu-vm-1TB:~/tools/classla-stanfordnlp$ python -m stanfordnlp.models.tagger --save_dir models/pos/ --save_name SETimes.SR --eval_file pretagged/SETimes.SR_ud.train+test.conllu --output_file pretagged/pos.SETimes.SR_ud.train+test.conllu --gold_file pretagged/SETimes.SR_ud.train+test.conllu --shorthand sr_set --mode predict
# Slovene lemmatization
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file ssj500k+Sloleks --eval_file pretagged/pos.ssj500k_ud.dev.conllu --output_file pretagged/pos.lemma.ssj500k_ud.dev.conllu --gold_file pretagged/pos.ssj500k_ud.dev.conllu --mode predict
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file ssj500k+Sloleks --eval_file pretagged/pos.ssj500k_ud.train+test.conllu --output_file pretagged/pos.lemma.ssj500k_ud.train+test.conllu --gold_file pretagged/pos.ssj500k_ud.train+test.conllu --mode predict
# Croatian lemmatization
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file hr500k+hrLex --eval_file pretagged/pos.hr500k_ud.dev.conllu --output_file pretagged/pos.lemma.hr500k_ud.dev.conllu --gold_file pretagged/pos.hr500k_ud.dev.conllu --mode predict
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file hr500k+hrLex --eval_file pretagged/pos.hr500k_ud.train+test.conllu --output_file pretagged/pos.lemma.hr500k_ud.train+test.conllu --gold_file pretagged/pos.hr500k_ud.train+test.conllu --mode predict
# Serbian lemmatization
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file SETimes.SR+srLex --eval_file pretagged/pos.SETimes.SR_ud.dev.conllu --output_file pretagged/pos.lemma.SETimes.SR_ud.dev.conllu --gold_file pretagged/pos.SETimes.SR_ud.dev.conllu --mode predict
python -m stanfordnlp.models.lemmatizer --model_dir models/lemma/ --model_file SETimes.SR+srLex --eval_file pretagged/pos.SETimes.SR_ud.train+test.conllu --output_file pretagged/pos.lemma.SETimes.SR_ud.train+test.conllu --gold_file pretagged/pos.SETimes.SR_ud.train+test.conllu --mode predict
```

Once you have all the training data preprocessed, you can train the parsers in the following manner:

```
python -m stanfordnlp.models.parser --save_dir models/depparse/ --save_name ssj500k_ud --wordvec_file ~/data/clarin.si-embed/embed.sl-token.ft.sg.vec.xz --train_file pretagged/pos.lemma.ssj500k_ud.train+test.conllu --eval_file pretagged/pos.lemma.ssj500k_ud.dev.conllu --gold_file pretagged/pos.lemma.ssj500k_ud.dev.conllu --shorthand sl_ssj --output_file temp.depparse.sl --mode train
python -m stanfordnlp.models.parser --save_dir models/depparse/ --save_name hr500k_ud --wordvec_file ~/data/clarin.si-embed/embed.hr-token.ft.sg.vec.xz --train_file pretagged/pos.lemma.hr500k_ud.train+test.conllu --eval_file pretagged/pos.lemma.hr500k_ud.dev.conllu --gold_file pretagged/pos.lemma.hr500k_ud.dev.conllu --shorthand hr_set --output_file temp.depparse.hr --mode train
python -m stanfordnlp.models.parser --save_dir models/depparse/ --save_name SETimes.SR_ud --wordvec_file ~/data/clarin.si-embed/embed.sr-token.ft.sg.vec.xz --train_file pretagged/pos.lemma.SETimes.SR_ud.train+test.conllu --eval_file pretagged/pos.lemma.SETimes.SR_ud.dev.conllu --gold_file pretagged/pos.lemma.SETimes.SR_ud.dev.conllu --shorthand sr_set --output_file temp.depparse.sr --mode train
```
