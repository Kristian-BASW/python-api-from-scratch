# Python API fra bunden med FastAPI og Swagger

**Dansk** | [English](README.md)

I denne guide bygger du et API til en lille opgaveliste. Du starter med ét endpoint og udvider det, så du kan oprette, hente, opdatere og slette opgaver. Til sidst afprøver du det hele i browseren med Swagger UI.

Du skal kunne skrive simple Python-funktioner og arbejde med lister og dictionaries. Brug Python 3.10 eller nyere, en editor og en terminal.

## 1. Forstå de vigtigste begreber

Et **API** gør det muligt for programmer at udveksle data. En klient sender en HTTP-request til en server, og serveren returnerer et response med en statuskode og typisk data i JSON-format.

Et **endpoint** er kombinationen af en HTTP-metode og en sti, eksempelvis `GET /tasks`.

| Metode | Formål i vores API | Endpoint |
| --- | --- | --- |
| `GET` | Hent opgaver | `/tasks` |
| `GET` | Hent én opgave | `/tasks/{task_id}` |
| `POST` | Opret en opgave | `/tasks` |
| `PUT` | Erstat en opgaves indhold | `/tasks/{task_id}` |
| `DELETE` | Slet en opgave | `/tasks/{task_id}` |

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

app = FastAPI(title="Opgave-API", version="1.0.0")


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

## 6. Udvid til et API med opgaver

Erstat **hele indholdet** af `main.py` med koden nedenfor. Læs først modellerne, derefter lageret og til sidst endpoints fra toppen og ned.

```python
from itertools import count

from fastapi import FastAPI, HTTPException, Response, status
from pydantic import BaseModel, Field

app = FastAPI(title="Opgave-API", version="1.0.0")


class TaskInput(BaseModel):
    title: str = Field(min_length=1, max_length=100)
    completed: bool = False


class Task(TaskInput):
    id: int


# Midlertidigt lager i hukommelsen.
tasks: dict[int, Task] = {}
task_ids = count(1)


@app.get("/")
def read_root():
    return {"message": "Mit første API virker!"}


@app.get("/tasks", response_model=list[Task])
def list_tasks(completed: bool | None = None):
    if completed is None:
        return list(tasks.values())
    return [task for task in tasks.values() if task.completed == completed]


@app.get("/tasks/{task_id}", response_model=Task)
def get_task(task_id: int):
    if task_id not in tasks:
        raise HTTPException(status_code=404, detail="Opgaven findes ikke")
    return tasks[task_id]


@app.post("/tasks", response_model=Task, status_code=status.HTTP_201_CREATED)
def create_task(task: TaskInput):
    new_task = Task(id=next(task_ids), **task.model_dump())
    tasks[new_task.id] = new_task
    return new_task


@app.put("/tasks/{task_id}", response_model=Task)
def update_task(task_id: int, task: TaskInput):
    if task_id not in tasks:
        raise HTTPException(status_code=404, detail="Opgaven findes ikke")
    updated_task = Task(id=task_id, **task.model_dump())
    tasks[task_id] = updated_task
    return updated_task


@app.delete("/tasks/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_task(task_id: int):
    if task_id not in tasks:
        raise HTTPException(status_code=404, detail="Opgaven findes ikke")
    del tasks[task_id]
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```

Gem filen, og opdater Swagger-siden. Hvis serveren er stoppet, starter du den igen med samme kommando som før.

Data ligger kun i hukommelsen i én serverproces. Alle opgaver forsvinder ved genstart, også når `--reload` genstarter serveren efter en kodeændring. Det gør eksemplet enkelt at undersøge; vedvarende lagring kræver eksempelvis en database.

### Sådan hænger koden sammen

