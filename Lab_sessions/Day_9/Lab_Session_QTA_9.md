# QTA lab session 9: Word embeddings


The goal of today’s lab session is to strenghten our understanding of
word embeddings. We’ll train a word embeddings model using the
**text2vec** library (Selivanov, Bickel & Wang, 2022) on a set of
speeches of European Commissioners and we’ll inspect these embeddings.
We can install this package using the `install.packages()` function.
We’ll also install the `Rtsne` package, which is used for visualizing
high-dimensional data (such as word embeddings) in a two-dimensional
space.

NB: Keep in mind that this lab session is meant for practice purposes
only. The word vectors that we’ll inspect require careful validation.

Let’s load the required packages first.

``` r
library(quanteda)
library(quanteda.textstats)
library(tidyverse)
library(text2vec)
library(Rtsne)
library(ggplot2)
```

## Preparing the data

Let’s first read in the Commission speeches. The speeches are in .Rdata
format, which is a native R format and can be read in using the `load()`
function. The data contains speeches of European Commissioners from 2007
to 2015.

``` r
load("european_commission.Rdata")

dim(commission_speeches)
```

    [1] 6140    2

``` r
names(commission_speeches)
```

    [1] "speaker" "text"   

We’ll tokenise the speeches.

``` r
corpus_speeches <- corpus(commission_speeches)
tokens_speeches <- tokens(corpus_speeches,
                          what = "word",
                          remove_punct = TRUE, 
                          remove_symbols = TRUE, 
                          remove_numbers = TRUE,
                          remove_url = TRUE,
                          remove_separators = TRUE,
                          split_hyphens = FALSE,
                          ) %>%
  tokens_remove(stopwords(source = "smart"), padding = FALSE) %>%
  tokens_tolower()
```

