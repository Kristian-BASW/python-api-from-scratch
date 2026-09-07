# Python API from scratch with FastAPI and Swagger

**English** | [Dansk](README.da.md)

In this guide, you will build an API for a small task list. You will start with one endpoint and expand it to create, retrieve, update, and delete tasks. Finally, you will try everything in your browser using Swagger UI.

You should be comfortable writing simple Python functions and working with lists and dictionaries. Use Python 3.10 or newer, an editor, and a terminal.

## 1. Understand the key concepts

An **API** allows programs to exchange data. A client sends an HTTP request to a server, and the server returns a response with a status code and usually data in JSON format.

An **endpoint** is the combination of an HTTP method and a path, such as `GET /tasks`.

| Method | Purpose in our API | Endpoint |
| --- | --- | --- |
| `GET` | Retrieve tasks | `/tasks` |
| `GET` | Retrieve one task | `/tasks/{task_id}` |
| `POST` | Create a task | `/tasks` |
| `PUT` | Replace a task's contents | `/tasks/{task_id}` |
| `DELETE` | Delete a task | `/tasks/{task_id}` |

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

app = FastAPI(title="Task API", version="1.0.0")


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

## 6. Expand the API to manage tasks

Replace the **entire contents** of `main.py` with the code below. Read the models first, then the storage, and finally the endpoints from top to bottom.

```python
from itertools import count

from fastapi import FastAPI, HTTPException, Response, status
from pydantic import BaseModel, Field

app = FastAPI(title="Task API", version="1.0.0")


class TaskInput(BaseModel):
    title: str = Field(min_length=1, max_length=100)
    completed: bool = False


class Task(TaskInput):
    id: int


# Temporary storage in memory.
tasks: dict[int, Task] = {}
task_ids = count(1)


@app.get("/")
def read_root():
    return {"message": "My first API works!"}


@app.get("/tasks", response_model=list[Task])
def list_tasks(completed: bool | None = None):
    if completed is None:
        return list(tasks.values())
    return [task for task in tasks.values() if task.completed == completed]


@app.get("/tasks/{task_id}", response_model=Task)
def get_task(task_id: int):
    if task_id not in tasks:
        raise HTTPException(status_code=404, detail="Task not found")
    return tasks[task_id]


@app.post("/tasks", response_model=Task, status_code=status.HTTP_201_CREATED)
def create_task(task: TaskInput):
    new_task = Task(id=next(task_ids), **task.model_dump())
    tasks[new_task.id] = new_task
    return new_task


@app.put("/tasks/{task_id}", response_model=Task)
def update_task(task_id: int, task: TaskInput):
    if task_id not in tasks:
        raise HTTPException(status_code=404, detail="Task not found")
    updated_task = Task(id=task_id, **task.model_dump())
    tasks[task_id] = updated_task
    return updated_task


@app.delete("/tasks/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_task(task_id: int):
    if task_id not in tasks:
        raise HTTPException(status_code=404, detail="Task not found")
    del tasks[task_id]
    return Response(status_code=status.HTTP_204_NO_CONTENT)
```

Save the file and refresh the Swagger page. If the server has stopped, start it again using the same command as before.

Data is stored only in memory in a single server process. All tasks disappear when the server restarts, including when `--reload` restarts it after a code change. This keeps the example simple to explore; persistent storage requires something like a database.

### How the code fits together

`TaskInput` describes the request data: a title of 1–100 characters and a `completed` field that defaults to `False`. `Task` inherits these fields and adds an id chosen by the server. `model_dump()` converts the model to a dictionary, and `**` passes the dictionary's fields as keyword arguments. FastAPI uses Pydantic models to read and validate JSON data. See [request bodies and models](https://fastapi.tiangolo.com/tutorial/body/).

`response_model` describes which fields the API should return and makes the response format visible in Swagger. See [response models](https://fastapi.tiangolo.com/tutorial/response-model/).

`raise HTTPException(...)` ends the request with a specific error status and an explanation. See [error handling](https://fastapi.tiangolo.com/tutorial/handling-errors/).

There are three ways to send input to our API:

| Input | Example | Meaning |
| --- | --- | --- |
| Path parameter | `/tasks/1` | `task_id` identifies a specific task |
| Query parameter | `/tasks?completed=true` | Filters the list to completed tasks |
| Request body | `{"title": "Read about APIs", "completed": false}` | JSON data for `POST` or `PUT` |

Our `PUT` replaces the task's contents. If `completed` is omitted, its value becomes `false`, even if the task was previously completed.

## 7. Test the full workflow in Swagger

Use **Try it out** and **Execute** for each request. Follow the steps in order without changing the Python file along the way.

1. Call `GET /tasks`. On a fresh server, you get `200` and an empty list: `[]`. Leave the `completed` query parameter unset to retrieve all tasks.
2. Call `POST /tasks` with this request body:

   ```json
   {
     "title": "Build my first API",
     "completed": false
   }
   ```

   You get `201` and the task with an `id`. The first task after a restart gets id `1`. Use the returned id for the remaining steps.

3. Call `GET /tasks/{task_id}` with the task's id. You get `200` and the task you created.
4. Call `PUT /tasks/{task_id}` with the same id and this body:

   ```json
   {
     "title": "Build my first API",
     "completed": true
   }
   ```

   You get `200` and the updated task.

5. Call `GET /tasks` with `completed` set to `true`. The task should appear in the list. Then try `false`; the task should now be filtered out.
6. Call `DELETE /tasks/{task_id}` with the same id. You get `204` with no response body.
7. Retrieve the deleted task again. You get `404` with `{"detail": "Task not found"}`.
8. Try creating a task with `{"title": ""}`. You get `422` because the title is empty. Also try `GET /tasks/abc`: the id must be an integer, so this also returns `422`.

| Status code | Meaning in this exercise |
| --- | --- |
| `200 OK` | The request succeeded |
| `201 Created` | The task was created |
| `204 No Content` | The task was deleted; there is no response content |
| `404 Not Found` | The task does not exist |
| `422 Unprocessable Entity` | The input does not satisfy the data model or parameter type |

## 8. Troubleshooting

| Problem | Solution |
| --- | --- |
| `No module named fastapi` or `uvicorn` | Activate `.venv` and run the installation from step 3 again. |
| `Could not import module "main"` | Run the command from the directory containing `main.py` and check the filename. |
| Port 8000 is already in use | Stop the old server, or use `python -m uvicorn main:app --reload --port 8001` and open port 8001 in your browser. |
| The browser cannot connect | Check that the server is still running and look for errors in the terminal. |
| Swagger shows old endpoints | Save the file, check the restart in the terminal, and refresh your browser. |
| A request returns `422` | Read the error's `detail`: it identifies the field that failed validation, among other information. |
| My tasks have disappeared | Storage is reset on restart. Create the tasks again using `POST`. |

## 9. Keep building

Once you can complete the test workflow, you can extend the API:

- Add an optional description to a task and check it in Swagger.
- Reject titles containing only spaces. The current length validation allows them.
- Store tasks in SQLite so they survive a restart.
- Add users and access control so each user can only view and modify their own tasks.

After following the guide, the project directory will contain `main.py`, `requirements.txt`, the README files, and `.venv/`. Do not commit the virtual environment to Git; the project's `.gitignore` already excludes `.venv`.