`TaskInput` beskriver request-data: en titel på 1–100 tegn og feltet `completed`, som som standard er `False`. `Task` arver disse felter og tilføjer et id, som serveren vælger. `model_dump()` laver modellen om til en dictionary, og `**` sender dictionaryens felter videre som navngivne argumenter. FastAPI bruger Pydantic-modeller til at læse og validere JSON-data. Se [request body og modeller](https://fastapi.tiangolo.com/tutorial/body/).

`response_model` beskriver, hvilke felter API'et skal returnere, og gør svarformatet synligt i Swagger. Se [response-modeller](https://fastapi.tiangolo.com/tutorial/response-model/).

`raise HTTPException(...)` afslutter requestet med en bestemt fejlstatus og en forklaring. Se [fejlhåndtering](https://fastapi.tiangolo.com/tutorial/handling-errors/).

Der er tre måder at sende input til vores API:

| Input | Eksempel | Betydning |
| --- | --- | --- |
| Path-parameter | `/tasks/1` | `task_id` angiver en bestemt opgave |
| Query-parameter | `/tasks?completed=true` | Filtrerer listen til færdige opgaver |
| Request body | `{"title": "Læs om API'er", "completed": false}` | JSON-data til `POST` eller `PUT` |

Vores `PUT` erstatter opgavens indhold. Hvis `completed` udelades, bliver værdien derfor `false`, også hvis opgaven tidligere var færdig.

## 7. Test hele forløbet i Swagger

Brug **Try it out** og **Execute** for hvert request. Kør trinene i rækkefølge uden at ændre Python-filen undervejs.

1. Kald `GET /tasks`. På en frisk server får du `200` og en tom liste: `[]`. Lad query-parameteren `completed` være udeladt for at hente alle opgaver.
2. Kald `POST /tasks` med denne request body:

   ```json
   {
     "title": "Byg mit første API",
     "completed": false
   }
   ```

   Du får `201` og opgaven med et `id`. Den første opgave efter en genstart får id `1`. Brug det returnerede id i resten af trinene.

3. Kald `GET /tasks/{task_id}` med opgavens id. Du får `200` og den oprettede opgave.
4. Kald `PUT /tasks/{task_id}` med samme id og denne body:

   ```json
   {
     "title": "Byg mit første API",
     "completed": true
   }
   ```

   Du får `200` og den opdaterede opgave.

5. Kald `GET /tasks` med `completed` sat til `true`. Opgaven skal være med i listen. Prøv derefter `false`; opgaven skal nu være filtreret fra.
6. Kald `DELETE /tasks/{task_id}` med samme id. Du får `204` uden response body.
7. Hent den slettede opgave igen. Du får `404` med `{"detail": "Opgaven findes ikke"}`.
8. Prøv at oprette en opgave med `{"title": ""}`. Du får `422`, fordi titlen er tom. Prøv også `GET /tasks/abc`: id'et skal være et heltal, så det giver også `422`.

| Statuskode | Betydning i øvelsen |
| --- | --- |
| `200 OK` | Requestet lykkedes |
| `201 Created` | Opgaven blev oprettet |
| `204 No Content` | Opgaven blev slettet; der er intet svarindhold |
| `404 Not Found` | Opgaven findes ikke |
| `422 Unprocessable Entity` | Input opfylder ikke datamodellen eller parametertypen |

## 8. Fejlfinding

| Problem | Løsning |
| --- | --- |
| `No module named fastapi` eller `uvicorn` | Aktiver `.venv`, og kør installationen fra trin 3 igen. |
| `Could not import module "main"` | Kør kommandoen fra mappen med `main.py`, og kontrollér filnavnet. |
| Port 8000 er optaget | Stop den gamle server, eller brug `python -m uvicorn main:app --reload --port 8001` og åbn port 8001 i browseren. |
| Browseren kan ikke forbinde | Kontrollér, at serveren stadig kører, og se efter fejl i terminalen. |
| Swagger viser gamle endpoints | Gem filen, kontrollér genstarten i terminalen, og opdater browseren. |
| Et request giver `422` | Læs fejlens `detail`: den angiver blandt andet feltet, som ikke kunne valideres. |
| Mine opgaver er væk | Lageret nulstilles ved genstart. Opret opgaverne igen med `POST`. |

## 9. Arbejd videre

Når du kan gennemføre testforløbet, kan du udvide API'et:

- Tilføj en valgfri beskrivelse til en opgave, og kontrollér den i Swagger.
- Afvis titler, der kun indeholder mellemrum. Den nuværende længdevalidering tillader dem.
- Gem opgaver i SQLite, så de overlever en genstart.
- Tilføj brugere og adgangskontrol, så hver bruger kun kan se og ændre egne opgaver.

Projektmappen vil efter guiden indeholde `main.py`, `requirements.txt`, `README.md` og `.venv/`. Det virtuelle miljø skal ikke med i Git; projektets `.gitignore` udelukker allerede `.venv`.
