---
layout: post
title: "LSTM classification model on reuters dataset with word vectors pretrained by fastText"
date: 2018-05-07
categories: [projects]
---

```python
import io
import numpy as np
from nltk.corpus import reuters
import fastText
from keras.models import Sequential
from keras.layers import Dense, Dropout, Activation, LSTM
import keras.backend as K
from nltk.tokenize import TweetTokenizer
```


    Using TensorFlow backend.


## 1. load word embeddings pre-trained by FB fastText
- English and French word embeddings downloaded from [fastText pre-trained word vecters](https://github.com/facebookresearch/fastText/blob/master/pretrained-vectors.md), stored locally at `'data/wiki.en.vec'` and `'data/wiki.fr.vec'`
- loading all vecs is too memory-expensive and I just take the 50000 most frequent words into account. Some codes are reused from MUSE [demo.ipynb](https://github.com/facebookresearch/MUSE/blob/master/demo.ipynb)


```python
def load_vec(emb_path, nmax=50000):
    vectors = []
    word2id = {}
    with io.open(emb_path, 'r', encoding='utf-8', newline='\n', errors='ignore') as f:
        next(f)
        for i, line in enumerate(f):
            word, vect = line.rstrip().split(' ', 1)
            vect = np.fromstring(vect, sep=' ')
            assert word not in word2id, 'word found twice'
            vectors.append(vect)
            word2id[word] = len(word2id)
            if len(word2id) == nmax:
                break
    id2word = {v: k for k, v in word2id.items()}
    embeddings = np.vstack(vectors)
    return embeddings, id2word, word2id
```


```python
en_path = 'data/wiki.en.vec'
fr_path = 'data/wiki.fr.vec'
nmax = 50000  # maximum number of word embeddings to load

en_embeddings, en_id2word, en_word2id = load_vec(en_path, nmax)
# fr_embeddings, fr_id2word, fr_word2id = load_vec(fr_path, nmax)

emb_dim = en_embeddings.shape[1]
```

## 2. generate training and testing data

- fastText use the tokenizer from Europarl processing tool to generate words, which is not yet integrated in the follwing processing
- use self-defined word2ngram function to generate words
- due to CPU|GPU|memory constraints and keras LSTM batch training requirements, padding or cliping each document to max_n_words = 100 first words 


```python
def word2ngram(word, min_n = None):
    if not word:
        return []
    min_n = min_n if min_n else len(word)-3
    ngram = []
    for n in range(len(word), min_n-1, -1):
        for i in range(0, len(word)-n+1):
            ngram.append(word[i:i+n])
    return ngram
```


```python
max_n_words = 100

documents = reuters.fileids()
docs_train = [d for d in documents if d.startswith('train')]
docs_test = [d for d in documents if d.startswith('test')]

category2id = {}
for i, c in enumerate(reuters.categories()):
    category2id[c] = i
n_categories = len(category2id)

data_train = {'xs': [], 'ys': []}
data_test = {'xs': [], 'ys': []}

tknzr = TweetTokenizer(preserve_case=0)

for data, docs in [(data_train, docs_train), (data_test, docs_test)]:
    for doc in docs:
        x = []
        words = tknzr.tokenize(reuters.raw(doc)) # how the words are preprocessed in the pretrained embeddings?
        for idx in range(max_n_words):
            flag = 0
            word = words[idx] if idx < len(words) else None
            for ngram in word2ngram(word):
                if ngram in en_word2id:
                    x.append(en_embeddings[en_word2id[ngram]])
                    flag = 1
                    break
            if not flag:
                x.append(np.zeros(emb_dim))
        data['xs'].append(x)

        y = [0] * n_categories
        for c in reuters.categories(doc):
            y[category2id[c]] = 1
        data['ys'].append(y)
    data['xs'] = np.array(data['xs'])
    data['ys'] = np.array(data['ys'])
```

## 3. train a multi-label classification model

- train a 2 layer LSTM for claasification
- multi-label error function is `binary_crossentropy` applied on sigmoid output layer
- using sklearn `label_ranking_average_precision_score` to evaluate the performance after each epoch since the binary_accuracy is misleading in this case - naive predictor got an accuracy of 98%


```python
# model configs - 2 layer LSTM with sigmoid as output activation for binary_crossentropy loss
emb_dropout_factor = 0.4
recurrent_dropout_factor = 0.2
LSTM_dropout_factor = 0.2
layer_dropout_factor = 0.2
LSTM_units = 128
lr = 0.001
lr_decay = 0.95
batch_size = 512
n_epochs = 5
```


```python
from keras.callbacks import Callback
from sklearn.metrics import label_ranking_average_precision_score

class LRAP(Callback):
    def on_epoch_end(self, epoch, logs={}):
        y_pred = (np.asarray(self.model.predict(self.validation_data[0])))   
        y_true = self.validation_data[1]
        val_LRAP = label_ranking_average_precision_score(y_true, y_pred)
        print " - val_LRAP: %f" %(val_LRAP)
        return
```


```python
model = Sequential()

model.add(Dropout(emb_dropout_factor, input_shape=(max_n_words, emb_dim)))
model.add(LSTM(LSTM_units, return_sequences=True, recurrent_dropout=recurrent_dropout_factor, dropout=LSTM_dropout_factor))
model.add(Dropout(layer_dropout_factor))
model.add(LSTM(LSTM_units, recurrent_dropout=recurrent_dropout_factor, dropout=LSTM_dropout_factor))
model.add(Dropout(layer_dropout_factor))
model.add(Dense(n_categories))
model.add(Activation('sigmoid'))

model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['binary_accuracy'])

# Train model
model.fit(data_train['xs'], data_train['ys'], batch_size=batch_size, epochs=n_epochs, 
          validation_data=(data_test['xs'], data_test['ys']), callbacks=[LRAP()])
```

    Train on 7769 samples, validate on 3019 samples
    Epoch 1/5
    7769/7769 [==============================] - 101s 13ms/step - loss: 0.5028 - binary_accuracy: 0.8632 - val_loss: 0.1666 - val_binary_accuracy: 0.9862
     - val_LRAP: 0.150455
    Epoch 2/5
    7769/7769 [==============================] - 91s 12ms/step - loss: 0.1061 - binary_accuracy: 0.9848 - val_loss: 0.0625 - val_binary_accuracy: 0.9862
     - val_LRAP: 0.503258
    Epoch 3/5
    7769/7769 [==============================] - 81s 10ms/step - loss: 0.0599 - binary_accuracy: 0.9858 - val_loss: 0.0534 - val_binary_accuracy: 0.9862
     - val_LRAP: 0.527394
    Epoch 4/5
    7769/7769 [==============================] - 81s 10ms/step - loss: 0.0544 - binary_accuracy: 0.9860 - val_loss: 0.0516 - val_binary_accuracy: 0.9862
     - val_LRAP: 0.533624
    Epoch 5/5
    7769/7769 [==============================] - 83s 11ms/step - loss: 0.0528 - binary_accuracy: 0.9859 - val_loss: 0.0509 - val_binary_accuracy: 0.9862
     - val_LRAP: 0.534528





    <keras.callbacks.History at 0x11cc84a50>




```python
# Evaluate model
score, acc = model.evaluate(data_test['xs'], data_test['ys'], batch_size=batch_size)
y_pred = model.predict(data_test['xs'])
    
print('Score: %1.4f' % score)
print('Accuracy: %1.4f' % acc)
print('LRAP: %1.4f' % label_ranking_average_precision_score(data_test['ys'], y_pred))
```

    3019/3019 [==============================] - 9s 3ms/step
    Score: 0.0509
    Accuracy: 0.9862
    LRAP: 0.5345


## 4. Extend the model to french 

- not implemented due to lack of GPU for the moment
- generate the mapping from french word embeddings to english by superviesd (Procrustes problem) or unsupervised (adversairal learning + refinement)
- map french text to vectors in english embeddings space and feed to the above model for classification
