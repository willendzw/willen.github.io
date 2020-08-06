---
layout: post
title:  "cmusphinx pocketSphinx 识别特定词汇"
author: Willen
categories: [ 语音识别 ]
tags: [ cmuphinx ]
comments: false
rating: false
---

# cmusphinx pocketSphinx 识别特定词汇

示例可执行程序pocketsphinx_continuous可以输入以下参数：

```
-infile 
设置需要识别的声音文件。例如：file.wav
-hmm 
设置指定的HMM模型，也就是声学模型。例如：model\en-us\en-us 
-lm 
设置指定的语言模型，例如：model\en-us\en-us.lm.bin
-dict 
设置指定的词典，例如：model\en-us\cmudict-en-us.dict
-keyphrase
设置关键词，例如：-keyphrase "oh mighty computer"
-kws
设置关键词列表，
请注意-kws与-lm和-jsgf选项冲突。您不能同时指定两者。
-kws_threshold
关键词识别阈值。必须为每个关键字指定阈值。对于较短的短语，您可以使用较小的阈值，例如`1e-1`，对于较长的短语，阈值必须更大，最大为`1e-50`。
-inmic 
设置输入源为麦克风，yes or no。

例子：
pocketsphinx_continuous -hmm model\zh-cn\
```



## Q: How to implement “Hot word listening”

CMUSphinx implements keyphrase spotting mode in pocketsphinx decoder. To recognize a single keyphrase you can run decoder in “keyphrase search” mode.

From command line try:

```
pocketsphinx_continuous -infile file.wav -keyphrase "oh mighty computer" -kws_threshold 1e-20
```

From the code:

```
ps_set_keyphrase(ps, "keyphrase_search", "oh mighty computer");
ps_set_search(ps, "keyphrase_search);
ps_start_utt();
/* process data */
```

You can also find examples for Python and Android/Java in our sources.

Threshold must be tuned for every keyphrase on a test data to get the right balance missed detections and false alarms. You can try values like 1e-5 to 1e-50.

For the best accuracy it is better to have keyphrase with 3-4 syllables. Too short phrases are easily confused.

You can also search for multiple keyphrase, create a file keyphrase.list like this:

```
oh mighty computer /1e-40/
hello world /1e-30/
other_phrase /other_phrase_threshold/
```

And use it in decoder with -kws configuration option.

```
pocketsphinx_continuous -inmic yes -kws keyphrase_list
```

This feature is not yet implemented in sphinx4 decoder.

# 建立语言模型

------

