# Evidence LAB03

## 1. Obiettivo compreso
Spiego con parole mie perché questo laboratorio prepara il progetto reale dei lab successivi.
Questo laboratorio ha lo scopo di introdurre una struttura reale di progetto basata su monorepo con frontend e backend separati ma gestiti insieme.
- Gestire più servizi in un unico repository
- Costruire immagini Docker indipendenti
- Automatizzare build e push tramite pipeline CI/CD

## 2. Struttura monorepo
Descrivo la struttura del repository FE/BE.
```bash
root/ 
├── frontend/ 
│       ├── app.py 
│       ├── requirements.txt 
│       └── Dockerfile 
│
└── backend/ 
        ├── app.py 
        ├── requirements.txt 
        └── Dockerfile
```
- Frontend: Espone endpoint per l'utente
- Backend: Contiene servizi API che gestisce la logica applicativa

## 3. Codice backend
Riporto il ruolo del backend e gli endpoint disponibili.
Il backend: applicazione flask che esponde endpoint REST per test e logica base.
Endpoint disponibili:
- /health -> verifica stato del servizio
- /work -> simula un elaborazione lato backend

## 4. Codice frontend
Riporto il ruolo del frontend e come chiama il backend.
Il frontend è anch'esso un applicazione flask che funge da "client" verso il backend.
Endpoint disponibili:
- /demo -> chiama il backend e restituisce la risposta
In questo modo si simula una vera architettura microservizi FE -> BE

## 5. Build locale
Descrivo come ho costruito localmente le due immagini.
Ho costruito le immagini Docker localmente con i seguenti comandi:

- docker build -t backend:local ./backend
- docker build -t frontend:local ./frontend

Questo mi ha permesso di verificare che:
- I Dockerfile sono corretti.
- Le dipendenze vengono installate correttamente

## 6. Test locale
Riporto gli output di:
- GET /health backend
1) {"service":"backend","status":"ok"}
- GET /work backend
1) {"message":"backend completed","processing_time":0.444,"service":"backend"}
---
- GET /health frontend
1) {"service":"frontend","status":"ok"}
- GET /demo frontend
1) {"backend_response":{"message":"backend completed","processing_time":0.779,"service":"backend"},"message":"frontend completed","service":"frontend"}

## 7. Push manuale iniziale
Descrivo i tag manuali pubblicati in ACR.

Ho effettuato il push manuale delle immagini su Azure Container Registry utilizzando:
- az acr login --name obsacrc830a3

- docker tag backend:local obsacrc830a3.azurecr.io/backend:v1
- docker push obsacrc830a3.azurecr.io/backend:v1

- docker tag frontend:local obsacrc830a3.azurecr.io/frontend:v1 
- docker push obsacrc830a3.azurecr.io/frontend:v1

## 8. Pipeline Azure DevOps
Descrivo le stage e i job della pipeline multi-image.
1) Validate Repository
- Verifica presenza file
- Controlla sintassi Pyhton

2) PublishImage
- Contiene due Job:
    - Build Backend:
        - build immagine backend
        - push su ACR
    - Build Frontend
        - build immagine frontend
        - push su ACR

PS: I JOB SONO STATI ESEGUITI I MODO SEQUENZIALE PER EVITARE ERRORI DI PARALLELISMO

## 9. Verifica finale ACR
Incollo l’output di:
- az acr repository show-tags --repository frontend
[
  "33",
  "lab03-manual"
]
- az acr repository show-tags --repository backend
[
  "33",
  "lab03-manual"
]

## 10. Problemi incontrati
Descrivo eventuali errori e come li ho risolti.
1) ERRORE: Bash su Windows agent

Errore:

/bin/bash: No such file or directory

1) RISOLTO: rimuovendo AzureCLI@2 con script bash e usando direttamente task bash.

---

2) ERRORE: parallelism Azure DevOps

Errore:

No parallel jobs available

2) RISOLTO: eseguendo i job in sequenza con dependsOn.