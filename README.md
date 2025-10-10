# Cat Facts 🐱

A lightweight multi-container application that combines cat facts with dog images for fun, random content displays.

## Overview

This application consists of two services:

- **Worker**: Fetches cat facts from the [Cat Facts API](https://catfact.ninja/fact) and random dog images from the [Dog CEO API](https://dog.ceo/api/breeds/image/random), storing them in a MySQL database every minute
- **Facts Frontend**: A Flask web interface that displays random cat fact and dog image combinations from the database

## Architecture

```
┌─────────────┐
│   Worker    │──► Fetches data every 60s
│  (Python)   │    from external APIs
└──────┬──────┘
       │
       ▼
  ┌─────────┐
  │  MySQL  │
  │Database │
  └────┬────┘
       │
       ▼
┌─────────────┐
│   Facts     │──► Displays random
│ Frontend    │    fact + image pairs
│  (Flask)    │
└─────────────┘
```

## Prerequisites

- Docker
- Docker Compose (recommended) or individual container orchestration
- MySQL database

## Environment Variables

Both services require the following MySQL connection parameters:

| Variable | Description | Example |
|----------|-------------|---------|
| `DB_HOST` | MySQL host address | `mysql` |
| `DB_USER` | Database username | `catfacts_user` |
| `DB_PASSWORD` | Database password | `secure_password` |
| `DB_NAME` | Database name | `catfacts_db` |

## Running the Application

### With Docker

**Build the images:**

```bash
# Build worker
cd worker
docker build -t catfacts-worker .

# Build facts frontend
cd ../facts
docker build -t catfacts-facts .
```

**Run the containers:**

```bash
# Start MySQL
docker run -d \
  --name catfacts-mysql \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=catfacts_db \
  -e MYSQL_USER=catfacts_user \
  -e MYSQL_PASSWORD=secure_password \
  mysql:latest

# Start worker
docker run -d \
  --name catfacts-worker \
  --link catfacts-mysql:mysql \
  -e DB_HOST=mysql \
  -e DB_USER=catfacts_user \
  -e DB_PASSWORD=secure_password \
  -e DB_NAME=catfacts_db \
  catfacts-worker

# Start facts frontend
docker run -d \
  --name catfacts-facts \
  --link catfacts-mysql:mysql \
  -e DB_HOST=mysql \
  -e DB_USER=catfacts_user \
  -e DB_PASSWORD=secure_password \
  -e DB_NAME=catfacts_db \
  -p 80:80 \
  catfacts-facts
```

Access the application at `http://localhost`

## Database Schema

The worker automatically creates the following table structure:

```sql
CREATE TABLE IF NOT EXISTS data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    fact LONGTEXT,
    image LONGTEXT
);
```

## How It Works

1. **Worker Service**:
   - Runs scheduled jobs every 60 seconds
   - Fetches a random cat fact from catfact.ninja
   - Fetches a random dog image URL from dog.ceo
   - Stores the pair in the MySQL database
   - Includes health checks for database connectivity and frontend availability

2. **Facts Frontend**:
   - Provides a web interface at the root path (`/`)
   - Queries the database for a random cat fact + dog image pair
   - Renders an HTML template displaying both

## API Dependencies

- [Cat Facts API](https://catfact.ninja/) - Provides random cat facts
- [Dog CEO API](https://dog.ceo/dog-api/) - Provides random dog images

## Tech Stack

- **Backend**: Python 3.12, Flask
- **Database**: MySQL
- **Containerization**: Docker
- **Dependencies**: 
  - `mysql-connector-python` for database operations
  - `requests` for API calls
  - `schedule` for job scheduling

## Development

To run locally without Docker:

1. Install dependencies:
```bash
pip install -r facts/app/requirements.txt
pip install -r worker/app/requirements.txt
```

2. Set environment variables for database connection

3. Run the worker:
```bash
python worker/app/worker.py
```

4. Run the facts frontend:
```bash
cd facts/app
flask run
```

## License

This project is open source and available under the standard MIT License.

## Contributing

Contributions are welcome! Feel free to submit issues or pull requests.