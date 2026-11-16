# Alzheimer's vs healthy EEG

Classifying people as Alzheimer's or healthy from their resting EEG, using brain-wave power features.

## Data

The data is from OpenNeuro, dataset ds004504 (Miltiadous et al., 2023). It has resting EEG recorded with 19 electrodes while people sat with their eyes closed. There are 88 people in three groups:

- 36 with Alzheimer's
- 23 with frontotemporal dementia
- 29 healthy

I used only the Alzheimer's and healthy groups, so 65 people in total. The frontotemporal group is not used. The average age is similar in both groups (66 and 68).

## The idea

Alzheimer's is linked to slower brain waves. So for each person I measure how strong each type of wave is and give those numbers to a model. Each person becomes 23 numbers:

- how much delta, theta, alpha and beta waves there are, in 5 brain areas (20 numbers)
- two slowing ratios: theta / alpha and (delta + theta) / (alpha + beta)
- the speed of the main alpha wave at the back of the head

## How I tested it

Each person is one row, so every test is on people the model has not seen. I used 5-fold cross-validation, repeated 10 times.

## Results

| Model | Accuracy |
|---|---|
| Logistic regression | 79.5% |
| SVM | 78.2% |
| Random forest | 73.4% |

Confusion matrix for logistic regression:

![confusion matrix](alz_confusion_matrix.png)

29 of 36 Alzheimer's people and 22 of 29 healthy people were found correctly.

## Libraries

- MNE: reads the EEG files and filters the signal
- SciPy: `welch` gives the power at each frequency
- NumPy and pandas: arrays and tables
- scikit-learn: the models and the testing
- Matplotlib: the plots
- openneuro-py: downloads the data


## Future scope

The model could be made more reliable by:

- adding connectivity features, which measure how much different parts of the brain move together
- bringing in the frontotemporal dementia group for a three-way classification
- testing on a second dataset from a different lab

## Run it

Open the notebook in Google Colab and run the cells from the top. It downloads the data by itself (about 2 GB).
