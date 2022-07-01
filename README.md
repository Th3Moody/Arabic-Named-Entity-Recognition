# Arabic Named Entity Recognition

*A named entity recognition model for Arabic text to recognize persons, locations, and organizations*

## Abstract
This a model that aims at recognizing different entities (persons, locations, organizations, and generic miscellaneous) in a given Arabic text using natural language processing techniques and with the help of Long-Short Term Memory recurrent neural network. The model scored 94.8% accuracy on test data. 

## Dataset
The dataset used was the [AQMAR Arabic Wikipedia Named Entity Corpus](https://www.cs.cmu.edu/%7Eark/ArabicNER/) that was provided by Behrang Mohit, Nathan Schneider, Rishav Bhowmick, Kemal Oflazer, and Noah Smith as part of the AQMAR project.

The dataset is a collection of Arabic Wikipedia pages about different topics. Each topic is in a separate text file that contains one Arabic word per line and its entity tag.

The entity tags are:
-	B-PER and I-PER for persons
-	B-LOC and I-LOC for locations
-	B-ORG and I-ORG for organizations
-	B-MIS and I-MIS for generic miscellaneous
-	O which means out of the problem context

**Example**
```
العصر B-MIS1
الذهبي I-MIS1
للإسلام O
تمتد O
لغاية O
القرن O
الرابع O
عشر O
و O
الخامس O
عشر O
الميلادي O
. O
```

## Cleaning
The cleaning process was split into two operations; cleaning tags, and cleaning the text itself.
### Cleaning Tags
According to the dataset [documentation](https://www.cs.cmu.edu/%7Eark/ArabicNER/corpus/README):
> Each article was tagged by 1 of 2 annotators. Annotators were encouraged to devise up to 3 article-specific entity classes to supplement the traditional four (PERson, ORGanization, LOCation, and generic MIScellaneous); these custom categories are summarized below each article.

This resulted in more tags that we aren’t interested in such as: B-ENGLISH, or MISX (where X is a digit) that means different entity across topics.

A function was created to normalize all tags and fix misspelled tags as well.

### Cleaning Text
Arabic text cleaning is bigger than cleaning normal Latin text due to more letter forms, and more punctuation and tashkeel (تشكيل) characters.
**Cleaning Steps:**
-	Remove Tashkeel
-	Remove Longation
-	Normalizing Arabic words by removing/replacing some characters.

## Looking for useful insights
After some investigations it was found that the provided dataset suffers from an unbalanced data problem. The tags that we are interested in combined represent 12.7% only of the whole data, while the remaining 87.3% are tagged as ‘O’.

But this problem is due to the nature of data itself; a normal Arabic sentence would contain two or three nouns at most and the rest of the sentence is just verbs, adjectives, or other out of the problem context words.

Thus, I had to be very careful while choosing the right sequence size for the LSTM model. I have 2,687 sentences and the max sentence length was 290 words. After applying some recleaning process the max sentence length became 271 words, and 83% of the sentences are 40 words or less. So, I chose 40 to be the sequence length.

## The Model
The model consist of embedding layer, one LSTM layer, and a distributed output layer. For the embedding layer, I used [AraVec](https://github.com/bakrianoo/aravec) (300d) model weights. AraVec was provided by Abu Bakr Soliman, Kareem Eissa, and Samhaa R.El-Beltagy.

The model scored nearly 94.75 for all train, validation, and test data. Check model summary below:
```
Model: "sequential"
_________________________________________________________________
 Layer (type)                Output Shape              Param #   
=================================================================
 embedding (Embedding)       (None, 40, 300)           5244300   
                                                                 
 lstm (LSTM)                 (None, 40, 300)           721200    
                                                                 
 time_distributed (TimeDistr  (None, 40, 9)            2709      
 ibuted)                                                         
                                                                 
=================================================================
Total params: 5,968,209
Trainable params: 723,909
Non-trainable params: 5,244,300
```

## Examples

```python
lstm_predict('منشئ المسجد هو أحمد بن طولون مؤسس الدولة الطولونية في مصر والشام، تعود أصوله إلى قبيلة التغزغز التركية، وكانت أُسرته تقيم في بخاري.')
```
```
output:
منشئ : O
المسجد : O
هو : O
أحمد : B-PER
بن : I-PER
طولون : I-PER
مؤسس : O
الدولة : B-LOC
الطولونية : O
في : O
مصر : O
والشام، : B-LOC
تعود : O
أصوله : O
إلى : O
قبيلة : O
التغزغز : O
التركية، : O
وكانت : O
أُسرته : O
تقيم : O
في : O
بخاري. : B-LOC
```

---
**Tags:** *Python, Numpy, Regex, LSTM, RNN, NN, AraVec, Gensim, TensorFlow*
