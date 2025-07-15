# QTA lab session 8: NLP processing in R


The goal of today’s lab session is to inspect the functionality of the
**udpipe** library. **UDPipe** (Wijffels, 2022) offers
‘language-agnostic tokenization, tagging, lemmatization and dependency
parsing of raw text’. We will focus on tagging and lemmatization in
particular and how these pre-processing steps may make further analysis
more precise. Lemmatizing generally works better than stemming,
especially for inflected languages such as German or French.
Part-of-Speech (POS) tags identify the type of word (noun, verb, etc) so
it can be used to e.g. analyse only the verbs (actions) or adjectives
and adverbs (descriptions).

Another library that was developed by the quanteda team and that has
similar functionality is **spacyr** (Benoit & Matsuo, 2020), an R
wrapper around the spaCy package in Python. See this
[link](https://spacyr.quanteda.io/articles/using_spacyr.html) for more
information on using **spacyr**.

Let’s load required packages first.

``` r
library(tidyverse)
library(quanteda)
library(quanteda.textplots)
library(quanteda.textmodels)
library(quanteda.textstats)
library(quanteda.sentiment)
library(seededlda)
library(udpipe)
```

The primary challenge for our purposes is to communicate between
**udpipe** and **quanteda**. In the following code block we first turn
our corpus into a dataframe called `inaugural_speeches_df` and save the
speeches – which are now stored in `inaugural_speeches_df$text` – as a
character vector called txt. **udpipe** works with character vectors.

``` r
inaugural_speeches <- data_corpus_inaugural

inaugural_speeches_df <- convert(inaugural_speeches,
                                 to = "data.frame")

txt <- inaugural_speeches_df$text
str(txt)
```

     chr [1:60] "Fellow-Citizens of the Senate and of the House of Representatives:\n\nAmong the vicissitudes incident to life n"| __truncated__ ...

Let’s apply the `udpipe` function to this `txt`. This function tags each
token in each speech, based on an English-language model which will be
downloaded into the working directory. We instruct `udpipe` to include
the doc_ids from our **quanteda** corpus object. This will help us later
on when we want to transform the output of our **udpipe** workflow back
into a corpus which we can inspect with **quanteda** functions.
As_tibble() is used to turn the output into a tibble, which is a more
user-friendly format than the default output of **udpipe**.

``` r
parsed_tokens <-  udpipe(txt, "english", 
                         doc_id = inaugural_speeches_df$doc_id) %>% 
  as_tibble()
```

Let’s inspect what’s inside parsed_tokens

``` r
head(parsed_tokens)
```

    # A tibble: 6 × 17
      doc_id    paragraph_id sentence_id sentence start   end term_id token_id token
      <chr>            <int>       <int> <chr>    <int> <int>   <int> <chr>    <chr>
    1 1789-Was…            1           1 Fellow-…     1     6       1 1        Fell…
    2 1789-Was…            1           1 Fellow-…     7     7       2 2        -    
    3 1789-Was…            1           1 Fellow-…     8    15       3 3        Citi…
    4 1789-Was…            1           1 Fellow-…    17    18       4 4        of   
    5 1789-Was…            1           1 Fellow-…    20    22       5 5        the  
    6 1789-Was…            1           1 Fellow-…    24    29       6 6        Sena…
    # ℹ 8 more variables: lemma <chr>, upos <chr>, xpos <chr>, feats <chr>,
    #   head_token_id <chr>, dep_rel <chr>, deps <chr>, misc <chr>

``` r
str(parsed_tokens)
```

    tibble [155,215 × 17] (S3: tbl_df/tbl/data.frame)
     $ doc_id       : chr [1:155215] "1789-Washington" "1789-Washington" "1789-Washington" "1789-Washington" ...
     $ paragraph_id : int [1:155215] 1 1 1 1 1 1 1 1 1 1 ...
     $ sentence_id  : int [1:155215] 1 1 1 1 1 1 1 1 1 1 ...
     $ sentence     : chr [1:155215] "Fellow-Citizens of the Senate and of the House of Representatives:" "Fellow-Citizens of the Senate and of the House of Representatives:" "Fellow-Citizens of the Senate and of the House of Representatives:" "Fellow-Citizens of the Senate and of the House of Representatives:" ...
     $ start        : int [1:155215] 1 7 8 17 20 24 31 35 38 42 ...
     $ end          : int [1:155215] 6 7 15 18 22 29 33 36 40 46 ...
     $ term_id      : int [1:155215] 1 2 3 4 5 6 7 8 9 10 ...
     $ token_id     : chr [1:155215] "1" "2" "3" "4" ...
     $ token        : chr [1:155215] "Fellow" "-" "Citizens" "of" ...
     $ lemma        : chr [1:155215] "fellow" "-" "citizen" "of" ...
     $ upos         : chr [1:155215] "ADJ" "PUNCT" "NOUN" "ADP" ...
     $ xpos         : chr [1:155215] "JJ" "HYPH" "NNS" "IN" ...
     $ feats        : chr [1:155215] "Degree=Pos" NA "Number=Plur" NA ...
     $ head_token_id: chr [1:155215] "3" "3" "0" "6" ...
     $ dep_rel      : chr [1:155215] "amod" "punct" "root" "case" ...
     $ deps         : chr [1:155215] NA NA NA NA ...
     $ misc         : chr [1:155215] "SpaceAfter=No" "SpaceAfter=No" NA NA ...

As you can see, this object is a dataframe that consists of
approximately 155,000 rows where each row is a token, and each column is
an annotation. For our purposes, the most relevant variables are:

- `doc_id` contains the document in which the token appeared;
- `token` – contains the actual token;
- `lemma` – contains the lemmatized token;
- `upos` – contains the part of speech of the token, such as adjective,
  verb, noun, etc.;
- `xpos` – contains more detailed part of speech tags in the Penn
  Treebank format, such as NN for noun, VB for verb, etc. Is more
  fine-grained than `upos`

Let’s select those variables for further analysis.

``` r
parsed_tokens <- parsed_tokens %>% 
  select(doc_id, token, upos, lemma)
```

Inspect how many nouns appear in the corpus

``` r
sum(parsed_tokens$upos == "NOUN")
```

    [1] 31100

Inspect how many verbs appear in the corpus

``` r
sum(parsed_tokens$upos == "VERB")
```

    [1] 15148

Inspect how many adjectives appear in the corpus

``` r
sum(parsed_tokens$upos == "ADJ")
```

    [1] 11781

We can also inspect all different POS tags in one go.

``` r
table(parsed_tokens$upos)
```


      ADJ   ADP   ADV   AUX CCONJ   DET  INTJ  NOUN   NUM  PART  PRON PROPN PUNCT 
    11781 19263  6371  9541  6970 16790    42 31100   529  3755 14148  2572 14467 
    SCONJ   SYM  VERB     X 
     2705    16 15148    17 

An interesting tag is `PROPN`or proper noun that refers to the name (or
part of the name) of a unique entity, be it an individual, a place, or
an object. To get a feel for what entities we can filter out the proper
nouns and then count and sort their lemmas using `count()` from
**tidyverse**

``` r
propns <- parsed_tokens %>%
  filter(upos == "PROPN")

propns %>% count(lemma, sort = TRUE)
```

    # A tibble: 512 × 2
       lemma         n
       <chr>     <int>
     1 States      311
     2 America     257
     3 United      171
     4 Congress    126
     5 God         111
     6 President    94
     7 Americans    84
     8 Mr.          31
     9 nation       29
    10 Chief        28
    # ℹ 502 more rows

Say we are only interested in the nouns in those speeches

``` r
nouns <- parsed_tokens %>%
  filter(upos == "NOUN")
```

Let’s display their lemmas in a Wordcloud. We’ll first use the `split()`
function from base R to divide the nouns per speech in a list. We then
use `as.tokens()` in **quanteda** to turn that list into a tokens
object. We can create a `dfm` and take it from there.

``` r
nouns_dfm <- split(nouns$lemma, nouns$doc_id) %>% 
  as.tokens() %>% 
  dfm() 


textplot_wordcloud(nouns_dfm, max_words = 50)
```

![](Lab_Session_QTA_8_files/figure-commonmark/wordcloud_lemmas_nouns-1.png)

Let’s do the same for verbs

``` r
verbs <- parsed_tokens %>%
  filter(upos == "VERB")

verbs_dfm <- split(verbs$lemma, verbs$doc_id) %>% 
  as.tokens() %>% dfm()

textplot_wordcloud(verbs_dfm, max_words = 50)
```

![](Lab_Session_QTA_8_files/figure-commonmark/wordcloud_lemmas_verbs-1.png)

If we want to stitch back together the metadata to our newly created
`nouns_dfm` and `verbs_dfm` we can do this as follows:

``` r
docvars(nouns_dfm) <- inaugural_speeches_df %>% 
  select(Year, President, FirstName, Party)

docvars(verbs_dfm) <- inaugural_speeches_df %>%
  select(Year, President, FirstName, Party)
```

We are now in a position to inspect these dfms. For example, we may be
interested in what sort of verbs distinguish Republican presidents from
Democratic presidents.

``` r
verbs_dfm_grouped <- verbs_dfm %>% 
  dfm_group(groups = Party) %>%
  dfm_subset(Party == "Democratic" | Party == "Republican")

verb_keyness <- textstat_keyness(verbs_dfm_grouped, target = "Republican")

textplot_keyness(verb_keyness,
                 n = 10,
                 color = c("red", "blue"))
```

![](Lab_Session_QTA_8_files/figure-commonmark/inspect_verbs_keyness-1.png)

Let’s apply a topic model to the nouns

``` r
lda_10 <- textmodel_lda(nouns_dfm, 
                       k = 10,
                       alpha = 1,
                       max_iter = 2000)
```

Let’s inspect this topic model

``` r
terms(lda_10, 10)
```

          topic1        topic2       topic3         topic4         topic5      
     [1,] "war"         "time"       "confidence"   "union"        "world"     
     [2,] "resource"    "freedom"    "mind"         "constitution" "people"    
     [3,] "force"       "citizen"    "good"         "power"        "nation"    
     [4,] "progress"    "liberty"    "providence"   "citizen"      "man"       
     [5,] "commerce"    "generation" "station"      "object"       "peace"     
     [6,] "tax"         "government" "hand"         "liberty"      "life"      
     [7,] "part"        "work"       "satisfaction" "state"        "freedom"   
     [8,] "sovereignty" "land"       "pledge"       "character"    "government"
     [9,] "improvement" "promise"    "duty"         "opinion"      "faith"     
    [10,] "person"      "economy"    "lesson"       "spirit"       "spirit"    
          topic6        topic7        topic8       topic9         topic10    
     [1,] "law"         "citizenship" "government" "civilization" "nation"   
     [2,] "business"    "capital"     "people"     "progress"     "today"    
     [3,] "policy"      "demand"      "country"    "capacity"     "day"      
     [4,] "legislation" "presence"    "duty"       "leadership"   "world"    
     [5,] "trade"       "body"        "citizen"    "belief"       "people"   
     [6,] "tariff"      "law"         "power"      "order"        "history"  
     [7,] "respect"     "knowledge"   "interest"   "ideal"        "child"    
     [8,] "race"        "wealth"      "nation"     "welfare"      "democracy"
     [9,] "part"        "other"       "party"      "service"      "century"  
    [10,] "currency"    "courage"     "law"        "republic"     "power"    

``` r
head(lda_10$theta, 10)
```

                         topic1      topic2     topic3     topic4     topic5
    1789-Washington 0.009933775 0.003311258 0.34105960 0.10927152 0.05960265
    1793-Washington 0.057142857 0.057142857 0.08571429 0.20000000 0.02857143
    1797-Adams      0.039792388 0.008650519 0.27162630 0.13840830 0.11418685
    1801-Jefferson  0.060827251 0.048661800 0.18491484 0.15328467 0.16058394
    1805-Jefferson  0.081871345 0.044834308 0.21442495 0.19688109 0.05847953
    1809-Madison    0.121323529 0.003676471 0.19117647 0.11397059 0.12500000
    1813-Madison    0.326086957 0.018115942 0.07608696 0.06521739 0.19565217
    1817-Monroe     0.249641320 0.001434720 0.05164993 0.14634146 0.01004304
    1821-Monroe     0.395698925 0.010752688 0.01827957 0.13118280 0.02150538
    1825-Adams      0.050546448 0.021857923 0.13387978 0.21038251 0.09836066
                         topic6      topic7    topic8      topic9     topic10
    1789-Washington 0.006622517 0.009933775 0.4304636 0.013245033 0.016556291
    1793-Washington 0.114285714 0.114285714 0.2857143 0.028571429 0.028571429
    1797-Adams      0.008650519 0.005190311 0.4013841 0.003460208 0.008650519
    1801-Jefferson  0.007299270 0.024330900 0.3454988 0.012165450 0.002433090
    1805-Jefferson  0.005847953 0.033138402 0.2923977 0.003898635 0.068226121
    1809-Madison    0.003676471 0.007352941 0.4117647 0.003676471 0.018382353
    1813-Madison    0.050724638 0.025362319 0.2065217 0.018115942 0.018115942
    1817-Monroe     0.002869440 0.012912482 0.5179340 0.005738881 0.001434720
    1821-Monroe     0.005376344 0.002150538 0.4118280 0.001075269 0.002150538
    1825-Adams      0.005464481 0.001366120 0.4494536 0.015027322 0.013661202

## Other languages

**updipe** allows you to work with pre-trained language models build for
more than 65 languages

<img src="language_models.png" style="width:65.0%"
alt="Language models" />

If you want to work with these models you first need to download them.
Let’s say I want to work with a Dutch corpus

``` r
udmodel_dutch <- udpipe_download_model(language = "dutch")

str(udmodel_dutch)
```

    'data.frame':   1 obs. of  5 variables:
     $ language        : chr "dutch-alpino"
     $ file_model      : chr "/Users/hjms/Documents/Teaching/Essex/2025/Labs/Lab_8/dutch-alpino-ud-2.5-191206.udpipe"
     $ url             : chr "https://raw.githubusercontent.com/jwijffels/udpipe.models.ud.2.5/master/inst/udpipe-ud-2.5-191206/dutch-alpino-"| __truncated__
     $ download_failed : logi FALSE
     $ download_message: chr "OK"

I can now start tagging with vector of Dutch documents

``` r
dutch_documents <- c(d1 = "AZ wordt kampioen dit jaar",
                     d2 = "Mark Rutte, de langstzittende premier van Nederland, is de nieuwe NAVO-baas")

parsed_tokens_dutch <-  udpipe(dutch_documents, udmodel_dutch) %>% 
  as_tibble()

head(parsed_tokens_dutch)
```

    # A tibble: 6 × 17
      doc_id paragraph_id sentence_id sentence    start   end term_id token_id token
      <chr>         <int>       <int> <chr>       <int> <int>   <int> <chr>    <chr>
    1 d1                1           1 AZ wordt k…     1     2       1 1        AZ   
    2 d1                1           1 AZ wordt k…     4     8       2 2        wordt
    3 d1                1           1 AZ wordt k…    10    17       3 3        kamp…
    4 d1                1           1 AZ wordt k…    19    21       4 4        dit  
    5 d1                1           1 AZ wordt k…    23    26       5 5        jaar 
    6 d2                1           1 Mark Rutte…     1     4       1 1        Mark 
    # ℹ 8 more variables: lemma <chr>, upos <chr>, xpos <chr>, feats <chr>,
    #   head_token_id <chr>, dep_rel <chr>, deps <chr>, misc <chr>

If I have already downloaded the a language, I can load it as follows
(if the model is in the current working directory – otherwise I will
need to give it the full path to the file)

``` r
udmodel_dutch <- udpipe_load_model(file = "dutch-alpino-ud-2.5-191206.udpipe")
```

## Exercises

For these exercises we will work with the `parsed_tokens` dataframe that
we created in the above script.

1.  Create a dataframe `adjs` that contains all adjectives that appear
    in the corpus of inaugural speeches.

<!-- -->

2.  Display the most occurring adjectives in the inaugural speeches
    using `count()`

<!-- -->

3.  Group the the adjectives by speech and turn them into a dataframe
    called `adjs_dfm`.

<!-- -->

4.  Append Year, President, FirstName and Party from
    `inaugural_speeches_df` as docvars to `adjs_dfm`

<!-- -->

5.  Inspect `adjs_dfm` using the NRC Emotion Association Lexicon. If you
    don’t recall how to do this, have a look back at lab session 4. Call
    the output of `dfm_lookuop` as `dfm_inaugural_NRC`.

<!-- -->

6.  Add the count of fear words as a variable `fear` to the docvars of
    `adjs_dfm`

**Advanced**

7.  Use tidyverse functions to display the mean number of fear words for
    Repulican and Democratic presidents (NB: normally we would divide
    this number by the total number of tokens in a speech). Have a look
    at [this link](https://dplyr.tidyverse.org/reference/group_by.html)
    for more info.

<!-- -->

8.  Download a language model of your choice and inspect a vector of a
    few sentences using `udpipe`
