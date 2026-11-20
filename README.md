# Alzheimer's vs healthy EEG
Alzheimer's recordings show slower brain waves, strongest at the back of the head. A logistic regression on 33 EEG features reached about 80% accuracy (AUC 0.86) with person-wise cross-validation on 65 people.

## Data

I used the public OpenNeuro dataset **ds004504**: resting-state, eyes-closed EEG recorded with 19 scalp electrodes. I used the cleaned (preprocessed) files from the dataset's `derivatives` folder, and I kept two groups:

- 36 people with Alzheimer's
- 29 healthy people

## What I did

1. Cut every recording into 20-second pieces.
2. Computed 33 features for each piece (listed below).
3. Averaged the pieces so each person has one row of features.
4. Compared the two groups feature by feature, with a Mann-Whitney test and a correction for testing many features at once (Benjamini-Hochberg).
5. Trained models and tested them with 5-fold cross-validation repeated 10 times. Each person is one row, so nobody is in both the training and test set.

**The 33 features**

| Group | How many | What it is |
|---|---|---|
| Relative band power | 20 | Share of total power in delta, theta, alpha and beta, for 5 brain regions (frontal, central, temporal, parietal, occipital) |
| Slowing ratios | 2 | Theta/alpha ratio, and (delta+theta)/(alpha+beta). Higher means slower brain waves |
| Alpha peak frequency | 1 | The speed of the strongest alpha wave at the back of the head |
| Alpha coherence | 10 | How in sync alpha waves are between each pair of regions |

## Results

### The brain waves are slower in Alzheimer's

23 of the 33 features differed between the groups after correction. The age of the two groups was similar (66.4 vs 67.9 years, p = 0.38), so this is not just an age effect.

| | Healthy (median) | Alzheimer's (median) |
|---|---|---|
| Occipital alpha (share of power) | 8.3% | 2.5% |
| Alpha peak frequency | 8.9 Hz | 6.7 Hz |
| Theta/alpha ratio | 1.65 | 2.93 |

![Average spectrum]("average_spectrum.png")

### Where it happens

The effect is strongest at the back of the head: much less alpha (Cohen's d = -1.27) and more delta (d = +1.06) in the occipital area. The front mainly shows extra theta (d = +0.76).

![Effect size by region]("C:\Users\HP\Pictures\Screenshots\effect_on_size.png")

### Can a model tell them apart?

| Model | Accuracy | AUC |
|---|---|---|
| Always guess the bigger group | 55.4% | 0.50 |
| Age only | 50.8% | not computed |
| Logistic regression | 79.5% (± 10.9) | 0.86 |
| SVM | 75.2% (± 13.1) | 0.85 |
| Random forest | 73.1% (± 11.6) | 0.84 |

The three models score about the same, and the differences are smaller than the spread between splits, so I don't claim one is better. The age-only model is at chance level, which tells me the models are not just learning age.

In one 5-fold run with logistic regression, the model found 31 of 36 Alzheimer's people and correctly cleared 21 of 29 healthy people.

![Confusion matrix]("C:\Users\HP\Pictures\Screenshots\confusion_matrix.png")

### What the model relied on

I averaged the logistic regression coefficients over the cross-validation folds. Occipital alpha, occipital delta, the alpha peak, theta power and parieto-occipital coherence came out near the top. Many features are correlated with each other (relative powers add up to 100%), so I read this as "these features work together" and not as a ranking of single biomarkers.

![Top features]("C:\Users\HP\Pictures\Screenshots\top_features.png")


## Limitations

- Parieto-occipital alpha coherence was *higher* in Alzheimer's, which I did not expect. I don't know the reason yet. It could be the slowed rhythm falling inside the 8-13 Hz window, or signal spreading through the skull. I did not test this.
- I computed coherence between region averages, which is sensitive to signal spreading (volume conduction).
- The same features could not separate Alzheimer's from frontotemporal dementia in a quick test, so I left that out of this project.

## Python libraries
numpy, pandas, scipy, scikit-learn, matplotlib, mne, openneuro-py.
