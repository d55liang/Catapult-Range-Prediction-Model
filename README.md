# Catapult Range Prediction Model

End-to-end pipeline that turns raw catapult experiment data into a validated range predictor.  
It imports measurements into a SQLite database, cleans and de-duplicates rows using SQL, removes outliers with a bin-based IQR method, then trains a **degree-3 polynomial regression** model to predict **Range (m)** from **Launch Angle (deg)**.

**Test-set performance (from the notebook):**  
- **MAE:** 0.935 m  
- **RMSE:** 1.898 m  
- **R²:** 0.961  

---

## What’s in this project

### Pipeline
1. **Ingest** `raw_data.csv` → writes table `Shots` into `catapult.db`
2. **SQL data cleaning**
   - Filter to a single projectile mass (`target_mass = 0.057 kg`)
   - Remove invalid angles (`< 0` or `> 90` or null/empty)
   - Remove invalid ranges (`< 0` or `> 50` or null/empty)
   - Remove duplicates (keeps latest row by `rowid`)
3. **Train/test split**
   - 80% train / 20% test (`random_state = 21`)
   - Saved to `Shots_train` and `Shots_test`
4. **Outlier removal (train only)**
   - Bin angles in **3° bins**
   - Apply **IQR rule** within each bin
   - Only apply IQR if the bin has **≥ 5 samples** (prevents deleting good sparse data)
5. **Model**
   - PolynomialFeatures **degree = 3** + LinearRegression
6. **Evaluation**
   - MAE, RMSE, R² on the held-out test set

---

## Data format

Place a file named `raw_data.csv` in the project root with **these columns**:

- `Mass_kg` (numeric, kg)
- `Angle_deg` (numeric, degrees)
- `Range_m` (numeric, meters)

> The notebook loads everything as strings first, then converts to numeric for modeling.

---

## How to run

### Option A: Run in Jupyter (recommended)
1. Put `raw_data.csv` in the same folder as the notebook.
2. Open and run all cells in:
   - `Catapult_Range_Prediction_Model.ipynb`

### Option B: Run headless (execute notebook)
```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute Catapult_Range_Prediction_Model.ipynb