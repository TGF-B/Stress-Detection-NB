# Stress Detection With BernouliNB
最近很流行玩人格测试，通过做一些题目然后判断出自己属于哪种人格，以便更快地找到适合自己的职业或者对象。在当下的快节奏社会，这确实是一个方便又有趣的测试方式。受此启发，我搭建一个分析情绪的预测模型，通过分析社交平台上的评论内容文本，来判断一下发帖人的情绪倾向。
## 导入数据集
```python
import pandas as pd
import numpy as np
data=pd.red_csv("https://raw.githubusercontent.com/amankharwal/Website-data/master/stress.csv") #reddit上的评论集
data.head()
```
## 数据预处理
```python
import nltk
import re
nltk.download('stopwords') #导入停止点，可以将句子分割成一个个单词
stemmer = nltk.SnowballStemmer("english")
from nltk.corpus import stopwords
import string
stopword=set(stopwords.words('english'))

def clean(text): #函数用来清除CSV文本中中含有的各种奇怪的符号
    text = str(text).lower()
    text = re.sub('\[.*?\]', '', text)
    text = re.sub('https?://\S+|www\.\S+', '', text)
    text = re.sub('<.*?>+', '', text)
    text = re.sub('[%s]' % re.escape(string.punctuation), '', text)
    text = re.sub('\n', '', text)
    text = re.sub('\w*\d\w*', '', text)
    text = [word for word in text.split(' ') if word not in stopword]
    text=" ".join(text)
    text = [stemmer.stem(word) for word in text.split(' ')]
    text=" ".join(text)
    return text
data["text"] = data["text"].apply(clean)
```
## 画云图
```python 
import matplotlib.pyplot as plt #用云图展现文本中各种关键词的出现频率
from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator
text = " ".join(i for i in data.text)
stopwords = set(STOPWORDS)
wordcloud = WordCloud(stopwords=stopwords, 
                      background_color="white").generate(text)
plt.figure( figsize=(15,10))
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.show()
```
![词云图](https://github.com/TGF-B/Stress-Detection-NB/blob/main/Figure_1.png)
## 模型搭建
  - 更改状态标注栏
  ```python
  data["label"] = data["label"].map({0: "No Stress", 1: "Stress"})#将测试集中的状态表述由符号改成文字。
  data = data[["text", "label"]]
  print(data.head())
  ```
   - 数据集分割
   ```python
   from sklearn.feature_extraction.text import CountVectorizer #CounterVectorizer负责将文本中的词语转换为词频矩阵，它通过fit_transform函数计算各个词语出现的次数
  from sklearn.model_selection import train_test_split
  x = np.array(data["text"])
  y = np.array(data["label"])
  cv = CountVectorizer()
  X = cv.fit_transform(x)
  xtrain, xtest, ytrain, ytest = train_test_split(X, y, 
                                                test_size=0.33, 
                                                random_state=42)
  ```
  - 用伯努利朴素贝叶斯搭建模型
  ```python
  from sklearn.naive_bayes import BernoulliNB
  model = BernoulliNB()
  model.fit(xtrain, ytrain)
  ```
## 测试
```python
user = input("Enter a Text: ")
data = cv.transform([user]).toarray()
output = model.predict(data)
print(output)
```
程序运行后输入一段话（英文），模型即可判断你的情绪是否有焦虑倾向
