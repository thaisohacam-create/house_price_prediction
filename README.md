Vietnamese Real Estate Price Prediction
Personal Machine Learning Project
This notebook builds a regression model to estimate Vietnamese real estate listing prices from listing attributes. It covers data inspection, cleaning, exploratory analysis, province-aware outlier handling, feature engineering, preprocessing, model training, and evaluation.
Detailed explanation of the workflow
1. Dataset inspection
The supplied CSV contains listing IDs, addresses, text-formatted prices, text-formatted areas, and room counts. The first inspection checks data types, missing values, and basic distributions before any transformations are applied.
2. Duplicate handling
Exact duplicate rows are removed first. Repeated product_id values are then reduced to one row, keeping the first occurrence. This prevents repeated listings from receiving disproportionate influence and reduces the risk of the same listing appearing in both training and test sets.
3. Price and area conversion
Price strings are converted to VND. The units tỷ/tỉ and triệu are converted to numeric multipliers, and comma decimal separators are normalized.
Rows labelled Giá thỏa thuận(negotiable price) are excluded instead of assigning the mean or median. These entries do not provide an observed sale/listing price. Imputing a target value would create synthetic labels and could teach the model an invented price. The trade-off is that the model cannot learn from those listings.
Area strings are converted to square metres. Rows with missing or invalid target prices, non-positive areas, or implausible room counts are removed.
4. Property type proxy
Listings with both bedrooms_num and bathrooms_num equal to zero are assigned the category Non-residential / unspecified. This follows the project's proposed rule, but it is only a proxy: some residential listings may have missing or unreported room counts. A dedicated property-type field would be more reliable.
5. Exploratory data analysis
Price and area distributions are strongly right-skewed, so logarithmic axes are used for visualization. The scatter plot helps inspect the broad relationship between area and price. These plots are descriptive; they do not establish causation.
6. Train/test split and province-aware outliers
The data is split before outlier detection. Outliers are identified only in the training set using log price per square metre and the 1.5 × IQR rule within each province/city.
A province with fewer than 30 training records uses the overall training-set quartiles as a fallback because its own quartile estimates may be unstable. Detected outliers are removed from training, while the test set remains unchanged. This avoids using test-set information to decide which training observations to retain.
Important limitation: The IQR rule identifies statistically unusual listings, not necessarily erroneous listings. Luxury properties and unusual land listings may be legitimate. Keeping the test set unchanged also means its extreme listings can make original-scale error metrics large.
7. Feature engineering and preprocessing
Area is transformed using log1p. Numeric features are expanded with degree-2 polynomial terms to allow simple nonlinear relationships, then standardized. Categorical variables (property_type, province, and local_area) are one-hot encoded.
The preprocessing transformer is fitted only on the training set and then applied to the test set. This prevents information from the test set from influencing imputation, scaling, or category learning. Unknown categories in the test set are safely ignored by the encoder.
8. Model and target transformation
A Linear Regression model is trained to predict log1p(price in billion VND). Predicting log price reduces the influence of very expensive listings during fitting. Predictions are converted back to billion VND for interpretation.
9. Evaluation
R² on log price measures fit in the transformed target space.
R² on original price measures fit after converting predictions back to the original price scale.
MAE is the average absolute prediction error, reported in billion VND.
RMSE penalizes large errors more strongly than MAE.
Median absolute error shows the typical middle absolute error and is less sensitive to extreme errors than the mean.
The two R² values answer different questions and should not be interchanged. A good-looking log-scale R² does not guarantee accurate predictions for very expensive listings.
