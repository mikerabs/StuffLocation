# ⚾ Stuff+ Modeling Pipeline (xRunValue-Based)

## 1. Data Acquisition
- Load pitch-level data (Statcast/Hawkeye)
- Features include:
  - `RelSpeed`, `SpinRate`, `InducedVertBreak`, `ABS_Horizontal`
  - `RelHeight`, `ABS_RelSide`, `Extension`, `Tilt`, etc.

## 2. Pitch Type Filtering and Labeling
- Split data into separate DataFrames for each pitch type:
  - `dfb2`, `dsi2`, `dsl2`, `dst2`, `dcb2`, `dch2`, `dct2`, `dsp2` (train)
  - `dfb3`, `dsi3`, `dsl3`, etc. (evaluation)

## 3. Feature Selection (Stuff-only)
- For each pitch type, use only movement-based features such as:
  - `RelSpeed`, `SpinRate`, `InducedVertBreak`, `ABS_Horizontal`
  - `RelHeight`, `ABS_RelSide`, `Extension`

## 4. Label Selection
- Use **xRunValue (expected run value)** as regression target
- Continuous value learned from pitch outcomes

## 5. Model Training (Per Pitch Type)
### A. XGBoost
- Train an `XGBRegressor` per pitch type
- Use standard hyperparameters (depth, subsample, learning rate, etc.)
- Predict `xRV_xgb` for training and evaluation sets

### B. Neural Network (Keras)
- Build a residual-feedforward NN with:
  - Dense layers, BatchNorm, ReLU, Dropout, Residual connections
- Use `StandardScaler` for input normalization
- Compile with:
  - Loss = `'mse'`
  - Metric = custom `r_squared`
- Train per pitch type with `EarlyStopping`
- Predict `xRV_nn` for training and evaluation sets

## 6. xRV Prediction Storage
- Add predictions (`xRV_xgb`, `xRV_nn`) as columns in the DataFrame

## 7. Stuff+ Calculation (Scaling Step)
- For each pitch:
  - `xRV_Scaled = xRV - max(xRV)`
  - `abs_scaled = abs(xRV_Scaled)`
  - `Stuff+ = (abs_scaled / abs_scaled.mean()) * 100`
- Done separately for:
  - XGBoost model → `Stuff_plus`
  - Neural network → `Stuff_plus_NN`

## 8. Model Evaluation
- Compute and track:
  - `R²` between predicted xRV and actual run value
  - `RMSE` and comparison to `std(y_test)`
- Compare performance across pitch types and models

## 9. Deployment and Interpretation
- Use `Stuff_plus` as standardized score of raw pitch quality
- High Stuff+ → High modeled effectiveness
- Supports:
  - Scouting
  - Pitch development
  - Team analytics
