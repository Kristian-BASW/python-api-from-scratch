# Python API fra bunden med FastAPI og Swagger

[English](README.md) | **Dansk**

I denne guide bygger du et API til en lille liste af entiteter. Du starter med ét endpoint og udvider det, så du kan oprette, hente, opdatere og slette entiteter. Til sidst afprøver du det hele i browseren med Swagger UI.

Du skal kunne skrive simple Python-funktioner og arbejde med lister og dictionaries. Brug Python 3.10 eller nyere, en editor og en terminal.

## 1. Forstå de vigtigste begreber

Et **API** gør det muligt for programmer at udveksle data. En klient sender en HTTP-request til en server, og serveren returnerer et response med en statuskode og typisk data i JSON-format.

Et **endpoint** er kombinationen af en HTTP-metode og en sti, eksempelvis `GET /<entity-path>`.

| Metode | Formål i vores API | Endpoint |
| --- | --- | --- |
| `GET` | Hent entiteter | `/<entity-path>` |
| `GET` | Hent én entitet | `/<entity-path>/{entity_id}` |
| `POST` | Opret en entitet | `/<entity-path>` |
| `PUT` | Erstat en entitets indhold | `/<entity-path>/{entity_id}` |
| `DELETE` | Slet en entitet | `/<entity-path>/{entity_id}` |

**FastAPI** er Python-frameworket, som håndterer requests. **Uvicorn** er serveren, der kører applikationen. FastAPI genererer en **OpenAPI-beskrivelse**, som **Swagger UI** bruger til at vise interaktiv dokumentation. Swagger UI følger med FastAPI, så du behøver ikke installere det separat. Se [FastAPIs introduktion](https://fastapi.tiangolo.com/tutorial/first-steps/).

## 2. Opret et virtuelt miljø

Åbn en terminal i denne projektmappe. Hvis du følger guiden uden at have hentet projektet, skal du først oprette og åbne en tom mappe.

Et virtuelt miljø holder projektets Python-pakker adskilt fra andre projekter.

**macOS og Linux:**

```bash
python3 --version
python3 -m venv .venv
source .venv/bin/activate
```

**Windows (PowerShell):**

```powershell
py --version
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Når miljøet er aktiveret, viser terminalen typisk `(.venv)`. Brug herefter `python` i kommandoerne på begge platforme. I VS Code skal du også vælge miljøet via **Python: Select Interpreter** i kommandopaletten.

Hvis PowerShell blokerer aktivering, kan du bruge miljøets Python direkte: Erstat `python` med `.\.venv\Scripts\python.exe` i de følgende kommandoer.

## 3. Installer FastAPI

```bash
python -m pip install "fastapi[standard]"
```

Pakken inkluderer blandt andet Uvicorn. Gem de installerede versioner, så miljøet kan genskabes:

```bash
python -m pip freeze > requirements.txt
```

På en anden computer kan du oprette et virtuelt miljø og installere pakkerne med `python -m pip install -r requirements.txt`.

## 4. Skriv dit første endpoint

Opret filen `main.py` i projektmappen:

```python
from fastapi import FastAPI

app = FastAPI(title="Python API", version="1.0.0")


@app.get("/")
def read_root():
    return {"message": "Mit første API virker!"}
```

`app` er din applikation. Dekoratoren `@app.get("/")` kobler et GET-request til stien `/` sammen med funktionen nedenunder. FastAPI omdanner funktionens dictionary til JSON.

Start serveren fra mappen med `main.py`:

```bash
python -m uvicorn main:app --reload
```

`main:app` betyder: Find objektet `app` i filen `main.py`. `--reload` genstarter serveren, når du gemmer ændringer, og er beregnet til lokal udvikling.

Lad terminalen køre, og åbn [http://127.0.0.1:8000](http://127.0.0.1:8000). Du bør se:

```json
{"message": "Mit første API virker!"}
```

Stop serveren med `Ctrl+C`, når du er færdig. Kommandoer, du vil køre imens, skal køres i en anden terminal med det virtuelle miljø aktiveret.

## 5. Afprøv dit API med Swagger UI

Åbn [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs), mens serveren kører.

1. Fold `GET /` ud.
2. Klik på **Try it out**.
3. Klik på **Execute**.
4. Find **Server response**. Statuskoden skal være `200`, og **Response body** skal indeholde beskeden fra før.

Swagger UI sender rigtige requests til din server. Under **Schemas** vises datamodeller, når vi tilføjer dem i næste trin. Den underliggende OpenAPI-beskrivelse findes på [/openapi.json](http://127.0.0.1:8000/openapi.json).

## 6. Udvid API'et

Prøv selv at lave nye endpoints med dekoratorerne:

`@app.post`

`@app.get`

`@app.put`

`@app.delete`

Prøv også at tilføje filtrering til dit GET-request.

## 7. Test hele forløbet i Swagger

Brug **Try it out** og **Execute** for hvert request. Kør trinene i rækkefølge uden at ændre Python-filen undervejs.

1. Kald `GET /<entity-path>`. På en frisk server får du `200` og en tom liste: `[]`. Lad query-parameteren `completed` være udeladt for at hente alle entiteter.
2. Kald `POST /<entity-path>` med denne request body:

   ```json
   {
     "property1": "Byg mit første API",
     "property2": false
   }
   ```

   Du får `201` og entiteten med et `id`. Den første entitet efter en genstart får id `1`. Brug det returnerede id i resten af trinene.

3. Kald `GET /<entity-path>/{entity_id}` med entitetens id. Du får `200` og den oprettede entitet.
4. Kald `PUT /<entity-path>/{entity_id}` med samme id og denne body:

   ```json
   {
     "property1": "Byg mit første API",
     "property2": true
   }
   ```

   Du får `200` og den opdaterede entitet.

5. Prøv nogle af de andre endpoints, du har oprettet.

| Statuskode | Betydning i øvelsen |
| --- | --- |
| `200 OK` | Requestet lykkedes |
| `201 Created` | Entiteten blev oprettet |
| `204 No Content` | Entiteten blev slettet; der er intet svarindhold |
| `404 Not Found` | Entiteten findes ikke |
| `422 Unprocessable Entity` | Input opfylder ikke datamodellen eller parametertypen |

## 8. Arbejd videre

Når du kan gennemføre testforløbet, kan du udvide API'et:

- Tilføj en valgfri egenskab til din entitet, og kontrollér den i Swagger.
- Afvis titler, der kun indeholder mellemrum. Den nuværende længdevalidering tillader dem.
- Gem entiteter i SQLite, så de overlever en genstart.
- Tilføj brugere og adgangskontrol, så hver bruger kun kan se og ændre egne entiteter.

Projektmappen vil efter guiden indeholde `main.py`, `requirements.txt`, README-filerne og `.venv/`. Det virtuelle miljø skal ikke med i Git; projektets `.gitignore` udelukker allerede `.venv`.
