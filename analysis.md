pip install numpy pandas #numerical help and data handling
pip install scikit-learn #for metrics and models
pip install matplotlib #plotting data visualization
pip install seaborn
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import json
from sklearn.preprocessing import MinMaxScaler, StandardScaler
from google.colab import drive
drive.mount('/content/drive')
from sklearn.cluster import KMeans


with open(r'/content/drive/MyDrive/user-wallet-transactions.json', 'r') as f:
    data = json.load(f)
df=pd.DataFrame(data)
print(df.shape)
print(df.columns.tolist())
  print(df['userWallet'].nunique())



  wallet_stats = {}

for i in range(len(df)): #iterating through json file
    row = df.iloc[i]
    wallet = row['userWallet']
    action = row['action']
    amount = 0





    try:
        amount = float(row['actionData'].get('amount', 0))
    except:
        amount = 0






    if wallet not in wallet_stats: #add to dictn made
        wallet_stats[wallet] = {
            'total_deposit': 0,
            'total_borrow': 0,
            'total_repay': 0,
            'num_liquidations': 0,
            'tx_count': 0
        }

    wallet_stats[wallet]['tx_count'] += 1

    if action == 'deposit':
        wallet_stats[wallet]['total_deposit'] += amount
    elif action == 'borrow':
        wallet_stats[wallet]['total_borrow'] += amount
    elif action == 'repay':
        wallet_stats[wallet]['total_repay'] += amount
    elif action == 'liquidationcall':
        wallet_stats[wallet]['num_liquidations'] += 1