NB: The next few steps draw on
[this](https://quanteda.io/articles/pkgdown/replication/text2vec.html)
**quanteda** tutorial.

We’ll select those features that occur at least 5 times.

``` r
feats <- dfm(tokens_speeches) %>%
    dfm_trim(min_termfreq = 5) %>%
    featnames()
```

We’ll now select these features from the tokenised speeches. We’ll set
`padding = TRUE` so that the resulting tokens object has the same number
of features as the original tokens object.

``` r
tokens_speeches <- tokens_select(tokens_speeches, 
                                 feats,
                                 padding = TRUE)
```

Let’s inspect which features occur most often

``` r
tokens_speeches %>%
  dfm() %>%
  topfeatures(n = 100,
              decreasing = TRUE,
              scheme = c("count")
)
```

         european                          eu        europe    commission 
            59741         57480         36221         28885         26391 
           member        states      economic        policy        market 
            21919         21113         20112         20031         18333 
            union          work        energy     important        growth 
            17782         15250         14912         14768         14553 
             make     countries         today     financial        people 
            14211         13889         13600         12862         12828 
          support         world          time          year        future 
            12806         11848         11796         11584         11424 
           social      national         years        public   development 
            11386         11285         11020         10909         10147 
          economy        global         level        crisis      research 
             9952          9892          9771          9664          9422 
       innovation    investment      citizens        change     political 
             9377          8942          8888          8875          8675 
       challenges      services        ladies        sector          good 
             8165          8113          8005          7987          7966 
        gentlemen           key        single       markets        ensure 
             7922          7805          7773          7666          7557 
            trade       council international         rules         clear 
             7524          7521          7413          7398          7379 
      cooperation          area          role        action       process 
             7332          7176          7052          6986          6929 
           common          part   competition      business     framework 
             6872          6862          6862          6758          6607 
             euro        system       climate   sustainable        issues 
             6544          6526          6500          6491          6488 
            state       forward      security     companies      strategy 
             6485          6416          6386          6350          6323 
             made        rights         areas     president    parliament 
             6299          6167          6126          6077          6063 
           reform      progress       working      approach      continue 
             6027          5976          5942          5940          5909 
              set     agreement      measures         means          jobs 
             5874          5862          5819          5814          5770 
         policies         place        strong       efforts          high 
             5589          5411          5356          5347          5331 
             open       country      regional          data           law 
             5318          5272          5186          5140          5109 

We’ll create a feature-co-occurrence matrix using the `fcm()` function
which calculates co-occurrences of features within a user-defined
context. We’ll choose a window size of 5, but other choices are
available. The `tri = TRUE` argument means that we will also include
co-occurrences of three words.

``` r
speeches_fcm <- fcm(tokens_speeches, 
                    context = "window", 
                    window = 5,
                    tri = TRUE)

dim(speeches_fcm)
```

    [1] 19571 19571

Let’s see what `speeches_fcm()` looks like.

``` r
speeches_fcm[1:5,1:5]
```

    Feature co-occurrence matrix of: 5 by 5 features.
               features
    features    dear president committee regions erasmus
      dear       750       270        64      32       5
      president    0      1336       108      53       3
      committee    0         0       214     310       5
      regions      0         0         0     580       4
      erasmus      0         0         0       0     120

*Dear* and *President* co-occur 270 times in the corpus. *Dear* and
*Regions* only 32 times.

## Fitting a GloVe model

We’ll now fit a GloVe vector model. GloVe is an unsupervised learning
algorithm for obtaining vector representations for words. Training is
performed on the feature co-occurrence matrix, which represents
information about global word-word co-occurrence statistics in a corpus.
The rank argument of the `GlobalVectors$new()` function specifies the
number of dimensions of the resulting word vectors. The `x_max` argument
is a hyperparameter that controls the weighting of the co-occurrence
counts. A common choice is 10, which we will use here.

We’ll estimate the GloVe embeddings using the `fit_transform()` method
of the `GlobalVectors` class. This method takes in the feature
co-occurrence matrix, the number of iterations to run, the convergence
tolerance, and the number of threads to use for parallel processing.

``` r
glove <- GlobalVectors$new(rank = 50, 
                           x_max = 10)

wv_main <- glove$fit_transform(speeches_fcm, 
                               n_iter = 10,
                               convergence_tol = 0.01, 
                               n_threads = 8)
```

    INFO  [20:09:11.418] epoch 1, loss 0.1562
    INFO  [20:09:14.374] epoch 2, loss 0.1143
    INFO  [20:09:17.276] epoch 3, loss 0.1026
    INFO  [20:09:20.150] epoch 4, loss 0.0964
    INFO  [20:09:23.076] epoch 5, loss 0.0925
    INFO  [20:09:25.984] epoch 6, loss 0.0899
    INFO  [20:09:28.877] epoch 7, loss 0.0879
    INFO  [20:09:31.792] epoch 8, loss 0.0864
    INFO  [20:09:34.769] epoch 9, loss 0.0852
    INFO  [20:09:37.687] epoch 10, loss 0.0842

``` r
dim(wv_main)
```

    [1] 19571    50

The model learns two sets of word vectors - main and context. They are
essentially the same.

``` r
wv_context <- glove$components
dim(wv_context)
```

    [1]    50 19571

Following recommendations in the **text2vec** package we sum these
vectors. We transpose the `wv_context` object so that it has the same
number of rows and columns as `wv_main`

``` r
word_vectors <- wv_main + t(wv_context)

dim(word_vectors)
```

    [1] 19571    50

We now have 50-dimension word_vectors for all approximantely 20,000
tokens in our corpus.

## Inspecting the GloVe model

Now it’s tme to inspect these word embeddings. For example, we find the
nearest neighbors of a word (or a set of words) of interest. Nearest
neighbors are those words that are most closely located in the vector
space. We can find those using by calculating cosine similarities
between the word vector of a target word and all other word vectors.

We’ll use a custom function
([source](https://s-ai-f.github.io/Natural-Language-Processing/Word-embeddings.html))
to finds these similar words It takes in three arguments: the target
word, the word_vectors object, and the number of neighbors we want to
inspect.

The find_similar_words() function calculates the cosine similarity
between the target word vector and all other word vectors in the
word_vectors object. It then sorts these similarities in descending
order and returns the top n similar words.

``` r
find_similar_words <- function(word, word_vectors, n = 10) {
  similarities <- word_vectors[word, , drop = FALSE] %>%
    sim2(word_vectors, y = ., method = "cosine")
  
  similarities[,1] %>% sort(decreasing = TRUE) %>% head(n)
}
```

The Commissioner speeches span the time period 2007–2015, a time of
upheaval in the EU. Let’s take a look at the nearest neighbors of
‘crisis’. The `drop = FALSE` argument ensure that crisis is not
converted to a vector.

``` r
find_similar_words("crisis", word_vectors)
```

           crisis     financial        facing      economic      response 
        1.0000000     0.7623151     0.7555791     0.7445364     0.7357258 
            faced         worst          face unprecedented     situation 
        0.7346225     0.7211939     0.7120763     0.6985381     0.6952249 

Crisis refers mostly to the Eurocrisis.

Let’s inspect the context of climate

``` r
find_similar_words("climate", word_vectors)
```

       climate     change     global  challenge      major developing adaptation 
     1.0000000  0.9206124  0.7145332  0.6973592  0.6627154  0.6427270  0.6296290 
    challenges     energy     action 
     0.6256344  0.6243404  0.6034091 

Global climate change needs to be addressed, that much is clear.

We can sum vectors to each find neighbors. Let’s add crisis + Ireland

``` r
crisis_Ireland <- word_vectors["crisis", , drop = FALSE] +
  word_vectors["ireland", , drop = FALSE] 
```

And find the nearest neighbors of this sum. The sim2 function calculates
the cosine similarity between the word vectors of the target word and
all other word vectors. The norm argument specifies the normalization
method to use. In this case, we use “l2” normalization, which is a
common choice for cosine similarity calculations.

``` r
cos_sim_crisis_Ireland <- sim2(x = word_vectors, y = crisis_Ireland, method = "cosine", norm = "l2")
head(sort(cos_sim_crisis_Ireland[,1], decreasing = TRUE), 20)
```

       crisis   ireland    greece    facing financial     shown      past      back 
    0.8966935 0.7833175 0.7364405 0.7221609 0.7054565 0.6986261 0.6964619 0.6807209 
    situation     years  economic beginning    forget difficult recession    recent 
    0.6777254 0.6760754 0.6742169 0.6639011 0.6600506 0.6590110 0.6567549 0.6557307 
       return   started      time      fact 
    0.6531277 0.6523152 0.6522399 0.6508440 

Interestingly, the result lists other countries that where also
struggling at the time.

What if we substract the Ireland vector from the crisis vector?

``` r
crisis_Ireland <- word_vectors["crisis", , drop = FALSE] -
  word_vectors["ireland", , drop = FALSE] 

cos_sim_crisis_Ireland <- sim2(x = word_vectors, y = crisis_Ireland, method = "cosine", norm = "l2")
head(sort(cos_sim_crisis_Ireland[,1], decreasing = TRUE), 10)
```

          crisis     response consequences  exacerbated       crises       urgent 
       0.7339710    0.6268996    0.6062634    0.6061828    0.6032195    0.5752578 
           faced         wake       ignore         face 
       0.5733309    0.5688283    0.5552337    0.5525405 

This time we get more general crisis terms.

Inspecting a word embeddings model like so can be useful for a few
different tasks:

1.  As a list of potential terms for dictionary construction;
2.  As an input to downstream QTA tasks;
3.  As a source for visualization.

Let’s take this first task an example. Perhaps we want to develop a
sentiment dictionary for Commissioner speeches, but we are less trusting
of off-the-shelf sentiment dictionaries because we suspect that these
may not capture how sentiment is expressed in Commissioner speeches. One
way to go is use a small seed dictionary of positive and negative words,
and use word embeddings to inspect what other words are close in the
embedding space to these seed words.

For example, we may take as positive words a small set of positive seed
words: *good*, *nice*, *excellent*, *positive*, *fortunate*, *correct*,
*superior*. And as negative words a small set of negative seed words:
*bad*, *nasty*, *poor*, *negative*, *wrong*, *unfortunate*

Let’s start by calculating the average vector for positive words

``` r
positive <- (word_vectors["good", , drop = FALSE] +
  word_vectors["nice", , drop = FALSE] +
  word_vectors["excellent", , drop = FALSE] +
  word_vectors["positive", , drop = FALSE] + 
  word_vectors["fortunate", , drop = FALSE] + 
  word_vectors["correct", , drop = FALSE] + 
  word_vectors["superior", , drop = FALSE]) /7
```

And for negative words.

``` r
negative <- (word_vectors["bad", , drop = FALSE] +
  word_vectors["nasty", , drop = FALSE] +
  word_vectors["poor", , drop = FALSE] +
  word_vectors["negative", , drop = FALSE] + 
  word_vectors["wrong", , drop = FALSE] + 
  word_vectors["unfortunate", , drop = FALSE]) /6
```

We can now inspect the neighbors of our ‘positive’ seed dictionary.

``` r
cos_sim_positive <- sim2(x = word_vectors, y = positive, method = "cosine", norm = "l2")
head(sort(cos_sim_positive[,1], decreasing = TRUE), 20)
```

          good  excellent      start        lot       time       hope    success 
     0.8382870  0.7610760  0.7582630  0.7477281  0.7351832  0.7099051  0.7033868 
          made      point      happy successful       hard      today       back 
     0.6949939  0.6912624  0.6823854  0.6819814  0.6811128  0.6794880  0.6746563 
       forward  convinced  extremely    results     future   positive 
     0.6744626  0.6709321  0.6663144  0.6566265  0.6534423  0.6510266 

This includes some words that seem useful such as encouraging and
opportunity and forward.

Let’s do the same for our ‘negative’ dictionary

``` r
cos_sim_negative <- sim2(x = word_vectors, y = negative, method = "cosine", norm = "l2")
head(sort(cos_sim_negative[,1], decreasing = TRUE), 20)
```

             bad consequences     negative        worse      effects       damage 
       0.7496833    0.7371557    0.7144980    0.7039093    0.6338216    0.6162128 
        worrying        avoid    suffering         risk    happening          hit 
       0.6065039    0.5922670    0.5901358    0.5898009    0.5853962    0.5773995 
            poor     dramatic        worst  experienced        wrong     affected 
       0.5730536    0.5649939    0.5636556    0.5605418    0.5601808    0.5565103 
           risks         lost 
       0.5563893    0.5538327 

## Calculate document embeddings

In a final step, let’s aggregate the embeddings at the document level to
obtain document embeddings. We can do this by averaging the word vectors
for all words in a document. This code is a bit involved. In separate
steps we do the following:

1)  Create a document-feature matrix (dfm) from the tokenized speeches;
2)  Generate an empty matrix to hold the document embeddings;
3)  For each document, extract the word counts as a named vector;
4)  Keep only the words (“valid words”) that appear in the document and
    are in the embedding vocabulary;
