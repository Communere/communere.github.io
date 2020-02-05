---
layout: post
title: Dealing with categorical features
categories: [ML, clustering, Unsupervised learning, Python]
datetime: 06-11-2019
author: Sepideh Nazemi
---

Encoding Categorical features is one of the fundamental feature engineerings processes when using machine learning techniques. There are different techniques and packages to perform Encoding, but it can be not very clear which one to use or what exactly their differences are.
Here, I am going to give a quick review of most common encoding techniques and how to apply them on the data.
The dataset, I am using to demonstrate, is Titanic dataset available on Kaggle.

## outline

- OneHot Encoding
- Lable Encodeing
- Get_dummiesg
- DictVectorizer
- OrdinalEncoder
- FeatureHashing

### OneHot Encoding

Provided by **Scikit-learn** library, it utilises the one-hot encoding (dummy) scheme. This transformer needs array-like of integers or strings as an input. The output will be a sparse matrix or an array with binary columns for each value of the categorical features.

We need to extract the categorical features and data from the dataset. Then, instantiate OneHotEncoder and fit and transform the data into the OneHotEncoder to transform the categorical data into 0/1 code.

In this example `df_categorical_data` contains 2 columns namely `'Sex'` and `'Deck'` where `'Sex'` can have female or male and Deck can have A, B, C, D, E, F, HA value respectively.

```
# import OneHotEncoder
from sklearn.preprocessing import OneHotEncoder
# instantiate OneHotEncoder
ohe = OneHotEncoder(categories='auto', sparse=False )

# apply OneHotEncoder on categorical feature columns
Categorical_ohe = ohe.fit_transform(df_categorical_data) # It returns an numpy array
print(Categorical_ohe.shape)
Categorical_ohe
```

<figure>
<img src="/images/EncodingImages/ohe-arrayOutput.JPG" />

  <figcaption style="text-align: center;"></figcaption>
</figure>

The output is an **array** with 9 columns covering all possible values that `'Sex'` and `'Deck'` can get.
We can retrive the name of the columns and convert the array to a dataframe if necesserly by utilising `get_feature_names()` attribute of the Onehot Encoder and DataFrame method of Panda API as follows:

```
cols = ohe.get_feature_names()  #get the column names using .get_feature_names() attribute.
df_Categorical_ohe = pd.DataFrame(Categorical_ohe, columns=cols)  #convert the result matrix to datafarem (if necessary)
```

figure>
<img src="/images/EncodingImages/ohe-array-dataframe.JPG" />

  <figcaption style="text-align: center;"> </figcaption>
</figure>

Note that when we use One-hot Encoding on a single column DataFrame, we need to let Numpy reshapes the DataFrame to make it compatible with the input format.

```
# apply OneHotEncoder on one categorical feature
Categorical_ohe1 = ohe.fit_transform(df['Embarked'].values.reshape(-1, 1)) # It returns an numpy array
# by using reshape(-1, 1), we indicate that we have 1 column but rows are unknown and we want that numpy figures the rows out such that it is compatible with the original shape.
Categorical_ohe1
```

<figure>
<img src="/images/EncodingImages/ohe-single-feature.JPG" />

  <figcaption style="text-align: center;"> </figcaption>
</figure>

### Lable Encodeing

