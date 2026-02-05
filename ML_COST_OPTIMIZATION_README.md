# Machine Learning Cost Optimization System 🤖💰

## Overview

This system uses **Linear Regression** to predict optimal delivery costs (`cost_per_section`) based on multiple factors, enabling dynamic pricing that adapts to real-time conditions.

---

## Features

### 1. **Dynamic Cost Prediction**
Automatically predicts optimal cost per section based on:
- **Distance** (km) - Longer distances = higher costs
- **Urgency** (deadline in hours) - Urgent deliveries = premium pricing
- **Demand** (active routes) - High demand = surge pricing
- **Time of Day** (peak hours) - Rush hour premium
- **Day of Week** (weekends) - Weekend premium

### 2. **Machine Learning Model**
- **Algorithm**: Linear Regression with Gradient Descent
- **Features**: 5 input features (distance, urgency, demand, peak hour, weekend)
- **Normalization**: Z-score normalization for better convergence
- **Training**: Uses historical transaction data
- **Validation**: 80/20 train-test split with performance metrics

### 3. **Performance Metrics**
- **MAE** (Mean Absolute Error) - Average prediction error in dollars
- **RMSE** (Root Mean Squared Error) - Penalizes large errors
- **R² Score** - How well the model fits the data (0-1, higher is better)

---

## File Structure

```
server.js/
├── utils/
│   └── costPrediction.js          # ML model implementation
├── scripts/
│   ├── create_ml_models_table.js  # Database setup
│   └── train_cost_model.js        # Training script
└── server.js                      # Integrated into main server
```

---

## Setup Instructions

### Step 1: Create Database Table

Run the database setup script to create the ML models table:

```bash
node server.js/scripts/create_ml_models_table.js
```

**What it does:**
- Creates `ml_models` table to store trained model coefficients
- Creates `model_training_history` table to track training sessions
- Initializes default cost predictor model with baseline coefficients

**Expected Output:**
```
✅ ml_models table created successfully!
✅ model_training_history table created successfully!
✅ Default cost predictor model initialized!
```

---

### Step 2: Train the Model (Initial Training)

**IMPORTANT:** You need at least **10 completed transactions** with cost data before training.

Run the training script:

```bash
node server.js/scripts/train_cost_model.js
```

**What it does:**
1. Fetches up to 1000 completed transactions from database
2. Extracts features (distance, urgency, demand, time factors)
3. Splits data into 80% training, 20% testing
4. Trains linear regression model using gradient descent
5. Evaluates model performance on test set
6. Saves trained model to database
7. Shows sample predictions

**Expected Output:**
```
📚 Fetching historical transaction data...
✅ Found 250 completed transactions

🔧 Preparing training data...
📊 Training set: 200 samples
📊 Test set: 50 samples

🎓 Starting model training...
Iteration 0, MSE: 125.4321
Iteration 100, MSE: 45.2134
...
✅ Model trained successfully!

📈 Evaluating model performance...
📊 Model Performance Metrics:
   MAE (Mean Absolute Error): $2.34
   RMSE (Root Mean Squared Error): $3.12
   R² Score: 0.782
   Test Samples: 50

✅ Excellent model performance! (R² > 0.7)
💾 Saving model to database...
✅ Model saved to database successfully!

🔮 Sample Predictions:
   Distance: 10km, Urgency: 2.0h, Demand: 3
   → Predicted Cost: $8.45/section
   ...
```

---

### Step 3: Start the Server

The server automatically loads the trained model on startup:

```bash
node server.js/server.js
```

**Expected Output:**
```
✅ Blockchain initialized
✅ Cost prediction model loaded
App is running on port 5001
```

---

## Usage

### Automatic Integration

Once the model is trained and the server is running, **cost predictions are automatic**!

When a driver clicks "GO" to start a task, the system:
1. Calculates distance to destination
2. Gets package deadline
3. Counts current active routes (demand)
4. Detects time of day and day of week
5. **Predicts optimal cost** using ML model
6. Creates transaction with predicted cost

**No manual intervention needed!** 🎉

---

## API Endpoints

