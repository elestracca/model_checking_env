# Model Checking Environment: Storm + Spot (Docker)

Ambiente Docker pronto all'uso per fare model checking probabilistico con [Storm](https://www.stormchecker.org/) / [stormpy](https://github.com/moves-rwth/stormpy) e traduzione LTL → automi con [Spot](https://spot.lre.epita.fr/), inclusa la GUI (matplotlib / PyQt / X11) per visualizzare grafici e automi.

## Servizi

L'ambiente è gestito tramite **Docker Compose** ([compose.yaml](compose.yaml)) e comprende tre servizi:

- **`spot`** — immagine con [Spot 2.11.6](https://spot.lre.epita.fr/) (LTL → automi), costruita da [spot/Dockerfile](spot/Dockerfile).
- **`storm`** — immagine Ubuntu con [Storm](https://github.com/moves-rwth/storm) + [stormpy](https://github.com/moves-rwth/stormpy) compilati da sorgente, definita in [storm/Dockerfile](storm/Dockerfile). È disponibile anche [storm/Dockerfile_stable](storm/Dockerfile_stable), che parte dall'immagine ufficiale `movesrwth/storm:stable` invece di compilare da sorgente (build più veloce, meno flessibilità sulla versione).
- **`merged_service`** — immagine finale che combina Storm e Spot (copia `/opt/spot-2.11.6` dall'immagine `spot_image` dentro `test_ubuntu`), definita in [merged/Dockerfile](merged/Dockerfile). È il servizio pensato per lanciare gli script che usano entrambi gli strumenti.

Un [.devcontainer/](.devcontainer/) è incluso per chi preferisce lavorare da VS Code con l'estensione [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).

## Prerequisiti

- [Docker](https://docs.docker.com/get-docker/) con il plugin Docker Compose.
- Per la GUI (matplotlib / PyQt / X11): un server X attivo sull'host e accesso a `/tmp/.X11-unix`. Su host Linux può essere necessario abilitare il display:
  ```sh
  xhost +local:docker
  ```

## Setup

1. Copia il file di esempio delle variabili d'ambiente:
   ```sh
   cp .env.example .env
   ```
   Il file `.env` viene letto automaticamente da `docker compose` e contiene le variabili usate dai servizi `storm` e `merged_service` per il rendering grafico (`DISPLAY`, `QT_X11_NO_MITSHM`, `XAUTHORITY`, `LIBGL_ALWAYS_SOFTWARE`, `XDG_RUNTIME_DIR`). Adatta i valori al tuo ambiente se necessario.

2. Se `~/.Xauthority` non esiste sull'host, verifica che il path sia corretto (in [compose.yaml](compose.yaml) è già referenziato come `${HOME}/.Xauthority`).

## Build delle immagini

Dalla cartella del progetto:
```sh
docker compose build
```
oppure costruire un singolo servizio:
```sh
docker compose build spot
docker compose build storm
docker compose build merged_service
```
> Nota: `merged_service` dipende dalle immagini `spot_image` e `test_ubuntu`, quindi `spot` e `storm` vanno costruiti **prima** di `merged_service`.

## Avviare il container di lavoro

Il servizio principale per eseguire gli script è `merged_service`. La cartella [storm_ws/](storm_ws/) viene montata in `/storm_ws` dentro il container ed è il punto in cui mettere i tuoi modelli PRISM e i tuoi script Python:
```sh
docker compose run --rm merged_service bash
```
Verrai droppato in una shell interattiva su `/storm_ws`. Le modifiche ai file sono visibili sia dall'host che dal container.

In alternativa è possibile entrare direttamente nei singoli servizi:
```sh
docker compose run --rm storm bash
docker compose run --rm spot bash
```

## Pulizia

```sh
docker compose down              # ferma i servizi
docker image rm storm_spot_merged test_ubuntu spot_image   # rimuove le immagini se vuoi rifare la build
```

