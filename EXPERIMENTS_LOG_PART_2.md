# Addendum 2021-11-22T14:15:59

The CLARIN repo was pulled anew to get the non-suitable dataset. Dataset construction will begin by reading all of the data, regardless of the duplicate flag.

I shall open a new notebook for this, namely 25_suitable_dataset_construction.

I note there are 1002 suitable documents and only 123 nonsuitable. Data has been saved as `/home/peterr/macocu/task5_webgenres/data/interim/suitable_tabular.csv`.



# Addendum 2021-11-23T10:10:26

The experiment is finished. 15 runs have been run on full datasets. If all the results are reshaped into a single matrix, the following figure is obtained:


<img src="images/25_cumulative_CM.png" alt="plot" width="300"/>

# Addendum 2021-11-25T12:44:54

The analysis below is wrong. Find out more in 

If f1_score is calculated across all runs and then averaged, we get **f1 = 0.954 +/- 0.0029**


### Comparison with dummy classifier

Due to enormous class imbalance we really should include a dummy classifer for comparison.

```
Dummy clf: strategy: 'stratified', 0.887 +/- 0.0105
Dummy clf: strategy: 'most_frequent', 0.942 +/- 0.0
```

### Are we better than dummy?

As before a MannWhitney U-test was used to estimate p-value. We get:
```
Dummy clf: strategy: 'stratified', 0.894 +/- 0.00945
	 p_value: 1.69e-06
Dummy clf: strategy: 'most_frequent', 0.942 +/- 0.0
	 p_value: 3.41e-07
```
Note: `stratified` strategy is stochastic, so the results are different from the results above, but they are within 1-sigma from eachother.


## Verdict

Our classifier is statistically significantly better than guessing, but unfortunately it is __only statistically significantly better than guessing__.

## Further possible actionables

