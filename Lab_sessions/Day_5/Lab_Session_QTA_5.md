# QTA lab Session 5: Supervised machine learning


This document walks you through an example of supervised machine
learning to predict which UK prime minister delivered a speech. For this
we’ll use UK prime minister speeches from the
[EUSpeech](https://dataverse.harvard.edu/dataverse/euspeech) dataset.

For the supervised machine learning exercise you will need to install
the `quanteda.textmodels`, `quanteda.textplots` and the
`quanteda.textstats` packages. We will also use the `tidyverse` library
to create training and test sets. Furthermore, we will use the `caret`
library to produce a confusion matrix. This library requires some
dependencies (i.e., functions from other packages), so if you are
working from your computer will need install it like so:
`install.packages('caret', dependencies = TRUE)`.

``` r
#load libraries
library(quanteda)
library(quanteda.textmodels)
library(quanteda.textplots)
library(quanteda.textstats)
library(tidyverse)
library(caret)
```

Let’s read in the speeches using the `read.csv()` function. We’ll also
construct a corpus and select speeches from Gordon Brown and David
Cameron. The speeches have been cleaned already so we can go ahead
directly.

``` r
speeches <- read.csv(file = "speeches_uk.csv", 
                     header = TRUE, 
                     stringsAsFactors = FALSE, 
                     sep = ",", 
                     encoding = "UTF-8")

#construct a corpus
corpus_pm <- corpus(speeches,
                    text_field = "text")

#select speeches from Cameron and Brown
corpus_brown_cameron <- corpus_subset(corpus_pm, speaker != "T. Blair")
```

Let’s tokenise this corpus and create a dfm called dfm_brown_cameron.
We’ll remove punctuation, symbols, numbers, urls, separators, and split
hyphens. We’ll also remove stopwords.

``` r
tokens_brown_cameron <- tokens(corpus_brown_cameron,
                            what = "word",
                            remove_punct = TRUE, 
                            remove_symbols = TRUE, 
                            remove_numbers = TRUE,
                            remove_url = TRUE,
                            remove_separators = TRUE,
                            split_hyphens = FALSE,
                            padding = FALSE
                            ) %>%
  tokens_remove(stopwords("en"))
```

In order to make this dfm less sparse, we will only select features that
appear in at least 1% of speeches

``` r
dfm_brown_cameron <- dfm(tokens_brown_cameron ) %>%
  dfm_trim(min_docfreq = 0.01, docfreq_type = "prop")

dim(dfm_brown_cameron)
```

    [1]  776 6832

We now have a dfm containing 776 speeches delivered by either Gordon
Brown or David Cameron and approximately 6832 tokens.

## Naive Bayes

Let’s see if we can build a classifier to predict if a speech is
delivered by Cameron or Brown. First, we’ll generate a vector of 250
random numbers selected from the vector 1:776. We’ll also append an id
variable`id_numeric` to our dfm.

**NB**: The `set.seed()` function makes sure that you can reproduce your
random samples.

``` r
#set.seed() allows us to reproduce randomly generated results 

set.seed(2)

#generate a sample of 250 numbers without replacement from 1 to 776

id_train <- sample(1:nrow(dfm_brown_cameron), 250, replace = FALSE)
head(id_train, 10)
```

     [1] 710 774 416 392 273 349 204 381 297 690

``` r
#create id variable
docvars(dfm_brown_cameron, "id_numeric") <- 1:ndoc(dfm_brown_cameron)

#take note of how many speeches were delivered by either Brown or Cameron
table(docvars(dfm_brown_cameron, "speaker"))
```


    D. Cameron   G. Brown 
           493        283 

We then take a sample of 250 speeches as our training data and turn it
into a dfm. The `%in%` operator produces a logical vector of the same
length as id_numeric, and contains a TRUE if `id_numeric[i]` appears in
id_train and FALSE otherwise.

``` r
# create a training set: a dfm of 250 documents with row numbers included in id_train
train_dfm <- dfm_subset(dfm_brown_cameron, id_numeric %in% id_train)

#create a test set: a dfm of 100 documents whose row numbers are *not* included in id_train by using the negation operator `!`
test_dfm <- dfm_subset(dfm_brown_cameron, !id_numeric %in% id_train)
test_dfm <- dfm_sample(test_dfm, 100, replace = FALSE)

#check whether there is no overlap between the train set and the test set using the which() function
which((docvars(train_dfm, "id_numeric")  %in% docvars(test_dfm, "id_numeric")))
```

    integer(0)

We can now train a Naive Bayes classifier on the training set using the
`textmodel_nb()` function. We’ll use the `smooth` argument to apply
Laplace smoothing. This is a technique to avoid zero probabilities in
the model. The `smooth` argument specifies the smoothing parameter,
which is usually set to 1. We’ll also use the `prior` argument to
specify the prior distribution of the classes; “docfreq” denotes the
prior probability of each class is the proportion of documents in the
training set that belong to that class. This is the default setting in
`textmodel_nb()`. The `distribution` argument specifies the distribution
of the features.

``` r
speaker_classifier_nb <- textmodel_nb(train_dfm, 
                                      y = docvars(train_dfm, "speaker"), 
                                      smooth = 1,
                                      prior = "docfreq",
                                      distribution = "multinomial")

summary(speaker_classifier_nb)
```


    Call:
    textmodel_nb.dfm(x = train_dfm, y = docvars(train_dfm, "speaker"), 
        smooth = 1, prior = "docfreq", distribution = "multinomial")

    Class Priors:
    (showing first 2 elements)
    D. Cameron   G. Brown 
           0.6        0.4 

    Estimated Feature Scores:
               european   council   focused    issues        uk renegotiation
    D. Cameron 0.002007 0.0010389 0.0002302 0.0009799 0.0015525     1.299e-04
    G. Brown   0.001721 0.0006327 0.0001519 0.0016704 0.0005441     6.327e-06
               migration    talked     last     night     come     back   shortly
    D. Cameron 1.594e-04 0.0002302 0.001960 0.0003070 0.001806 0.002190 3.542e-05
    G. Brown   3.164e-05 0.0006897 0.002449 0.0002151 0.002449 0.001759 6.327e-05
                  first afternoon discussed   ongoing    crisis    facing    winter
    D. Cameron 0.003188 0.0001948 0.0004368 5.903e-05 0.0004368 0.0001889 4.722e-05
    G. Brown   0.003410 0.0001645 0.0004745 4.429e-05 0.0016767 0.0004239 2.531e-05
                   still     many  migrants   coming   europe   around  arriving
    D. Cameron 0.0012751 0.002586 5.313e-05 0.001116 0.001842 0.001358 4.722e-05
    G. Brown   0.0009491 0.002765 1.392e-04 0.001152 0.001316 0.001367 1.898e-05
                     via   eastern mediterranean
    D. Cameron 1.771e-05 1.181e-04     5.903e-05
    G. Brown   3.164e-05 6.327e-05     6.327e-06

Let’s analyze if we can predict whether speeches in the test set are
from Cameron or Brown. We’ll use the `predict()` function to predict the
speakers.

``` r
#Naive Bayes can only take features that occur both in the training set and the test set. We can make the features identical by passing train_dfm to dfm_match() as a pattern.

matched_dfm <- dfm_match(test_dfm, features = featnames(train_dfm))

#predict speaker 
pred_speaker_classifier_nb <- predict(speaker_classifier_nb, 
                                      newdata = matched_dfm, 
                                      type = "class")

head(pred_speaker_classifier_nb)
```

       text121    text748    text629    text326    text722    text228 
    D. Cameron   G. Brown   G. Brown D. Cameron   G. Brown D. Cameron 
    Levels: D. Cameron G. Brown

We could also predict the probability the model assigns to each class.
We could use this to create an ROC curve.

``` r
#predict probability of speaker
pred_prob_classifier_nb <- predict(speaker_classifier_nb, 
                                   newdata = matched_dfm, 
                                   type = "probability")

head(pred_prob_classifier_nb)
```

             
    docs         D. Cameron      G. Brown
      text121  1.000000e+00 3.651118e-107
      text748 8.513519e-157  1.000000e+00
      text629 2.306891e-125  1.000000e+00
      text326  1.000000e+00  3.879671e-09
      text722 4.265353e-109  1.000000e+00
      text228  1.000000e+00  5.575147e-42

Let’s see how well our classifier did by producing a confusion matrix. A
confusion matrix is a table that is often used to describe the
performance of a classification model on a set of test data for which
the true values are known.

``` r
tab_class_nb <- table(predicted_speaker_nb = pred_speaker_classifier_nb, 
                      actual_speaker = docvars(test_dfm, "speaker"))

print(tab_class_nb)
```

                        actual_speaker
    predicted_speaker_nb D. Cameron G. Brown
              D. Cameron         55        0
              G. Brown           11       34

So it appears we are somewhat successful at predicting whether a speech
is delivered by Cameron or Brown. Our accuracy is 90%.

Let’s have a look at the most predictive features for Cameron in the
complete corpus.

``` r
# Step 1: Extract the conditional probabilities. This code extracts the conditional probabilities of each feature given each class from the Naive Bayes model, transposes the result, and converts it to a data frame.

feature_probs <- as.data.frame(t(speaker_classifier_nb$param))

# Step 2: Add a column for the feature names (words)
feature_probs$feature <- rownames(feature_probs)

# Step 3: Compute log odds difference
# Avoid division by zero or log(0) by adding a small constant (if necessary)
epsilon <- 1e-10  # tiny number for stability

feature_probs$log_odds_diff <- log((feature_probs$`D. Cameron` + epsilon) /
                                   (feature_probs$`G. Brown` + epsilon))

# Step 4: Get the most distinctive features

# Top 20 words most associated with D. Cameron (positive log odds). This code orders the features by the log odds difference in descending order and selects the top 20 features most associated with Cameron.
top_cameron <- feature_probs[order(-feature_probs$log_odds_diff), ][1:20, ]
print(top_cameron)
```

                    D. Cameron     G. Brown       feature log_odds_diff
    syria         0.0008972633 1.265446e-05         syria      4.261332
    assad         0.0002892494 6.327232e-06         assad      3.822411
    syrian        0.0002833463 6.327232e-06        syrian      3.801792
    isil          0.0002715402 6.327232e-06          isil      3.759232
    muslims       0.0002066067 6.327232e-06       muslims      3.485939
    earn          0.0003128616 1.265446e-05          earn      3.207743
    kuwait        0.0003069585 1.265446e-05        kuwait      3.188695
    nick          0.0001534793 6.327232e-06          nick      3.188687
    dementia      0.0002892494 1.265446e-05      dementia      3.129272
    eurozone      0.0006965597 3.163616e-05      eurozone      3.091850
    grips         0.0001357701 6.327232e-06         grips      3.066085
    renegotiation 0.0001298671 6.327232e-06 renegotiation      3.021633
    tech          0.0001298671 6.327232e-06          tech      3.021633
    bosnia        0.0001239640 6.327232e-06        bosnia      2.975114
    intelligence  0.0005962079 3.163616e-05  intelligence      2.936286
    pays          0.0001180610 6.327232e-06          pays      2.926323
    hs2           0.0001180610 6.327232e-06           hs2      2.926323
    default       0.0001121579 6.327232e-06       default      2.875030
    charities     0.0003187646 1.898170e-05     charities      2.820973
    manifesto     0.0001062549 6.327232e-06     manifesto      2.820963

``` r
# Top 20 words most associated with G. Brown (negative log odds)
top_brown <- feature_probs[order(feature_probs$log_odds_diff), ][1:20, ]
print(top_brown)
```

                  D. Cameron     G. Brown     feature log_odds_diff
    indistinct  1.180610e-05 0.0013540276  indistinct     -4.742214
    downturn    5.903048e-06 0.0006390504    downturn     -4.684498
    barroso     5.903048e-06 0.0003796339     barroso     -4.163722
    scientific  5.903048e-06 0.0002973799  scientific     -3.919525
    taleban     5.903048e-06 0.0002594165     taleban     -3.782949
    knives      5.903048e-06 0.0002404348      knives     -3.706964
    brown       2.951524e-05 0.0011199200       brown     -3.636102
    low-carbon  5.903048e-06 0.0001834897  low-carbon     -3.436673
    pittsburgh  5.903048e-06 0.0001771625  pittsburgh     -3.401582
    zimbabwe    1.180610e-05 0.0003416705    zimbabwe     -3.365223
    visible     5.903048e-06 0.0001708353     visible     -3.365214
    gordon      3.541829e-05 0.0009870482      gordon     -3.327488
    supervision 1.180610e-05 0.0003163616 supervision     -3.288262
    much.tags   5.903048e-06 0.0001455263   much.tags     -3.204872
    knife       1.770915e-05 0.0004049428       knife     -3.129659
    copenhagen  1.770915e-05 0.0003986156  copenhagen     -3.113911
    havens      5.903048e-06 0.0001328719      havens     -3.113900
    heroes      5.903048e-06 0.0001202174      heroes     -3.013817
    antisocial  1.180610e-05 0.0002151259  antisocial     -2.902599
    volatility  5.903048e-06 0.0001075629  volatility     -2.902591

## Logistic regression

Let’s try a different classifier, a logistic regression. This regression
is not any different a logistic regression you may have come across
estimating a regression model for a binary dependent variable, but this
time we use it solely for prediction purposes.

``` r
speaker_classifier_lr <- textmodel_lr(train_dfm, 
                                      y = docvars(train_dfm, "speaker"))

#predict speaker 
pred_speaker_classifier_lr <- predict(speaker_classifier_lr, 
                                   newdata = matched_dfm, 
                                   type = "class")

#confusion matrix
tab_class_lr <- table(predicted_speaker_lr = pred_speaker_classifier_lr, 
                       actual_speaker = docvars(test_dfm, "speaker"))
print(tab_class_lr)
```

                        actual_speaker
    predicted_speaker_lr D. Cameron G. Brown
              D. Cameron         65        6
              G. Brown            1       28

We now need to decide which of these classifiers works best for us. As
discussed in class we would need to check precision, recall and F1
scores. The `confusionMatrix()` function in the `caret` package does
exactly that.

``` r
confusionMatrix(tab_class_nb, mode = "prec_recall")
```

    Confusion Matrix and Statistics

                        actual_speaker
    predicted_speaker_nb D. Cameron G. Brown
              D. Cameron         55        0
              G. Brown           11       34
                                              
                   Accuracy : 0.89            
                     95% CI : (0.8117, 0.9438)
        No Information Rate : 0.66            
        P-Value [Acc > NIR] : 1.123e-07       
                                              
                      Kappa : 0.7727          
                                              
     Mcnemar's Test P-Value : 0.002569        
                                              
                  Precision : 1.0000          
                     Recall : 0.8333          
                         F1 : 0.9091          
                 Prevalence : 0.6600          
             Detection Rate : 0.5500          
       Detection Prevalence : 0.5500          
          Balanced Accuracy : 0.9167          
                                              
           'Positive' Class : D. Cameron      
                                              

``` r
confusionMatrix(tab_class_lr, mode = "prec_recall")
```

    Confusion Matrix and Statistics

                        actual_speaker
    predicted_speaker_lr D. Cameron G. Brown
              D. Cameron         65        6
              G. Brown            1       28
                                              
                   Accuracy : 0.93            
                     95% CI : (0.8611, 0.9714)
        No Information Rate : 0.66            
        P-Value [Acc > NIR] : 1.615e-10       
                                              
                      Kappa : 0.8383          
                                              
     Mcnemar's Test P-Value : 0.1306          
                                              
                  Precision : 0.9155          
                     Recall : 0.9848          
                         F1 : 0.9489          
                 Prevalence : 0.6600          
             Detection Rate : 0.6500          
       Detection Prevalence : 0.7100          
          Balanced Accuracy : 0.9042          
                                              
           'Positive' Class : D. Cameron      
                                              

Based on the F1 score, our logistic regression classifier is performing
slightly better than predictions from our Naive Bayes classifier, so,
all else equal we would go with logistic regression.

## TF-IDF weighting

In order to improve on our predictions, we may think of other ways to
represent our documents. A common approach is to produce a
feature-weighted dfm by calculating the term-frequency-inverse document
frequency (tfidf) for each token. The intuition behind this
transformation is that it gives a higher weight to tokens that occur
often in a particular document but not much in other documents, compared
to tokens that occur often in a particular document but also in other
documents. Tf-idf weighting is done through `dfm_tfidf()`.

``` r
train_dfm_weighted <- dfm_tfidf(train_dfm)
matched_dfm_weighted <- dfm_tfidf(matched_dfm)

speaker_classifier_weighted_lr <- textmodel_lr(train_dfm_weighted, 
                                               y = docvars(train_dfm_weighted, "speaker"), 
                                               smooth = 1,
                                               prior = "docfreq",
                                               distribution = "multinomial")

pred_speaker_classifier_weigthed_lr <- predict(speaker_classifier_weighted_lr, 
                                               newdata = matched_dfm_weighted, 
                                               type = "class")

tab_class_weighted_lr <- table(predicted_speaker = pred_speaker_classifier_weigthed_lr, 
                               actual_speaker = docvars(test_dfm, "speaker"))

print(tab_class_weighted_lr)
```

                     actual_speaker
    predicted_speaker D. Cameron G. Brown
           D. Cameron         65        5
           G. Brown            1       29

We’ll, we indeed did slightly better this time.

## Exercises

For this set of exercises we will use the `data_corpus_moviereviews`
corpus that is stored in **quanteda**. This dataset contains 2000 move
reviews which are labeled as positive or negative. We’ll try to predict
these labels using supervised machine learning.

Apply `table()` to the sentiment variable appended to
`data_corpus_moviereviews` to inspect how many reviews are labelled
positive and negative.

Check the distribution of the number of tokens across reviews by
applying `ntoken()` to the corpus and then produce a histogram using
`hist()` .

Tokenise the corpus and save it as `tokens_reviews`.

Create a document-feature matrix and call it `dfm_reviews`

Apply `topfeatures()` to this dfm and get the 50 most frequent features.

Repeat this step, but get the 25 most frequent features from reviews
labelled as “pos” and “neg”, using the `groups` function in the
`topfeatures()` function.

## Supervised Machine Learning

Set a seed (`set.seed()`) to ensure reproducibility.

Create a new dfm with a random sample of 1500 reviews. We will use this
dfm as a training set. Call it `train_movies_dfm`. Use the sampling code
we used above.

Create another dfm with the remaining 500 reviews. Call it
`test_movies_dfm`. This will be our test set.

Apply `textmodel_nb()` to the dfm consisting of 1500 documents. Use
“sentiment” to train the classifier. Call the output `tmod_nb`

Predict the sentiment of the remaining 500 documents by using
`predict()`. Call the output `prediction`.

Create a cross-table/confusion matrix to assess the classification
performance using `table()`.
