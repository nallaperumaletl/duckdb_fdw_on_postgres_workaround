```bash
#!/bin/bash

# source:
#https://www.linkedin.com/pulse/loading-parquet-postgresql-via-duckdb-testing-queries-alvaro-huarte-uqcif


# Set PostgreSQL and DuckDB versions
POSTGRES_VERSION=16
DUCKDB_VERSION=1.0.0

# Create a log file
LOG_FILE="install_duckdb_fdw.log"
exec > >(tee -a "${LOG_FILE}") 2>&1

# Log the start of the script
echo "Starting duckdb_fdw installation: $(date)"

# Update and install dependencies
echo "Updating system and installing dependencies..."
sudo apt-get update
sudo apt-get install -y \
  build-essential \
  cmake \
  postgresql-${POSTGRES_VERSION} \
  postgresql-server-dev-${POSTGRES_VERSION} \
  postgresql-client-${POSTGRES_VERSION} \
  postgresql-contrib-${POSTGRES_VERSION}
    

# Clone duckdb_fdw repository
echo "Cloning duckdb_fdw repository..."
git clone https://github.com/alitrack/duckdb_fdw.git

# Navigate to the cloned repository
cd duckdb_fdw || exit

# Download and extract the DuckDB library
echo "Downloading and extracting DuckDB library..."
wget -c https://github.com/duckdb/duckdb/releases/download/v${DUCKDB_VERSION}/libduckdb-linux-amd64.zip
unzip -o -d . libduckdb-linux-amd64.zip

# Copy DuckDB shared library to PostgreSQL's lib directory
echo "Copying DuckDB shared library..."
sudo cp libduckdb.so $(pg_config --libdir)

# Build and install the duckdb_fdw extension
echo "Building and installing duckdb_fdw extension..."
make USE_PGXS=1
sudo make install USE_PGXS=1

# Set PostgreSQL environment variables
export POSTGRES_HOST_AUTH_METHOD='trust'

# Ensure the extension artifacts are correctly installed
echo "Copying extension artifacts..."
sudo cp duckdb_fdw.so /usr/lib/postgresql/${POSTGRES_VERSION}/lib/
sudo cp duckdb_fdw.control /usr/share/postgresql/${POSTGRES_VERSION}/extension/
sudo cp duckdb_fdw*.sql /usr/share/postgresql/${POSTGRES_VERSION}/extension/

# Log completion
echo "duckdb_fdw installation is complete: $(date)"
echo "Logs saved to ${LOG_FILE}"```

