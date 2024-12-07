import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime
import tkinter as tk
from tkinter import messagebox

# Wczytanie danych z pliku CSV
try:
    df = pd.read_csv('poland_data.csv', encoding='utf-8')
except FileNotFoundError:
    messagebox.showerror("Błąd",
                         "Nie znaleziono pliku 'poland_data.csv'. Upewnij się, że plik znajduje się w tym samym folderze co skrypt.")
    exit()


# Funkcja do parsowania numeru PESEL i wyciągania płci oraz daty urodzenia
def parse_pesel(pesel):
    pesel = str(pesel)
    if len(pesel) != 11 or not pesel.isdigit():
        return None, None
    year = int(pesel[0:2])
    month = int(pesel[2:4])
    day = int(pesel[4:6])

    # Określenie stulecia na podstawie miesiąca
    if 1 <= month <= 12:
        century = 1900
    elif 21 <= month <= 32:
        century = 2000
        month -= 20
    elif 41 <= month <= 52:
        century = 2100
        month -= 40
    elif 61 <= month <= 72:
        century = 2200
        month -= 60
    elif 81 <= month <= 92:
        century = 1800
        month -= 80
    else:
        return None, None  # Nieprawidłowy miesiąc

    year += century

    # Płeć na podstawie 10. cyfry PESEL
    gender_digit = int(pesel[9])
    gender = 'Mężczyzna' if gender_digit % 2 == 1 else 'Kobieta'

    # Data urodzenia
    try:
        birth_date = datetime(year, month, day)
    except ValueError:
        return None, None  # Nieprawidłowa data

    return gender, birth_date


# Zastosowanie funkcji do DataFrame
df['Gender'], df['Birth Date'] = zip(*df['PESEL'].apply(parse_pesel))

# Usunięcie rekordów z brakującymi danymi
df.dropna(subset=['Gender', 'Birth Date', 'Power Consumption (kWh)', 'House Size (m2)'], inplace=True)

# Konwersja kolumn do typów numerycznych
df['Power Consumption (kWh)'] = pd.to_numeric(df['Power Consumption (kWh)'], errors='coerce')
df['House Size (m2)'] = pd.to_numeric(df['House Size (m2)'], errors='coerce')

# Obliczenie wieku
today = datetime.now()
df['Age'] = df['Birth Date'].apply(lambda x: (today - x).days // 365 if x else None)

# Usunięcie rekordów z brakującym wiekiem
df.dropna(subset=['Age'], inplace=True)

# Lista unikalnych lokalizacji
locations = df['Lokalizacja'].unique().tolist()
locations.append('Cała Polska')


# Funkcja do tworzenia wykresów
def create_plots(location):
    # Filtrowanie danych dla wybranej lokalizacji
    if location == 'Cała Polska':
        data = df.copy()
    else:
        data = df[df['Lokalizacja'] == location]

    if data.empty:
        messagebox.showinfo("Informacja", f"Brak danych dla lokalizacji: {location}")
        return

    # Obliczenie zużycia energii na m2
    data['Consumption per m2'] = data['Power Consumption (kWh)'] / data['House Size (m2)']

    # Definicja przedziałów wiekowych
    bins = [0, 20, 40, 60, 80, 100, 120]
    labels = ['0-20', '21-40', '41-60', '61-80', '81-100', '101+']
    data['Age Range'] = pd.cut(data['Age'], bins=bins, labels=labels, include_lowest=True)

    # Grupowanie danych dla pierwszego wykresu
    group1 = data.groupby(['Age Range', 'Gender'])['Consumption per m2'].mean().unstack()

    # Reindeksacja, aby uwzględnić wszystkie przedziały wiekowe
    group1 = group1.reindex(labels)

    # Tworzenie wykresu 1 (liniowego)
    fig, axs = plt.subplots(1, 3, figsize=(18, 6))

    # Mapowanie przedziałów wiekowych na wartości numeryczne
    age_ranges_numeric = []
    for label in labels:
        if '-' in label:
            age_numeric = int(label.split('-')[0]) + (int(label.split('-')[1]) - int(label.split('-')[0])) / 2
        else:
            age_numeric = int(label.rstrip('+')) + 5  # Przykładowa wartość środka przedziału dla '101+'
        age_ranges_numeric.append(age_numeric)

    # Tworzenie wykresu 1
    for gender in ['Kobieta', 'Mężczyzna']:
        if gender in group1.columns:
            axs[0].plot(age_ranges_numeric, group1[gender], marker='o', label=gender)

    axs[0].set_title('Średnie zużycie energii na m²')
    axs[0].set_xlabel('Przedział wiekowy')
    axs[0].set_ylabel('Zużycie energii (kWh/m²)')
    axs[0].set_xticks(age_ranges_numeric)
    axs[0].set_xticklabels(labels)
    axs[0].legend(title='Płeć')

    # Przygotowanie danych dla drugiego wykresu
    top_names = data['First Name'].value_counts().head(5)

    # Tworzenie wykresu 2 (kołowego)
    axs[1].pie(top_names, labels=top_names.index, autopct='%1.1f%%', startangle=90)
    axs[1].set_title('5 najbardziej popularnych imion')

    # Grupowanie danych dla trzeciego wykresu (średnie zużycie energii dla kobiet i mężczyzn)
    group3 = data.groupby('Gender')['Power Consumption (kWh)'].mean()

    # Tworzenie wykresu 3 (kolumnowego)
    group3.plot(kind='bar', ax=axs[2], color=['lightblue', 'lightgreen'])
    axs[2].set_title('Średnie zużycie energii na domostwo (kWh)')
    axs[2].set_xlabel('Płeć')
    axs[2].set_ylabel('Średnie zużycie energii (kWh)')

    plt.tight_layout()
    plt.show()


# Tworzenie głównego okna aplikacji
root = tk.Tk()
root.title('Wybierz lokalizację')

# Dodanie paska przewijania jeśli jest wiele lokalizacji
canvas = tk.Canvas(root)
scrollbar = tk.Scrollbar(root, orient="vertical", command=canvas.yview)
scroll_frame = tk.Frame(canvas)

scroll_frame.bind(
    "<Configure>",
    lambda e: canvas.configure(
        scrollregion=canvas.bbox("all")
    )
)

canvas.create_window((0, 0), window=scroll_frame, anchor='nw')
canvas.configure(yscrollcommand=scrollbar.set)

# Tworzenie przycisków dla każdej lokalizacji
for loc in locations:
    button = tk.Button(scroll_frame, text=loc, width=20, command=lambda l=loc: create_plots(l))
    button.pack(pady=2)

canvas.pack(side="left", fill="both", expand=True)
scrollbar.pack(side="right", fill="y")

root.mainloop()