### 1. Get Model Status
```http
GET /api/ml/cost-model/status
Authorization: Admin only
```

**Response:**
```json
{
  "status": "active",
  "model_type": "linear_regression",
  "trained_at": "2026-02-05T10:30:00.000Z",
  "is_active": true,
  "metrics": {
    "mae": 2.34,
    "rmse": 3.12,
    "r_squared": 0.782
  },
  "coefficients": {
    "intercept": 5.23,
    "distance": 0.47,
    "urgency": 0.31,
    "demand": 0.19,
    "hour_peak": 1.52,
    "weekend": 1.98
  }
}
```

---

### 2. Retrain Model
```http
POST /api/ml/cost-model/train
Authorization: Admin only
```

**Response:**
```json
{
  "success": true,
  "message": "Model trained successfully",
  "training_data": {
    "training_samples": 200,
    "test_samples": 50
  },
  "metrics": {
    "mae": 2.34,
    "rmse": 3.12,
    "r_squared": 0.782
  },
  "training_duration_seconds": 3
}
```

**When to Retrain:**
- After accumulating 50+ new completed transactions
- When pricing strategy changes
- When cost patterns shift (seasonal, market changes)
- Monthly or quarterly (recommended)

---

### 3. Test Prediction
```http
POST /api/ml/cost-model/predict
Authorization: Any authenticated user

Body:
{
  "distance_km": 50,
  "deadline_hours": 6,
  "current_demand": 8
}
```

**Response:**
```json
{
  "predicted_cost_per_section": 12.45,
  "input": {
    "distance_km": 50,
    "deadline_hours": 6,
    "current_demand": 8
  }
}
```

---

### 4. Get Training History
```http
GET /api/ml/training-history
Authorization: Admin only
```

**Response:**
```json
{
  "history": [
    {
      "history_id": 3,
      "model_name": "cost_predictor",
      "training_samples": 200,
      "test_samples": 50,
      "mae": 2.34,
      "rmse": 3.12,
      "r_squared": 0.782,
      "training_duration_seconds": 3,
      "trained_at": "2026-02-05T10:30:00.000Z"
    }
  ],
  "total_trainings": 1
}
```

---

## How It Works

### Linear Regression Formula

```
cost = β₀ + β₁×distance + β₂×urgency + β₃×demand + β₄×peak_hour + β₅×weekend
```

Where:
- **β₀** (intercept) = Base cost
- **β₁** (distance coefficient) = Cost increase per km
- **β₂** (urgency coefficient) = Premium for urgent deliveries
- **β₃** (demand coefficient) = Surge pricing factor
- **β₄** (peak_hour coefficient) = Rush hour premium
- **β₅** (weekend coefficient) = Weekend premium

### Feature Engineering

#### 1. **Distance** (Normalized)
```javascript
normalized_distance = (distance_km - mean) / std
```

#### 2. **Urgency** (Normalized)
```javascript
urgency_hours = (deadline - now) / (1000 * 60 * 60)
normalized_urgency = (urgency_hours - mean) / std
```

#### 3. **Demand** (Normalized)
```javascript
current_demand = COUNT(active_routes)
normalized_demand = (current_demand - mean) / std
```

#### 4. **Peak Hour** (Binary)
```javascript
is_peak = (hour >= 7 && hour <= 9) || (hour >= 17 && hour <= 19) ? 1 : 0
```

#### 5. **Weekend** (Binary)
```javascript
is_weekend = (day === Sunday || day === Saturday) ? 1 : 0
```

---

## Training Algorithm

### Gradient Descent

```javascript
for (iteration = 0; iteration < 1000; iteration++) {
    predictions = X × θ
    errors = predictions - y
    
    for (each coefficient θⱼ) {
        gradient = (1/m) × Σ(error × feature)
        θⱼ = θⱼ - learning_rate × gradient
    }
}
```

**Parameters:**
- Learning rate: 0.01
- Iterations: 1000
- Convergence check: Every 100 iterations

---

## Example Scenarios

### Scenario 1: Normal Delivery
```javascript
Input:
- Distance: 30 km
- Deadline: 24 hours
- Demand: 3 routes
- Time: 2 PM Tuesday

Predicted Cost: $6.50/section
```