5)  If there are valid words, calculate the mean of their embeddings and
    store it in the document embeddings matrix;

``` r
# Create a document-feature matrix (dfm) from the tokenized speeches
dfm_speeches <- dfm(tokens_speeches)

# Generate an empty matrix
doc_embeddings <- matrix(NA, nrow = ndoc(dfm_speeches), ncol = ncol(word_vectors))
rownames(doc_embeddings) <- docnames(dfm_speeches)

dim(doc_embeddings)
```

    [1] 6140   50

``` r
# For each document
for (i in seq_len(ndoc(dfm_speeches))) {
  # Extract word counts as named vector
  doc_vector <- as.numeric(dfm_speeches[i, ])
  names(doc_vector) <- colnames(dfm_speeches)

  # Keep words that appear in the document and are in the embedding vocabulary
  present_words <- names(doc_vector)[doc_vector > 0]
  valid_words <- intersect(present_words, rownames(word_vectors))

   if (length(valid_words) > 0) {
    embeddings <- word_vectors[valid_words, , drop = FALSE]
    doc_embeddings[i, ] <- colMeans(embeddings)
  } else {
    doc_embeddings[i, ] <- NA  # or zeros
  }
}
```

We could now use these document embeddings as input to a downstream
task, such as clustering (unsupervised) or classification (supervised).

