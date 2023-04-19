
FROM python:3.9

WORKDIR /app

RUN pip install poetry
COPY ./pyproject.toml .

RUN poetry config virtualenvs.create false && poetry install

COPY ./app.py ./app.py

CMD [ "python", "./app.py" ]
