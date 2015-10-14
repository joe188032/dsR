---
title: "Text Analytics: Pre-processing Exercise"
author: "Shu-Kai Hsieh, National Taiwan Uni."
date: "2015 年 10 月 15 日"
output: html_document
---


## 文本發掘基本：流程複習

---

#### 練習用 `tm` 等相關套件

- 建立 *corpus object*
- 了解中英文文本的前處理工作
- 基本的語料探索分析


#### 語料文本：總統的語言

- 美國總統就職演說 `inaugTexts.txt` (A corpus of US presidential inaugural addresses from 1789-2013).
- 台灣總統元旦與國慶文告 

> 注意：Text Analytics 是人機合作的藝術，不一定要依賴大數據。
> 想一個比較的點。當成第一份小組合作作業。


---

- 安裝並載入可能用得到的套件


```r
library(tm) # Popular framework for text mining.
library(SnowballC) # Provides wordStem() for stemming.  
library(RColorBrewer) # Generate palette of colours for plots.
library(ggplot2)
library(Rgraphviz) # Correlation plots.
library(wordcloud)
```

- 建立給 `tm` 用的 **語料庫物件**


```r
file.path <- list.files("../data/week5/usP/", "*.*")
usP <- Corpus(VectorSource(file.path))
#class(usP)
#summary(usP)
#inspect(usP[2])
```


- 前處理（依研究與應用需要轉換資料）

  - `tm map()` 這個函式所帶的各種參數可以執行不同的前處理工作。


```r
# getTransformations()
usP <- tm_map(usP, removeNumbers)
usP <- tm_map(usP, removePunctuation)
usP <- tm_map(usP, removeWords, stopwords("english"))
# 也可以自己決定停用詞
# usP <- tm_map(usP, removeWords, c("own", "stop", "words"))
usP <- tm_map(usP, stripWhitespace)
usP <- tm_map(usP, stemDocument)
```

  - 我們也可以利用 `content_transformer()` 來自訂想要的前處理函式。
  

```r
# 把 \n 刪掉
removal <- content_transformer(function(x, pattern) gsub(pattern, "", x)) 
usP <- tm_map(usP, removal, "\n")
inspect(usP[2])
```

```
## <<VCorpus (documents: 1, metadata (corpus/indexed): 0/0)>>
## 
## [[1]]
## NULL
```

```r
# 替換 
#toString <- content_transformer(function(x, from, to) gsub(from, to, x))
#usP <- tm_map(usP, toString, "specific transform", "ST")
#usP <- tm_map(usP, toString, "other specific transform", "OST")
```

#### 詞類標記
  
- 可以用 `koRpus` 或 `openNLP` 來處理歐美語言。
- 中文可用 `jiebaR` (粗粒度) `jseg.py` (細粒度) 




```bash
sed -n '6,13p' dickens-clean.txt > dickens-sample.txt
```




```r
####### 
## Use koRpus package
## 但要自行安裝 Tree Tagger

# The koRpus package uses the TreeTagger for POS tagging
library(koRpus)

### perform POS tagging

text.tagged <- treetag("../data/week5/dickens-sample.txt",
                       treetagger = "manual", 
                       lang="en",
                       TT.options = list(path="~/Linguistic.Data.Science/tools/TreeTagger/",
                                         preset="en"))



# inspect text.tagged
options(width=150)
text.tagged@TT.res
```



```r
library(NLP)
library(openNLP)
library(openNLPmodels.en)
```



```r
# dickens-sample.txt
dickens <- readLines(file.choose())
dickens.1 <- paste(dickens, collapse = " ")


## For many kinds of text processing it is sufficient, even preferable to use base R classes. But for NLP we are obligated to use the String class. We need to convert our bio variable to a string.
dickens.2 <- as.String(dickens.1)


sent_anno <- Maxent_Sent_Token_Annotator()
word_anno <- Maxent_Word_Token_Annotator()

# These annotators form a “pipeline” for annotating the text in our dickens.2 variable.
usP_annotations <- annotate(usP,list(sent_anno, word_anno))
head(usP_annotations,5)
pos_anno <- Maxent_POS_Tag_Annotator()

usP_annotations.pos <- annotate(usP_annotations, pos_anno, usP_annotations)
usP.pos <- subset(usP_annotations.pos, type =="word")
tags <- sapply(usP.pos$features, `[[`, "POS")
tags
table(tags)
```




## 中文處理

- 注意編碼
- 斷詞與詞類標記