Let’s cluster these document embeddings in a two-dimensional space using
the t-sne algorithm. t-SNE is a technique for dimensionality reduction
that is particularly well-suited for visualizing high-dimensional data.
Let’s assume that we want to cluster the documents into 3 clusters.
We’ll use the `kmeans()` function to perform k-means clustering on the
two-dimensional t-SNE embeddings. This code is again a bit involved, but
it does the following: 1) Set a random seed for reproducibility; 2)
Apply t-SNE to the document embeddings to reduce them to two dimensions;
3) Perform k-means clustering on the two-dimensional embeddings,
specifying the number of clusters (centers); 4) Create a data frame with
the t-SNE coordinates, document names, and cluster labels; 5) Plot the
t-SNE coordinates with points colored by cluster labels and labeled with
document names.

``` r
set.seed(42)
doc_embeddings_2d <- Rtsne(doc_embeddings, 
                           dims = 2, 
                           perplexity = 30, 
                           verbose = TRUE, 
                           check_duplicates = FALSE)

kmeans_result <- kmeans(doc_embeddings_2d$Y, centers = 3)
cluster_labels <- kmeans_result$cluster



df <- data.frame(
  x = doc_embeddings_2d$Y[, 1],
  y = doc_embeddings_2d$Y[, 2],
  word = docnames(dfm_speeches),
  cluster = factor(cluster_labels)
)

doc_embeddings_plot <- ggplot(df, aes(x = x, y = y, label = word, color = cluster)) +
  geom_point(alpha = 0.6) +
  geom_text(check_overlap = TRUE, size = 2, vjust = 1.5) +
  theme_minimal()

print(doc_embeddings_plot)
```

