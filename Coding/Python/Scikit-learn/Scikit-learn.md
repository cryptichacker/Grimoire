---
tags: [coding, python, ml, scikit-learn]
type: cheatsheet
source: compiled reference (scikit-learn)
last-verified: 2026-08-27
---

# Scikit-learn

## Up
- [[Python]]

scikit-learn is the standard classical-ML library for Python — a consistent `fit`/`predict`/`transform` API over preprocessing, models, model selection, and metrics. Built on [[NumPy]]/[[Pandas]]. Install: `pip install scikit-learn`. Import as `sklearn`.

---

## The Estimator API (one pattern for everything)

```python
model.fit(X_train, y_train)      # learn from data
model.predict(X_test)            # predict labels/values
model.predict_proba(X_test)      # class probabilities (classifiers)
model.score(X_test, y_test)      # default metric (accuracy / R²)

transformer.fit(X); transformer.transform(X)   # preprocessing
transformer.fit_transform(X)                    # both in one call
```

`X` is a 2-D array/DataFrame `(n_samples, n_features)`; `y` is a 1-D target.

---

## Train/Test Split & Cross-Validation

```python
from sklearn.model_selection import train_test_split, cross_val_score, KFold

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y)

scores = cross_val_score(model, X, y, cv=5, scoring="accuracy")
print(scores.mean(), scores.std())

from sklearn.model_selection import StratifiedKFold, cross_validate
cross_validate(model, X, y, cv=5, scoring=["accuracy", "f1_macro"])
```

---

## Preprocessing

```python
from sklearn.preprocessing import (StandardScaler, MinMaxScaler,
    OneHotEncoder, LabelEncoder, OrdinalEncoder)
from sklearn.impute import SimpleImputer

StandardScaler().fit_transform(X)        # zero mean, unit variance
MinMaxScaler().fit_transform(X)          # scale to [0, 1]
OneHotEncoder(handle_unknown="ignore").fit_transform(X_cat)
SimpleImputer(strategy="mean").fit_transform(X)   # fill missing

# fit on train, apply to test (avoid leakage!)
scaler = StandardScaler().fit(X_train)
X_train_s = scaler.transform(X_train)
X_test_s  = scaler.transform(X_test)     # never fit on test data
```

---

## Common Models

```python
# Classification
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.tree import DecisionTreeClassifier
from sklearn.naive_bayes import GaussianNB

clf = RandomForestClassifier(n_estimators=200, max_depth=None, random_state=42)
clf.fit(X_train, y_train)
clf.feature_importances_

# Regression
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.ensemble import RandomForestRegressor

reg = Ridge(alpha=1.0).fit(X_train, y_train)

# Clustering (unsupervised)
from sklearn.cluster import KMeans, DBSCAN
km = KMeans(n_clusters=3, n_init="auto", random_state=42).fit(X)
km.labels_; km.cluster_centers_

# Dimensionality reduction
from sklearn.decomposition import PCA
X_2d = PCA(n_components=2).fit_transform(X)
```

---

## Pipelines & ColumnTransformer (the right way)

```python
from sklearn.pipeline import Pipeline, make_pipeline
from sklearn.compose import ColumnTransformer

num = ["age", "income"]; cat = ["city", "plan"]

pre = ColumnTransformer([
    ("num", StandardScaler(), num),
    ("cat", OneHotEncoder(handle_unknown="ignore"), cat),
])

pipe = Pipeline([
    ("prep", pre),
    ("model", LogisticRegression(max_iter=1000)),
])

pipe.fit(X_train, y_train)      # preprocessing + model as one object
pipe.predict(X_test)
```

Pipelines prevent data leakage (the scaler is re-fit on each CV fold) and make deployment a single `joblib.dump(pipe, ...)`.

---

## Hyperparameter Tuning

```python
from sklearn.model_selection import GridSearchCV, RandomizedSearchCV

grid = {"model__C": [0.1, 1, 10], "model__penalty": ["l2"]}
search = GridSearchCV(pipe, grid, cv=5, scoring="f1_macro", n_jobs=-1)
search.fit(X_train, y_train)
search.best_params_; search.best_score_; search.best_estimator_

# note the "model__C" prefix: <step_name>__<param> inside a pipeline
```

---

## Evaluation Metrics

```python
from sklearn.metrics import (accuracy_score, precision_score, recall_score,
    f1_score, confusion_matrix, classification_report, roc_auc_score,
    mean_squared_error, mean_absolute_error, r2_score)

# classification
y_pred = clf.predict(X_test)
accuracy_score(y_test, y_pred)
print(classification_report(y_test, y_pred))
confusion_matrix(y_test, y_pred)
roc_auc_score(y_test, clf.predict_proba(X_test)[:, 1])

# regression
mean_squared_error(y_test, y_pred, squared=False)   # RMSE
r2_score(y_test, y_pred)
```

| Task | Go-to metrics |
|---|---|
| Balanced classification | accuracy, F1 |
| Imbalanced classification | precision/recall, F1, ROC-AUC, PR-AUC |
| Regression | RMSE, MAE, R² |
| Clustering | silhouette_score, adjusted_rand_score |

---

## Persistence

```python
import joblib
joblib.dump(pipe, "model.joblib")
pipe = joblib.load("model.joblib")
```

---

## Typical Workflow

```python
# 1. split → 2. build pipeline (preprocess + model) → 3. cross-validate
# 4. GridSearchCV to tune → 5. evaluate best on held-out test → 6. persist
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=.2, stratify=y, random_state=42)
search = GridSearchCV(pipe, grid, cv=5, n_jobs=-1).fit(X_tr, y_tr)
print(classification_report(y_te, search.predict(X_te)))
joblib.dump(search.best_estimator_, "model.joblib")
```

---

## Tips

- **Never fit any transformer on test data** — fit on train, transform both. Pipelines enforce this automatically.
- Set `random_state` everywhere for reproducibility.
- `n_jobs=-1` uses all CPU cores for parallelisable estimators/searches.
- Scale features for distance/gradient models (SVM, KNN, linear, neural); tree ensembles don't need scaling.
- For big data / GPUs or gradient boosting at scale, look at XGBoost, LightGBM, or CatBoost (sklearn-compatible APIs).
