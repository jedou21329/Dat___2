# app.py
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

# -----------------------------
# Configuration de la page
# -----------------------------
st.set_page_config(
    page_title="🇲🇷 Dashboard Macroéconomique – Mauritanie",
    page_icon="📊",
    layout="wide"
)

# Palette bleue
BLEU_FONCE = "#003366"
BLEU_MOYEN = "#1f77b4"
ROUGE = "#c00000"

# -----------------------------
# Titre et introduction
# -----------------------------
st.title("🇲🇷 Dashboard Macroéconomique – Mauritanie")
st.markdown("""
Ce dashboard présente l’évolution des principaux indicateurs économiques de la Mauritanie, 
basé sur des données de la **Banque Centrale de Mauritanie (BCM)**, du **FMI** et de la **Banque Mondiale**.
""")

# -----------------------------
# Chargement et nettoyage des données
# -----------------------------
@st.cache_data
def load_and_clean_data():
    df = pd.read_csv("macro_mauritanie_complet_1960_2024.csv")
    df_clean = df.groupby("Année", as_index=False).first()
    df_clean["Année"] = df_clean["Année"].astype(int)
    return df_clean.sort_values("Année").reset_index(drop=True)

df = load_and_clean_data()

# -----------------------------
# Sélection de la période
# -----------------------------
st.sidebar.header("🔧 Paramètres")
annee_min, annee_max = st.sidebar.slider(
    "Sélectionner la période",
    min_value=int(df["Année"].min()),
    max_value=int(df["Année"].max()),
    value=(2000, 2024)
)

# -----------------------------
# GRAPHIQUE 1 : Recettes fiscales (% PIB)
# -----------------------------
st.subheader("1. Recettes fiscales (% du PIB)")
df_rec = df[(df["Année"] >= 2007) & (df["Année"] <= 2024)]
df_rec = df_rec.dropna(subset=["Recettes_fiscales_pct_PIB"])

if not df_rec.empty:
    fig1, ax1 = plt.subplots(figsize=(12, 5))
    bars = ax1.bar(df_rec["Année"], df_rec["Recettes_fiscales_pct_PIB"], color=BLEU_MOYEN)
    for bar in bars:
        height = bar.get_height()
        ax1.text(bar.get_x() + bar.get_width()/2, height + 0.1, f"{height:.1f}%", 
                 ha="center", va="bottom", fontsize=8, color=BLEU_FONCE)
    ax1.set_xlabel("Année")
    ax1.set_ylabel("% du PIB")
    ax1.set_title("Recettes fiscales (% du PIB, 2007–2024)", weight="bold", color=BLEU_FONCE)
    ax1.grid(True, axis='y', alpha=0.3)
    plt.xticks(rotation=45)
    st.pyplot(fig1)
else:
    st.warning("Données indisponibles pour les recettes fiscales.")

# -----------------------------
# GRAPHIQUE 2 : Croissance vs Inflation
# -----------------------------
st.subheader("2. Croissance économique vs Inflation")
df_macro = df[(df["Année"] >= annee_min) & (df["Année"] <= annee_max)]
df_macro = df_macro.dropna(subset=["Croissance_PIB_pct", "Inflation_pct"])

if not df_macro.empty:
    fig2, ax1 = plt.subplots(figsize=(12, 6))
    
    # Croissance
    ax1.plot(df_macro["Année"], df_macro["Croissance_PIB_pct"], 
             color=BLEU_FONCE, marker='o', linewidth=2.5, label="Croissance PIB (%)")
    for i, row in df_macro.iterrows():
        ax1.text(row["Année"], row["Croissance_PIB_pct"] + 0.3, f"{row['Croissance_PIB_pct']:.1f}%", 
                 ha="center", va="bottom", fontsize=7, color=BLEU_FONCE)
    ax1.set_xlabel("Année")
    ax1.set_ylabel("Croissance PIB (%)", color=BLEU_FONCE)
    ax1.tick_params(axis='y', labelcolor=BLEU_FONCE)
    
    # Inflation
    ax2 = ax1.twinx()
    ax2.plot(df_macro["Année"], df_macro["Inflation_pct"], 
             color=ROUGE, marker='s', linestyle='--', linewidth=2.5, label="Inflation (%)")
    for i, row in df_macro.iterrows():
        ax2.text(row["Année"], row["Inflation_pct"] - 0.6, f"{row['Inflation_pct']:.1f}%", 
                 ha="center", va="top", fontsize=7, color=ROUGE)
    ax2.set_ylabel("Inflation (%)", color=ROUGE)
    ax2.tick_params(axis='y', labelcolor=ROUGE)
    
    # Annotations historiques (si dans la plage)
    events = {2008: "Crise", 2015: "Fer", 2020: "Pandémie", 2022: "Ukraine"}
    for year, label in events.items():
        if annee_min <= year <= annee_max:
            ax1.axvline(year, color="gray", linestyle=":", alpha=0.7)
            ax1.text(year, ax1.get_ylim()[0] + 1, label, rotation=90, color="gray", fontsize=8)
    
    ax1.set_title(f"Croissance vs Inflation ({annee_min}–{annee_max})", weight="bold", color=BLEU_FONCE)
    st.pyplot(fig2)
else:
    st.warning("Données insuffisantes pour la période sélectionnée.")

# -----------------------------
# Sources
# -----------------------------
st.markdown("---")
st.subheader("📚 Sources")
st.markdown("""
- **Banque Centrale de Mauritanie (BCM)** – Rapports annuels, bulletins économiques  
- **Fonds Monétaire International (FMI)** – Consultations Article IV (2022, 2023)  
- **Banque Mondiale** – World Development Indicators
""")
st.caption("© 2026 – Projet académique | Jedou Mohamed Bebacar | Master SSD, Université de Nouakchott")