`n-gram` 指文本中连续出现的 n 个语词。`n-gram` 语法模型是基于 (n-1) 阶马尔可夫链的一种概率语言模型，通过 n 个语词出现的概率来推断语句的结构。当 n 分别为 1、2、3 时，又分别称为unigram（一元语法）、bigram（二元语法）与 trigram（三元语法）。

基于 `n-gram`，对文字样本进行数据归纳。文字样本源自美国第九任总统威廉·亨利·哈里森的就职演说（http://www.pythonscraping.com/files/inaugurationSpeech.txt）。

***

## 建立 `2-gram` 模型

使用 `2-gram` 模型，统计 `2-gram` 序列的频率。

```python
# _*_coding:utf-8_*_

from urllib.request import urlopen
from bs4 import BeautifulSoup
import re
import string
import operator

# 清晰文本
def cleanInput(input):
    input = re.sub(r'\n+', ' ', input)
    input = re.sub(r'\s+', ' ', input)
    input = re.sub(r'\[[0-9]*\]', ' ', input)
    input = bytes(input, 'utf-8')
    input = input.decode('ascii', 'ignore')
    input = input.split(' ')
    cleanInput = []
    for item in input:
        item = item.strip(string.punctuation)
        if len(item) > 1 or (item.lower() == 'a' or item.lower() == 'i'):
            cleanInput.append(item)
    return cleanInput

# n-gram 模型
def ngrams(input, n):
    input = cleanInput(input)
    output = {}
    for i in range(len(input)+1-n):
        newNGram = ' '.join(input[i:i+n])
        if isCommon(newNGram):
            continue
        if newNGram not in output: 
            output[newNGram] = 0
        output[newNGram] += 1
    return output

content = str(urlopen("http://pythonscraping.com/files/inaugurationSpeech.txt").read(), 'utf-8')
ngrams = ngrams(content, 2)
sortedNGrams = sorted(ngrams.items(), key=operator.itemgetter(1), reverse=True)
print(sortedNGrams)

```

此时的输出结果：

```python
[('of the', 212), ('in the', 62), ('to the', 60), ('by the', 41), ('the Constitution', 32), ('of our', 29), ('to be', 26), ('from the', 24), ('the people', 23), ('that the', 23), ('of a', 22), ('and the', 22), ('may be', 19), ('of their', 19), ('of that', 18), ('for the', 16), ('of its', 16), ('of power', 16), ('have been', 16), ('is the', 16), ('with the', 15), ('it is', 15), ('all the', 14), ('which they', 13),
...
('their subjects', 2), ('can never', 2), ('of mine', 2), ('States but', 2), ('used in', 2), ('that character', 2), ('their former', 2), ('who has', 2), ('his own', 2), ('respects the', 2), ('free and', 2), ('authorities of', 2), ('as collisions', 2), ('their passions', 2), ('their country', 2), ('is to', 2), ('its existence', 2), ('all its', 2), ('its members', 2), ('whole The', 2), ('be exercised', 2),
```

其中，`of the`、`in the`、`to the` 等看起来并不重要。需要将其过滤。

***

## 美式英语语料库

美国杨百翰大学的语言学教授 Mark Davies 一直在维护当代美式英语语料库。借助语料库，可以分别出 “有意的” 和  “没意义的” 单词。现只采用其中前 100 个单词。

```python
# 常用语料库
def isCommon(ngram):
    commonWords = [
        "the", "be", "and", "of", "a", "in", "to", "have", "it", "i",
        "that", "for", "you", "he", "with", "on", "do", "say", "this", "they",
        "is", "an", "at", "but", "we", "his", "from", "that", "not", "by",
        "she", "or", "as", "what", "go", "their", "can", "who", "get", "if",
        "would", "her", "all", "my", "make", "about", "know", "will", "as", "up",
        "one", "time", "has", "been", "there", "year", "so", "think", "when", "which",
        "them", "some", "me", "people", "take", "out", "into", "just", "see", "him",
        "your", "come", "could", "now", "than", "like", "other", "how", "then", "its",
        "our", "two", "more", "these", "want", "way", "look", "first", "also", "new",
        "because", "day", "more", "use", "no", "man", "find", "here", "thing", "give",
    ]
    for word in ngram.split():
        if word in commonWords:
            return True
    return False
```