Provided by **Scikit-learn** library, this class of transformer can be used to transform numerical and non-numerical labels into numerical labels. In contrast with one-hot Encoder, which provides a one-hot coding scheme (‘one-of-K’), LableEncoder provides labels with values between 0 and n_classes-1. It is advised to use this Encoder for target values rather than input values.
The output type of lableEncoder is similar to its input, but you are restricted to the [array-like](https://scikit-learn.org/stable/glossary.html) of shape(n_sample,). So, in case, you want to encode a DataFrame with 2 columns you should first convert it to an array-like of shape(n_sample,), perhaps using `raval`. I provide a couple of different scenario in the followings:

```
from sklearn.preprocessing import LabelEncoder

# instantiate a LabelEncoder object
le = LabelEncoder()

# feed a coulmn of the datafarme which has a binary class feature i.e male and female
df['Sex'] = le.fit_transform(df['Sex'])

# applying LabelEncoder on multiple class features
df['Deck'] = le.fit_transform(df['Deck'])

df[['Sex','Deck']].sample(6)

```

**Note that in this case the result is a DataFrame as was the input.**

<figure>
<img src="/images/EncodingImages/Labelencoder-ex1-dfsinglecolumn-multi-singleclass.JPG" />

  <figcaption style="text-align: center;"> </figcaption>
</figure>
 
 To demonstrate how to deal with other shapes of input, I applied LabelEncoder directly to a DataFrame containing 2 columns. The following code shows different stage results for clarity.

```
df_categorical_data = df[['Sex', 'Deck']]  # extracting just 2 of categorical features from the main dataframe
#Convert the dataframe into an array-like of (n_shape,) utilising raval()
df_categorical_data_arryLike = np.ravel(df_categorical_data)
print('The output is an array of shape {} as follows \n {}'.format(df_categorical_data_arryLike.shape, df_categorical_data_arryLike))

#apply fit_transform on the result above
eEncoded_Sex_Deck = le.fit_transform(df_categorical_data_arryLike)

print('The output is an array of shape {} as follows \n {}'.format(eEncoded_Sex_Deck.shape, eEncoded_Sex_Deck))
print(eEncoded_Sex_Deck.shape)

#we can reshape the result utilising .reshape so, the result make better sences
leEncoded_Sex_Deck.reshape(1309,2)
```

<figure>
<img src="/images/EncodingImages/Labelencoder-2dimentiondataframe.JPG" />

  <figcaption style="text-align: center;"></figcaption>
</figure>

Using LabelEncoder for multi-class features may not be the right choice as it may unintentionally bring the notion of ordering among values. For example, in our case, Deck F and B have been encoded by 5 and 1 respectively; However, we can not say F is higher than B. This may result into poor performance in the proposed machine learning model.

### Get_dummies

Provided by **Panda** library, is the most straightforward encoding model. It is intelligent enough to recognise all the categorical features at once in the DataFrame and provide a one-hot scheme code for the categorical data. However, you may not want to encode everything as `get_dummies` does!
To demoneste, I've extracted the features that I was willing to encode them.

```
import pandas as pd
df_categorical_data = df[['Sex', 'Deck','Embarked']]
pd.get_dummies(df_categorical_data)

```

<figure>
<img src="/images/EncodingImages/Get_dummies.JPG" />

  <figcaption style="text-align: center;"></figcaption>
</figure>

### DictVectorizer

Provided by **Scikit-learn** library, this class of transformer can provide binary one-hot code for non-numerical (string) labels; For the numerical labels, it is required to follow the encoding by OneHotEncoding model to complete the encoding process.
The input of this transformer must be a dictionary-like (feature-value) objects, and output will be an array or a scipy.sparse metrix.

Following our example, first, we need to convert the DataFarme to a dictionary-like object as follows. Note that DictVectorizer is intelligent enough to recognise categorical features in the DataFrame. However, for the sake of presentation, I've extracted a couple of categorical features from the main DataFarme.

```
# convert datafarme to dictionary
df_categorical_data = df[['Sex','Embarked']]
df_categorical_data_dic = df_categorical_data.to_dict(orient='records') # turn each row as key-value pairs
# print
df_categorical_data_dic
```

<figure>
<img src="/images/EncodingImages/DicExample.JPG" />

  <figcaption style="text-align: center;"></figcaption>
</figure>

Importing the required library, we can initiate a DictVectorizer object and apply it's fit_transform on the dictionary.

```
from sklearn.feature_extraction import DictVectorizer
# instantiate a Dictvectorizer object
dv_df_categorical_data = DictVectorizer(sparse=False)
# apply Dictvectorizer on the Dictionary
df_categorical_data_encoded = dv_df_categorical_data.fit_transform(df_categorical_data_dic)
print (df_categorical_data_encoded)
```

<figure>
<img src="/images/EncodingImages/DicvectorizerOutput.JPG" />

  <figcaption style="text-align: center;"></figcaption>
</figure>

As you can see, the output is an array. To interpret the output, it is beneficial to indicate which column represents which feature by utilising `vocabulary_` attribute as follow:

```
# vocabulary
vocab = dv_df_categorical_data.vocabulary_
# show vocab
vocab
```

<figure>
<img src="/images/EncodingImages/VocabOutput.JPG" />

  <figcaption style="text-align: center;"></figcaption>
</figure>

### OrdinalEncoder

Provided by **Scikit-learn** library, this transformer provides a single column of integers (0 to n_categories - 1) per each feature. Note that this transformer can not distinguish between categorical and non-categorical features. So, it is beneficial to extract the categorical features that you want to encode before starting the encoding process.
The input to this transformer should be an array-like of the categorical features with integers or strings values. The output will be an array. Note that estimators may interpret the order among values as an actual order among the categorical feature values which is not often the case and leed to poor performance or estimation.

```
df_categorical_data = df[['Sex','Deck','Embarked']]
from sklearn.preprocessing import OrdinalEncoder

# instantiate a LabelEncoder object
enc = OrdinalEncoder()

##apply fit_transform on the categorical featurs
enc.fit_transform(df_categorical_data)
```

<figure>
<img src="/images/EncodingImages/OrdinalEncodeingExample.JPG" />

  <figcaption style="text-align: center;"></figcaption>
</figure>

To retrive encoded features, we can use `categories_` attribut. Also, it is possible to inverse the transformation to retrive the original values as follow:

```
enc.categories_
enc.inverse_transform(enc.fit_transform(df_categorical_data))

```

### FeatureHashing

Provided by **Scikit-learn** library, The class FeatureHasher utilises " Feature hashing" or "Hashing trick" and provides a high-speed and memory-efficient encoding approach. So, it can be an alternative to DictVectorizer encoder or CountVectorizer in a large, online learning scenario. By default, the input is a dictionary-like of the categorical features. However, it can accept single strings. Check this [link](https://scikit-learn.org/stable/modules/feature_extraction.html) for more information. The output always will be a scipy.spars matrix which we should define the number of its columns when initiating the encoder object. The number of columns should not be too small or large as it may cause low performance or collision in the result.

Following our example, we can apply FeatureHashing as follow. Note that we need to convert our categorical data into a Dictionary.

```
# convert datafarme to dictionary
df_categorical_data = df[['Deck','Sex']]
df_categorical_data_dic = df_categorical_data.to_dict(orient='records') # turn each row as key-value pairs
#df_categorical_data_dic


from sklearn.feature_extraction import FeatureHasher
# instantiate a FeatureHasher object with the number of output columns
h = FeatureHasher(n_features=10)
#apply fit_transform on the categorical featurs dictionary
f = h.fit_transform(df_categorical_data_dic)
f.toarray()[:20]

```

<figure>
<img src="/images/EncodingImages/FeatureHashing.JPG" />

  <figcaption style="text-align: center;"></figcaption>
</figure>

In contrast with "OrdinalEncoder", FeatureHashing does not have inverse_transform attribute as it does not remember what the input feature looked like. That is the price paid for its high speed and low memory usage.

## Conclusion

As demonstrated, there are a variety of standard and typical encoding schemes where you can decide based on their required input or provided output type, speed and memory efficiency, their simplicity and more importantly your categorical data essence. It is also possible to combine different schemes or come up with your idea to encode the categorical data. You can check this [Link](https://www.kaggle.com/shahules/an-overview-of-encoding-techniques) for more idea about encoding the categorical features.
