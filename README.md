[![build status](http://img.shields.io/travis/datasciencecampus/pyGrams/master.svg?style=flat)](https://travis-ci.org/datasciencecampus/pyGrams)
[![Build status](https://ci.appveyor.com/api/projects/status/oq49c4xuhd8j2mfp/branch/master?svg=true)](https://ci.appveyor.com/project/IanGrimstead/patent-app-detect/branch/master)
[![codecov](https://codecov.io/gh/datasciencecampus/pyGrams/branch/master/graph/badge.svg)](https://codecov.io/gh/datasciencecampus/pyGrams)
[![LICENSE.](https://img.shields.io/badge/license-OGL--3-blue.svg?style=flat)](http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/)

# pyGrams 

<p align="center"><img align="center" src="meta/images/pygrams-logo.png" width="400px"></p>

## Description of tool

This python-based app (`pygrams.py`) is designed to extract popular or emergent n-grams/terms (words or short phrases) from free text within a large (>1,000) corpus of documents. Example corpora of granted patent document abstracts are included for testing purposes.

The app pipeline (more details in the user option section):
1. **[Input Text Data](#input-text-data)** Text data can be input by several text document types (ie. csv, xls, pickled python dataframes, etc) 
2. **TFIDF Dictionary**  This is the processed list of terms (ngrams) out of the whole corpus. These terms are the columns of the [TFIDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) sparse matrix. The user can control the following parameters: minimum document frequency, stopwords, ngram range. 
3. **TFIDF Computation** Grab a coffee if your text corpus is long (>1 million docs) :)
4. **Filters** These are filters to use on the computed TFIDF matrix. They consist of document filters and term filters
   1. **Document Filters** These filters work on document level. Examples are: date range, column features (eg. cpc classification), document length normalisation and time weighting.
   2. **Term Filters** These filters work on term level. Examples are: search terms list (eg. pharmacy, medicine, chemist)
5. **Mask the TFIDF Matrix** Apply the filters to the TFIDF matrix
6. **Emergence Calculations** Options include [Porter 2018](https://www.researchgate.net/publication/324777916_Emergence_scoring_to_identify_frontier_RD_topics_and_key_players) emergence calculations or curve fitting. 
7. **Emergence Forecasts** Options include ARIMA, linear and quadratic regression, Holt-Winters, LSTMs. 
8. **Outputs** The default 'report' output is a ranked and scored list of 'popular' ngrams or emergent ones if selected. Other outputs include a 'graph summary', word cloud and an html document as emergence report.

## Installation guide

pyGrams.py has been developed to work on both Windows and MacOS. To install:

1. Please make sure Python 3.6 is installed and set in your path.  

   To check the Python version default for your system, run the following in command line/terminal:

   ```
   python --version
   ```

   **_Note_**: If Python 2.x is the default Python version, but you have installed Python 3.x, your path may be setup to use `python3` instead of `python`.

2. To install pyGrams packages and dependencies, from the root directory (./pyGrams) run:

   ``` 
   pip install -e
   ```

   This will install all the libraries and run some tests. If the tests pass, the app is ready to run. If any of the tests fail, please email [ons.patent.explorer@gmail.com](mailto:ons.patent.explorer@gmail.com) with a screenshot of the failure so that we may get back to you, or alternatively open a [GitHub issue here](https://github.com/datasciencecampus/pyGrams/issues).

### System requirements

We have stress-tested `pygrams.py` using Windows 10 (64-bit) with 8GB memory (VM hosted on 2.1GHz Xeon E5-2620). We observed a linear increase in both execution time and memory usage in relation to number of documents analysed, resulting in:

- Processing time: 41.2 documents/sec
- Memory usage: 236.9 documents/MB

For the sample files, this was recorded as:

- 1,000 documents: 0:00:37
- 10,000 documents: 0:04:45 (285s); 283MB
- 100,000 documents: 0:40:10 (2,410s); 810MB
- 500,000 documents: 3:22:08 (12,128s); 2,550MB

## User guide

pyGrams is command line driven, and called in the following manner:

```
python pygrams.py
```

### Input Text Data

#### Selecting the document source (-ds, -pt)

This argument is used to select the corpus of documents to analyse. The default source is a pre-created random 1,000 patent dataset from the USPTO, `USPTO-random-1000.pkl.bz2`. 

Pre-created datasets of 100, 1,000, 10,000, 100,000, and 500,000 patents are available in the `./data` folder:

- ```USPTO-random-100.pkl.bz2```
- ```USPTO-random-1000.pkl.bz2```
- ```USPTO-random-10000.pkl.bz2```
- ```USPTO-random-100000.pkl.bz2```
- ```USPTO-random-500000.pkl.bz2```

For example, to load the 10,000 pickled dataset for patents, use:

```
python pygrams.py -ds=USPTO-random-10000.pkl.bz2
```

To use your own document dataset, either place in the `./data` folder of pyGrams or change the path using `-pt`. File types currently supported are:

- pkl.bz2: compressed pickle file containing a dataset
- xlsx: new Microsoft excel format
- xls: old Microsoft excel format
- csv: comma separated value file (with headers)

Datasets should contain the following columns:

|Column			            |	    Required?  |
| :------------------------ | -------------------:|
|Unique ID                  |       Yes           |
|Free text field            |       Yes           |
|Date                       |       Optional      |
|Boolean fields (Yes/No)    |       Optional      |

The unique ID field should contain unique identifiers for each row of the dataset. The free text field can be any free text, for example an abstract. The date field should be in the format `YYYY-MM-DD HH:MM:SS`. The boolean fields can be any Yes/No data (there may be multiple)

#### Selecting the column header names (-nh, -th, -dh)

When loading a document dataset, you will need to provide the column header names for each, using:

- `-nh`: unique ID column (default is 'id')
- `-th`: free text field column (default is 'text')
- `-dh`: date column (default is 'date')

For example, for a corpus of book blurbs you could use:

```
python pygrams.py -nh='book_name' -th='blurb' -dh='published_date'
```

#### Word filters (-fh, -fb)

If you want to filter results, such as for female, British, authors in the above example, you can specify the boolean (yes/no) column names you wish to filter by, and the type of filter you want to apply, using:

- `-fh`: the list of boolean fields (default is None)
- `-fb`: the type of filter (choices are `'union'` (default), where all fields need to be 'yes', or `'intersection'`, where any field can be 'yes') 

```
python pygrams.py -fh=['female','british'] -fb='union'
```

#### Time filters (-df, -dt)

This argument can be used to filter documents to a certain timeframe. For example, the below will restrict the document cohort to only those from 20 Feb 2000 up to now (the default start date being 1 Jan 1900).

```
python pygrams.py -df=2000/01/20
```

The following will restrict the document cohort to only those between 1 March 2000 and 31 July 2016.

```
python pygrams.py -df=2000/03/01 -dt=2016/07/31
```

### TF-IDF Parameters 

#### N-gram selection (-mn, -mx)

An n-gram is a contiguous sequence of n items ([source](https://en.wikipedia.org/wiki/N-gram)). N-grams can be unigrams (single words, e.g., vehicle), bigrams (sequences of two words, e.g., aerial vehicle), trigrams (sequences of three words, e.g., unmanned aerial vehicle) or any n number of continuous terms. 

The following arguments will set the n-gram limit to be unigrams, bigrams or trigrams (the default).

```
python pygrams.py -mn=1 -mx=3
```

To analyse only unigrams:

```
python pygrams.py -mn=1 -mx=1
```

#### Maximum document frequency (-mdf)

Terms identified are filtered by the maximum number of documents that use this term; the default is 0.3, representing an upper limit of 30% of documents containing this term. If a term occurs in more that 30% of documents it is rejected.

For example, to set the maximum document frequency to 5% (the default), use:

```
python pygrams.py -mdf 0.05
```

By using a small (5% or less) maximum document frequency for unigrams, this may help remove generic words, or stop words.

#### TFIDF score mechanics (-p)

By default the TFIDF score will be calculated per n-gram as the sum of the TF-IDF values over all documents for the selected n-gram. However you can select:

- `median`: the median value
- `max`: the maximum value
- `sum`: the sum of all values
- `avg`: the average, over non zero values

To choose an average scoring for example, use:

```
python pygrams.py -p='avg'
```

#### Normalise by document length (-ndl)

This option normalises the TFIDF scores by document length.

```
python pygrams.py -ndl
```

#### Time-weighting (-t)

This option applies a linear weight that starts from 0.01 and ends at 1 between the time limits.

```
python pygrams.py -t
```

### Outputs Parameters (-o)

The default option outputs a report of top ranked terms. Additional command line arguments provide alternative options, for example a word cloud or force directed graph (fdg) output. The option 'all', produces all:

```
python pygrams.py -o='report'
python pygrams.py -o='wordcloud'
python pygrams.py -o='fdg'
python pygrams.py -o='table'
python pygrams.py -o='tfidf'
python pygrams.py -o='termcounts'
python pygrams.py -o='all'
```

The output options generate:

- `report` (default): a text file containing top n terms (default is 250 terms, see `-np` for more details)
- `wordcloud`: a word cloud containing top n terms (default is 250 terms, see `-nd` for more details)
- `fdg`: a force-directed graph containing top n terms (default is 250 terms, see `-nf` for more details)
- `table`: an XLS spreadsheet to compare term rankings
- `tfidf`: a pickle of the TFIDF matrix
- `termcounts`: a pickle of term counts per week
- `all`: all of the above

Note that all outputs are generated in the `outputs` subfolder. Below are some example outputs:

#### Report ('report')

The report will output the top n number of terms (default is 250) and their associated [TFIDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) score. Below is an example for patent data, where only bigrams have been analysed.

|Term			            |	    TFIDF Score  |
| :------------------------ | -------------------:|
|1. fuel cell               |       2.143778      |
|2. heat exchanger          |       1.697166      |
|3. exhaust gas             |       1.496812      |
|4. combustion engine       |       1.480615      |
|5. combustion chamber      |       1.390726      |
|6. energy storage          |       1.302651      |
|7. internal combustion     |       1.108040      |
|8. positive electrode      |       1.100686      |
|9. carbon dioxide          |       1.092638      |
|10. control unit           |       1.069478      |

#### Wordcloud ('wordcloud')

A wordcloud, or tag cloud, is a novel visual representation of text data, where words (tags) importance is shown with font size and colour. Here is a [wordcloud](https://raw.githubusercontent.com/datasciencecampus/pygrams/master/outputs/wordclouds/wordcloud_tech.png) using patent data. The greater the TFIDF score, the larger the font size of the term.

#### Force directed graph ('fdg')

This output provides an interactive HTML graph. The graph shows connections between terms that are generally found in the same documents.

#### TFIDF matrix ('tfidf')

The TFIDF matrix can be saved as a pickle file, containing a list of four items:

- The TFIDF sparse matrix
- List of unique terms
- List of document publication dates
- List of unique document IDs

#### Term counts matrix

Of use for further processing, the number of patents containing a term in a given week
(stored as a matrix) can be output as a pickle file, containing a list of four items:
- Term counts per week (sparse matrix)
- List of terms (column heading)
- List of number of patents in that week
- List of patent publication dates (row heading)

### Patent specific support

#### Choosing CPC classification

This subsets the chosen patents dataset to a particular Cooperative Patent Classification (CPC) class, for example Y02. The Y02 classification is for "technologies or applications for mitigation or adaptation against climate change". 
An example script is:

```
python pygrams.py -cpc=Y02 -ps=USPTO-random-10000.pkl.bz2
```

In the console the number of subset patents will be stated. For example, for `python pygrams.py -cpc=Y02 -ps=USPTO-random-10000.pkl.bz2` the number of Y02 patents is 197. Thus, the TFIDF will be run for 197 patents.

### Config files

There are three configuration files available inside the config directory:

- stopwords_glob.txt
- stopwords_n.txt
- stopwords_uni.txt

The first file (stopwords_glob.txt) contains stopwords that are applied to all n-grams. The second file contains stopwords that are applied to all n-grams for n > 1 (bigrams and trigrams). The last file (stopwords_uni.txt) contains stopwords that apply only to unigrams. The users can append stopwords into this files, to stop undesirable output terms.

### Folder structure

- pygrams.py is the main python program file in the root folder (Pygrams).
- README.md is this markdown readme file in the root folder
- pipeline.py in the scripts folder provides the main program sequence along with pygrams.py.
- The 'data' folder is where to place the source text data files.
- The 'outputs' folder contains all the program outputs.
- The 'config' folder contains the stop word configuration files.
- The setup file in the root folder, along with the meta folder, contain installation related files.
- The test folder contains unit tests.

## Help

A help function details the range and usage of these command line arguments:

```
python pygrams.py -h
```

The help output is included below. This starts with a summary of arguments:

```
python pygrams.py -h
usage: pygrams.py [-h] [-ds DOC_SOURCE] [-it INPUT_TFIDF] [-th TEXT_HEADER]
                  [-dh DATE_HEADER] [-fc FILTER_COLUMNS]
                  [-fb {union,intersection}] [-df DATE_FROM] [-dt DATE_TO]
                  [-mn {1,2,3}] [-mx {1,2,3}] [-mdf MAX_DOCUMENT_FREQUENCY]
                  [-p {median,max,sum,avg}] [-ndl] [-t]
                  [-o [{graph,wordcloud,report,tfidf,termcounts} [{graph,wordcloud,report,tfidf,termcounts} ...]]]
                  [-on OUTPUTS_NAME] [-wt WORDCLOUD_TITLE] [-nltk NLTK_PATH]
                  [-np NUM_NGRAMS_REPORT] [-nd NUM_NGRAMS_WORDCLOUD]
                  [-nf NUM_NGRAMS_FDG] [-cpc CPC_CLASSIFICATION]

extract popular n-grams (words or short phrases) from a corpus of documents
```
It continues with a detailed description of the arguments:
```
optional arguments:
  -h, --help            show this help message and exit
  -ds DOC_SOURCE, --doc_source DOC_SOURCE
                        the document source to process (default: USPTO-
                        random-1000.pkl.bz2)
  -it INPUT_TFIDF, --input_tfidf INPUT_TFIDF
                        Load a pickled TFIDF output instead of creating TFIDF
                        by processing a document source (default: None)
  -th TEXT_HEADER, --text_header TEXT_HEADER
                        the column name for the free text (default: abstract)
  -dh DATE_HEADER, --date_header DATE_HEADER
                        the column name for the date (default: None)
  -fc FILTER_COLUMNS, --filter_columns FILTER_COLUMNS
                        list of columns with binary entries by which to filter
                        the rows (default: None)
  -fb {union,intersection}, --filter_by {union,intersection}
                        Returns filter: intersection where all are 'Yes' or
                        '1'or union where any are 'Yes' or '1' in the defined
                        --filter_columns (default: union)
  -df DATE_FROM, --date_from DATE_FROM
                        The first date for the document cohort in YYYY/MM/DD
                        format (default: None)
  -dt DATE_TO, --date_to DATE_TO
                        The last date for the document cohort in YYYY/MM/DD
                        format (default: None)
  -mn {1,2,3}, --min_ngrams {1,2,3}
                        the minimum ngram value (default: 1)
  -mx {1,2,3}, --max_ngrams {1,2,3}
                        the maximum ngram value (default: 3)
  -mdf MAX_DOCUMENT_FREQUENCY, --max_document_frequency MAX_DOCUMENT_FREQUENCY
                        the maximum document frequency to contribute to TF/IDF
                        (default: 0.05)
  -p {median,max,sum,avg}, --pick {median,max,sum,avg}
                        Everything is computed over non zero values (default:
                        sum)
  -ndl, --normalize_doc_length
                        normalize tf-idf scores by document length (default:
                        False)
  -t, --time            weight terms by time (default: False)
  -o [{graph,wordcloud,report,tfidf,termcounts} [{graph,wordcloud,report,tfidf,termcounts} ...]], --output [{graph,wordcloud,report,tfidf,termcounts} [{graph,wordcloud,report,tfidf,termcounts} ...]]
                        Note that this can be defined multiple times to get
                        more than one output. termcounts represents the term
                        frequency component of tfidf (default: ['report'])
  -on OUTPUTS_NAME, --outputs_name OUTPUTS_NAME
                        outputs filename (default: out)
  -wt WORDCLOUD_TITLE, --wordcloud_title WORDCLOUD_TITLE
                        wordcloud title (default: Popular Terms)
  -nltk NLTK_PATH, --nltk_path NLTK_PATH
                        custom path for NLTK data (default: None)
  -np NUM_NGRAMS_REPORT, --num_ngrams_report NUM_NGRAMS_REPORT
                        number of ngrams to return for report (default: 250)
  -nd NUM_NGRAMS_WORDCLOUD, --num_ngrams_wordcloud NUM_NGRAMS_WORDCLOUD
                        number of ngrams to return for wordcloud (default:
                        250)
  -nf NUM_NGRAMS_FDG, --num_ngrams_fdg NUM_NGRAMS_FDG
                        number of ngrams to return for fdg graph (default:
                        250)
  -cpc CPC_CLASSIFICATION, --cpc_classification CPC_CLASSIFICATION
                        the desired cpc classification (for patents only)
                        (default: None)
```

## Acknowledgements

### Patent data

Patent data was obtained from the [United States Patent and Trademark Office (USPTO)](https://www.uspto.gov) through the [Bulk Data Storage System (BDSS)](https://bulkdata.uspto.gov). In particular we used the `Patent Grant Full Text Data/APS (JAN 1976 - PRESENT)` dataset, using the data from 2004 onwards in XML 4.* format.

### scikit-learn usage

Sections of this code are based on [scikit-learn](https://github.com/scikit-learn/scikit-learn) sources.

### Knockout JavaScript library

The [Knockout](http://knockoutjs.com/) JavaScript library is used with our force-directed graph output.

### WebGenresForceDirectedGraph

The [WebGenresForceDirectedGraph](https://github.com/Aeternia-ua/WebGenresForceDirectedGraph) 
project by Iryna Herasymuk is used to generate the force directed graph output.

### 3rd Party Library Usage

Various 3rd party libraries are used in this project; these are listed
on the [dependencies](https://github.com/datasciencecampus/pygrams/network/dependencies) page, whose contributions we gratefully acknowledge. 

