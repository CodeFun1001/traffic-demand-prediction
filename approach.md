# ***Traffic Demand Prediction – Approach Documentation***





### **1. Problem Understanding**



The objective was to predict normalized traffic demand values for unseen road segments and time intervals using spatial, temporal, environmental and infrastructure-related attributes.



Since traffic demand is influenced by multiple interacting factors such as road type, number of lanes, geographical location, weather conditions and time of day, the solution focused on combining domain-driven feature engineering with ensemble tree-based machine learning models.



### 2\. Data Preprocessing



The first stage involved ensuring data quality and consistency.



##### Missing Value Handling



The following imputations were performed:



* Temperature → Median value
* Weather → Mode value
* RoadType → Mode value



This prevented information loss while preserving overall feature distributions.



###### Dataset Integration



Train and test datasets were temporarily combined during preprocessing to ensure identical transformations and feature generation.



After preprocessing, the datasets were separated again for model training and prediction.



### **3. Feature Engineering**



Feature engineering contributed significantly to model performance.



##### Temporal Features



Timestamp values were converted into:



* Hour
* Minute
* Time Slot (96 intervals per day)



To capture cyclic traffic behaviour:



* hour\_sin
* hour\_cos
* time\_sin
* time\_cos



were generated using sine/cosine transformations.



These encodings preserve the continuity between the last and first time intervals of a day.



#### **Geospatial Features**



Geohash values were decoded into:



* Latitude
* Longitude



To capture locality-level traffic patterns:



* Geohash prefix clustering
* Geo-cluster identifiers



were generated.



Additional cluster-level statistics:



* geo\_mean\_demand
* geo\_std\_demand



were derived to represent average traffic behaviour within spatial regions.



#### **Infrastructure Features**



Road infrastructure strongly influences traffic flow.



The following interaction features were created:



* highway\_lanes
* street\_lanes
* vehicle\_lane\_interaction



These features allow the model to capture relationships between road capacity and vehicle movement restrictions.



#### **Categorical Encoding**



Categorical variables were converted using:



* One-Hot Encoding for Weather
* One-Hot Encoding for RoadType



Binary variables were converted to numerical representations.



### **4. Exploratory Data Analysis**



Correlation analysis and visual exploration revealed several important insights.



##### Key Findings



RoadType emerged as the strongest predictor of demand.



Observed correlations:



* RoadType\_Highway ≈ +0.80
* RoadType\_Residential ≈ –0.78



Highway segments consistently exhibited significantly higher traffic demand compared to residential roads.



Average predicted demand by road type:



|***Road Type***|***Average Demand***|
|-|-|
|Highway|0.5765|
|Street|0.2725|
|Residential|0.0629|



Temporal patterns were also present.



Demand varied across time slots and showed recurring daily behaviour. However, the magnitude of variation across road types was substantially larger than the variation across time intervals, explaining why road infrastructure features received higher model importance scores.



Weather-related variables exhibited relatively weak direct correlations with demand.



### **5. Model Development**



Multiple machine learning models were evaluated.



##### **Baseline Models:**

* Random Forest Regressor
* XGBoost Regressor
* LightGBM Regressor
* CatBoost Regressor
* Validation Strategy



A 5-Fold Cross Validation framework was used.



**Evaluation Metric** : R² Score



This provided a robust estimate of generalization performance while reducing dependence on a single train-validation split.



### **6. Hyperparameter Optimization**



Optuna was used to optimize XGBoost hyperparameters.



The search space included:



* n\_estimators
* max\_depth
* learning\_rate
* subsample
* colsample\_bytree
* min\_child\_weight
* gamma
* reg\_alpha
* reg\_lambda



A total of 60 optimization trials were executed.



### **7. Final Model**



The best-performing model was XGBoost.



##### **Performance :** 



|***Model***|***CV R²***|
|-|-|
|Random Forest|0.8906|
|LightGBM|0.9157|
|CatBoost|0.8919|
|XGBoost|0.9220|
|Tuned XGBoost|0.9322|



Final predictions were generated using:



1. 65% Tuned XGBoost
2. 35% LightGBM



ensemble averaging.



Predictions were clipped to the valid target range : **\[0,1]**



before submission generation.



### **8. Data Drift Analysis**



A Kolmogorov-Smirnov (KS) based drift analysis was performed between training and test datasets.



**Findings:**



* Weather distributions remained stable.
* Temperature distributions remained stable.
* Time-related variables exhibited expected shifts.
* Road-type distributions showed moderate variation.



These shifts were consistent with natural differences between observation windows and did not indicate data quality issues.



### **9. Tools and Libraries**



* Python
* Pandas
* NumPy
* Scikit-Learn
* XGBoost
* LightGBM
* CatBoost
* Optuna
* SciPy
* Matplotlib
* Seaborn

PyGeoHash

