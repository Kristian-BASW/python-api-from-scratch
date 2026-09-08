# Python API from scratch with FastAPI and Swagger

**English** | [Dansk](README.da.md)

In this guide, you will build an API for a small entity list. You will start with one endpoint and expand it to create, retrieve, update, and delete entities. Finally, you will try everything in your browser using Swagger UI.

You should be comfortable writing simple Python functions and working with lists and dictionaries. Use Python 3.10 or newer, an editor, and a terminal.

## 1. Understand the key concepts

An **API** allows programs to exchange data. A client sends an HTTP request to a server, and the server returns a response with a status code and usually data in JSON format.

An **endpoint** is the combination of an HTTP method and a path, such as `GET /<entity-path>`.

| Method | Purpose in our API | Endpoint |
| --- | --- | --- |
| `GET` | Retrieve entities | `/<entity-path>` |
| `GET` | Retrieve one entity | `/<entity-path>/{entity_id}` |
| `POST` | Create a entity | `/<entity-path>` |
| `PUT` | Replace a entity's contents | `/<entity-path>/{entity_id}` |
| `DELETE` | Delete a entity | `/<entity-path>/{entity_id}` |

**FastAPI** is the Python framework that handles requests. **Uvicorn** is the server that runs the application. FastAPI generates an **OpenAPI description**, which **Swagger UI** uses to display interactive documentation. Swagger UI is included with FastAPI, so you do not need to install it separately. See [FastAPI's introduction](https://fastapi.tiangolo.com/tutorial/first-steps/).

## 2. Create a virtual environment

Open a terminal in this project directory. If you are following the guide without downloading the project, first create and open an empty directory.

A virtual environment keeps the project's Python packages separate from other projects.

**macOS and Linux:**

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

Once the environment is activated, the terminal usually displays `(.venv)`. From now on, use `python` in commands on both platforms. In VS Code, also select the environment using **Python: Select Interpreter** in the command palette.

If PowerShell blocks activation, you can use the environment's Python directly: replace `python` with `.\.venv\Scripts\python.exe` in the following commands.

## 3. Install FastAPI

```bash
python -m pip install "fastapi[standard]"
```

This package includes Uvicorn, among other tools. Save the installed versions so you can recreate the environment:

```bash
python -m pip freeze > requirements.txt
```

On another computer, you can create a virtual environment and install the packages using `python -m pip install -r requirements.txt`.

## 4. Write your first endpoint

Create a file named `main.py` in the project directory:

```python
from fastapi import FastAPI

app = FastAPI(title="Python API", version="1.0.0")


@app.get("/")
def read_root():
    return {"message": "My first API works!"}
```

`app` is your application. The `@app.get("/")` decorator connects a GET request to the path `/` with the function below it. FastAPI converts the function's dictionary to JSON.

Start the server from the directory containing `main.py`:

```bash
python -m uvicorn main:app --reload
```

`main:app` means: find the object `app` in the file `main.py`. `--reload` restarts the server when you save changes and is intended for local development.

Leave the terminal running and open [http://127.0.0.1:8000](http://127.0.0.1:8000). You should see:

```json
{"message": "My first API works!"}
```

Stop the server with `Ctrl+C` when you are done. To run other commands while the server is running, use another terminal with the virtual environment activated.

## 5. Try your API with Swagger UI

Open [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) while the server is running.

1. Expand `GET /`.
2. Click **Try it out**.
3. Click **Execute**.
4. Find **Server response**. The status code should be `200`, and **Response body** should contain the message from before.

Swagger UI sends real requests to your server. **Schemas** will display data models once we add them in the next step. The underlying OpenAPI description is available at [/openapi.json](http://127.0.0.1:8000/openapi.json).

## 6. Expand the API 

Try to make some new endpoints with the annotations

`@app.post`

`@app.get`

`@app.put`

`@app.delete`

Try to make some filtering on the get request aswell.


## 7. Test the full workflow in Swagger

Use **Try it out** and **Execute** for each request. Follow the steps in order without changing the Python file along the way.

1. Call `GET /<entity-path>`. On a fresh server, you get `200` and an empty list: `[]`. Leave the `completed` query parameter unset to retrieve all entities.
2. Call `POST /<entity-path>` with this request body:

   ```json
   {
     "property1": "Build my first API",
     "property2": false
   }
   ```

   You get `201` and the entity with an `id`. The first entity after a restart gets id `1`. Use the returned id for the remaining steps.

3. Call `GET /<entity-path>/{entity_id}` with the entities id. You get `200` and the entity you created.
4. Call `PUT /<entity-path>/{entity_id}` with the same id and this body:

   ```json
   {
     "property1": "Build my first API",
     "property2": true
   }
   ```

   You get `200` and the updated entity.

5. Try some of the other endpoints you have created


`422`.

| Status code | Meaning in this exercise |
| --- | --- |
| `200 OK` | The request succeeded |
| `201 Created` | The entity was created |
| `204 No Content` | The entity was deleted; there is no response content |
| `404 Not Found` | The entity does not exist |
| `422 Unprocessable Entity` | The input does not satisfy the data model or parameter type |

## 8. Keep building

Once you can complete the test workflow, you can extend the API:

- Add an optional property to your entity and check it in Swagger.
- Reject titles containing only spaces. The current length validation allows them.
- Store entities in SQLite so they survive a restart.
- Add users and access control so each user can only view and modify their own entities.

After following the guide, the project directory will contain `main.py`, `requirements.txt`, the README files, and `.venv/`. Do not commit the virtual environment to Git; the project's `.gitignore` already excludes `.venv`.
