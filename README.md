# Fault Detection - ALFA Dataset

# Indice

- [Introduzione](#introduzione)
- [Dataset](#dataset)
- [Sviluppo](#sviluppo)
  - [Preprocessing](#preprocessing)
  - [Estrazione features diagnostiche](#estrazione-features-diagnostiche)
  - [Classificazione](#classificazione)
- [Risultati](#risultati)
- [Conclusioni](#conclusioni)

# Introduzione

L’obiettivo del progetto è quello di progettare un classificatore diagnostico dei guasti, basandosi sui dati presenti nel dataset ALFA (scaricabile da [qui](https://kilthub.cmu.edu/articles/dataset/ALFA_A_Dataset_for_UAV_Fault_and_Anomaly_Detection/12707963)).

Il codice presente nel file `FaultDetection_ALFA_Dataset.ipynb` nasce per essere eseguito in ambiente [Colab](https://colab.research.google.com/) e utilizzando [Google Drive](https://drive.google.com/) per caricare e salvare i file necessari. All’interno del file sono presenti i commenti del codice in lingua inglese. Oltre al codice è condivisa la cartella `failures` contente i file e le cartelle prodotte dal codice. Per analisi maggiormente approfondite si rimanda alla relazione del progetto presente nel repository.

---

The goal of the project is to design a fault diagnostic classifier based on the data present in the ALFA dataset (downloadable from [here](https://kilthub.cmu.edu/articles/dataset/ALFA_A_Dataset_for_UAV_Fault_and_Anomaly_Detection/12707963)).

The code in the file `FaultDetection_ALFA_Dataset.ipynb` is intended to be executed in a [Colab](https://colab.research.google.com/) environment and uses [Google Drive](https://drive.google.com/) to upload and save the necessary files. The code comments inside the file are in English. In addition to the code, the `failures` folder is shared, which contains the files and folders produced by the code. For more in-depth analysis, please refer to the project report in the repository

# Dataset

Il progetto è stato svolto sul dataset [AirLab Failure and Anomaly (ALFA)](https://theairlab.org/alfa-dataset/), un set di dati raccolti da diversi voli autonomi di un UAV (Unmanned Aerial Vehicle) ad ala fissa, che descrivono i guasti alle superfici di controllo e ai motori dell’aeromobile. In particolare si è lavorato solo su uno dei quattro tipi di dato presente nel dataset: `processed`. Il dataset `processed`contiene sequenze di volo elaborate per contenere solo dati di volo autonomo e per includere i dati di ground truth solo in caso di guasto, ogni file contiene una sequenza di volo con al massimo un guasto. La seguente tabella mostra i tipi di guasto presenti nel dataset insieme al numero di voli e ai secondi di volo con e senza guasto:

| Tipo di guasto              | Numero di casi testati | Tempo di volo prima del guasto (s) | Tempo di volo con guasto (s) |
| --------------------------- | ---------------------- | ---------------------------------- | ---------------------------- |
| Engine Full Power Loss      | 23                     | 2282                               | 362                          |
| Rudder Stuck to Left        | 1                      | 60                                 | 9                            |
| Rudder Stuck to Right       | 2                      | 107                                | 32                           |
| Elevator Stuck at Zero      | 2                      | 181                                | 23                           |
| Left Aileron Stuck at Zero  | 3                      | 228                                | 183                          |
| Right Aileron Stuck at Zero | 4                      | 442                                | 231                          |
| Both Ailerons Stuck at Zero | 1                      | 66                                 | 36                           |
| Rudder & Aileron at Zero    | 1                      | 116                                | 27                           |
| No Fault                    | 10                     | 558                                | -                            |
| Total                       | 47                     | 3935                               | 777                          |

# Sviluppo

## Preprocessing

Per ognuno dei 47 casi di volo presenti nel dataset sono stata selezionati solo i file CSV più significativi:

- "`failure_status.csv`"
- "`nav_info-airspeed.csv`"
- "`nav_info-errors.csv`"
- "`nav_info-pitch.csv`"
- "`nav_info-roll.csv`"

- "`nav_info-velocity.csv`"
- "`nav_info-yaw.csv`"
- "`local_position-velocity.csv`"
- "`imu-data.csv`"

In particolare per ognuno dei file sono stati utilizzati le seguenti colonne:

- `failure_status-<name>`
  - `%time`
  - `field.data`
- `nav_info-airspeed`
  - `field.commanded`
  - `field.measured`
- `nav_info-roll`
  - `field.commanded`
  - `field.measured`
- `nav_info-pitch`
  - `field.commanded`
  - `field.measured`
- `nav_info-yaw`
  - `field.commanded`
  - `field.measured`
- `nav_info_velocity`

  - `field.des_x`
  - `field.des_y`
  - `field.des_z`
  - `field.meas_x`
  - `field.meas_y`
  - `field.meas_z`

- `nav_info_errors`
  - `field.alt_error`
  - `field.aspd_error`
  - `field.xtrack_error`
- `imu-data`
  - `field.orientation.x`
  - `field.orientation.y`
  - `field.orientation.z`
  - `field.linear_acceleration.x`
  - `field.linear_acceleration.y`
  - `field.linear_acceleration.z`
- `local_position-velocity`
  - `field.twist.angular.x`
  - `field.twist.angular.y`
  - `field.twist.angular.z`
  - `field.twist.linear.x`
  - `field.twist.linear.y`
  - `field.twist.linear.z`

**Nota bene**: Nel caso in cui sono presenti i dati dei valori desiderati e dei valori reali (es: `field.commanded` e `field.measured`) è stato calcolato un delta tra i due valori ed è stato considerato solo quest’ultimo valore.

I valori sono campionati a frequenze diverse, per questa motivazione si è effettuato un resample dei dati alla frequenza più alta presente nei sensori (pari a 20Hz) utilizzando al tecnica del **forward fill** per i dati mancanti.

## Estrazione features diagnostiche

Per ognuno dei diversi tipi di guasto (’`engine`’, ‘`ailerons`’, ‘`rudder`’, ‘`elevator`’) sono state calcolate le features diagnostiche nel tempo e in frequenza, il calcolo è stato eseguito su finestre di volo di durata 1s con overlap di 0.5s.

Le **feature nel tempo** sono:

- Mean
- Variance
- MAE (Mean Absolute Error)
- RMS (Root Mean Square)
- RMSE (Root Mean Square Error)

- 25° percentile
- 50° percentile
- 75° percentile
- Kurtosis
- Skewness
- Impulse Factor

Le **feature in frequenza**, che vengono calcolate dopo aver calcolato la trasformata di Fourier dei dati, sono:

- Peak Frequency
- Peak Power
- Band Power

- Mean
- 25° percentile
- 50° percentile
- 75° percentile

Le features nel tempo e in frequenza vengono salvate nei file ‘`failures/features/time_<fault>`' e ‘`failures/features/frequency_<fault>`', per poi effettuare il merge dei due file che viene salvato nel file ‘`failures/features/<fault>`'.

Al termine della fase di preprocessing otteniamo un unico dataframe effettuando la concatenazione dei file ‘`failures/features/<fault>`'. Il dataframe a sua volta viene salvato in un file CSV (’`failures/features/all.csv`'), che contiene le feature nel tempo e in frequenza di ogni tipo di classe, la tabella sottostante mostra il tipo di guasto e la rispettiva label.

| Tipo di guasto              | Label |
| --------------------------- | ----- |
| Engine Full Power Loss      | 1     |
| Rudder Stuck to Left        | 2     |
| Rudder Stuck to Right       | 3     |
| Elevator Stuck at Zero      | 4     |
| Left Aileron Stuck at Zero  | 5     |
| Right Aileron Stuck at Zero | 6     |
| Both Ailerons Stuck at Zero | 7     |
| Rudder & Aileron at Zero    | 8     |
| No Fault                    | 0     |

## Classificazione

La fase di classificazione inizia con la normalizzazione dei dati e con la selezione di un numero ridotte di colonne utilizzando il metodo `SelectKBest` di `sklearn` che utilizza la `f_regression` per selezionare le K colonne più significative del dataframe, nel nostro caso sono state selezionate 50 colonne.

Al termine di queste operazioni otteniamo un dataframe (salvato nel file ‘`failures/features/column50.csv`') suddiviso come mostrato nella tabella sottostante:

| Tipo di guasto              | Label | Numero di elementi |
| --------------------------- | ----- | ------------------ |
| Engine Full Power Loss      | 1     | 718                |
| Rudder Stuck to Left        | 2     | 18                 |
| Rudder Stuck to Right       | 3     | 64                 |
| Elevator Stuck at Zero      | 4     | 46                 |
| Left Aileron Stuck at Zero  | 5     | 252                |
| Right Aileron Stuck at Zero | 6     | 319                |
| Both Ailerons Stuck at Zero | 7     | 71                 |
| Rudder & Aileron at Zero    | 8     | 54                 |
| No Fault                    | 0     | 6721               |

Come facilmente osservabile il dataset è fortemente sbilanciato, ragione per la quale sono state effettuati svariate classificazioni utilizzando diversi metodi di bilanciamento, di classificatori e di dimensioni di training e test set, al fine di ottenere i migliori risultati possibili.

In linea generale, il dataset è stato diviso in training set e test set con percentuali 85%-15%, i classificatori utilizzati sono stati:

- Random Forest Classifier
- K-Neighbors Classifier
- Decision Tree Classifier
- Logistic Regression
- SVC
- MLP Classifier

Sono state effettuate classificazioni anche effettuando sia undersamplig che upsampling dei dati per ottenere diversi dataframe maggiormente bilanciati.

# Risultati

Di seguito vengo mostrati i risultati delle diverse classificazioni effettuate.

## Undersampling di tutte le classi

Inizialmente si è proceduto con la classificazione effettuando il bilanciamento mediante undersampling delle classi, portando tutte le classi a un numero di campioni pari a quello della classe meno popolosa ( guasto ‘`rudder_left`' pari a 18 campioni).

I risultati di accuracy ottenuti sono i seguenti:

```
Accuracy: 0.68 	 ---> RandomForestClassifier
Accuracy: 0.6 	 ---> DecisionTreeClassifier
Accuracy: 0.56 	 ---> KNeighborsClassifier
Accuracy: 0.44 	 ---> SVC
Accuracy: 0.44 	 ---> LogisticRegression
Accuracy: 0.48 	 ---> MLPClassifier
```

![Untitled](images/results/Untitled%201.png)

![Untitled](images/results/Untitled%202.png)

```
RandomForestClassifier Classification Report:
              precision    recall  f1-score   support

           0       1.00      0.67      0.80         6
           1       0.00      0.00      0.00         1
           2       0.80      1.00      0.89         4
           3       1.00      1.00      1.00         2
           4       0.40      1.00      0.57         2
           5       0.67      0.80      0.73         5
           6       0.00      0.00      0.00         1
           7       0.00      0.00      0.00         2
           8       1.00      0.50      0.67         2

    accuracy                           0.68        25
   macro avg       0.54      0.55      0.52        25
weighted avg       0.69      0.68      0.66        25

DecisionTreeClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.80      0.67      0.73         6
           1       0.00      0.00      0.00         1
           2       1.00      1.00      1.00         4
           3       1.00      1.00      1.00         2
           4       0.67      1.00      0.80         2
           5       0.50      0.60      0.55         5
           6       0.00      0.00      0.00         1
           7       0.00      0.00      0.00         2
           8       0.00      0.00      0.00         2

    accuracy                           0.60        25
   macro avg       0.44      0.47      0.45        25
weighted avg       0.59      0.60      0.59        25

KNeighborsClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.40      0.33      0.36         6
           1       0.00      0.00      0.00         1
           2       1.00      1.00      1.00         4
           3       1.00      1.00      1.00         2
           4       0.50      1.00      0.67         2
           5       1.00      0.40      0.57         5
           6       0.00      0.00      0.00         1
           7       0.00      0.00      0.00         2
           8       0.50      1.00      0.67         2

    accuracy                           0.56        25
   macro avg       0.49      0.53      0.47        25
weighted avg       0.62      0.56      0.55        25

SVC Classification Report:
              precision    recall  f1-score   support

           0       0.00      0.00      0.00         6
           1       0.50      1.00      0.67         1
           2       1.00      1.00      1.00         4
           3       1.00      1.00      1.00         2
           4       0.29      1.00      0.44         2
           5       0.67      0.40      0.50         5
           6       0.00      0.00      0.00         1
           7       0.00      0.00      0.00         2
           8       0.00      0.00      0.00         2

    accuracy                           0.44        25
   macro avg       0.38      0.49      0.40        25
weighted avg       0.42      0.44      0.40        25

LogisticRegression Classification Report:
              precision    recall  f1-score   support

           0       0.00      0.00      0.00         6
           1       0.00      0.00      0.00         1
           2       1.00      1.00      1.00         4
           3       1.00      1.00      1.00         2
           4       0.40      1.00      0.57         2
           5       0.67      0.40      0.50         5
           6       0.00      0.00      0.00         1
           7       0.00      0.00      0.00         2
           8       0.33      0.50      0.40         2

    accuracy                           0.44        25
   macro avg       0.38      0.43      0.39        25
weighted avg       0.43      0.44      0.42        25

MLPClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.25      0.17      0.20         6
           1       0.00      0.00      0.00         1
           2       1.00      1.00      1.00         4
           3       1.00      1.00      1.00         2
           4       0.50      0.50      0.50         2
           5       0.75      0.60      0.67         5
           6       0.00      0.00      0.00         1
           7       0.00      0.00      0.00         2
           8       0.50      0.50      0.50         2

    accuracy                           0.48        25
   macro avg       0.44      0.42      0.43        25
weighted avg       0.53      0.48      0.50        25
```

## Undersampling con solo le classi maggiormente rappresentate

Considerati i risultati non incoraggianti ottenuti, si è deciso di effettuare una classificazione prendendo in considerazione solo le 4 classi maggiormente rappresentate (`no_fault`, `engine`, `aileron_left`, `aileron_right`), sempre effettuando una sampling pari al numero dalla classe meno rappresentata (`aileron_left` con 252 campioni). I risultati ottenuti sono i seguenti:

```
Accuracy: 0.86 	 ---> RandomForestClassifier
Accuracy: 0.75 	 ---> DecisionTreeClassifier
Accuracy: 0.69 	 ---> KNeighborsClassifier
Accuracy: 0.54 	 ---> SVC
Accuracy: 0.47 	 ---> LogisticRegression
Accuracy: 0.62 	 ---> MLPClassifier
```

![Untitled](images/results/Untitled%203.png)

![Untitled](images/results/Untitled%204.png)

```
RandomForestClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.84      0.69      0.76        39
           1       0.76      0.91      0.83        35
           5       0.94      0.89      0.92        37
           6       0.88      0.93      0.90        41

    accuracy                           0.86       152
   macro avg       0.86      0.86      0.85       152
weighted avg       0.86      0.86      0.85       152

DecisionTreeClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.69      0.64      0.67        39
           1       0.69      0.83      0.75        35
           5       0.80      0.76      0.78        37
           6       0.82      0.78      0.80        41

    accuracy                           0.75       152
   macro avg       0.75      0.75      0.75       152
weighted avg       0.75      0.75      0.75       152

KNeighborsClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.70      0.54      0.61        39
           1       0.78      0.83      0.81        35
           5       0.58      0.76      0.66        37
           6       0.73      0.66      0.69        41

    accuracy                           0.69       152
   macro avg       0.70      0.70      0.69       152
weighted avg       0.70      0.69      0.69       152

SVC Classification Report:
              precision    recall  f1-score   support

           0       0.48      0.62      0.54        39
           1       0.62      0.74      0.68        35
           5       0.46      0.32      0.38        37
           6       0.59      0.49      0.53        41

    accuracy                           0.54       152
   macro avg       0.54      0.54      0.53       152
weighted avg       0.54      0.54      0.53       152

LogisticRegression Classification Report:
              precision    recall  f1-score   support

           0       0.46      0.56      0.51        39
           1       0.52      0.69      0.59        35
           5       0.43      0.41      0.42        37
           6       0.43      0.24      0.31        41

    accuracy                           0.47       152
   macro avg       0.46      0.47      0.46       152
weighted avg       0.46      0.47      0.45       152

MLPClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.66      0.59      0.62        39
           1       0.63      0.77      0.69        35
           5       0.54      0.41      0.46        37
           6       0.65      0.73      0.69        41

    accuracy                           0.62       152
   macro avg       0.62      0.62      0.62       152
weighted avg       0.62      0.62      0.62       152
```

## Nessun sampling

Abbiamo effettuato una classificazione lasciando il dataframe originale senza effettuare operazioni di sampling, I risultati ottenuti sono stati:

```
Accuracy: 0.9 	 ---> RandomForestClassifier
Accuracy: 0.89 	 ---> KNeighborsClassifier
Accuracy: 0.85 	 ---> DecisionTreeClassifier
```

![Untitled](images/results/Untitled%205.png)

![Untitled](images/results/Untitled%206.png)

```
RandomForestClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.91      0.98      0.94      1010
           1       0.87      0.75      0.81       101
           2       1.00      0.80      0.89         5
           3       1.00      1.00      1.00         8
           4       1.00      0.12      0.22         8
           5       0.67      0.26      0.38        38
           6       0.88      0.44      0.59        50
           7       1.00      0.44      0.62         9
           8       0.00      0.00      0.00        11

    accuracy                           0.90      1240
   macro avg       0.81      0.53      0.60      1240
weighted avg       0.89      0.90      0.89      1240

KNeighborsClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.91      0.96      0.94      1010
           1       0.74      0.74      0.74       101
           2       1.00      1.00      1.00         5
           3       1.00      0.88      0.93         8
           4       0.50      0.38      0.43         8
           5       0.42      0.21      0.28        38
           6       0.68      0.42      0.52        50
           7       1.00      0.67      0.80         9
           8       1.00      0.36      0.53        11

    accuracy                           0.89      1240
   macro avg       0.81      0.62      0.69      1240
weighted avg       0.87      0.89      0.88      1240

DecisionTreeClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.93      0.90      0.91      1010
           1       0.63      0.72      0.67       101
           2       0.83      1.00      0.91         5
           3       1.00      0.88      0.93         8
           4       0.43      0.38      0.40         8
           5       0.36      0.39      0.38        38
           6       0.44      0.56      0.49        50
           7       0.45      0.56      0.50         9
           8       0.50      0.55      0.52        11

    accuracy                           0.85      1240
   macro avg       0.62      0.66      0.64      1240
weighted avg       0.86      0.85      0.85      1240
```

## Oversampling dei dati e undersampling di `no_fault`

Un ulteriore tentativo di classificazione ci ha portato a creare un training set in cui ogni classe avesse lo stesso numero di campioni della classe `engine`, questo è stato possibile effetuando l’undersampling dei campioni `no_fault` e al tempo stesso l’oversampling delle altre classi tramite `SMOTE`. In questo caso i risultati ottenuti sono stati:

```
Accuracy: 0.83 	 ---> RandomForestClassifier
Accuracy: 0.71 	 ---> DecisionTreeClassifier
Accuracy: 0.7 	 ---> KNeighborsClassifier
Accuracy: 0.52 	 ---> SVC
Accuracy: 0.41 	 ---> LogisticRegression
Accuracy: 0.59 	 ---> MLPClassifier
```

![Untitled](images/results/Untitled%207.png)

![Untitled](images/results/Untitled%208.png)

```
RandomForestClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.88      0.70      0.78       110
           1       0.82      0.91      0.86       107
           3       1.00      0.92      0.96        12
           4       0.38      0.60      0.46         5
           5       0.87      0.91      0.89        45
           6       0.80      0.90      0.84        39
           7       0.90      0.82      0.86        11
           8       0.77      1.00      0.87        10

    accuracy                           0.83       339
   macro avg       0.80      0.84      0.81       339
weighted avg       0.84      0.83      0.83       339

DecisionTreeClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.70      0.60      0.65       110
           1       0.76      0.81      0.78       107
           3       1.00      0.92      0.96        12
           4       0.67      0.40      0.50         5
           5       0.69      0.73      0.71        45
           6       0.60      0.69      0.64        39
           7       0.67      0.73      0.70        11
           8       0.73      0.80      0.76        10

    accuracy                           0.71       339
   macro avg       0.73      0.71      0.71       339
weighted avg       0.72      0.71      0.71       339

KNeighborsClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.76      0.57      0.65       110
           1       0.87      0.78      0.82       107
           3       1.00      0.92      0.96        12
           4       0.27      0.60      0.37         5
           5       0.52      0.64      0.57        45
           6       0.64      0.72      0.67        39
           7       0.65      1.00      0.79        11
           8       0.41      0.90      0.56        10

    accuracy                           0.70       339
   macro avg       0.64      0.77      0.68       339
weighted avg       0.74      0.70      0.71       339

SVC Classification Report:
              precision    recall  f1-score   support

           0       0.71      0.42      0.53       110
           1       0.79      0.65      0.71       107
           3       0.92      0.92      0.92        12
           4       0.11      0.80      0.20         5
           5       0.37      0.33      0.35        45
           6       0.37      0.38      0.37        39
           7       0.34      0.91      0.50        11
           8       0.15      0.40      0.22        10

    accuracy                           0.52       339
   macro avg       0.47      0.60      0.47       339
weighted avg       0.62      0.52      0.54       339

LogisticRegression Classification Report:
              precision    recall  f1-score   support

           0       0.57      0.43      0.49       110
           1       0.71      0.51      0.59       107
           2       0.00      0.00      0.00         0
           3       0.77      0.83      0.80        12
           4       0.09      0.80      0.15         5
           5       0.24      0.22      0.23        45
           6       0.11      0.03      0.04        39
           7       0.21      0.91      0.34        11
           8       0.17      0.30      0.21        10

    accuracy                           0.41       339
   macro avg       0.32      0.45      0.32       339
weighted avg       0.49      0.41      0.43       339

MLPClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.69      0.47      0.56       110
           1       0.72      0.74      0.73       107
           3       1.00      0.92      0.96        12
           4       0.23      0.60      0.33         5
           5       0.41      0.42      0.42        45
           6       0.33      0.46      0.39        39
           7       0.73      1.00      0.85        11
           8       0.44      0.70      0.54        10

    accuracy                           0.59       339
   macro avg       0.57      0.66      0.60       339
weighted avg       0.62      0.59      0.60       339
```

## Oversampling dei dati

Un’ulteriore training è stato effettuato con l’oversampling dei dati grazie alla funzione `SMOTE`, al fine di portare tutte le classi allo stesso numero di campioni della classe `no_fault`.

```
Accuracy: 0.91 	 ---> RandomForestClassifier
Accuracy: 0.77 	 ---> KNeighborsClassifier
Accuracy: 0.84 	 ---> DecisionTreeClassifier
```

![Untitled](images/results/Untitled%209.png)

![Untitled](images/results/Untitled%2010.png)

```
RandomForestClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.97      0.93      0.95      1010
           1       0.69      0.94      0.80       101
           2       1.00      1.00      1.00         5
           3       1.00      1.00      1.00         8
           4       0.50      0.62      0.56         8
           5       0.59      0.68      0.63        38
           6       0.76      0.88      0.81        50
           7       0.89      0.89      0.89         9
           8       1.00      0.73      0.84        11

    accuracy                           0.91      1240
   macro avg       0.82      0.85      0.83      1240
weighted avg       0.93      0.91      0.92      1240

KNeighborsClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.97      0.75      0.85      1010
           1       0.58      0.86      0.70       101
           2       1.00      1.00      1.00         5
           3       0.80      1.00      0.89         8
           4       0.35      1.00      0.52         8
           5       0.16      0.55      0.25        38
           6       0.41      0.84      0.55        50
           7       0.53      0.89      0.67         9
           8       0.43      0.82      0.56        11

    accuracy                           0.77      1240
   macro avg       0.58      0.86      0.66      1240
weighted avg       0.88      0.77      0.80      1240

DecisionTreeClassifier Classification Report:
              precision    recall  f1-score   support

           0       0.96      0.86      0.91      1010
           1       0.59      0.77      0.67       101
           2       0.83      1.00      0.91         5
           3       0.89      1.00      0.94         8
           4       0.42      0.62      0.50         8
           5       0.31      0.58      0.40        38
           6       0.45      0.74      0.56        50
           7       0.45      0.56      0.50         9
           8       0.50      0.27      0.35        11

    accuracy                           0.84      1240
   macro avg       0.60      0.71      0.64      1240
weighted avg       0.88      0.84      0.85      1240
```

# Conclusioni

Lo scopo di questo progetto era l'esecuzione di una classificazione delle diverse classi di guasto presenti nel dataset AirLab Failure and Anomaly (ALFA). È stato preso in considerazione il dataset processed disponibile e, dopo un'opportuna analisi, sono stati selezionati gli attributi più rilevanti. Dopo aver uniformato i diversi dataframe sulla stessa frequenza di campionamento, sono state estratte le features nel tempo e in frequenza su finestre temporali di 1 secondo con overlap di 0.5 secondi. Una volta unite tutte le informazioni su un unico dataframe, è stato possibile procedere con la classificazione. A tal proposito, sono stati eseguiti diversi test con l'obiettivo di trovare le configurazioni migliori. L'esecuzione dell'undersampling di tutto il dataset in base alla classe meno numerosa non ha portato a risultati soddisfacenti, in quanto risulta esserci un evidente sbilanciamento tra le classi. Si è deciso, pertanto, di procedere con una classificazione prendendo in considerazione solo le classi con il maggior numero di elementi, bilanciando il dataset in base alla classe meno popolosa. In questo test, si nota un netto miglioramento delle performance.
Nonostante ciò, si è comunque deciso di procedere con un addestramento che comprendesse tutte le classi, cercando di risolvere il problema dello sbilanciamento.
Si è effettuata, quindi, una classificazione completa senza alcun tipo di bilanciamento. In questo test, sono stati ottenuti ottimi valori di accuracy e di precision.
Infine, è stata applicata la tecnica SMOTE di oversampling delle classi meno numerose effettuando due test: il primo differenzia dal secondo in quanto è stato effettuato un undersampling della classe No Fault. I risultati di entrambi test sono soddisfacenti.

In tutti i test effettuati, il Random Forest si è mostrato il classificatore migliore. Escludendo il primo, i restanti test hanno mostrato una buona classificazione.

Per analisi maggiormente approfondite si rimanda alla relazione del progetto presente nel repository.