***

## 数据统计

再次基础上，添加一下功能：

* 获取 `n-gram` 语词所在语句
* 出现频率大于 2 的语词 及其 所在语句

```python
# 获取 n-gram 语词所在语句
def getFirstSentenceContaining(ngram, content):
    sentences = content.split(".")
    for sentence in sentences: 
        if ngram in sentence:
            return sentence
    return ""

# 出现频率大于2的语词 及其 所在语句
topWords=[]
for i in range(len(sortedNGrams)):
    if sortedNGrams[i][1] > 2:
        topWords.append(sortedNGrams[i][0])
print(topWords)

sentences = set()
for word in topWords:
    sentences.add(getFirstSentenceContaining(word, content))
for i in list(sentences):
    print(i)
```

获得出现频率大于 2 的语词如下：

```python
['United States', 'I shall', 'It was', 'It may', 'executive department', 'General Government', 'called upon', 'Chief Magistrate', 'country The', 'I conceive', 'Mr Jefferson', 'legislative body', 'same causes', 'Government should', 'whole country']
```

及其所在语句：

```txt
It may be observed, however, as a general remark, that republics can commit no greater error than to adopt or continue any feature in their systems of government which may be calculated to create or increase the lover of power in the bosoms of those to whom necessity obliges them to commit the management of their affairs; and surely nothing is more likely to produce such a state of mind than the long continuance of an office of high trust

It was the remark of a Roman consul in an early period of that celebrated Republic that a most striking contrast was observable in the conduct of candidates for offices of power and trust before and after obtaining them, they seldom carrying out in the latter case the pledges and promises made in the former

It could not but have occurred to the Convention that in a country so extensive, embracing so great a variety of soil and climate, and consequently of products, and

which from the same causes must ever exhibit a great difference in the amount of the population of its various sections, calling for a great diversity in the employments of the people, that the legislation of the majority might not always justly regard the rights and interests of the minority, and that acts of this character might be passed under an express grant by the words of the Constitution, and therefore not within the competency of the judiciary to declare void; that however enlightened and patriotic they might suppose from past experience the members of Congress might be, and however largely partaking, in the general, of the liberal feelings of the people, it was impossible to expect that bodies so constituted should not sometimes be controlled by local interests and sectional feelings

Called from a retirement which I had supposed was to continue for the residue of my life to fill the chief executive office of this great and free nation, I appear before you, fellow-citizens, to take the oaths which the Constitution prescribes as a necessary qualification for the performance of its duties; and in obedience to a custom coeval with our Government and what I believe to be your expectations I proceed to present to you a summary of the principles which will govern me in the discharge of the duties which I shall be called upon to perform


It may be said, indeed, that the Constitution has given to the Executive the power to annul the acts of the legislative body by refusing to them his assent

The presses in the necessary employment of the Government should never be used "to clear the guilty or to varnish crime

On the contrary, it is our duty to encourage them to the extent of our constitutional authority to apply their best means and cheerfully to make all necessary sacrifices and submit to all necessary burdens to fulfill their engagements and maintain their credit, for the character and credit of the several States form a part of the character and credit of the whole country

The Constitution of the United States is the instrument containing this grant of power to the several departments composing the Government
 Such a one was afforded by the executive department constituted by the Constitution

It would not become me to say that the fears of these patriots have been already realized; but as I sincerely believe that the tendency of measures and of men's opinions for some years past has been in that direction, it is, I conceive, strictly proper that I should take this occasion to repeat the assurances I have heretofore given of my determination to arrest the progress of that tendency if it really exists and restore the Government to its pristine health and vigor, as far as this can be effected by any legitimate exercise of the power placed in my hands

The General Government has seized upon none of the reserved rights of the States

Although the fiat of the people has gone forth proclaiming me the Chief Magistrate of this glorious Union, nothing upon their part remaining to be done, it may be thought that a motive may exist to keep up the delusion under which they may be supposed to have acted in relation to my principles and opinions; and perhaps there may be some in this assembly who have come here either prepared to condemn those I shall now deliver, or, approving them, to doubt the sincerity with which they are now uttered
```

***

参考：

《python 网络数据采集》