# paste.py 🐍

<a href="https://kuma.fosscu.org/status/pastepy" target="_blank"><img src="https://badgen.net/badge/status/paste.py/green?icon=lgtm" alt=""></a>

<hr>

paste.py 🐍 is a simple pastebin written in Python and powered by FastAPI.

## 🤔 Prerequisites

- `python3`
- `pdm`
- `docker` and Docker Compose plugin if you want to run the full local stack with containers

## 🐍 Python Version Support

This project is designed to work best with:

- **Python 3.11.3**

Check your Python version with:

```bash
python --version
```

### Installing Python 3.11.3 with `pyenv`

Managing Python versions is easiest with [pyenv](https://github.com/pyenv/pyenv).

If you do not have `pyenv`, install it using their [official guide](https://github.com/pyenv/pyenv).

Then install the recommended Python version:

```bash
pyenv install 3.11.3
```

> When you enter this project directory later, `pyenv` can automatically pick the version from the `.python-version` file.

## ⚙️ Environment Variables

The application reads configuration from environment variables and also supports loading them from a local `.env` file.

### Required variables

- `MINIO_CLIENT_LINK` — S3-compatible endpoint used by the app
- `MINIO_ACCESS_KEY` — object storage access key
- `MINIO_SECRET_KEY` — object storage secret key
- `MINIO_BUCKET_NAME` — object storage bucket name
- `BASE_URL` — public base URL for your deployment, used in API responses and homepage examples
- `SQLALCHEMY_DATABASE_URL` — database connection string

### Optional variables

- `SOURCE_CODE_URL` — source code link shown on the homepage  
  Default: `https://github.com/FOSS-Community/paste.py`

### Example `.env`

```bash
MINIO_CLIENT_LINK=http://127.0.0.1:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin123
MINIO_BUCKET_NAME=pastebucket
BASE_URL=http://127.0.0.1:8080
SOURCE_CODE_URL=https://github.com/FOSS-Community/paste.py
SQLALCHEMY_DATABASE_URL=postgresql://postgres:mytestpassword@127.0.0.1:5432/pastedb
```

## 📦 Setup

### Local setup with Docker

This repository includes a development Docker Compose stack under `dev/docker-compose.yml` with:

- PostgreSQL
- RustFS / S3-compatible object storage
- the `paste.py` application

#### Run the development stack

1. Clone the repository:

```bash
git clone https://github.com/FOSS-Community/paste.py.git
cd paste.py
```

2. Start the development stack:

```bash
docker compose -f dev/docker-compose.yml up --build
```

3. Open the app:

- Homepage: `http://127.0.0.1:8082`
- Web form: `http://127.0.0.1:8082/web`
- Health check: `http://127.0.0.1:8082/health`

> The Docker development stack publishes the app on port `8082`, so the compose file sets `BASE_URL=http://127.0.0.1:8082`.

#### Stop the development stack

```bash
docker compose -f dev/docker-compose.yml down
```

#### Run the published Docker image

You can also run the published image directly, but the app still requires a database and S3-compatible object storage to already be available.

Example:

```bash
docker run -d \
  -p 8080:8080 \
  --name pastepy \
  -e MINIO_CLIENT_LINK=http://host.docker.internal:9000 \
  -e MINIO_ACCESS_KEY=minioadmin \
  -e MINIO_SECRET_KEY=minioadmin123 \
  -e MINIO_BUCKET_NAME=pastebucket \
  -e BASE_URL=http://127.0.0.1:8080 \
  -e SOURCE_CODE_URL=https://github.com/FOSS-Community/paste.py \
  -e SQLALCHEMY_DATABASE_URL=postgresql://postgres:mytestpassword@host.docker.internal:5432/pastedb \
  mrsunglasses/pastepy
```

> `host.docker.internal` works well on macOS and Windows. On Linux, replace it with the actual host address or attach the container to a Docker network where your database and object storage are reachable by service name.

If you already have a `.env` file with the required variables, you can also use:

```bash
docker run -d \
  -p 8080:8080 \
  --name pastepy \
  --env-file .env \
  mrsunglasses/pastepy
```

### Local setup without Docker

#### Setting up the project with PDM

[PDM](https://pdm.fming.dev/latest/) is used for dependency management in this project.

1. Clone the repository:

```bash
git clone https://github.com/FOSS-Community/paste.py.git
cd paste.py
```

2. Install dependencies:

```bash
pdm install
```

3. Create a `.env` file and set the required variables.

4. Run the app:

```bash
pdm run start
```

For development with auto-reload:

```bash
pdm run dev
```

## 🧪 Development and Testing

### Install Git hooks

This project uses `pre-commit` hooks to help maintain code quality.

```bash
pre-commit install
```

### Run tests

```bash
pdm run test
```

### Run database migrations

```bash
pdm run migrate
```

### Test the running server

If you started the app locally without Docker on port `8080`:

```bash
curl http://127.0.0.1:8080/health
```

If you started the Docker development stack:

```bash
curl http://127.0.0.1:8082/health
```

## 🗒️ Usage

### Using the CLI

> `curl` is required to use the CLI examples below.

Replace `<BASE_URL>` with your deployment URL. This should match the value you set in `BASE_URL`.

- Paste a file named `file.txt`

```bash
curl -X POST -F "file=@file.txt" <BASE_URL>/file
```

- Paste from stdin

```bash
echo "Hello, world." | curl -X POST -F "file=@-" <BASE_URL>/file
```

- Delete an existing paste

```bash
curl -X DELETE <BASE_URL>/paste/<id>
```

### Using the web interface

Open:

```bash
<BASE_URL>/web
```

### API notes

- `POST <BASE_URL>/paste` — create a paste from raw body content
- `GET <BASE_URL>/paste/<id>` — retrieve a paste as plain text
- `DELETE <BASE_URL>/paste/<id>` — delete a paste

The homepage examples and returned paste URLs are generated using `BASE_URL`, so set it to the public URL you want users to see.

The homepage "Source Code" link is controlled by `SOURCE_CODE_URL`.

## 🤝 Contributing

> Important: please read the [Code of Conduct](CODE_OF_CONDUCT.md) and [Contributing Guidelines](CONTRIBUTING.md) before contributing to `paste.py`.

- Feel free to open an issue for clarifications, bug reports, or suggestions.

<hr>

For more API usage and shell examples, run the project locally and visit your configured homepage at `<BASE_URL>`.