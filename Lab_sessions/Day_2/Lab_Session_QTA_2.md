# QTA Lab Session 2: String operations and inspecting a corpus


## String operations

`R` stores text as a `string` or `character` vector. It is important to
become comfortable with string operations, since they allow you to clean
a string before you start analyzing your text.

These string operations require familiarity with regular expressions in
`R` as well as with functions in the **stringr** library.[^1]

Packages such as **quanteda** include some string cleaning functions as
well but knowledge of regular expressions and string operations allow
you to deal much better with with the specifics of your text. That said,
these operations usually involve lots of trial and error, Google, or
conversations with ChatGPT to figure out how to do certain things. But
they help you clean your data, which will save you lot of headaches
later on. Let’s have a look at a set of useful functions.

First load the `stringr` library in `R`.

``` r
library(stringr)
```

Then create a string vector called shopping_list:

``` r
shopping_list <- c("4 bananas", " 136 2 Apples", "20 oranges", "1 Milk", "2 eggs")
```

Vectors are basic objects in `R` which contain a set of values of the
same type (character, numeric, factor, etc.) The shopping_list contains
five character. Check that this is true with the `str()` function:

``` r
str(shopping_list)
```

     chr [1:5] "4 bananas" " 136 2 Apples" "20 oranges" "1 Milk" "2 eggs"

