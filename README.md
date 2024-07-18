# Data Science --Forecast-California-Housing-Market

In progress...
import pandas as pd

# Przykładowy DataFrame
data = {
    'Type': ['Sell', 'Buy', 'Sell', 'Buy', 'Sell', 'Sell'],
    'Option Type': ['Call', 'Call', 'Put', 'Put', 'Call', 'Call'],
    'Strike': [100, 100, 150, 150, 200, 200],
    'Trade Date': ['2023-01-01', '2023-01-01', '2023-01-02', '2023-01-02', '2023-01-03', '2023-01-03'],
    'Maturity Date': ['2024-01-01', '2024-01-01', '2024-02-02', '2024-02-02', '2024-03-03', '2024-03-03'],
    'Folder': ['commodity', 'STR_COM', 'commodity', 'STR_COM', 'commodity', 'commodity'],
    'Deal Number': [1, 2, 3, 4, 5, 6]
}

df = pd.DataFrame(data)

# Filtrujemy tylko interesujące nas kolumny
df = df[['Type', 'Option Type', 'Strike', 'Trade Date', 'Maturity Date', 'Folder', 'Deal Number']]

# Dodajemy kolumnę na numer transakcji, która odpowiada
df['Matched Deal Number'] = 'XXX'

# Grupowanie po kolumnach 'Type', 'Option Type', 'Strike', 'Trade Date' i 'Maturity Date'
grouped = df.groupby(['Option Type', 'Strike', 'Trade Date', 'Maturity Date'])

for name, group in grouped:
    sells = group[group['Type'] == 'Sell']
    buys = group[group['Type'] == 'Buy']
    
    for _, sell in sells.iterrows():
        # Znalezienie odpowiadającego Buy z różnym Folder
        matching_buys = buys[(buys['Folder'] != sell['Folder']) & (buys['Matched Deal Number'] == 'XXX')]
        if not matching_buys.empty:
            buy = matching_buys.iloc[0]
            df.loc[df['Deal Number'] == sell['Deal Number'], 'Matched Deal Number'] = buy['Deal Number']
            df.loc[df['Deal Number'] == buy['Deal Number'], 'Matched Deal Number'] = sell['Deal Number']
            buys = buys[buys['Deal Number'] != buy['Deal Number']]  # Usuwamy dopasowaną transakcję z dalszych dopasowań
    
    for _, buy in buys.iterrows():
        # Znalezienie odpowiadającego Sell z różnym Folder
        matching_sells = sells[(sells['Folder'] != buy['Folder']) & (sells['Matched Deal Number'] == 'XXX')]
        if not matching_sells.empty:
            sell = matching_sells.iloc[0]
            df.loc[df['Deal Number'] == buy['Deal Number'], 'Matched Deal Number'] = sell['Deal Number']
            df.loc[df['Deal Number'] == sell['Deal Number'], 'Matched Deal Number'] = buy['Deal Number']
            sells = sells[sells['Deal Number'] != sell['Deal Number']]  # Usuwamy dopasowaną transakcję z dalszych dopasowań

# Zapisanie wyników do pliku CSV
df.to_csv('matched_transactions.csv', index=False)

print(df)
