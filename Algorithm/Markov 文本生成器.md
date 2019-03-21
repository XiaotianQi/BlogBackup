Markov 文本生成器（Markov text generator）是基于分析大量随机事件的马尔科夫模型。随机的特点是一个离散事件发生后，另一个离散事件将在前一个事件的条件下，以一定的概率发生。

## 马尔可夫链

### 定义

维基百科上的定义如下：

> 马尔可夫链（Markov chain），又称离散时间马尔可夫链（discrete-time Markov chain，缩写为DTMC），因俄国数学家安德烈·马尔可夫得名，为**状态空间中经过从一个状态到另一个状态的转换的随机过程**。该过程要求具备**“无记忆”**的性质：**下一状态的概率分布只能由当前状态决定**，在时间序列中它前面的事件均与之无关。这种特定类型的“无记忆性”称作**马尔可夫性质**。马尔科夫链作为实际过程的统计模型具有许多应用。
> 在马尔可夫链的每一步，系统根据概率分布，可以从一个状态变到另一个状态，也可以保持当前状态。状态的改变叫做转移，与不同的状态改变相关的概率叫做**转移概率**。随机漫步就是马尔可夫链的例子。随机漫步中每一步的状态是在图形中的点，每一步可以移动到任何一个相邻的点，在这里移动到每一个点的概率都是相同的（无论之前漫步路径是如何的）。 

马尔可夫链是具有马尔可夫性质的随机过程。这也意味着，状态改变是概率性的，未来的状态仅仅依赖当前的状态。

***

## Markov 文本生成器

### 训练模型示例

以如下语句训练模型：

```txt
I eat aplles.U eat bananas.
U eat bananas.I eat aplles.
```

状态转移：

![状态转移](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Algorithm/Markov.jpg)

状态分布：

||I|U|eat|apples|bananas|.|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|I|0|0|1|0|0|0|
|U|0|0|1|0|0|0|
|eat|0|0|0|0.5|0.5|0|
|apples|0|0|0|0|0|1|
|bananas|0|0|0|0|0|1|
|.| 0.5  |0.5|0|0|0|0|

***

### 代码实现

```python
from urllib.request import urlopen
from random import randint
import re

def buildWordDict(text):
    text = re.sub(r'\n', ' ', text)
    text = re.sub(r'\"', '', text)

    punctuation = [',', '.', ';', ':']
    for symbol in punctuation:
        text = text.replace(symbol, ' '+symbol+' ')
    
    words = text.split(' ')
    words = [word for word in words if word !='']

    wordDict = {}
    for i in range(len(words)-1):
        if words[i] not in wordDict:
            wordDict[words[i]] = {}
        if words[i+1] not in wordDict[words[i]]:
            wordDict[words[i]][words[i+1]] = 0
        wordDict[words[i]][words[i+1]] += 1
    
    return wordDict

def wordListSum(wordList):
    sum = 0
    for value in wordList.values():
        sum += value
    return sum

def retrieveRandomWord(wordList):
    randIndex = randint(1, wordListSum(wordList))
    for word, value in wordList.items():
        randIndex -= value
        if randIndex <= 0:
            return word

with open(r'C:\Users\bnwse\Desktop\inaugurationSpeech.txt') as f:
    text = f.read()
    wordDict = buildWordDict(text)

    length = 100
    chain = ""
    currentWord = "I"
    for i in range(0, length):
        chain += currentWord+" "
        currentWord = retrieveRandomWord(wordDict[currentWord])

    print(chain)

```

```txt
I possess a breath can conceive to be quite as well adapted as long as power : The broad foundation upon which they might be worthy representatives of its existence and to that instrument to you . When the others . Fellow-citizens , to prevent the careful culture of the lesser Asia would be , but by whom he is an argument as collisions of power : The sagacious mind of those of our ancestors derived from the support of our system . When the adoption of the complicated intrigues of any of its correction is harsh , placed in
```

***

由马尔可夫链生成的文本和真实文本之间的差异在于语法规则和语义含义。马尔可夫链在生成文本时足够好，有时会提供语法和语义上正确的语句。 



***



参考：



[Markov Chains Explained Visually](http://setosa.io/ev/markov-chains/)