The `stringr` library contains many useful functions for working with
character values, which are listed in this [cheat
sheet](https://github.com/rstudio/cheatsheets/blob/master/strings.pdf).
Let’s go through a few examples, starting with the `str_extract()`
function for basic pattern matching.

`str_extract()` takes in two arguments: a string and a pattern to look
for. For each string it returns the pattern it found. Let’s see if it
finds a n in each of the strings:

``` r
str_extract(shopping_list, "n")
```

    [1] "n" NA  "n" NA  NA 

We can use `str_which()` to find the indexes of strings that contain a
pattern match:

``` r
str_which(shopping_list, "n")
```

    [1] 1 3

Using `str_locate()` we can find the position of the first match in each
string:

``` r
str_locate(shopping_list, "n")
```

         start end
    [1,]     5   5
    [2,]    NA  NA
    [3,]     7   7
    [4,]    NA  NA
    [5,]    NA  NA

Using `str_locate_all()` we can find the position of all matches in each
string:

``` r
str_locate_all(shopping_list, "n")
```

    [[1]]
         start end
    [1,]     5   5
    [2,]     7   7

    [[2]]
         start end

    [[3]]
         start end
    [1,]     7   7

    [[4]]
         start end

    [[5]]
         start end

Functions in the `stringr` library can work with regular expressions to
extract content from a string vector in a more systematic way. For
example, run the following line of code:

``` r
str_extract(shopping_list, "\\d")
```

    [1] "4" "1" "2" "1" "2"

Here `d` is a regular expression which refers to any number (**NB**:
without the escape character `\\` it would just refer to the letter d).
See what happens when you replace `d` with `d+` like so:
`str_extract(shopping_list, "\\d+")`. What does the + do?

``` r
str_extract_all(shopping_list, "\\d+")
```

    [[1]]
    [1] "4"

    [[2]]
    [1] "136" "2"  

    [[3]]
    [1] "20"

    [[4]]
    [1] "1"

    [[5]]
    [1] "2"

``` r
#your answer here
```

Let’s turn to alphabetic characters (aka letters). The regular
expression `[a-z]` refers to any lower case letter. The `+` symbol after
`[a-z]` refers to one or more lower case letters. The `{3,4}` refers to
a range of 3 to 4 lower case letters. The regular expression `[A-z]`
refers to any upper or lower case letter. The `\\b` refers to a word
boundary. Let’s see how these work in practice:

``` r
#extract the first lower case charachter in each string
str_extract(shopping_list, "[a-z]")
```

    [1] "b" "p" "o" "i" "e"

``` r
#extract lower case characters one or more times (again note the "+" symbol after "[a-z]")
str_extract(shopping_list, "[a-z]+")
```

    [1] "bananas" "pples"   "oranges" "ilk"     "eggs"   

``` r
#extract up to four lower case letters occurring in a row
str_extract(shopping_list, "[a-z]{3,4}")
```

    [1] "bana" "pple" "oran" "ilk"  "eggs"

``` r
#extract up to four upper OR lower case letters
str_extract(shopping_list, "[A-z]{1,4}")
```

    [1] "bana" "Appl" "oran" "Milk" "eggs"

``` r
#extract all letters in each string
str_extract_all(shopping_list, "[A-z]+")
```

    [[1]]
    [1] "bananas"

    [[2]]
    [1] "Apples"

    [[3]]
    [1] "oranges"

    [[4]]
    [1] "Milk"

    [[5]]
    [1] "eggs"

``` r
#extract all numbers in each string
str_extract_all(shopping_list, "\\d+")
```

    [[1]]
    [1] "4"

    [[2]]
    [1] "136" "2"  

    [[3]]
    [1] "20"

    [[4]]
    [1] "1"

    [[5]]
    [1] "2"

Note that str_extract_all generates a list of character strings as
output. A list is a data structure in R that can hold different types of
elements, including vectors, matrices, and other lists. Each element in
the list corresponds to an element in the original vector, and each
element in the list is itself a character vector containing the matches
found by `str_extract_all()`. This can be simplified into a character
matrix using the simplify command:

``` r
str_extract_all(shopping_list, "\\b[A-z]+\\b", 
                simplify = TRUE)
```

         [,1]     
    [1,] "bananas"
    [2,] "Apples" 
    [3,] "oranges"
    [4,] "Milk"   
    [5,] "eggs"   

``` r
str_extract_all(shopping_list, "\\d", 
                simplify = TRUE)
```

         [,1] [,2] [,3] [,4]
    [1,] "4"  ""   ""   ""  
    [2,] "1"  "3"  "6"  "2" 
    [3,] "2"  "0"  ""   ""  
    [4,] "1"  ""   ""   ""  
    [5,] "2"  ""   ""   ""  

Let’s have a look at the `str_replace()` function, which replaces a
pattern in a string with another pattern. This function takes in three
arguments: *string, pattern and replacement*. The string is the text you
want to modify, the pattern is the text you want to replace, and the
replacement is the text you want to replace it with.

Let’s replace the first vowel in each string with a dash. And then
replace all vowels with a dash:

``` r
#replace first vowel
str_replace(shopping_list, "[aeiou]", "-")
```

    [1] "4 b-nanas"     " 136 2 Appl-s" "20 -ranges"    "1 M-lk"       
    [5] "2 -ggs"       

``` r
#replace all vowels
str_replace_all(shopping_list, "[aeiou]", "-")
```

    [1] "4 b-n-n-s"     " 136 2 Appl-s" "20 -r-ng-s"    "1 M-lk"       
    [5] "2 -ggs"       

In *R*, you write regular expressions as strings, sequences of
characters surrounded by quotes (““) or single quotes (’’). Characters
like +, ?, ^, and . have a special meaning as regular expressions and
cannot be represented directly in an R string (see the RegEx [cheat
sheet](https://github.com/rstudio/cheatsheets/blob/master/strings.pdf)
for more examples). In order to match them literally, they need to be
preceded by two backslashes: \`\`\\”.

Let’s start with an example of a vector of names that for some reason
contain dots and plus signs.

``` r
name_list <- c("Jo.hn", "Anna.", "Si.+si", "Ma.ria")
```

Compare the output of these two calls

``` r
str_replace(name_list, ".", "-")
```

    [1] "-o.hn"  "-nna."  "-i.+si" "-a.ria"

``` r
str_replace(name_list, "\\.", "-")
```

    [1] "Jo-hn"  "Anna-"  "Si-+si" "Ma-ria"

Ccompare the output of these two calls:

``` r
str_replace(name_list, ".+", "-")
```

    [1] "-" "-" "-" "-"

``` r
str_replace(name_list, "\\.\\+", "-")
```

    [1] "Jo.hn"  "Anna."  "Si-si"  "Ma.ria"

## Inspecting a corpus

For the next part of this script we’ll first need to load the
**quanteda** package. **quanteda** is a package for the quantitative
analysis of textual data. It is a powerful package that allows you to
preprocess, analyze, and visualize text data.

``` r
library(quanteda)
```

**quanteda** has several in-built corpora we can use for exercises. One
of these corpora is `data_corpus_inaugural` which contains the inaugural
speeches of all the American presidents in a corpus format. Type
`summary(data_corpus_inaugural)` and inspect the object.

``` r
summary(data_corpus_inaugural,  n = 10)
```

    Corpus consisting of 60 documents, showing 10 documents:

                Text Types Tokens Sentences Year  President   FirstName
     1789-Washington   625   1537        23 1789 Washington      George
     1793-Washington    96    147         4 1793 Washington      George
          1797-Adams   826   2577        37 1797      Adams        John
      1801-Jefferson   717   1923        41 1801  Jefferson      Thomas
      1805-Jefferson   804   2380        45 1805  Jefferson      Thomas
        1809-Madison   535   1261        21 1809    Madison       James
        1813-Madison   541   1302        33 1813    Madison       James
         1817-Monroe  1040   3677       121 1817     Monroe       James
         1821-Monroe  1259   4886       131 1821     Monroe       James
          1825-Adams  1003   3147        74 1825      Adams John Quincy
                     Party
                      none
                      none
                Federalist
     Democratic-Republican
     Democratic-Republican
     Democratic-Republican
     Democratic-Republican
     Democratic-Republican
     Democratic-Republican
     Democratic-Republican

Let’s make a copy of this corpus. We’ll save it in our working
environment as an object called `speeches_inaugural`

``` r
speeches_inaugural <- data_corpus_inaugural
```

We can inspect the content of the first inaugural speech using
`as.character()` function:

``` r
as.character(speeches_inaugural)[1]
```

                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                1789-Washington 
    "Fellow-Citizens of the Senate and of the House of Representatives:\n\nAmong the vicissitudes incident to life no event could have filled me with greater anxieties than that of which the notification was transmitted by your order, and received on the 14th day of the present month. On the one hand, I was summoned by my Country, whose voice I can never hear but with veneration and love, from a retreat which I had chosen with the fondest predilection, and, in my flattering hopes, with an immutable decision, as the asylum of my declining years  -  a retreat which was rendered every day more necessary as well as more dear to me by the addition of habit to inclination, and of frequent interruptions in my health to the gradual waste committed on it by time. On the other hand, the magnitude and difficulty of the trust to which the voice of my country called me, being sufficient to awaken in the wisest and most experienced of her citizens a distrustful scrutiny into his qualifications, could not but overwhelm with despondence one who (inheriting inferior endowments from nature and unpracticed in the duties of civil administration) ought to be peculiarly conscious of his own deficiencies. In this conflict of emotions all I dare aver is that it has been my faithful study to collect my duty from a just appreciation of every circumstance by which it might be affected. All I dare hope is that if, in executing this task, I have been too much swayed by a grateful remembrance of former instances, or by an affectionate sensibility to this transcendent proof of the confidence of my fellow citizens, and have thence too little consulted my incapacity as well as disinclination for the weighty and untried cares before me, my error will be palliated by the motives which mislead me, and its consequences be judged by my country with some share of the partiality in which they originated.\n\nSuch being the impressions under which I have, in obedience to the public summons, repaired to the present station, it would be peculiarly improper to omit in this first official act my fervent supplications to that Almighty Being who rules over the universe, who presides in the councils of nations, and whose providential aids can supply every human defect, that His benediction may consecrate to the liberties and happiness of the people of the United States a Government instituted by themselves for these essential purposes, and may enable every instrument employed in its administration to execute with success the functions allotted to his charge. In tendering this homage to the Great Author of every public and private good, I assure myself that it expresses your sentiments not less than my own, nor those of my fellow citizens at large less than either. No people can be bound to acknowledge and adore the Invisible Hand which conducts the affairs of men more than those of the United States. Every step by which they have advanced to the character of an independent nation seems to have been distinguished by some token of providential agency; and in the important revolution just accomplished in the system of their united government the tranquil deliberations and voluntary consent of so many distinct communities from which the event has resulted can not be compared with the means by which most governments have been established without some return of pious gratitude, along with an humble anticipation of the future blessings which the past seem to presage. These reflections, arising out of the present crisis, have forced themselves too strongly on my mind to be suppressed. You will join with me, I trust, in thinking that there are none under the influence of which the proceedings of a new and free government can more auspiciously commence.\n\nBy the article establishing the executive department it is made the duty of the President \"to recommend to your consideration such measures as he shall judge necessary and expedient.\" The circumstances under which I now meet you will acquit me from entering into that subject further than to refer to the great constitutional charter under which you are assembled, and which, in defining your powers, designates the objects to which your attention is to be given. It will be more consistent with those circumstances, and far more congenial with the feelings which actuate me, to substitute, in place of a recommendation of particular measures, the tribute that is due to the talents, the rectitude, and the patriotism which adorn the characters selected to devise and adopt them. In these honorable qualifications I behold the surest pledges that as on one side no local prejudices or attachments, no separate views nor party animosities, will misdirect the comprehensive and equal eye which ought to watch over this great assemblage of communities and interests, so, on another, that the foundation of our national policy will be laid in the pure and immutable principles of private morality, and the preeminence of free government be exemplified by all the attributes which can win the affections of its citizens and command the respect of the world. I dwell on this prospect with every satisfaction which an ardent love for my country can inspire, since there is no truth more thoroughly established than that there exists in the economy and course of nature an indissoluble union between virtue and happiness; between duty and advantage; between the genuine maxims of an honest and magnanimous policy and the solid rewards of public prosperity and felicity; since we ought to be no less persuaded that the propitious smiles of Heaven can never be expected on a nation that disregards the eternal rules of order and right which Heaven itself has ordained; and since the preservation of the sacred fire of liberty and the destiny of the republican model of government are justly considered, perhaps, as deeply, as finally, staked on the experiment entrusted to the hands of the American people.\n\nBesides the ordinary objects submitted to your care, it will remain with your judgment to decide how far an exercise of the occasional power delegated by the fifth article of the Constitution is rendered expedient at the present juncture by the nature of objections which have been urged against the system, or by the degree of inquietude which has given birth to them. Instead of undertaking particular recommendations on this subject, in which I could be guided by no lights derived from official opportunities, I shall again give way to my entire confidence in your discernment and pursuit of the public good; for I assure myself that whilst you carefully avoid every alteration which might endanger the benefits of an united and effective government, or which ought to await the future lessons of experience, a reverence for the characteristic rights of freemen and a regard for the public harmony will sufficiently influence your deliberations on the question how far the former can be impregnably fortified or the latter be safely and advantageously promoted.\n\nTo the foregoing observations I have one to add, which will be most properly addressed to the House of Representatives. It concerns myself, and will therefore be as brief as possible. When I was first honored with a call into the service of my country, then on the eve of an arduous struggle for its liberties, the light in which I contemplated my duty required that I should renounce every pecuniary compensation. From this resolution I have in no instance departed; and being still under the impressions which produced it, I must decline as inapplicable to myself any share in the personal emoluments which may be indispensably included in a permanent provision for the executive department, and must accordingly pray that the pecuniary estimates for the station in which I am placed may during my continuance in it be limited to such actual expenditures as the public good may be thought to require.\n\nHaving thus imparted to you my sentiments as they have been awakened by the occasion which brings us together, I shall take my present leave; but not without resorting once more to the benign Parent of the Human Race in humble supplication that, since He has been pleased to favor the American people with opportunities for deliberating in perfect tranquillity, and dispositions for deciding with unparalleled unanimity on a form of government for the security of their union and the advancement of their happiness, so His divine blessing may be equally conspicuous in the enlarged views, the temperate consultations, and the wise measures on which the success of this Government must depend. " 

This produces the text of George Washington’s first inaugural speech.
Metadata such as year, speaker, etc. are stored in a corpus object as
*docvars*, and can be accessed like so:

``` r
#year
docvars(speeches_inaugural, "Year")
```

     [1] 1789 1793 1797 1801 1805 1809 1813 1817 1821 1825 1829 1833 1837 1841 1845
    [16] 1849 1853 1857 1861 1865 1869 1873 1877 1881 1885 1889 1893 1897 1901 1905
    [31] 1909 1913 1917 1921 1925 1929 1933 1937 1941 1945 1949 1953 1957 1961 1965
    [46] 1969 1973 1977 1981 1985 1989 1993 1997 2001 2005 2009 2013 2017 2021 2025

``` r
#party
head(docvars(speeches_inaugural, "Party"), 10)
```

     [1] none                  none                  Federalist           
     [4] Democratic-Republican Democratic-Republican Democratic-Republican
     [7] Democratic-Republican Democratic-Republican Democratic-Republican
    [10] Democratic-Republican
    Levels: Democratic Democratic-Republican Federalist none Republican Whig

Using the `table()` function we can inspect the number of presidents of
each party:

``` r
#number of presidents of each party
table(docvars(speeches_inaugural, "Party"))
```


               Democratic Democratic-Republican            Federalist 
                       22                     7                     1 
                     none            Republican                  Whig 
                        2                    25                     3 

We can also inspect the number of documents in the corpus using the
`ndoc()` function:

``` r
ndoc(speeches_inaugural)
```

    [1] 60

Subsetting a corpus is easy using the `corpus_subset()` function. Note
the `==` operator here. In `R` `=` is primarily used for assignment
within function calls. It assigns values to variables. It can also be
used for variable assignment, but this usage is less common compared to
the `<-` operator, which is the preferred assignment operator in R. The
`==` operator in `R` is used for comparison. It checks if two values are
equal and returns a logical value (TRUE or FALSE).

Using the `==` operator, we can create an object that only contains the
inaugural speech of Donald Trump and call it `trump_inaugural`

``` r
trump_inaugural <- corpus_subset(speeches_inaugural, President == "Trump")

ndoc(trump_inaugural)
```

    [1] 2

As you can see, Trump appears twice in the corpus, once for his first
inaugural speech in 2017 and once for his second inaugural speech in
2025. We can inspect the content of this corpus using `as.character()`:

``` r
as.character(trump_inaugural)[1]
```

                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             2017-Trump 
    "Chief Justice Roberts, President Carter, President Clinton, President Bush, President Obama, fellow Americans, and people of the world: thank you.\n\nWe, the citizens of America, are now joined in a great national effort to rebuild our country and restore its promise for all of our people.\n\nTogether, we will determine the course of America and the world for many, many years to come.\n\nWe will face challenges. We will confront hardships. But we will get the job done.\n\nEvery four years, we gather on these steps to carry out the orderly and peaceful transfer of power, and we are grateful to President Obama and First Lady Michelle Obama for their gracious aid throughout this transition. They have been magnificent. Thank you.\n\nToday's ceremony, however, has very special meaning. Because today we are not merely transferring power from one Administration to another, or from one party to another - but we are transferring power from Washington DC and giving it back to you, the people.\n\nFor too long, a small group in our nation's Capital has reaped the rewards of government while the people have borne the cost.\n\nWashington flourished - but the people did not share in its wealth.\n\nPoliticians prospered - but the jobs left, and the factories closed.\n\nThe establishment protected itself, but not the citizens of our country.\n\nTheir victories have not been your victories; their triumphs have not been your triumphs; and while they celebrated in our nation's capital, there was little to celebrate for struggling families all across our land.\n\nThat all changes - starting right here, and right now, because this moment is your moment: it belongs to you.\n\nIt belongs to everyone gathered here today and everyone watching all across America.\n\nThis is your day. This is your celebration.\n\nAnd this, the United States of America, is your country.\n\nWhat truly matters is not which party controls our government, but whether our government is controlled by the people.\n\nJanuary 20, 2017, will be remembered as the day the people became the rulers of this nation again.\n\nThe forgotten men and women of our country will be forgotten no longer.\n\nEveryone is listening to you now.\n\nYou came by the tens of millions to become part of a historic movement the likes of which the world has never seen before.\n\nAt the center of this movement is a crucial conviction: that a nation exists to serve its citizens.\n\nAmericans want great schools for their children, safe neighborhoods for their families, and good jobs for themselves.\n\nThese are just and reasonable demands of righteous people and a righteous public.\n\nBut for too many of our citizens, a different reality exists: mothers and children trapped in poverty in our inner cities; rusted-out factories scattered like tombstones across the landscape of our nation; an education system, flush with cash, but which leaves our young and beautiful students deprived of all knowledge; and the crime and the gangs and the drugs that have stolen too many lives and robbed our country of so much unrealized potential.\n\nThis American carnage stops right here and stops right now.\n\nWe are one nation - and their pain is our pain. Their dreams are our dreams; and their success will be our success. We share one heart, one home, and one glorious destiny.\n\nThe oath of office I take today is an oath of allegiance to all Americans.\n\nFor many decades, we've enriched foreign industry at the expense of American industry; subsidized the armies of other countries while allowing for the very sad depletion of our military; we've defended other nations' borders while refusing to defend our own; and spent trillions and trillions of dollars overseas while America's infrastructure has fallen into disrepair and decay.\n\nWe've made other countries rich while the wealth, strength, and confidence of our country has dissipated over the horizon.\n\nOne by one, the factories shuttered and left our shores, with not even a thought about the millions and millions of American workers that were left behind.\n\nThe wealth of our middle class has been ripped from their homes and then redistributed all across the world.\n\nBut that is the past. And now we are looking only to the future.\n\nWe assembled here today are issuing a new decree to be heard in every city, in every foreign capital, and in every hall of power.\n\nFrom this day forward, a new vision will govern our land.\n\nFrom this day forward, it's going to be only America first, America first.\n\nEvery decision on trade, on taxes, on immigration, on foreign affairs, will be made to benefit American workers and American families.\n\nWe must protect our borders from the ravages of other countries making our products, stealing our companies, and destroying our jobs. Protection will lead to great prosperity and strength.\n\nI will fight for you with every breath in my body - and I will never, ever let you down.\n\nAmerica will start winning again, winning like never before.\n\nWe will bring back our jobs. We will bring back our borders. We will bring back our wealth. And we will bring back our dreams.\n\nWe will build new roads, and highways, and bridges, and airports, and tunnels, and railways all across our wonderful nation.\n\nWe will get our people off of welfare and back to work - rebuilding our country with American hands and American labor.\n\nWe will follow two simple rules: buy American and hire American.\n\nWe will seek friendship and goodwill with the nations of the world - but we do so with the understanding that it is the right of all nations to put their own interests first.\n\nWe do not seek to impose our way of life on anyone, but rather to let it shine as an example for everyone to follow.\n\nWe will reinforce old alliances and form new ones - and unite the civilized world against radical Islamic terrorism, which we will eradicate from the face of the Earth.\n\nAt the bedrock of our politics will be a total allegiance to the United States of America, and through our loyalty to our country, we will rediscover our loyalty to each other.\n\nWhen you open your heart to patriotism, there is no room for prejudice.\n\nThe Bible tells us: \"How good and pleasant it is when God's people live together in unity.\"\n\nWe must speak our minds openly, debate our disagreements honestly, but always pursue solidarity.\n\nWhen America is united, America is totally unstoppable.\n\nThere should be no fear - we are protected, and we will always be protected.\n\nWe will be protected by the great men and women of our military and law enforcement and, most importantly, we are protected by God.\n\nFinally, we must think big and dream even bigger.\n\nIn America, we understand that a nation is only living as long as it is striving.\n\nWe will no longer accept politicians who are all talk and no action - constantly complaining but never doing anything about it.\n\nThe time for empty talk is over.\n\nNow arrives the hour of action.\n\nDo not let anyone tell you it cannot be done. No challenge can match the heart and fight and spirit of America.\n\nWe will not fail. Our country will thrive and prosper again.\n\nWe stand at the birth of a new millennium, ready to unlock the mysteries of space, to free the Earth from the miseries of disease, and to harness the energies, industries and technologies of tomorrow.\n\nA new national pride will stir ourselves, lift our sights, and heal our divisions.\n\nIt is time to remember that old wisdom our soldiers will never forget: that whether we are black or brown or white, we all bleed the same red blood of patriots, we all enjoy the same glorious freedoms, and we all salute the same great American Flag.\n\nAnd whether a child is born in the urban sprawl of Detroit or the windswept plains of Nebraska, they look up at the same night sky, they fill their heart with the same dreams, and they are infused with the breath of life by the same almighty Creator.\n\nSo to all Americans, in every city near and far, small and large, from mountain to mountain, and from ocean to ocean, hear these words:\n\nYou will never be ignored again.\n\nYour voice, your hopes, and your dreams, will define our American destiny. And your courage and goodness and love will forever guide us along the way.\n\nTogether, we will make America strong again.\n\nWe will make America wealthy again.\n\nWe will make America proud again.\n\nWe will make America safe again.\n\nAnd, yes, together, we will make America great again. Thank you, God bless you, and God bless America." 

``` r
as.character(trump_inaugural)[2]
```

                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     2021-Biden.txt 
    "Chief Justice Roberts, Vice President Harris, Speaker Pelosi, Leader Schumer, Leader McConnell, Vice President Pence, distinguished guests, and my fellow Americans.\n\nThis is America's day.\n\nThis is democracy's day.\n\nA day of history and hope.\n\nOf renewal and resolve.\n\nThrough a crucible for the ages America has been tested anew and America has risen to the challenge.\n\nToday, we celebrate the triumph not of a candidate, but of a cause, the cause of democracy.\n\nThe will of the people has been heard and the will of the people has been heeded.\n\nWe have learned again that democracy is precious.\n\nDemocracy is fragile.\n\nAnd at this hour, my friends, democracy has prevailed.\n\nSo now, on this hallowed ground where just days ago violence sought to shake this Capitol's very foundation, we come together as one nation, under God, indivisible, to carry out the peaceful transfer of power as we have for more than two centuries.\n\nWe look ahead in our uniquely American way – restless, bold, optimistic – and set our sights on the nation we know we can be and we must be.\n\nI thank my predecessors of both parties for their presence here.\n\nI thank them from the bottom of my heart.\n\nYou know the resilience of our Constitution and the strength of our nation.\n\nAs does President Carter, who I spoke to last night but who cannot be with us today, but whom we salute for his lifetime of service.\n\nI have just taken the sacred oath each of these patriots took — an oath first sworn by George Washington.\n\nBut the American story depends not on any one of us, not on some of us, but on all of us.\n\nOn \"We the People\" who seek a more perfect Union.\n\nThis is a great nation and we are a good people.\n\nOver the centuries through storm and strife, in peace and in war, we have come so far. But we still have far to go.\n\nWe will press forward with speed and urgency, for we have much to do in this winter of peril and possibility.\n\nMuch to repair.\n\nMuch to restore.\n\nMuch to heal.\n\nMuch to build.\n\nAnd much to gain.\n\nFew periods in our nation's history have been more challenging or difficult than the one we're in now.\n\nA once-in-a-century virus silently stalks the country.\n\nIt's taken as many lives in one year as America lost in all of World War II.\n\nMillions of jobs have been lost.\n\nHundreds of thousands of businesses closed.\n\nA cry for racial justice some 400 years in the making moves us. The dream of justice for all will be deferred no longer.\n\nA cry for survival comes from the planet itself. A cry that can't be any more desperate or any more clear.\n\nAnd now, a rise in political extremism, white supremacy, domestic terrorism that we must confront and we will defeat.\n\nTo overcome these challenges – to restore the soul and to secure the future of America – requires more than words.\n\nIt requires that most elusive of things in a democracy:\n\nUnity.\n\nUnity.\n\nIn another January in Washington, on New Year's Day 1863, Abraham Lincoln signed the Emancipation Proclamation.\n\nWhen he put pen to paper, the President said, \"If my name ever goes down into history it will be for this act and my whole soul is in it.\"\n\nMy whole soul is in it.\n\nToday, on this January day, my whole soul is in this:\n\nBringing America together.\n\nUniting our people.\n\nAnd uniting our nation.\n\nI ask every American to join me in this cause.\n\nUniting to fight the common foes we face:\n\nAnger, resentment, hatred.\n\nExtremism, lawlessness, violence.\n\nDisease, joblessness, hopelessness.\n\nWith unity we can do great things. Important things.\n\nWe can right wrongs.\n\nWe can put people to work in good jobs.\n\nWe can teach our children in safe schools.\n\nWe can overcome this deadly virus.\n\nWe can reward work, rebuild the middle class, and make health care\n\nsecure for all.\n\nWe can deliver racial justice.\n\nWe can make America, once again, the leading force for good in the world.\n\nI know speaking of unity can sound to some like a foolish fantasy.\n\nI know the forces that divide us are deep and they are real.\n\nBut I also know they are not new.\n\nOur history has been a constant struggle between the American ideal that we are all created equal and the harsh, ugly reality that racism, nativism, fear, and demonization have long torn us apart.\n\nThe battle is perennial.\n\nVictory is never assured.\n\nThrough the Civil War, the Great Depression, World War, 9/11, through struggle, sacrifice, and setbacks, our \"better angels\" have always prevailed.\n\nIn each of these moments, enough of us came together to carry all of us forward.\n\nAnd, we can do so now.\n\nHistory, faith, and reason show the way, the way of unity.\n\nWe can see each other not as adversaries but as neighbors.\n\nWe can treat each other with dignity and respect.\n\nWe can join forces, stop the shouting, and lower the temperature.\n\nFor without unity, there is no peace, only bitterness and fury.\n\nNo progress, only exhausting outrage.\n\nNo nation, only a state of chaos.\n\nThis is our historic moment of crisis and challenge, and unity is the path forward.\n\nAnd, we must meet this moment as the United States of America.\n\nIf we do that, I guarantee you, we will not fail.\n\nWe have never, ever, ever failed in America when we have acted together.\n\nAnd so today, at this time and in this place, let us start afresh.\n\nAll of us.\n\nLet us listen to one another.\n\nHear one another.\n\nSee one another.\n\nShow respect to one another.\n\nPolitics need not be a raging fire destroying everything in its path.\n\nEvery disagreement doesn't have to be a cause for total war.\n\nAnd, we must reject a culture in which facts themselves are manipulated and even manufactured.\n\nMy fellow Americans, we have to be different than this.\n\nAmerica has to be better than this.\n\nAnd, I believe America is better than this.\n\nJust look around.\n\nHere we stand, in the shadow of a Capitol dome that was completed amid the Civil War, when the Union itself hung in the balance.\n\nYet we endured and we prevailed.\n\nHere we stand looking out to the great Mall where Dr. King spoke of his dream.\n\nHere we stand, where 108 years ago at another inaugural, thousands of protestors tried to block brave women from marching for the right to vote.\n\nToday, we mark the swearing-in of the first woman in American history elected to national office – Vice President Kamala Harris.\n\nDon't tell me things can't change.\n\nHere we stand across the Potomac from Arlington National Cemetery, where heroes who gave the last full measure of devotion rest in eternal peace.\n\nAnd here we stand, just days after a riotous mob thought they could use violence to silence the will of the people, to stop the work of our democracy, and to drive us from this sacred ground.\n\nThat did not happen.\n\nIt will never happen.\n\nNot today.\n\nNot tomorrow.\n\nNot ever.\n\nTo all those who supported our campaign I am humbled by the faith you have placed in us.\n\nTo all those who did not support us, let me say this: Hear me out as we move forward. Take a measure of me and my heart.\n\nAnd if you still disagree, so be it.\n\nThat's democracy. That's America. The right to dissent peaceably, within the guardrails of our Republic, is perhaps our nation's greatest strength.\n\nYet hear me clearly: Disagreement must not lead to disunion.\n\nAnd I pledge this to you: I will be a President for all Americans.\n\nI will fight as hard for those who did not support me as for those who did.\n\nMany centuries ago, Saint Augustine, a saint of my church, wrote that a people was a multitude defined by the common objects of their love.\n\nWhat are the common objects we love that define us as Americans?\n\nI think I know.\n\nOpportunity.\n\nSecurity.\n\nLiberty.\n\nDignity.\n\nRespect.\n\nHonor.\n\nAnd, yes, the truth.\n\nRecent weeks and months have taught us a painful lesson.\n\nThere is truth and there are lies.\n\nLies told for power and for profit.\n\nAnd each of us has a duty and responsibility, as citizens, as Americans, and especially as leaders – leaders who have pledged to honor our Constitution and protect our nation — to defend the truth and to defeat the lies.\n\nI understand that many Americans view the future with some fear and trepidation.\n\nI understand they worry about their jobs, about taking care of their families, about what comes next.\n\nI get it.\n\nBut the answer is not to turn inward, to retreat into competing factions, distrusting those who don't look like you do, or worship the way you do, or don't get their news from the same sources you do.\n\nWe must end this uncivil war that pits red against blue, rural versus urban, conservative versus liberal.\n\nWe can do this if we open our souls instead of hardening our hearts.\n\nIf we show a little tolerance and humility.\n\nIf we're willing to stand in the other person's shoes just for a moment.\n\nBecause here is the thing about life: There is no accounting for what fate will deal you.\n\nThere are some days when we need a hand.\n\nThere are other days when we're called on to lend one.\n\nThat is how we must be with one another.\n\nAnd, if we are this way, our country will be stronger, more prosperous, more ready for the future.\n\nMy fellow Americans, in the work ahead of us, we will need each other.\n\nWe will need all our strength to persevere through this dark winter.\n\nWe are entering what may well be the toughest and deadliest period of the virus.\n\nWe must set aside the politics and finally face this pandemic as one nation.\n\nI promise you this: as the Bible says weeping may endure for a night but joy cometh in the morning.\n\nWe will get through this, together\n\nThe world is watching today.\n\nSo here is my message to those beyond our borders: America has been tested and we have come out stronger for it.\n\nWe will repair our alliances and engage with the world once again.\n\nNot to meet yesterday's challenges, but today's and tomorrow's.\n\nWe will lead not merely by the example of our power but by the power of our example.\n\nWe will be a strong and trusted partner for peace, progress, and security.\n\nWe have been through so much in this nation.\n\nAnd, in my first act as President, I would like to ask you to join me in a moment of silent prayer to remember all those we lost this past year to the pandemic.\n\nTo those 400,000 fellow Americans – mothers and fathers, husbands and wives, sons and daughters, friends, neighbors, and co-workers.\n\nWe will honor them by becoming the people and nation we know we can and should be.\n\nLet us say a silent prayer for those who lost their lives, for those they left behind, and for our country.\n\nAmen.\n\nThis is a time of testing.\n\nWe face an attack on democracy and on truth.\n\nA raging virus.\n\nGrowing inequity.\n\nThe sting of systemic racism.\n\nA climate in crisis.\n\nAmerica's role in the world.\n\nAny one of these would be enough to challenge us in profound ways.\n\nBut the fact is we face them all at once, presenting this nation with the gravest of responsibilities.\n\nNow we must step up.\n\nAll of us.\n\nIt is a time for boldness, for there is so much to do.\n\nAnd, this is certain.\n\nWe will be judged, you and I, for how we resolve the cascading crises of our era.\n\nWill we rise to the occasion?\n\nWill we master this rare and difficult hour?\n\nWill we meet our obligations and pass along a new and better world for our children?\n\nI believe we must and I believe we will.\n\nAnd when we do, we will write the next chapter in the American story.\n\nIt's a story that might sound something like a song that means a lot to me.\n\nIt's called \"American Anthem\" and there is one verse stands out for me:\n\n\"The work and prayers\n\nof centuries have brought us to this day\n\nWhat shall be our legacy?\n\nWhat will our children say?…\n\nLet me know in my heart\n\nWhen my days are through\n\nAmerica\n\nAmerica\n\nI gave my best to you.\"\n\nLet us add our own work and prayers to the unfolding story of our nation.\n\nIf we do this then when our days are through our children and our children's children will say of us they gave their best.\n\nThey did their duty.\n\nThey healed a broken land.\n\nMy fellow Americans, I close today where I began, with a sacred oath.\n\nBefore God and all of you I give you my word.\n\nI will always level with you.\n\nI will defend the Constitution.\n\nI will defend our democracy.\n\nI will defend America.\n\nI will give my all in your service thinking not of power, but of possibilities.\n\nNot of personal interest, but of the public good.\n\nAnd together, we shall write an American story of hope, not fear.\n\nOf unity, not division.\n\nOf light, not darkness.\n\nAn American story of decency and dignity.\n\nOf love and of healing.\n\nOf greatness and of goodness.\n\nMay this be the story that guides us.\n\nThe story that inspires us.\n\nThe story that tells ages yet to come that we answered the call of history.\n\nWe met the moment.\n\nThat democracy and hope, truth and justice, did not die on our watch but thrived.\n\nThat our America secured liberty at home and stood once again as a beacon to the world.\n\nThat is what we owe our forebearers, one another, and generations to follow.\n\nSo, with purpose and resolve we turn to the tasks of our time.\n\nSustained by faith.\n\nDriven by conviction.\n\nAnd, devoted to one another and to this country we love with all our hearts.\n\nMay God bless America and may God protect our troops.\n\nThank you, America." 

If the documents are clean enough (i.e., with correct interpunction
etc.), then it is easy in **quanteda** to break down a document on a
sentence to sentence basis using `corpus_reshape()`, which reshapes the
corpus to a different level of granularity, in this case to the level of
sentences.

``` r
trump_sentence <- corpus_reshape(trump_inaugural, to =  "sentences")

ndoc(trump_sentence)
```

    [1] 304

As you can see, Trump’s two inaugural speeches consisted of 304
sentences.

Before we preprocess our texts, we first need to tokenize it using
`tokens()`. Tokenization is the process of breaking a text into smaller
units, such as words.

``` r
tokens_speeches_inaugural <- tokens(speeches_inaugural)
```

Using `tokens_compound()` we can create multiword expressions. Multiword
expressions are phrases that consist of more than one word. Let’s say we
want to make certain that references to the `United States of America`
are recognized as such in subsequent analyses. We can do so using the
following line of code:

``` r
tokens_speeches_inaugural <- tokens_compound(tokens_speeches_inaugural, phrase("United States of America"))
```

Let’s see how often each President referred to the United States of
America in their inaugural speeches:

``` r
str_count(as.character(speeches_inaugural), "United States of America")
```

     [1] 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
    [39] 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 1 1 1 2 1 1

Apparently, this is a turn of phrase that is used more often in recent
years, but not so much in the past.

Using the `kwic()` function, we can inspect the context in which the
United States of America is used in these speeches. The `kwic()`
function takes in a tokenized text and a pattern to look for. It returns
a keyword-in-context (KWIC) object, which is a data frame that shows the
context in which the pattern occurs. In this case we’ll look at the
context in which the United States of America of 10 words before and
after the phrase:

``` r
kwic(tokens_speeches_inaugural, 
     pattern = phrase("United_States_of_America"),
     window = 10)  %>%
  tail()
```

    Keyword-in-context with 6 matches.                                                                           
         [2009-Obama, 2685]             you. God bless you. And God bless the |
         [2013-Obama, 2313]     God bless you, and may He forever bless these |
          [2017-Trump, 347]         . This is your celebration. And this, the |
         [2017-Trump, 1140] of our politics will be a total allegiance to the |
         [2025-Trump, 1051]            . And, we must meet this moment as the |
     [2021-Biden.txt, 1051]            . And, we must meet this moment as the |
                                                                            
     United_States_of_America | .                                           
     United_States_of_America | .                                           
     United_States_of_America | , is your country. What truly matters is not
     United_States_of_America | , and through our loyalty to our country, we
     United_States_of_America | . If we do that, I guarantee you,           
     United_States_of_America | . If we do that, I guarantee you,           

## Excercise: inspecting a corpus

For these exercises we’ll use the `speeches_inaugural` corpus that we
just created.

Explore `corpus_subset()` and filter only speeches delivered since 1990.
Store this object as `speeches_inaug_since1990`.

``` r
#your answer here
```

Explore `corpus_reshape()`. Reshape `speeches_inaugural_since1990`to the
level of sentences and store this object as `sentences_inaug_since1990`.
What is the number of sentences (= number of documents) in this text
corpus?

``` r
#your answer here
```

Inspect how often references to the `pursuit of happiness` occur in this
corpus:

Tokenize the sentence-level text corpus. Make sure that references to
the `Supreme Court`, `United States of America`, and
`pursuit of happiness` are included as multiword expressions.

``` r
#your answer here
```

Use corpus_reshape() and change the unit of `speeches_inaug_since1990`
to the level of paragraphs. How many paragraphs does this corpus
contain?

``` r
#your answer here
```

## Excercise: cleaning a text vector

The local zoo has made an inventory of its animal stock.[^2] However,
the zoo keepers did a messy job with writing up totals as you can see
below. You are hired to clean up the mess using *R*.

``` r
zoo <- c("bear x2", "Ostric7", "platypus x60", "x7 Eliphant", "x16 conDOR")
```

Use the functions in the `stringr` to clean up the string, taking out
typos. Generate a dataframe with the following variables: *animal*
(character), *number* (numeric).

``` r
#your answer here
```

Plot this data using `ggplot2`, call the resulting plot `zoo_plot`, and
save it in a file called `zoo_plot.png` in the current working
directory.

``` r
#your answer here
```

[^1]: `R` is open source with different developers working on similar
    issues, as a result of which there can be multiple packages that do
    the same thing. For example, functions in base `R` and the
    **stringi** package also let you manipulate strings in `R`. However,
    the syntax of the function calls in these different packages is
    different.

[^2]: This exercise is based on an example from Automated Data
    Collection With R, by Munzert *et al* (2015).
