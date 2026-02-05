# 🚀 Quick Start Guide: ML Cost Optimization

## Prerequisites
- MySQL database running
- Node.js installed
- At least 10 completed transactions in database (for real training)

---

## Step-by-Step Setup (5 minutes)

### 1️⃣ Test the ML Model (Optional)

First, test if the model works correctly:

```bash
node server.js/scripts/test_cost_prediction.js
```

**Expected Output:**
- Training with sample data
- 5 test predictions
- Model coefficients
- Performance metrics
- Exported model JSON

---

### 2️⃣ Create Database Table

```bash
node server.js/scripts/create_ml_models_table.js
```

**Expected Output:**
```
📋 Creating ML models table...
✅ ml_models table created successfully!
✅ model_training_history table created successfully!
✅ Default cost predictor model initialized!
```

**What happens:**
- Creates `ml_models` table
- Creates `model_training_history` table  
- Initializes default model with baseline coefficients

---

### 3️⃣ Train the Model

**IMPORTANT:** You need at least 10 completed transactions!

```bash
node server.js/scripts/train_cost_model.js
```

**If you don't have enough data:**
```
❌ Insufficient data for training. Need at least 10 completed transactions.
   Current count: 3

💡 Tip: Create more transactions and complete them to train the model.
```

**If successful:**
```
✅ Found 250 completed transactions
🎓 Starting model training...
✅ Model trained successfully!
📊 Model Performance Metrics:
   MAE: $2.34
   RMSE: $3.12
   R² Score: 0.782
✅ Model saved to database successfully!
```

---

### 4️⃣ Start the Server

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

## 🎯 How to Use

### Automatic Integration (No Code Changes Needed!)

The ML model is **automatically used** when creating transactions:

1. **Driver logs in**
2. **Selects a container and plans route**
3. **Clicks "GO" on a task**
4. **System automatically:**
   - Calculates distance
   - Checks deadline urgency
   - Counts active routes (demand)
   - Detects peak hours / weekends
   - **Predicts optimal cost using ML** ✨
   - Creates transaction with predicted cost

---

## 📡 API Endpoints (For Testing)

### Check Model Status
```bash
# Get model info and performance
curl http://localhost:5001/api/ml/cost-model/status
```

### Test a Prediction
```bash
# Predict cost for specific scenario
curl -X POST http://localhost:5001/api/ml/cost-model/predict \
  -H "Content-Type: application/json" \
  -d '{
    "distance_km": 50,
    "deadline_hours": 6,
    "current_demand": 8
  }'
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

### Retrain Model (Admin Only)
```bash
# Retrain with latest transaction data
curl -X POST http://localhost:5001/api/ml/cost-model/train
```

### View Training History
```bash
# See all past training sessions
curl http://localhost:5001/api/ml/training-history
```

---

## 🧪 Testing Scenarios

### Scenario 1: Normal Delivery
```bash
curl -X POST http://localhost:5001/api/ml/cost-model/predict \
  -H "Content-Type: application/json" \
  -d '{
    "distance_km": 30,
    "deadline_hours": 24,
    "current_demand": 3
  }'
```
**Expected:** ~$6-8/section

### Scenario 2: Urgent Rush
```bash
curl -X POST http://localhost:5001/api/ml/cost-model/predict \
  -H "Content-Type: application/json" \
  -d '{
    "distance_km": 50,
    "deadline_hours": 2,
    "current_demand": 10
  }'
```
**Expected:** ~$15-20/section (higher due to urgency + demand)

### Scenario 3: Long Distance
```bash
curl -X POST http://localhost:5001/api/ml/cost-model/predict \
  -H "Content-Type: application/json" \
  -d '{
    "distance_km": 100,
    "deadline_hours": 48,
    "current_demand": 2
  }'
```
**Expected:** ~$12-16/section (distance-based)

---

## 📊 Understanding Results

### Good Model Performance:
- **R² > 0.7**: Excellent! Model predictions are very accurate
- **R² > 0.5**: Good! Model is reliable
- **MAE < $5**: Predictions within $5 of actual costs

### Poor Model Performance:
- **R² < 0.3**: Needs more training data or better features
- **MAE > $10**: Large prediction errors

**Solution:** Collect more diverse transaction data and retrain

---

## 🔄 Maintenance Schedule

### When to Retrain:

| Transaction Volume | Retrain Frequency |
|-------------------|-------------------|
| 100+ new/week | Weekly |
| 50-100 new/week | Bi-weekly |
| 20-50 new/week | Monthly |
| < 20 new/week | Quarterly |

### How to Retrain:
```bash
# Option 1: Run script
node server.js/scripts/train_cost_model.js

# Option 2: Use API (from browser console or Postman)
fetch('/api/ml/cost-model/train', { method: 'POST' })
```

---

## ❌ Troubleshooting

### "Insufficient training data"
**Problem:** Less than 10 completed transactions  
**Solution:** Complete more delivery transactions first

### "Model not loaded" on server start
**Problem:** Database table doesn't exist or no model saved  
**Solution:**
```bash
node server.js/scripts/create_ml_models_table.js
node server.js/scripts/train_cost_model.js
```

### Predictions seem wrong
**Problem:** Model needs retraining with recent data  
**Solution:** 
```bash
node server.js/scripts/train_cost_model.js
```

### R² score is low
**Problem:** Not enough diverse training data  
**Solution:**
1. Ensure transaction costs are realistic
2. Complete transactions at different times (peak/off-peak)
3. Vary distances and urgency levels
4. Accumulate 50+ transactions before retraining

---

## 🎓 What the Model Learns

The model analyzes patterns like:

| Pattern | What It Learns |
|---------|---------------|
| Distance | "Longer routes cost more" |
| Urgency | "2-hour deadline = premium pricing" |
| Demand | "10 active routes = surge pricing" |
| Peak Hours | "Evening rush = higher costs" |
| Weekends | "Saturday delivery = premium" |

**Example Learning:**
- After 100 transactions, model discovers: "50km + 2hr deadline = $15/section"
- After 500 transactions, model refines: "Same conditions on Friday 5PM = $18/section"

---

## 💡 Tips for Better Predictions

1. **Set realistic deadlines** - Model uses urgency as a key factor
2. **Complete transactions promptly** - More data = better predictions
3. **Vary delivery times** - Train model on different scenarios
4. **Retrain regularly** - Keep model updated with latest patterns
5. **Monitor R² score** - Retrain if it drops below 0.5

---

## 📈 Success Metrics

After implementing ML cost optimization, you should see:

✅ **Dynamic pricing** adapting to conditions  
✅ **Higher revenue** during peak demand  
✅ **Competitive pricing** during off-peak  
✅ **Consistent cost logic** across all transactions  
✅ **Automated pricing** with no manual calculations  

---

## 🎉 You're All Set!

The ML cost optimization system is now:
- ✅ Installed
- ✅ Trained (if you had data)
- ✅ Integrated into transaction flow
- ✅ Ready to predict costs automatically

**Every new transaction will use ML predictions! 🚀**

---

## 📚 Additional Resources

- Full documentation: [ML_COST_OPTIMIZATION_README.md](ML_COST_OPTIMIZATION_README.md)
- Test script: `server.js/scripts/test_cost_prediction.js`
- Training script: `server.js/scripts/train_cost_model.js`
- Model code: `server.js/utils/costPrediction.js`

---

**Happy Optimizing! 💰🤖**
