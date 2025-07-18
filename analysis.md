import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import json
from sklearn.preprocessing import MinMaxScaler, StandardScaler
from google.colab import drive
drive.mount('/content/drive')
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




        wallet_features = pd.DataFrame.from_dict(wallet_stats, orient='index') # dict made to df
wallet_features.reset_index(inplace=True) #all wallets in single col
wallet_features.rename(columns={'index': 'userWallet'}, inplace=True)

# Add ratios
wallet_features['repay_to_borrow_ratio'] = wallet_features['total_repay'] / (wallet_features['total_borrow'] + 1)
wallet_features['borrow_to_deposit_ratio'] = wallet_features['total_borrow'] / (wallet_features['total_deposit'] + 1)
wallet_features.replace([np.inf, -np.inf], 0, inplace=True)

print(" Wallet:", wallet_features.shape)





# Filter out wallets with no borrowing or no repaying activity
wallet_features = wallet_features[wallet_features['total_borrow'] > 0]
wallet_features = wallet_features[wallet_features['total_repay'] > 0]




X = wallet_features[['total_deposit', 'total_borrow', 'total_repay', 'repay_to_borrow_ratio',
                     'borrow_to_deposit_ratio', 'num_liquidations', 'tx_count']]
scaler = MinMaxScaler()
X_scaled = scaler.fit_transform(X)

from sklearn.cluster import KMeans
kmeans = KMeans(n_clusters=5, random_state=42, n_init=10)
wallet_features['cluster'] = kmeans.fit_predict(X_scaled)

# Rank clusters by behavior (high repay ratio is better)
cluster_order = wallet_features.groupby('cluster')['repay_to_borrow_ratio'].mean().sort_values()

# Assign credit score ranges based on clusters
score_scale = np.linspace(100, 1000, len(cluster_order))
score_map = {cluster: int(score) for cluster, score in zip(cluster_order.index, score_scale)}
wallet_features['score'] = wallet_features['cluster'].map(score_map)




###HISTOGRAM
plt.figure(figsize=(8, 5))
sns.histplot(wallet_features['score'], bins=10, kde=True)
plt.title("Wallet Credit Score Distribution")
plt.xlabel("Credit Score")
plt.ylabel("Number of Wallets")
plt.grid(True)
plt.tight_layout()
plt.show()
https://github.com/user-attachments/assets/9eaecf88-976a-4a16-b175-c37b7846a20e





bins = [0, 300, 700, 1000]
labels = ['High Risk (0–300)', 'Moderate (300–700)', 'Trustworthy (700–1000)']

wallet_features['score_band'] = pd.cut(wallet_features['score'], bins=bins, labels=labels, include_lowest=True)
score_counts = wallet_features['score_band'].value_counts().sort_index()

colors = ['#ff4c4c', '#ffcc00', '#33cc33']

plt.figure(figsize=(10, 6))
sns.barplot(x=score_counts.index, y=score_counts.values, palette=colors)

for i, val in enumerate(score_counts.values):
    plt.text(i, val + 5, str(val), ha='center', va='bottom', fontweight='bold')

plt.title("Credit Score Distribution with Behavioral Bands", fontsize=14)
plt.xlabel("Credit Score Range", fontsize=12)
plt.ylabel("Number of Wallets", fontsize=12)
plt.grid(axis='y', linestyle='--', alpha=0.6)
plt.tight_layout()
plt.show()