### Scenario 2: Urgent Rush Hour
```javascript
Input:
- Distance: 50 km
- Deadline: 2 hours (urgent!)
- Demand: 10 routes (high!)
- Time: 5 PM Friday (peak!)

Predicted Cost: $18.75/section
```

### Scenario 3: Long Weekend Delivery
```javascript
Input:
- Distance: 100 km
- Deadline: 48 hours
- Demand: 2 routes
- Time: 10 AM Saturday

Predicted Cost: $14.20/section
```

---

## Benefits

### 1. **Dynamic Pricing** 💰
- Automatically adjusts to market conditions
- Maximizes revenue during high demand
- Competitive pricing during low demand

### 2. **Fair Pricing** ⚖️
- Distance-based pricing ensures fairness
- Urgency premium for rush deliveries
- Transparent pricing factors

### 3. **Data-Driven** 📊
- Learns from historical patterns
- Improves over time with more data
- Adapts to changing conditions

### 4. **Automated** 🤖
- No manual cost calculations
- Consistent pricing logic
- Reduces human error

---

## Maintenance

### Regular Retraining

**Recommended Schedule:**
- **Weekly**: If you have 100+ new transactions/week
- **Monthly**: For moderate transaction volumes
- **Quarterly**: For low transaction volumes

**How to Retrain:**
```bash
# Option 1: Run training script
node server.js/scripts/train_cost_model.js

# Option 2: Use API endpoint
curl -X POST http://localhost:5001/api/ml/cost-model/train \
  -H "Cookie: your-session-cookie"
```

### Monitor Performance

Check model metrics regularly:
```bash
curl http://localhost:5001/api/ml/cost-model/status
```

**Good Performance:**
- R² > 0.7: Excellent
- R² > 0.5: Good
- R² > 0.3: Acceptable
- R² < 0.3: Needs improvement (retrain or collect more data)

---

## Troubleshooting

### Issue: "Insufficient training data"
**Cause:** Less than 10 completed transactions
**Solution:** Complete more transactions before training

### Issue: "Model not loaded"
**Cause:** Database table not created or model not trained
**Solution:** 
1. Run `create_ml_models_table.js`
2. Run `train_cost_model.js`

### Issue: Low R² score
**Cause:** Not enough data or poor feature quality
**Solution:**
- Collect more diverse transactions
- Ensure accurate cost data in historical transactions
- Check that deadlines are being set properly

### Issue: Predictions too high/low
**Cause:** Model needs retraining with recent data
**Solution:** Retrain the model using API or script

---

## Future Enhancements

### Potential Improvements:
1. **Additional Features**
   - Weather conditions
   - Traffic patterns
   - Fuel prices
   - Driver experience level

2. **Advanced Models**
   - Polynomial regression for non-linear patterns
   - Random Forest for complex interactions
   - Neural networks for deep learning

3. **Real-time Updates**
   - Automatic retraining on schedule
   - Online learning (update model incrementally)
   - A/B testing different pricing strategies

4. **Business Intelligence**
   - Cost trend analysis
   - Revenue optimization reports
   - Demand forecasting

---

## Technical Details

### Dependencies
- **Node.js**: JavaScript runtime
- **MySQL**: Database storage
- **Express**: Web framework
- **No external ML libraries**: Pure JavaScript implementation

### Performance
- **Training Time**: 2-5 seconds for 1000 samples
- **Prediction Time**: < 1ms per prediction
- **Memory Usage**: Minimal (~1MB for model storage)

### Scalability
- Handles 10,000+ transactions efficiently
- Can be optimized with batch predictions
- Database-backed model storage for persistence

---

## Credits

**Implementation**: Linear Regression with Gradient Descent  
**Language**: Pure JavaScript (No ML libraries required)  
**Framework**: Node.js + Express  
**Database**: MySQL  

---

## Support

For issues or questions:
1. Check logs in terminal
2. Verify database table exists
3. Ensure sufficient training data (10+ transactions)
4. Check API endpoints return proper responses

**Happy Optimizing! 🚀💰**
