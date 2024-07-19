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






************


import pandas as pd

# Przykładowe dane
data = {
    'Deal Number': ['1', '2', '3', '4', '5', '6'],
    'Matched Deal Number': ['X2', 'X3, 4, X5', 'X1', '5', '4', 2.0],
    'Folder Short Name': ['OTHER', 'COMMODITY', 'OTHER', 'OTHER', 'COMMODITY', 'OTHER'],
    # Inne kolumny mogą być dodane tutaj
}

df = pd.DataFrame(data)

# Funkcja do usuwania prefiksu 'X' i konwersji na string
def clean_matched_deal_number(x):
    if isinstance(x, float):
        return str(int(x))
    elif isinstance(x, str):
        return x.replace('X', '')
    return str(x)

# Konwersja wszystkich wartości na stringi i czyszczenie kolumny 'Matched Deal Number'
df['Matched Deal Number'] = df['Matched Deal Number'].apply(lambda x: ', '.join([clean_matched_deal_number(i) for i in str(x).split(', ')]))

# Oddzielanie transakcji nie będących w folderze 'COMMODITY'
non_commodity_deals = df[df['Folder Short Name'] != 'COMMODITY']

# Funkcja do tworzenia nowej struktury DataFrame
def match_transactions(df, non_commodity_deals):
    result = []
    for _, deal in non_commodity_deals.iterrows():
        result.append(deal.tolist() + [''] * (len(df.columns) + 1 - len(deal)))  # Dodanie wiersza z transakcją
        matched_deal_numbers = deal['Matched Deal Number'].split(', ')
        for match in matched_deal_numbers:
            matched_deals = df[df['Deal Number'] == match.strip()]
            for _, matched_deal in matched_deals.iterrows():
                result.append([''] * len(deal) + matched_deal.tolist())  # Dodanie wiersza z dopasowaną transakcją
        result.append([''] * (len(deal) + len(df.columns)))  # Dodanie pustego wiersza
    return pd.DataFrame(result, columns=list(df.columns) + [''] * (len(result[0]) - len(df.columns)))

# Tworzenie nowego DataFrame
matched_df = match_transactions(df, non_commodity_deals)

# Wyświetlanie wyników
print(matched_df)

# Zapisanie wyników do pliku CSV (opcjonalnie)
matched_df.to_csv('matched_transactions.csv', index=False)
