# NLP Application: SwiftKey Data

## Data Science Capstone
## Johns Hopkins University

This repository contains all code and products that are necessary to build a predictive application for natural language processing. We will demonstrate the application of several data science tasks to a dataset from the SwiftKey company. We include all scripts that were used to obtain, pre-process, analize and make predictions from the Data. 

Please refer to all R scripts in the webpage [https://github.com/corderoai/JHU_DataScienceCapstone](https://github.com/corderoai/JHU_DataScienceCapstone).


# Journal of Tasks

### Code to load r scripts

`source()`

## Task 0 - Understanding the problem

The first step in analyzing any new data set is figuring out: 
(a) what data you have and (b) what are the standard tools and models used for that type of data. Make sure you have downloaded the data from Coursera before heading for the exercises. This exercise uses the files named LOCALE.blogs.txt where LOCALE is the each of the four locales en_US, de_DE, ru_RU and fi_FI. The data is from a corpus called HC Corpora. See the About the Corpora reading for more details. The files have been language filtered but may still contain some foreign text.

### Packages for NLP

[tokenizers](https://cran.r-project.org/web/packages/tokenizers/vignettes/introduction-to-tokenizers.html)

[readr](https://readr.tidyverse.org/)

[tm](https://cran.r-project.org/web/packages/tm/tm.pdf)

[]()

[]()

```
# We obtained the Data from the web.

downloadFinalProjectFile <- function(){
  fileUrl <- "https://d396qusza40orc.cloudfront.net/dsscapstone/dataset/Coursera-SwiftKey.zip"
  destinationPath <- "./data/Coursera-SwiftKey.zip"
  
  if(!file.exists("./data")){
      dir.create("./data")
    }
  
  if(!file.exists("./data/Coursera-SwiftKey.zip")){
    download.file(fileUrl, destfile = destinationPath, method = "curl")  #for binary files.
  }
  else
  {
    print("Source files has already been downloaded! will place the new one in the backup folder in data.")
    dir.create("./data/backup")
    download.file(fileUrl, destfile = "./data/backup/Coursera-SwiftKey.zip", method = "curl")
  }  
  dateDownloaded <- date()
  unzip( destinationPath, exdir="./data", list =TRUE)                
}
```
Please refer to `the downloadFiles.R` script.

## Task 1 - Getting and cleaning the data
Large databases comprising of text in a target language are commonly used when generating language models for various purposes. In this exercise, you will use the English database but may consider three other databases in German, Russian and Finnish.

The goal of this task is to get familiar with the databases and do the necessary cleaning. After this exercise, you should understand what real data looks like and how much effort you need to put into cleaning the data. When you commence on developing a new language, the first thing is to understand the language and its peculiarities with respect to your target. You can learn to read, speak and write the language. Alternatively, you can study data and learn from existing information about the language through literature and the internet. At the very least, you need to understand how the language is written: writing script, existing input methods, some phonetic knowledge, etc.

Note that the data contain words of offensive and profane meaning. They are left there intentionally to highlight the fact that the developer has to work on them.

Tasks to accomplish

Tokenization - identifying appropriate tokens such as words, punctuation, and numbers. Writing a function that takes a file as input and returns a tokenized version of it.

Profanity filtering - removing profanity and other words you do not want to predict.

### Some checks

- Length of file en_US.blogs.txt:

`theSize <- (file.info("./data/final/en_US/en_US.blogs.txt")$size)`
`print(theSize/1024/1024)`

- Number of lines in en_US.twitter.txt:

We read the file in chuncks to have a faster task...

`connection <- file("./data/final/en_US/en_US.twitter.txt", "rb")`
```
numberOfLines <- 0L

while (length(chunk <- readBin(connection, "raw", 65536)) > 0) 
{
    numberOfLines <- numberOfLines + sum(chunk == as.raw(10L))
}
close(connection)
numberOfLines
```
- Longest length of the line seen in any of three en_US files:

```
readTotalNumberOfLinesInFile <- function(inFile)
{
  totalLines = 0
  
  connection <- file(inFile, "r")
  totalData <- readLines(connection, encoding = "UTF-8", skipNul = TRUE)
  close(connection)
  
  totalLines <- summary(nchar(totalData))[6]
  
  return(totalLines)  
}

readTotalNumberOfLinesInFile("./data/final/en_US/en_US.blogs.txt")
readTotalNumberOfLinesInFile("./data/final/en_US/en_US.news.txt")
readTotalNumberOfLinesInFile("./data/final/en_US/en_US.twitter.txt")
```

- In the en_US twitter data set, if you divide the number of lines where the word "love" (all lowercase) occurs by the number of lines the word "hate" (all lowercase) occurs, about what do you get?

```
fullDataUSTwitter <- readFullFile("./data/final/en_US/en_US.twitter.txt")

# class(fullDataUSTwitter)
# str(fullDataUSTwitter)
# Notice how fullDataUSTwitter is just a vector of character types(each line is an element of the vector being a string of characters including all words in a line)

rowsOfLove <- grep("love", fullDataUSTwitter)
rowsOfHate <- grep("hate", fullDataUSTwitter) 
loveHateRatio <- length(rowsOfLove)/length(rowsOfHate)
loveHateRatio 
```

- The one tweet in the en_US twitter data set that matches the word "biostats" says what?

```
lineNumber <- grep("biostats", fullDataUSTwitter)
fullDataUSTwitter[lineNumber]

# Or just:

lineOfCharacters <- grep("biostats", fullDataUSTwitter, value = T)
lineOfCharacters
```

- How many tweets have the exact characters "A computer once beat me at chess, but it was no match for me at kickboxing". (I.e. the line matches those characters exactly.)

```
length(grep("A computer once beat me at chess, but it was no match for me at kickboxing", fullDataUSTwitter))
```
We could have done all the previous operations by reading the Data line by line. The advantage of such approach would be low main memory consumption but at the gruesome cost of higher elapsed time.

Please refer to the `readData.R` script.

## Task 2 - Exploratory Data Analysis

The first step in building a predictive model for text is understanding the distribution and relationship between the words, tokens, and phrases in the text. The goal of this task is to understand the basic relationships you observe in the data and prepare to build your first linguistic models.

Tasks to accomplish

Exploratory analysis - perform a thorough exploratory analysis of the data, understanding the distribution of words and relationship between the words in the corpora.
Understand frequencies of words and word pairs - build figures and tables to understand variation in the frequencies of words and word pairs in the data.

### Initial Review

Initially, we need to consider the dimmensions between all dataset files we have:

```
displayNumberOfLinesBarPlot <- function(){
  suppressMessages(library(ggplot2))
  
  dataPath <- "./data/final/en_US/"
  source("readData.R")
  numberLinesBlogs <- readTotalNumberOfLinesInFile(paste0(dataPath, "en_US.blogs.txt"))
  numberLinesNews <- readTotalNumberOfLinesInFile(paste0(dataPath, "en_US.news.txt"))
  numberLinesTwitter <- readTotalNumberOfLinesInFile(paste0(dataPath, "en_US.twitter.txt"))
  
  df <- data.frame( c(numberLinesBlogs, numberLinesNews, numberLinesTwitter))
  df$File <- c("US_Blogs.txt", "US_News.txt", "US_Twitter.txt")
  colnames(df)[1] <- "Counts"
  colors1 <- c("red", "blue", "green")
  
  print(df)
  
  ggplot(df, aes(x=File, y=Counts)) + geom_bar(stat='identity', fill = colors1) + ylab("Number of Lines") + xlab("File") + ggtitle("Number of Lines for all US Files") 
}

displayNumberOfLinesBarPlot()
```

We observe that the twitter file is the largest:

```
printLengthsOfFilesTable <- function(){
  dataPath <- "./data/final/en_US/"
  source("readData.R")
  numberLinesBlogs <- readTotalNumberOfLinesInFile(paste0(dataPath, "en_US.blogs.txt"))
  numberLinesNews <- readTotalNumberOfLinesInFile(paste0(dataPath, "en_US.news.txt"))
  numberLinesTwitter <- readTotalNumberOfLinesInFile(paste0(dataPath, "en_US.twitter.txt"))
  
  df <- data.frame( c(numberLinesBlogs, numberLinesNews, numberLinesTwitter))
  df$File <- c("US_Blogs.txt", "US_News.txt", "US_Twitter.txt")
  
  # print(df)
  knitr::kable(df, col.names = gsub("[.]", " ", names(df)))
}

printLengthsOfFilesTable ()
```

It is necessary to review the size of each file in Mbs as well to have an impression of the density of lines in each file:

```
displaySizeOfFilesBarPlot <- function(){
  dataPath <- "./data/final/en_US/"
  
  sizeBlogs <- (file.info(paste0(dataPath, "en_US.blogs.txt"))$size)
  sizeNews <- (file.info(paste0(dataPath, "en_US.news.txt"))$size)
  sizeTwitter <- (file.info(paste0(dataPath, "en_US.twitter.txt"))$size)
  
  df <- data.frame( c(sizeBlogs/1024/1024, sizeNews/1024/1024, sizeTwitter/1024/1024))
  df$File <- c("US_Blogs.txt", "US_News.txt", "US_Twitter.txt")
  colnames(df)[1] <- "Size"
  colors1 <- c("red", "blue", "green")
  # print(df)
  
  ggplot(df, aes(x=File, y=Size)) + geom_bar(stat='identity', fill = colors1) + ylab("Size of File in MBs") + xlab("File") + ggtitle("File Sizes for US Files") 
}

displaySizeOfFilesBarPlot()

displaySizeOfFilesTable <- function(){
  dataPath <- "./data/final/en_US/"
  
  sizeBlogs <- (file.info(paste0(dataPath, "en_US.blogs.txt"))$size)
  sizeNews <- (file.info(paste0(dataPath, "en_US.news.txt"))$size)
  sizeTwitter <- (file.info(paste0(dataPath, "en_US.twitter.txt"))$size)
  
  df <- data.frame( c(sizeBlogs/1024/1024, sizeNews/1024/1024, sizeTwitter/1024/1024))
  df$File <- c("US_Blogs.txt", "US_News.txt", "US_Twitter.txt")
  colnames(df)[1] <- "Size in MBs"
  
  # print(df)
   knitr::kable(df, col.names = gsub("[.]", " ", names(df)))
} 

displaySizeOfFilesTable()
```
Interestingly enough we can see that the blogs and news files are larger in Byte size than the twitter ones. This is logical since twitter has a fixed length of characters per tweet whereas in the other cases the length of the posts is open for the writer. 

In order to check the later assumption we will find some relevant statistics regarding the lines in the files:

```
displayLineStatisticsForFiles <- function(){
  fullDataUSTwitter <- readFullFile("./data/final/en_US/en_US.twitter.txt")
  fullDataUSBlogs <- readFullFile("./data/final/en_US/en_US.blogs.txt")
  fullDataUSNews <- readFullFile("./data/final/en_US/en_US.news.txt")
  
  df <- data.frame( File = character(), Maximum.Number.Of.Characters = numeric(), Total.Number.Of.Words = numeric()) 
  
  nchars <- lapply(fullDataUSTwitter, nchar)
  maxchars <- which.max(nchars)
  wordCount <- sum(sapply(strsplit(fullDataUSTwitter, "\\s+"), length))
  
  df <- rbind(df, c("en_US.twitter.txt", maxchars, wordCount) )
  
  nchars <- lapply(fullDataUSBlogs, nchar)
  maxchars <- which.max(nchars)
  wordCount <- sum(sapply(strsplit(fullDataUSBlogs, "\\s+"), length))
  
  df <- rbind(df, c("en_US.blogs.txt", maxchars, wordCount) )
  
  nchars <- lapply(fullDataUSNews, nchar)
  maxchars <- which.max(nchars)
  wordCount <- sum(sapply(strsplit(fullDataUSNews, "\\s+"), length))
  
  df <- rbind(df, c("en_US.news.txt", maxchars, wordCount) )
  
  colnames(df) <- c("File", "Maximum.Number.Of.Words.In.A.Line", "Total.Number.Of.Words")
  
  #print(df)
  knitr::kable(df, col.names = gsub("[.]", " ", names(df)))
}

displayLineStatisticsForFiles()
```

Notice how the maximum number of words in a line in the twitter file was only 26. Which sounds completely logic given the maximum character restriction.


### Cleaning Up the Data

We form the Corpus and remove all unnecessary features from Data. We also sample the Data in order to have a faster analisis, however we may as well include all contents with all lines for the analysis. We also apply a censorship function in order to remove all bad words, punctuations or other features we decided irrelvant for this project. The list of obscene words was taken from the repository [https://raw.githubusercontent.com/LDNOOBW/List-of-Dirty-Naughty-Obscene-and-Otherwise-Bad-Words/master/en](https://raw.githubusercontent.com/LDNOOBW/List-of-Dirty-Naughty-Obscene-and-Otherwise-Bad-Words/master/en).

The following function make it possible:

```
## These functions are meant to be used for any dataset

createCorpus <- function(textData_in_Memory){
  corpora <- paste(textData_in_Memory, collapse=" ")
  corpora <- VectorSource(corpora)
  corpora <- Corpus(corpora)
  
  return(corpora)
}

cleanCorpus <- function(rawCorpus) {
  
  rawCorpus <- tm_map(rawCorpus, content_transformer(tolower))
  
  toSpace <- content_transformer(function(x, pattern) gsub(pattern, " ", x))
  rawCorpus <- tm_map(rawCorpus, toSpace, "/|@|//|$|:|:)|*|&|!|?|_|-|#|")
  
  rawCorpus <- tm_map(rawCorpus, removeNumbers)
  rawCorpus <- tm_map(rawCorpus, removeWords, stopwords("english"))
  rawCorpus <- tm_map(rawCorpus, removePunctuation)
  rawCorpus <- tm_map(rawCorpus, stripWhitespace)
  
  source("badwords.R")
  rawCorpus <- tm_map(rawCorpus, removeWords, VectorSource(badwords))
  
  #rawCorpus <- tm_map(rawCorpus, stemDocument)
  
  return (rawCorpus)
}
```

### Questions to consider

1. Some words are more frequent than others - what are the distributions of word frequencies?

We consider the top 25 words that we found over a sampled subset of the original Data from merging the three datasets. We set a variable to extract only a percentage of lines from the original file (trying to practice a sampling task, however we may as well consider all data for exploratory analisis since no performance issues are critical in this case). We then show a plot containing the frequencies of most used words.

```
## Obtain all frequencies for all words in the Corpus

calculateFrequencyOfWords <- function (theCorpus) {
  csparse <- DocumentTermMatrix(theCorpus)
  cmatrix <- as.matrix(csparse)   
  
  wordFrequencies <- colSums(cmatrix)
  wordFrequencies <- as.data.frame(sort(wordFrequencies, decreasing=TRUE))
  wordFrequencies$word <- rownames(wordFrequencies)
  colnames(wordFrequencies) <- c("Frequency", "Word")
  
  return (wordFrequencies)
}

options(warn=-1)
library(tm)
library(wordcloud)
library(ggplot2)
source("readData.R")

### Read all files at once
  fullDataUSTwitter <- readFullFile("./data/final/en_US/en_US.twitter.txt")
  fullDataUSBlogs <- readFullFile("./data/final/en_US/en_US.blogs.txt")
  fullDataUSNews <- readFullFile("./data/final/en_US/en_US.news.txt")
  
  ### Sample the Data
  set.seed(13)
  sampleSizePercentage <- .00000625    # For each file we sample the given percentage of lines (from 0 to 1, e.g. .6 is sixty percent). 
  
  sampledDataUSTwitter <- sample(fullDataUSTwitter, round(sampleSizePercentage*length(fullDataUSTwitter)), replace = F)
  sampledDataUSBlogs <- sample(fullDataUSBlogs, round(sampleSizePercentage*length(fullDataUSBlogs)), replace = F)
  sampledDataUSNews <- sample(fullDataUSNews, round(sampleSizePercentage*length(fullDataUSNews)), replace = F) 
  
  ### Optional, we may as well use any sampled subset (sampledDataUSTwitter, sampledDataUSBlogs, sampledDataUSNews)
  ### This decision has to take into consideration other factors to be investigated outside the raw Data. e.g. Expert opinion.
  sampledDataTwitterBlogsNews <- c(sampledDataUSTwitter, sampledDataUSBlogs, sampledDataUSNews)
  
  textData <- sampledDataTwitterBlogsNews
  
  USCorpus <- createCorpus(textData)
  USCorpus <- cleanCorpus(USCorpus)
  
  allWordFrequencies <- calculateFrequencyOfWords(USCorpus)
  top25Words <- allWordFrequencies[1:25,]
  
  #Plot results of word frequencies
  p <- ggplot(data=top25Words, aes(x=reorder(Word,Frequency), y=Frequency,
               fill=factor(reorder(Word,-Frequency))))+ geom_bar(stat="identity") 
  p + xlab("Word") +labs(title = "Top 25 Words: US Data") +theme(legend.title=element_blank()) + coord_flip()
```

We present the results in a graphic manner as a word cloud with the top 100 most used words:

```
# Show a word Cloud for the 100 most used words

wordcloud(allWordFrequencies$Word[1:100], allWordFrequencies$Frequency[1:100], colors=brewer.pal(8, "Dark2"))
```


2. What are the frequencies of 2-grams and 3-grams in the dataset?

According to [https://en.wikipedia.org/wiki/N-gram](https://en.wikipedia.org/wiki/N-gram) an n-gram is a contiguous sequence of n items from a given sample of text or speech. The items can be phonemes, syllables, letters, words or base pairs according to the application. The n-grams typically are collected from a text or speech corpus. When the items are words, n-grams may also be called shingles. Using Latin numerical prefixes, an n-gram of size 1 is referred to as a "unigram"; size 2 is a "bigram" (or, less commonly, a "digram"); size 3 is a "trigram".

For the effects of this project we calculate the frequencies of the n-grams the following way (the unigrams are the same as just word frequencies, we choose the top 25):

```
### Calculate the unigrams, bigrams and trigrams using this generalized function version for an n-gram
getN_grams <- function(textData_in_Memory, numberOfGrams){
  theTokens<- tokens(textData_in_Memory,what ="word", remove_numbers = TRUE, remove_punct = TRUE, 
                     remove_separators = TRUE, remove_symbols = TRUE )
  theTokens <- tokens_tolower(theTokens)
  theTokens <- tokens_select(theTokens, stopwords(), selection ="remove") 
  
  unigram <- tokens_ngrams(theTokens, n = numberOfGrams)  
  unigram.dfm <- dfm(unigram, tolower =TRUE, remove = stopwords("english"), remove_punct = TRUE)   
  
  return( unigram.dfm )
}

library(quanteda)

numberOfWords = 25
  #Obtain the top N Unigrams
  unigrams <- getN_grams( textData, 1) 
  unigramsDf <- stack(topfeatures(unigrams, numberOfWords))
  unigramsDf <- setNames(stack(topfeatures(unigrams, numberOfWords))[2:1], c('Words','Frequency'))
    
  p <- ggplot(data=unigramsDf, aes(x=reorder(Words,Frequency), y=Frequency,
                                   fill=factor(reorder(Words,-Frequency))))+ geom_bar(stat="identity") 
  p + xlab("Word") +labs(title = "Top 25 UNIGRAMS: US Data") +theme(legend.title=element_blank()) + coord_flip()
```

Now, we count the top 25 bigrams:

```
#Obtain the Bigrams
  bigrams <- getN_grams( textData, 2) 
  bigramsDf <- stack(topfeatures(bigrams, numberOfWords))
  bigramsDf <- setNames(stack(topfeatures(bigrams, numberOfWords))[2:1], c('Words','Frequency'))
  
  p <- ggplot(data=bigramsDf, aes(x=reorder(Words,Frequency), y=Frequency,
                                   fill=factor(reorder(Words,-Frequency))))+ geom_bar(stat="identity") 
  p + xlab("Word") +labs(title = "Top 25 BIGRAMS: US Data") +theme(legend.title=element_blank()) + coord_flip()
```

Finally we obtain the frequencies for the trigrams:

```
#Obtain the Trigrams
  trigrams <- getN_grams( textData, 3) 
  trigramsDf <- stack(topfeatures(trigrams, numberOfWords))
  trigramsDf <- setNames(stack(topfeatures(trigrams, numberOfWords))[2:1], c('Words','Frequency'))
  
  p <- ggplot(data=trigramsDf, aes(x=reorder(Words,Frequency), y=Frequency,
                                  fill=factor(reorder(Words,-Frequency))))+ geom_bar(stat="identity") 
  p + xlab("Word") +labs(title = "Top 25 TRIGRAMS: US Data") +theme(legend.title=element_blank()) + coord_flip()
```


3. How many unique words do you need in a frequency sorted dictionary to cover 50% of all word instances in the language? 90%?

In the following plot we can obtain the coverage for 50% and 90% by checking the x axis (for completeness we show all other percentages):

```
### Calculate Coverage. You receive a corpus and in turn the function obtains how many words are needed to cover a given percentage of the full corpus.
getCoverage <- function(theCorpus, percentage){
  
  
  allWordFrequencies <- calculateFrequencyOfWords(theCorpus)
  totalNumberOfWords <- sum(allWordFrequencies$Frequency)
  
  currentPercentage <- 0
  counter <- 1
  numberOfWords <- 0
  
  while(currentPercentage < percentage){
    numberOfWords <- numberOfWords + allWordFrequencies$Frequency[counter]
    currentPercentage <- numberOfWords / totalNumberOfWords
    
    counter <- counter + 1
    
    return(counter)
  }
}  

#Obtain coverage for any given percentage
  percents <- seq(from = .05, to =.95, by = .05)
  wordCovers <- c()
  
  for (i in 1:length(percents)){
    wordCovers <- c(wordCovers, getCoverage(USCorpus, percents[i]))
  }
  
  q <- qplot(percents,wordCovers, geom=c("line","point")) + geom_text(aes(label=wordCovers), hjust=1.35, vjust=-0.1)+ xlab("Percentage") + ylab("Times that Words Appear") + labs(title = paste( "Coverage for ", sum(allWordFrequencies$Frequency), "Words")) 
  q +scale_x_continuous(breaks=percents)
```

4. How do you evaluate how many of the words come from foreign languages?

As a first approach in order to evaluate words that come from foreign languages, we may access a dictionary or idiom list for that language, then we could either remove them from the corpus or identify them as foreign words.

5. Can you think of a way to increase the coverage -- identifying words that may not be in the corpora or using a smaller number of words in the dictionary to cover the same number of phrases?

We may use another machine learning method to learn the vocabulary and habits of the user in order to offer a more personalized prediction, and thus, increasing coverage. The coverage of other spatio temporal features such as location: We may also use location services in order to recommend nearby places or other features that might be of interest. Finally, we may learn the relationship between stopwords and other nouns and verbs that might be correlated.


Please refer to the `exploratoryAnalisis.R` script in this repo.
