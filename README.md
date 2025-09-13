#!/usr/bin/env python3
import os
import argparse
from collections import defaultdict
import matplotlib.pyplot as plt
import pandas as pd
from github import Github




def fetch_events(username, gh):
user = gh.get_user(username)
events = user.get_events()
return list(events)




def contributions_by_month(events):
counts = defaultdict(int)
for ev in events:
if ev.type == 'PushEvent':
created = ev.created_at
key = created.strftime('%Y-%m')
try:
commits = len(ev.payload.get('commits', []))
except Exception:
commits = 1
counts[key] += commits
df = pd.Series(counts).sort_index()
if df.empty:
return pd.DataFrame({'month': [], 'commits': []})
df.index = pd.to_datetime(df.index + '-01')
df = df.rename('commits').reset_index().rename(columns={'index': 'month'})
return df




def fetch_languages(username, gh):
user = gh.get_user(username)
repos = user.get_repos()
lang_agg = defaultdict(int)
for r in repos:
try:
langs = r.get_languages()
for lang, bytes_count in langs.items():
lang_agg[lang] += bytes_count
except Exception:
continue
return lang_agg




def plot_contributions(df, outpath):
plt.figure(figsize=(10,4))
if df.empty:
plt.text(0.5, 0.5, 'No contribution data available', ha='center', va='center')
plt.axis('off')
else:
plt.bar(df['month'].dt.strftime('%Y-%m'), df['commits'])
plt.xticks(rotation=45, ha='right')
plt.xlabel('Month')
plt.ylabel('Commits')
plt.title('Contributions by month (from public events)')
plt.tight_layout()
main()
