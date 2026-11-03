# 🌍 Mondial: profili statistici dei Paesi del Mondo

Progetto per il corso di **Statistica per i Big Data – Laboratorio di Informatica**
Università degli Studi di Salerno · Anno Accademico 2023/2024

Analisi statistica del database relazionale **Mondial**, che raccoglie informazioni geografiche, demografiche, economiche, politiche e religiose sui Paesi del mondo. Il progetto unisce interrogazioni SQL, analisi esplorativa, regressione lineare e un semplice modello di classificazione per rispondere a quattro domande di ricerca.

---

## 📑 Indice

- [Obiettivi dell'analisi](#-obiettivi-dellanalisi)
- [Dataset](#-dataset)
- [Tecnologie utilizzate](#-tecnologie-utilizzate)
- [Struttura del repository](#-struttura-del-repository)
- [Come eseguire il progetto](#-come-eseguire-il-progetto)
- [Pipeline di analisi](#-pipeline-di-analisi)
- [Risultati principali](#-risultati-principali)
- [Limiti e possibili sviluppi](#-limiti-e-possibili-sviluppi)
- [Team](#-team)

---

## 🎯 Obiettivi dell'analisi

| Focus | Domanda di ricerca |
|-------|--------------------|
| **1** | Esiste una relazione tra il tasso di crescita della popolazione e il tasso di mortalità infantile? |
| **2** | Quanto è forte il legame tra il PIL e la popolazione? |
| **3** | Che tipo di associazione sussiste tra la religione predominante e la forma di governo di un Paese? |
| **4** | La religione predominante nei Paesi dipendenti coincide con quella dei Paesi da cui dipendono? |

---

## 🗄️ Dataset

Il database **Mondial** è ospitato pubblicamente sul [Relational Dataset Repository](https://relational-data.org/) ed è accessibile tramite MySQL con credenziali guest.

Delle numerose tabelle disponibili (`country`, `economy`, `population`, `politics`, `religion`, `encompasses`, `borders`, `city`, `river`, `lake`, …) l'analisi utilizza:

| Tabella | Campi utilizzati |
|---------|------------------|
| `country` | `Name`, `Code`, `Population` |
| `encompasses` | `Continent`, `Percentage` |
| `economy` | `GDP` |
| `population` | `Infant_Mortality`, `Population_Growth` |
| `politics` | `Government`, `Dependent` |
| `religion` | `Name`, `Percentage` |

### Query di estrazione

Per ogni Paese vengono selezionati il **continente prevalente** e la **religione predominante** (quelli con percentuale massima):

```sql
SELECT encompasses.Continent, country.Name AS Nation, country.Code,
       country.Population, politics.Government, politics.Dependent, economy.GDP,
       population.Infant_Mortality, population.Population_Growth,
       religion.Name AS Religion
FROM country
JOIN encompasses ON country.Code = encompasses.Country
JOIN economy     ON country.Code = economy.Country
JOIN population  ON country.Code = population.Country
JOIN politics    ON country.Code = politics.Country
JOIN religion    ON country.Code = religion.Country
WHERE religion.Percentage = (
    SELECT MAX(r2.Percentage) FROM religion AS r2
    WHERE r2.Country = country.Code
) AND encompasses.Percentage = (
    SELECT MAX(e2.Percentage) FROM encompasses AS e2
    WHERE e2.Country = country.Code
);
```

Il dataset risultante contiene circa **209 Paesi** con le colonne:
`Continent`, `Nation`, `Code`, `Population`, `Government`, `Dependent`, `GDP`, `Infant_Mortality`, `Population_Growth`, `Religion`.

---

## 🛠️ Tecnologie utilizzate

- **Python 3** (sviluppato su Google Colab)
- **MySQL** – estrazione dei dati
- [`SQLAlchemy`](https://www.sqlalchemy.org/) + [`mysql-connector-python`](https://pypi.org/project/mysql-connector-python/) – connessione al database
- [`pandas`](https://pandas.pydata.org/) – manipolazione dei dati
- [`NumPy`](https://numpy.org/) – trasformazioni logaritmiche
- [`seaborn`](https://seaborn.pydata.org/) – visualizzazione
- [`SciPy`](https://scipy.org/) – regressione lineare (`stats.linregress`)
- [`scikit-learn`](https://scikit-learn.org/) – classificatore Naive Bayes gaussiano

---

## 📁 Struttura del repository

```
.
├── mondial_analysis.py      # Script di analisi (esportato dal notebook Colab)
├── presentazione_mondial.pdf        # Slide di presentazione del progetto
└── README.md
```

---

## ▶️ Come eseguire il progetto

1. **Clona il repository**
   ```bash
   git clone https://github.com/simonesquizzato/mondial-analysis.git
   cd mondial-analysis
   ```

2. **Installa le dipendenze**
   ```bash
   pip install mysql-connector-python sqlalchemy pandas numpy scipy seaborn scikit-learn
   ```

3. **Esegui l'analisi**
   Lo script è stato generato da un notebook Google Colab: il modo più semplice per eseguirlo è caricarlo su [Colab](https://colab.research.google.com/) o Jupyter ed eseguire le celle in ordine.

   > ⚠️ Se lo esegui come script Python standard, rimuovi la riga `pip install mysql-connector-python` (valida solo in un notebook) e aggiungi `print(...)` o `plt.show()` dove necessario per visualizzare output e grafici.

La connessione al database avviene con le credenziali pubbliche di sola lettura:

```python
engine = create_engine(
    "mysql+mysqlconnector://guest:relational@db.relational-data.org/Mondial"
)
```

---

## 🔬 Pipeline di analisi

### Analisi esplorativa (EDA)
- Statistiche descrittive (`describe()`) delle variabili quantitative.
- Trasformazione logaritmica di `GDP` e `Population` (`log_GDP`, `log_Population`) per ridurre la forte asimmetria delle distribuzioni.
- `pairplot` delle variabili quantitative colorato per continente.

### Focus 1 – Crescita della popolazione vs mortalità infantile
- Rimozione dei valori nulli e dei valori oltre il **95° percentile** per limitare l'effetto degli outlier.
- Regressione lineare semplice `Infant_Mortality ~ Population_Growth`.
- **Classificatore Naive Bayes gaussiano** che predice il continente a partire dalle due variabili (split train/test 80/20, `random_state=99`).

### Focus 2 – PIL vs popolazione
- Regressione lineare tra `log_Population` e `log_GDP`.

### Focus 3 – Religione e forma di governo
- Individuazione delle **cinque forme di governo più diffuse** e, per ciascuna, della religione predominante più frequente.
- Confronto del PIL (per continente) tra i Paesi che presentano una di queste coppie governo-religione e i restanti.

### Focus 4 – Paesi dipendenti
- Join tra Paesi dipendenti e Paesi da cui dipendono (tramite `Dependent` ↔ `Code`).
- Calcolo della percentuale di corrispondenza tra le rispettive religioni predominanti.

---

## 📊 Risultati principali

### Statistiche descrittive

| | Population | GDP | Infant_Mortality | Population_Growth |
|---|---|---|---|---|
| **count** | 209 | 203 | 202 | 203 |
| **mean** | 2.56e+07 | 1.55e+05 | 45.52 | 1.73 |
| **mediana** | 4.39e+06 | 1.04e+04 | 33.65 | 1.71 |
| **min** | 48 | 7.8 | 2.2 | −3.30 |
| **max** | 1.21e+09 | 7.25e+06 | 149.7 | 16.49 |

### Focus 1
- Coefficiente di correlazione **r ≈ 0.52**: relazione positiva moderata, i Paesi con crescita demografica più elevata tendono ad avere una mortalità infantile più alta.
- Il classificatore Naive Bayes raggiunge un'**accuratezza ≈ 73.2%** nel predire il continente sul test set.

### Focus 2
- Coefficiente di correlazione **r ≈ 0.87** tra `log_Population` e `log_GDP`: relazione lineare forte (in scala logaritmica) tra dimensione della popolazione e PIL.

### Focus 3
Religione predominante più frequente per le cinque forme di governo più diffuse:

| Forma di governo | Religione predominante |
|------------------|------------------------|
| British Overseas Territories | Protestant |
| constitutional monarchy | Roman Catholic |
| federal republic | Roman Catholic |
| parliamentary democracy | Roman Catholic |
| republic | Roman Catholic |

Il confronto del PIL mostra che, in Europa, i Paesi selezionati hanno un PIL medio (≈ 218.000) superiore a quello dei restanti (≈ 131.000), mentre in Africa, America e Asia la media è più alta tra i Paesi non selezionati. Data la numerosità ridotta di alcuni gruppi (es. 1 Paese in Oceania, 2 in Asia), i confronti vanno letti con cautela.

### Focus 4
- La religione predominante di un Paese dipendente coincide con quella del Paese da cui dipende nel **≈ 52.4%** dei casi.
- Esempi: *Aruba* → Paesi Bassi (Roman Catholic, ✅); *Puerto Rico* → USA (Roman Catholic vs Protestant, ❌).

---

## ⚠️ Limiti e possibili sviluppi

- I dati di Mondial non sono aggiornati e possono contenere valori mancanti o incoerenti.
- Per continente e religione viene considerato solo il valore con percentuale massima; in caso di parità la query può restituire più righe per lo stesso Paese.
- Le forme di governo sono stringhe testuali non normalizzate (es. varianti di "parliamentary democracy"), il che frammenta le categorie.
- Il classificatore usa solo due variabili e un singolo split: una cross-validation e l'aggiunta di altre feature (es. `log_GDP`) potrebbero migliorarne la valutazione.
- Le correlazioni misurano associazioni lineari e non implicano causalità.

---

## 👥 Team

| Nome | Contatto |
|------|----------|
| Cristian Avagnano | c.avagnano@studenti.unisa.it |
| Mattia Avino | m.avino25@studenti.unisa.it |
| Simone Squizzato | s.squizzato1@studenti.unisa.it |

---

*Progetto realizzato a scopo didattico per l'esame di Laboratorio di Informatica per il corso di laurea in Statistica per i Big Data – Università degli Studi di Salerno.*
