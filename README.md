# DeFi-Credit-Score-Model


Hey there! I'm Harshal  
This is my attempt to build a credit scoring model using DeFi wallet data from Aave V2. I'm still learning all this, so the code is simple, the logic is straightforward, and I’ve tried to keep everything beginner-friendly and honest.

---

##  What This Project Is About ##

We were given a big JSON file containing a bunch of transactions from users interacting with Aave V2. Each user (wallet) performs actions like:
- `deposit`
- `borrow`
- `repay`
- `liquidationcall` 

The goal?  
To analyze how wallets behave and give each one a **credit score from 0 to 1000** based only on how they've interacted with Aave.

---

## How I Broke It Down (My Thought Process) ##

### 1. **Loading the Data**
Started off by loading the raw JSON data using pandas. The file was huge, so I took it step by step — just looked at the keys, cleaned the rows, and kept only what I needed.

### 2. **Building Wallet Stats**
For each wallet, I tracked:
- How much it deposited
- How much it borrowed
- How much it repaid
- How many times it got liquidated
- How many total transactions it made

I stored all this in a dictionary. Felt like building a mini profile for every wallet.

### 3. **Creating Some Ratios**
To make things more meaningful, I added two key features:
- `repay_to_borrow_ratio`
- `borrow_to_deposit_ratio`

These helped me compare users fairly, even if they had very different total amounts.

### 4. **Clustering Wallets (KMeans)**
Since I didn’t have any labels (like "good user" or "bad user"), I used **KMeans** to cluster them into 5 groups based on their behavior.

I sorted the clusters based on how well they repaid, and assigned each cluster a score between 100 and 1000 — higher score = better behavior.

---

## 📊 Credit Score Distribution

I made a simple chart to see how many wallets fall into which score range:

-  **0–300**: Risky / bot-like behavior  
-  **300–700**: Average users  
-  **700–1000**: Clean, responsible wallets

You can find more details and visualizations in the [`analysis.md`](./analysis.md) file.

---

## 🔧 Tech & Libraries I Used

I kept things light:

```bash
pandas
numpy
matplotlib
seaborn
scikit-learn