* Proper (wandb) or partial (only #epochs) hyperparameter optimization?
* Repeating the training on dedup data? Repeat full grid searching? (i.e. {train_dedup, train_full} × {test_dedup, test_full})

# Addendum 2021-11-23T12:48:55

Wrong predictions of all 15 runs have been analyzed by the following protocol:
* For every run, we look at what instances have been misclassified.
* We log their primary label and the true and predicted value, although we only need the primary label, as the sets of suitable primary labels and unsuitable primary labels are disjoint.
* The labels are counted. This produces the following table:

| primary                    | misclassified_counts |
| :------------------------- | -------------------: |
| Non-textual                |                  125 |
| Not Slovene                |                  116 |
| Machine Translation        |                   76 |
| Too Short/Incoherent       |                   63 |
| Too Long                   |                   51 |
| HTML Source Code           |                   30 |
| Forum                      |                   23 |
| Promotion of a Product     |                   17 |
| List of Summaries/Excerpts |                   16 |
| Generated Text             |                   15 |
| Instruction                |                   14 |
| Information/Explanation    |                    6 |
| News/Reporting             |                    4 |
| Announcement               |                    4 |
| Legal/Regulation           |                    2 |
| Opinion/Argumentation      |                    2 |
| Encoding Issues            |                    1 |

* This is not very informative as we do not know how frequent these labels are in the test split, so further analysis is needed.
* For all of the test split we count the labels appearing in the first column of the table above. This number is then multiplied by 15 (because we had 15 runs and therefore have 15 times more misclassified instances, which should be accounted for.)
* Numbers of misclassified counts are then divided by the number of counts in the test split. The final result looks like this:

| primary                    | misclassified_counts | counts_in_test_split | misclassification_ratio |
| :------------------------- | -------------------: | -------------------: | ----------------------: |
| HTML Source Code           |                   30 |                   30 |                       1 |
| Generated Text             |                   15 |                   15 |                       1 |
| Too Long                   |                   51 |                   60 |                    0.85 |
| Not Slovene                |                  116 |                  165 |                 0.70303 |
| Too Short/Incoherent       |                   63 |                   90 |                     0.7 |
| Non-textual                |                  125 |                  180 |                0.694444 |
| Machine Translation        |                   76 |                  135 |                0.562963 |
| Instruction                |                   14 |                  150 |               0.0933333 |
| Forum                      |                   23 |                  375 |               0.0613333 |
| Announcement               |                    4 |                  120 |               0.0333333 |
| Promotion of a Product     |                   17 |                  750 |               0.0226667 |
| List of Summaries/Excerpts |                   16 |                  840 |               0.0190476 |
| Encoding Issues            |                    1 |                   60 |               0.0166667 |
| Legal/Regulation           |                    2 |                  135 |               0.0148148 |
| Information/Explanation    |                    6 |                  690 |              0.00869565 |
| News/Reporting             |                    4 |                  630 |              0.00634921 |
| Opinion/Argumentation      |                    2 |                  660 |               0.0030303 |

The type of error can be inferred from the label. E.g., for label `Forum` the original instance was labeled as suitable and since it ended up in our table, the classifier misclassified it as nonsuitable.

### Inverted question: which primaries do not get misclassified?

Again a table was constructed:

| primary                    | not_misclassified_counts | counts_in_test_split | not_misclassification_ratio |
| :------------------------- | -----------------------: | -------------------: | --------------------------: |
| Correspondence             |                       90 |                   90 |                           1 |
| Promotion of Services      |                      195 |                  195 |                           1 |
| Recipe                     |                       60 |                   60 |                           1 |
| Research Article           |                       75 |                   75 |                           1 |
| Call                       |                       30 |                   30 |                           1 |
| Interview                  |                       30 |                   30 |                           1 |
| Lyrical                    |                       15 |                   15 |                           1 |
| Invitation                 |                      165 |                  165 |                           1 |
| Other                      |                      195 |                  195 |                           1 |
| Promotion                  |                      210 |                  210 |                           1 |
| FAQ                        |                       15 |                   15 |                           1 |
| Opinionated News           |                      525 |                  525 |                           1 |
| Review                     |                       60 |                   60 |                           1 |
| Opinion/Argumentation      |                      658 |                  660 |                     0.99697 |
| News/Reporting             |                      626 |                  630 |                    0.993651 |
| Information/Explanation    |                      684 |                  690 |                    0.991304 |
| Legal/Regulation           |                      133 |                  135 |                    0.985185 |
| Encoding Issues            |                       59 |                   60 |                    0.983333 |
| List of Summaries/Excerpts |                      824 |                  840 |                    0.980952 |
| Promotion of a Product     |                      733 |                  750 |                    0.977333 |
| Announcement               |                      116 |                  120 |                    0.966667 |
| Forum                      |                      352 |                  375 |                    0.938667 |
| Instruction                |                      136 |                  150 |                    0.906667 |
| Machine Translation        |                       59 |                  135 |                    0.437037 |
| Non-textual                |                       55 |                  180 |                    0.305556 |
| Too Short/Incoherent       |                       27 |                   90 |                         0.3 |
| Not Slovene                |                       49 |                  165 |                     0.29697 |
| Too Long                   |                        9 |                   60 |                        0.15 |

## Summary:

The nonsuitable primary labels from least successfully classified to most successfully classified: 

| primary              | misclassified_counts | counts_in_test_split | misclassification_ratio |
| :------------------- | -------------------: | -------------------: | ----------------------: |
| HTML Source Code     |                   30 |                   30 |                       1 |
| Generated Text       |                   15 |                   15 |                       1 |
| Too Long             |                   51 |                   60 |                    0.85 |
| Not Slovene          |                  116 |                  165 |                 0.70303 |
| Too Short/Incoherent |                   63 |                   90 |                     0.7 |
| Non-textual          |                  125 |                  180 |                0.694444 |
| Machine Translation  |                   76 |                  135 |                0.562963 |
| Encoding Issues      |                    1 |                   60 |               0.0166667 |

# Addendum 2021-11-24T12:32:46: Possible improvements to the paper

Stylistic unification: now we have `dataset`, but also `label set`?

To do:
* With Taja work out the dataset structure:
* * One DS for unsuitable, one DS for suitable (original labels, 25 labels, with my latest splits.)
* After the dataset is published somewhere, put together a repo with demo code for training.



Ideas for further experimentation: *on hold for now*
* Randomly initialize Sloberta and XLM Roberta and then fine-tune to show that pretraining is not the driving factor behind the transformers behaviour
* Take a checkpoint trained on another language and fine tune it






# Taja and Peter's brainstorming on dataset publication

* Originally formatted paragraphs.
* CLARIN repo prerequisites: json, utf-8, zazipano
* Add token count, sentence count? Ask Nikola.
* Briefly check for missing fields, empty paragraph lists... (Don't forget to check )

Fields:
segmentation: suitable: `train:dev:test` as is, unsuitable: assign new split by primary_level_2 only
instead of `primary`, `secondary`, `tertiary`: `primary_level_1`, `primary_level_2`,... `secondary_level_1`, `secondary_level_2`... `tertiary_level_1`,...
Add field `domain`, but leave URL in as well
`split`: which split does the instance belong to.



# Addendum 2021-11-29T08:53:54

Taja identified some secondary and tertiary labels that were input incorrectly.

Iffy secondary labels:
* Promotion of **s**ervices -> ... Services
* Opinionated **n**ews -> ... News
* Research **a**rticle -> ... Article
* Promotion of a **p**roduct -> ... Product


A corrective measure has since been implemented in `26_dataset_creation.ipynb` and the dataset was constructed anew.

A new repo has been initiated for demo purposes. It loads the dataset from disc (to be improved upon after publication) and then trains a `simpletransformers` model. A brief evaluation part was added to illustrate the performance being approximately equat to what we describe in the paper. In the demo repo number of epochs was raised from 30 to 90 to account for loss of data after changing format (in fasttext format every instance appeared in 3 copies.)


# Addendum 2021-11-30T11:03:29

For publication to HuggingFace dataset hub some decisions will have to be made.

* top level file structure will have to be `test.json`, `dev.json`, and `train.json`.
* we have to decide if we want to publish also the nonsuitable documents.
* what metadata to keep? minimal setup: `label` and `text`, e.g. primary_level_2 and joined paragraphs.
* I feel there isn't much use in keeping the `paragraphs` structure as is. I suggest we deduplicate paragraphs and join them with the standard `<p/>` tag.


# Addendum 2021-12-01T13:37:29

CLARIN repo has README.txt, although it is actually a .md file. Can this be changed?