## Exercises

Estimate new word vectors but this time on a feature co-occurrence
matrix with a window size of 5 but with more weight given to words when
they appear closer to the target word (see the *count* and *weight*
arguments in `fcm()`. To estimate this model comment out the code chunk
below to run the model)

``` r
#speeches_fcm_weighted <- fcm(tokens_speeches, 
#                    context = "window", 
#                    count = "weighted", 
#                    weights = 1 / (1:5),
#                    tri = TRUE)


#glove <- GlobalVectors$new(rank = 50, 
#                           x_max = 10)

#wv_main_weighted <- glove$fit_transform(speeches_fcm_weighted, 
#                                        n_iter = 10,
#                                        convergence_tol = 0.01, 
#                                        n_threads = 8)

#wv_context_weighted <- glove$components

#word_vectors_weighted <- wv_main_weighted + t(wv_context_weighted)
```

2.  Compare the nearest neighbors for crisis in both the original and
    the new model. Are they any different?

``` r
#find_similar_words("crisis", word_vectors)
#find_similar_words("crisis", word_vectors_weighted)
```

3.  Inspect the nearest neighbors for Greece, Portugal, Spain and Italy
    and substract the vectors for Netherlands, Germany, Denmark and
    Austria

``` r
#southern_northern  <- (word_vectors["greece", , drop = FALSE] +
#  word_vectors["portugal", , drop = FALSE] +
#  word_vectors["spain", , drop = FALSE] +
#  word_vectors["italy", , drop = FALSE] -
#  word_vectors["netherlands", , drop = FALSE] -
#  word_vectors["germany", , drop = FALSE] -
#  word_vectors["denmark", , drop = FALSE] -
#  word_vectors["austria", , drop = FALSE])


#cos_sim_southern_northern <- sim2(x = word_vectors, y = southern_northern, method = "cosine", norm = "l2")
#head(sort(cos_sim_southern_northern[,1], decreasing = TRUE), 20)
```

4.  And turn this vector around

``` r
#northern_southern  <- (word_vectors["netherlands", , drop = FALSE] +
#  word_vectors["germany", , drop = FALSE] +
# word_vectors["denmark", , drop = FALSE] +
# word_vectors["austria", , drop = FALSE] -
# word_vectors["greece", , drop = FALSE] -
#  word_vectors["portugal", , drop = FALSE] -
#  word_vectors["spain", , drop = FALSE] -
#  word_vectors["italy", , drop = FALSE])


#cos_sim_northern_southern <- sim2(x = word_vectors, y = northern_southern, method = "cosine", norm = "l2")
#head(sort(cos_sim_northern_southern[,1], decreasing = TRUE), 20)
```

5.  Inspect these word vectors further. If you receive a
    `subscript out of bounds` error, it means that the word does not
    appear in the corpus.
