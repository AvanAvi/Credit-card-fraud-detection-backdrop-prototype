# Credit-card-fraud-detection-backdrop-prototype

"""
Welcome to the world of Credit Card Fraud Detection.
This is a magical place where frauds get caught red-handed and innocents stay untouched. 
"""

# Importing necessary libraries (aka tools needed for the magical work)
import sys
import numpy
import pandas
import matplotlib
import seaborn
import sklearn
import scipy

# Printing the versions of our magical tools to keep everything in check
print('Python :{}'.format(sys.version))
print('Numpy :{}'.format(numpy.__version__))
print('Pandas :{}'.format(pandas.__version__))
print('Matplotlib :{}'.format(matplotlib.__version__))
print('Seaborn :{}'.format(seaborn.__version__))
print('Scipy :{}'.format(scipy.__version__))
print('Sklearn :{}'.format(sklearn.__version__))

# Setting up the magic spell (importing more specific libraries)
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Importing our crystal ball (the dataset)
data = pd.read_csv('/content/creditcard.csv')

# Checking out the elements of the crystal ball 
print(data.columns)     # prints all the column names in the dataset
print(data.shape)       # prints the structure of the dataset i.e columns X rows
print(data.describe())  # prints the schema of the dataset

# More checking out. Repetition leads to perfection, they say!
print(data.columns)
print(data.shape)
print(data.describe())

# We're going on a trip in our favourite rocket ship, zooming through the data. Here, we sample 50% of the data for analysis. 
data = data.sample(frac = 0.5, random_state =1)
print(data.shape)

# Time to get a bird's eye view of our data. Histograms show us the frequency distribution of our data.
data.hist(figsize = (20,20))
plt.show()

# Time to separate the good guys (Valid transactions) and the bad guys (Fraud transactions) 
Fraud = data[data['Class']== 1]
Valid = data[data['Class']== 0]

# Just some ratio magic to see the proportion of bad guys in the total population. Spoiler alert: it's quite low!
outlier_fraction = len(Fraud) / float(len(Valid))
print(outlier_fraction)

# Remember the saying, "Show me your friends and I'll tell you who you are"? Let's see who hangs out with whom.
print('Fraud cases :{}'.format(len(Fraud)))
print('Valid cases :{}'.format(len(Valid)))

# Alright, enough of the fun and games. Time for some serious correlation matrix stuff. Let's see who's related to whom and how much. 
cor_mat = data.corr()
fig = plt.figure(figsize = (12,9))
sns.heatmap(cor_mat, vmax = .8, square = True)

# Getting all the columns from the dataframe, it's like a roll-call time in school.
columns = data.columns.tolist()

# Filter the columns to remove data we don't want. Not everyone gets to play, sorry.
columns = [c for c in columns if c not in ["Class"]]

# The prediction variable, the one we're going to predict. Yes, very imaginative name.
target = "Class"

# Defining our X's and Y's. X is our features and Y is our label.
X = data[columns]
Y = data[target]

print(X.shape)
print(Y.shape)

# Remember the childhood game of "Spot the difference"? Well, this is it, for adults. We're going to use these models to spot the outliers i.e. fraud cases in our data.
from sklearn.metrics import classification_report,accuracy_score
from sklearn.ensemble import IsolationForest
from sklearn.neighbors import LocalOutlierFactor

# Define a random state (aka seed for our random number generator) for reproducibility 
state = 1

# Outlier detection methods, aka the game tools for our "Spot the difference" game.
classifiers = {
    "Isolation Forest": IsolationForest(max_samples=len(X),
                                      contamination = outlier_fraction,
                                       random_state= state),
     "Local Outlier Factor": LocalOutlierFactor(
     n_neighbors= 20,
     contamination = outlier_fraction)
}

# Using imputation magic to replace any NaN values. No NaNs allowed in this game!
from sklearn.impute import SimpleImputer
imputer = SimpleImputer(strategy='mean')
imputer = imputer.fit(X)
X = imputer.transform(X)

# Convert the data back into a pandas dataframe, because pandas are cute and also because we need to use the dropna() function. 
X_df = pd.DataFrame(X)
X_df = X_df.dropna()
X = X_df.to_numpy()

# No NaNs allowed! Begone NaNs from the data. We then define our X's and Y's again. Yes, we love our X's and Y's, don't we?
data = data.dropna()
X = data.drop('Class', axis=1)
Y = data['Class']

# Convert the X's and Y's to values. Just some pandas magic.
X = X.values
Y = Y.values

# Running the models. Finally! Let's spot those differences, shall we?
n_outliers = len(Fraud)
for i, (clf_name, clf) in enumerate(classifiers.items()):
    # Fit the data and tag outliers
    if clf_name == "Local Outlier Factor":
        y_pred = clf.fit_predict(X)
        scores_pred = clf.negative_outlier_factor_
    else:
        clf.fit(X)
        scores_pred = clf.decision_function(X)
        y_pred = clf.predict(X)

    # Reshape the prediction values to 0 for valid and 1 for fraud
    y_pred[y_pred == 1] = 0
    y_pred[y_pred == -1] = 1

    # Count the errors if any. Let's see how well our game went!
    n_errors = (y_pred != Y).sum()

    # Print the name of the model and the number of errors it made during the game.
    print('{}: {}'.format(clf_name,n_errors))
    
    # Print the accuracy score. The higher, the better.
    print(accuracy_score(Y,y_pred))
    
    # Print the classification report. This is like the game report. It shows the Precision, Recall, F1-score and Support for both the classes. 
    print(classification_report(Y,y_pred))

"""
There you go! You've reached the end of this exciting credit card fraud detection game. Hope you enjoyed the game as much as I did.
Remember, "With great data, comes great responsibility". So, use your powers wisely! ;) 
"""