- 关键字清单
  - [在PocketSphinx中使用关键字列表](https://cmusphinx.github.io/wiki/tutoriallm/#using-keyword-lists-with-pocketsphinx)
- 文法
  - [建立语法](https://cmusphinx.github.io/wiki/tutoriallm/#building-a-grammar)
  - [在PocketSphinx中使用语法](https://cmusphinx.github.io/wiki/tutoriallm/#using-your-grammar-with-pocketsphinx)
- 语言模型
  - [建立统计语言模型](https://cmusphinx.github.io/wiki/tutoriallm/#building-a-statistical-language-model)
  - [文字准备](https://cmusphinx.github.io/wiki/tutoriallm/#text-preparation)
  - [使用SRILM训练ARPA模型](https://cmusphinx.github.io/wiki/tutoriallm/#training-an-arpa-model-with-srilm)
  - [用CMUCLMTK训练ARPA模型](https://cmusphinx.github.io/wiki/tutoriallm/#training-an-arpa-model-with-cmuclmtk)
  - [使用Web服务构建简单的语言模型](https://cmusphinx.github.io/wiki/tutoriallm/#building-a-simple-language-model-using-a-web-service)
  - [使用其他语言模型工具箱](https://cmusphinx.github.io/wiki/tutoriallm/#using-other-language-model-toolkits)
  - [将模型转换为二进制格式](https://cmusphinx.github.io/wiki/tutoriallm/#converting-a-model-into-the-binary-format)
  - [在PocketSphinx中使用您的语言模型](https://cmusphinx.github.io/wiki/tutoriallm/#using-your-language-model-with-pocketsphinx)
  - [在Sphinx4中使用您的语言模型](https://cmusphinx.github.io/wiki/tutoriallm/#using-your-language-model-with-sphinx4)

------

语言模型是配置的重要组成部分，它告诉解码器哪些单词序列可以识别。

有几种类型的模型：关键字列表，语法和统计语言模型以及语音语言模型。它们具有不同的功能和性能属性。您可以根据需要选择任何解码模式，甚至可以在运行时在各种模式之间切换。有关更多详细信息，请参见[Pocketsphinx教程](https://cmusphinx.github.io/wiki/tutorialpocketsphinx/#searches)。

## 关键字清单

Pocketsphinx支持关键字发现模式，您可以在其中指定要查找的关键字列表。此模式的优点是您可以为每个关键字指定阈值，以便可以连续语音检测到关键字。所有其他模式都将尝试从语法中检测单词，即使您使用的不是语法中的单词也是如此。典型的关键字列表如下所示：

```
oh mighty computer /1e-40/
hello world /1e-30/
other phrase /1e-20/
```

必须为每个关键字指定阈值。对于较短的短语，您可以使用较小的阈值，例如`1e-1`，对于较长的短语，阈值必须更大，最大为`1e-50`。如果您的关键短语很长（大于10个音节），建议将其拆分并分别为每个部分定位。必须调整阈值，以在错误警报和错过的检测之间取得平衡。最好的方法是使用预先录制的音频文件。常见的调整过程如下：

1. 长时间录制，很少出现关键字和其他声音。您可以拍摄电影声音或其他声音。音频的长度应约为1小时。

2. 在该文件上运行关键字发现，每个关键字的阈值不同，请使用以下命令：

   ```
    pocketsphinx_continuous -infile <your_file.wav> -keyphrase <your keyphrase> \
     -kws_threshold <your_threshold> -time yes
      pocketsphinx_continuous -infile 1.wav -keyphrase "Wi-fi on" \
     -kws_threshold <your_threshold> -time yes
   ```

   该命令将打印许多行，其中一些是具有检测时间和置信度的关键字。您还可以选择禁用其他日志，`-logfn your_file.log`以避免混乱。

3. 从关键字发现结果中算出您遇到了多少次虚假警报和错过的检测。

4. 选择错误警报和漏检最少的阈值。

为了获得最佳准确性，最好将关键字短语包含3-4个音节。太短的短语很容易混淆。

仅Pocketsphinx支持关键字列表，sphinx4无法处理它们。

### 在PocketSphinx中使用关键字列表

要在命令行中使用关键字列表，请使用`-kws`选项指定它。您还可以使用一个`-keyphrase`选项来指定单个关键字。

在Python中，您可以在配置对象中指定选项，也可以为关键字短语添加命名搜索：

```
decoder.set_kws('keyphrase', kws_file)
decoder.set_search('keyphrase')
```

在Android中，它看起来类似于：

```
recognizer.setKws('keyphrase', kwsFile);
recognizer.startListening('keyphrase')
```

请注意`-kws`与`-lm`和`-jsgf`选项冲突。您不能同时指定两者。

## 文法

语法描述了一种非常简单的命令和控制语言。它们通常是手工编写的或在代码中自动生成的。语法通常没有单词序列的概率，但是可能会加权某些元素。它们可以使用Java语音语法格式（[JSGF](https://en.wikipedia.org/wiki/JSGF)）创建，并且通常具有*.gram*或*.jsgf之*类的文件扩展名。

语法允许您非常精确地指定可能的输入，例如，某个单词只能重复两次或三遍。但是，如果您的用户不小心跳过了语法要求的单词，那么这种严格性可能有害。在这种情况下，整个识别将失败。因此，最好使语法更灵活。除了列出短语以外，还可以列出允许任意排序的单词。避免使用许多规则和案例的非常复杂的语法。这只会减慢识别器的速度，您可以改用简单的规则。过去，语法需要花费大量精力进行调整，正确分配变体等。庞大的VXML咨询行业就是如此。

### 建立语法

语法通常以Java语音语法格式（JSGF）手动编写：

```
#JSGF V1.0;

grammar hello;
public <greet> = (good morning | hello) ( bhiksha | evandro | rita | will );
```

有关JSGF的更多信息，请参见[W3C](http://www.w3.org/TR/jsgf/)的 [完整文档](http://www.w3.org/TR/jsgf/)。

### 在PocketSphinx中使用语法

要在命令行中使用语法，请使用`-jsgf`选项将其指定。

在Python中，您可以在配置对象中指定选项，也可以添加语法的命名搜索：

```
decoder.set_jsgf('grammar', jsgf_file)
decoder.set_search('grammar')
```

在Android中，这看起来很相似：

```
recognizer.setJsgf('grammar', jsgfFile);
recognizer.startListening('grammar')
```

请注意`-jsgf`与`-kws`和`-jsgf`选项冲突。您不能同时指定两者。

## 语言模型

统计语言模型描述了更复杂的语言。它们包含单词和单词组合的概率。这些概率是根据样本数据估算的，并自动具有一定的灵活性。词汇表中的每种组合都是可能的，尽管每种组合的概率都会变化。例如，如果您从单词列表中创建统计语言模型，即使您并非故意这样做，它仍将允许解码单词组合。

总体而言，建议将统计语言模型用于自由格式输入，其中用户可以用自然语言说任何话。与语法相比，它们需要的工程工作量更少。您只列出可能的句子。例如，您可能会列出“ 21”和“ 33”之类的数字，而统计语言模型也将以一定的概率允许“ 31”。

通常，现代语音识别界面趋于更加自然，并避免了上一代的命令和控制风格。因此，大多数界面设计人员更喜欢使用统计语言模型来识别自然语言，而不是使用老式的VXML语法。

关于设计VUI界面的主题，您可能会对以下书籍感兴趣：[成为一台比坏人更好的机器：在Jetsonian时代的黄昏时的语音识别和其他异国用户界面，](http://www.amazon.com/Better-Good-Machine-Than-Person/dp/1932558098) 作者Bruce Balentine。

建立统计语言模型的方法有很多。当您的数据集很大时，使用CMU语言建模工具包是有意义的。当模型较小时，可以使用快速的在线Web服务。如果您需要特定的选项，或者只想使用自己喜欢的工具构建ARPA模型，也可以使用它。

语言模型可以三种不同的格式存储和加载：*文本 [ARPA](https://cmusphinx.github.io/wiki/arpaformat)*格式，*二进制BIN*格式和*二进制DMP*格式。ARPA格式占用更多空间，但是可以对其进行编辑。ARPA文件具有`.lm`扩展名。二进制格式占用的空间少得多，并且加载速度更快。二进制文件具有`.lm.bin`扩展名。也可以在这些格式之间进行转换。DMP格式已过时，不建议使用。

### 建立统计语言模型

### 文字准备

首先，您需要准备大量干净的文本。展开缩写，将数字转换为单词，清除非单词项目。例如，清理Wikipedia XML转储，您可以使用特殊的Python脚本，例如[Wikiextractor](https://github.com/attardi/wikiextractor)。要清理HTML页面，您可以尝试 [BoilerPipe](http://code.google.com/p/boilerpipe)。这是一个很好的软件包，专门用于从HTML提取文本。

有关如何根据Wikipedia文本创建语言模型的示例，请参见此[博客文章](http://trulymadlywordly.blogspot.ru/2011/03/creating-text-corpus-from-wikipedia.html)。电影字幕也是口语的好来源。

完成语言建模过程后，请将您的语言模型提交到CMUSphinx项目。我们很乐意分享！

普通话和其他类似语言的语言建模与英语基本相同，但需要另外考虑。区别在于输入文本必须按单词分段。提供了一个分段工具和一个相关的单词列表来完成此任务。

### 使用SRILM训练ARPA模型

使用SRI语言建模工具包（[SRILM](http://www.speech.sri.com/projects/srilm/)）训练模型很容易。这就是为什么我们推荐它。此外，SRILM是迄今为止最先进的工具包。要训练模型，可以使用以下命令：

```
ngram-count -kndiscount -interpolate -text train-text.txt -lm your.lm
```

您可以在以后修剪模型以减小模型的大小：

```
ngram -lm your.lm -prune 1e-8 -write-lm your-pruned.lm
```

经过训练后，值得在测试数据上测试模型的复杂性：

```
ngram -lm your.lm -ppl test-text.txt
```

### 用CMUCLMTK训练ARPA模型

您需要下载并安装CMUSphinx（CMUCLMTK）的语言模型工具包。有关详细信息，请参见[下载页面](https://cmusphinx.github.io/wiki/download)。

创建语言模型的过程如下：

1）准备将用于生成语言模型的参考文本。语言模型工具箱期望其输入为规范化文本文件的形式，并以`<s>`和`</s>` 标记分隔。许多输入过滤器可用于特定的语料库，例如总机，ISL和NIST会议以及HUB5笔录。结果应该是由句子的开始和结束标记界定的一组句子：`<s>`和`</s>`。这是一个例子：

```
<s> generally cloudy today with scattered outbreaks of rain and drizzle
persistent and heavy at times </s>
<s> some dry intervals also with hazy sunshine especially in eastern parts in
the morning </s>
<s> highest temperatures nine to thirteen Celsius in a light or moderate mainly
east south east breeze </s>
<s> cloudy damp and misty today with spells of rain and drizzle in most places
much of this rain will be light and patchy but heavier rain may develop in the
west later </s>
```

更多数据将产生更好的语言模型。`weather.txt`sphinx4中的文件（用于生成天气语言模型）包含近100,000个句子。

2）生成词汇文件。这是文件中所有单词的列表：

```
 text2wfreq < weather.txt | wfreq2vocab > weather.tmp.vocab
```

3）您可能需要编辑词汇文件以删除单词（数字，拼写错误，名字）。如果发现拼写错误，则最好在输入笔录中修复它们。

4）如果您要使用封闭的词汇表语言模型（这种语言模型没有为未知单词提供条件），则应从输入成绩单中删除句子，这些句子包含不在词汇表文件中的单词。

5）使用以下命令生成ARPA格式语言模型：

```
  text2idngram -vocab weather.vocab -idngram weather.idngram < weather.closed.txt
  idngram2lm -vocab_type 0 -idngram weather.idngram -vocab weather.vocab -arpa weather.lm
```

6）生成CMU二进制形式（BIN）：

```
 sphinx_lm_convert -i weather.lm -o weather.lm.bin
```

### 使用Web服务构建简单的语言模型

如果您的语言是英语且文本很小，则有时使用Web服务来构建它会更方便。以这种方式构建的语言模型对于简单的命令和控制任务非常有用。首先，您需要创建一个语料库。

“语料库”只是用于训练语言模型的句子列表。例如，我们将假设的语音控制任务用于移动Internet设备。我们想告诉它“打开浏览器”，“新电子邮件”，“前进”，“后退”，“下一个窗口”，“最后一个窗口”，“打开音乐播放器”之类的内容。因此，我们将从创建一个名为的文件开始`corpus.txt`：

```
open browser
new e-mail
forward
backward
next window
last window
open music player
```

然后转到[LMTool页面](http://www.speech.cs.cmu.edu/tools/lmtool-new.html)。只需单击*“浏览...”*按钮，选择`corpus.txt`您创建的文件，然后单击*“ COMPILE KNOWLEDGE BASE”*。

您应该看到一个包含一些状态消息的页面，然后是一个名为*“ Sphinx知识库”*的页面。该页面将包含标题为*“字典”*和*“语言模型”的链接*。下载这些文件并记下它们的名称（它们应由4位数字组成，后跟扩展名 `.dic`和`.lm`）。现在，您可以使用PocketSphinx测试新创建的语言模型。

### 使用其他语言模型工具箱

有许多工具包可从文本文件创建ARPA n-gram语言模型。

您可以尝试一些工具包：

- [国税局](https://sourceforge.net/projects/irstlm/)
- [MITLM](http://code.google.com/p/mitlm/)

如果您正在训练大型词汇语音识别系统，则将在[单独的页面中](https://cmusphinx.github.io/wiki/tutoriallmadvanced)概述[有关大规模语言模型](https://cmusphinx.github.io/wiki/tutoriallmadvanced)的语言模型训练。

创建ARPA文件后，您可以将模型转换为二进制格式，以加快加载速度。

### 将模型转换为二进制格式

为了快速加载大型模型，您可能希望将它们转换为二进制格式，这样可以节省解码器的初始化时间。对于小型模型，这不是必需的。Pocketsphinx和sphinx3可以使用该`-lm`选项处理它们两者。Sphinx4通过lm文件的扩展名自动检测格式。

ARPA格式和BINARY格式可以相互转换。您可以使用`sphinx_lm_convert`sphinxbase中的命令生成另一个文件：

```
sphinx_lm_convert -i model.lm -o model.lm.bin
sphinx_lm_convert -i model.lm.bin -ifmt bin -o model.lm -ofmt arpa
```

您也可以通过这种方式将旧的DMP模型转换为二进制格式。

在下一节中，我们将介绍如何使用，测试和改进您创建的语言模型。

### 在PocketSphinx中使用您的语言模型

如果安装了PocketSphinx，将有一个名为的程序`pocketsphinx_continuous`，可以从命令行运行该程序 以识别语音。假设它是安装在 `/usr/local`，你的语言模型和字典被称为`8521.dic`和`8521.lm`并放置在当前文件夹，请尝试运行下面的命令：

```
pocketsphinx_continuous -inmic yes -lm 8521.lm -dict 8521.dic
```

这将使用您的新语言模型，字典和默认声学模型。在Windows上，还必须使用以下`-hmm`选项指定声学模型文件夹：

```
bin/Release/pocketsphinx_continuous.exe -inmic yes -lm 8521.lm -dict 8521.dic -hmm model/en-us/en-us
```

您会看到很多诊断消息，然后是暂停，然后是输出 *“ READY…”*。现在，您可以尝试讲一些命令。它应该能够完全准确地识别它们。否则，您的麦克风或声卡可能会出现问题。

### 在Sphinx4中使用您的语言模型

在Sphinx4高级API中，您需要在配置中指定语言模型的位置：

```
configuration.setLanguageModelPath("file:8754.lm");
```

如果模型在资源中，则可以使用以下命令进行引用`"resource:URL"`：

```
configuration.setLanguageModelPath("resource:/com/example/8754.lm");
```