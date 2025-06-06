## BloodHound-Legacy

### Deprecation Notice

This repository will be archived in the near future.

[BloodHound-Legacy | Github](https://github.com/SpecterOps/BloodHound-Legacy)

[BloodHound-Legacy | Docs](https://bloodhound.readthedocs.io/en/latest/index.html)

### Windows Install

#### Install Java11

Download the Windows installer for Oracle JDK 11 from https://www.oracle.com/java/technologies/javase-jdk11-downloads.html (needs an Oracle account).

#### Install neo4j

> Neo4j 5 suffers from severe performance regression issues. Until further notice, please use the latest Neo4j 4.4.x version

1. Download the latest neo4j 4.x Community Server Edition zip from https://neo4j.com/download-center

2. Setting System Environment Variables

   ```cmd
   NEO4J_CONF = path to neo4j
   NEO4J_CONF = path to neo4j
   ```

3. Run the following command

   ```
   neo4j.bat install-service
   net start neo4j
   ```

### Download the BloodHound GUI

   1. Download the latest version of the BloodHound GUI from https://github.com/BloodHoundAD/BloodHound/releases
   2. Unzip the folder and double click BloodHound.exe
   3. Authenticate with the credentials you set up for neo4j

### Alternative: Build the BloodHound GUI

   1. Install NodeJS from https://nodejs.org/en/download/
   2. Install electron-packager

   ```
   C:\> npm install -g electron-packager
   ```

   1. Clone the BloodHound GitHub repo:

   ```
   C:\> git clone https://github.com/BloodHoundAD/BloodHound
   ```

   1. From the root BloodHound directory, run npm install

   ```
   C:\> npm install
   ```

   1. Build BloodHound with npm run build:win32

   ```
   C:\> npm run build:win32
   ```

### SharpHound

SharpHound is the official data collector for BloodHound. It is written in C# and uses native Windows API functions and LDAP namespace functions to collect data from domain controllers and domain-joined Windows systems.

Download the pre-compiled SharpHound binary and PS1 version at https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors

You can view the source code for SharpHound and build it from source by visiting the SharpHound repo at https://github.com/BloodHoundAD/SharpHound

## BloodHound CE

[BloodHound | Github](https://github.com/SpecterOps/BloodHound)

[BloodHound | Support](https://bloodhound.specterops.io/home)

### Docker Compose

[BloodHound | Docker Compose](https://support.bloodhoundenterprise.io/hc/en-us/articles/17468450058267-Install-BloodHound-Community-Edition-with-Docker-Compose)

#### Download docker-compose.yml

#####  One-line command for Steps 1 & 2

```bash
curl -L https://ghst.ly/getbhce | docker compose -f - up
```

##### On Linux/Mac

```bash
curl -L https://ghst.ly/getbhce > docker-compose.yml
```

##### On Windows, from CMD

```cmd
curl -L https://ghst.ly/getbhce > docker-compose.yml
```

##### On Windows, from PowerShell

```
Invoke-WebRequest -Uri https://ghst.ly/getbhce -OutFile docker-compose.yaml
```

##### Build

```bash
docker compose pull && docker compose up .

docker compose up -d
docker compose logs
# Find Initial Password
docker compose logs | grep "Initial Password"
```

1. In a browser, navigate to http://localhost:8080/ui/login. Login with the username `admin` and the randomly generated password from the logs.

#### Change Network

Note: The default `docker-compose.yml` example binds only to localhost (127.0.0.1). If you want to access BHCE outside of localhost, you'll need to follow the instructions in examples/docker-compose/README.md to configure the host binding for the container.

[https://ghst.ly/getbhce | docker-compose.yml](https://raw.githubusercontent.com/SpecterOps/bloodhound/main/examples/docker-compose/docker-compose.yml)

```yaml
# Copyright 2023 Specter Ops, Inc.
#
# Licensed under the Apache License, Version 2.0
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# SPDX-License-Identifier: Apache-2.0

services:
  app-db:
    image: docker.io/library/postgres:16
    environment:
      - PGUSER=${POSTGRES_USER:-bloodhound}
      - POSTGRES_USER=${POSTGRES_USER:-bloodhound}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD:-bloodhoundcommunityedition}
      - POSTGRES_DB=${POSTGRES_DB:-bloodhound}
    # Database ports are disabled by default. Please change your database password to something secure before uncommenting
    # ports:
    #   - 127.0.0.1:${POSTGRES_PORT:-5432}:5432
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "pg_isready -U ${POSTGRES_USER:-bloodhound} -d ${POSTGRES_DB:-bloodhound} -h 127.0.0.1 -p 5432"
        ]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  graph-db:
    image: docker.io/library/neo4j:4.4.42
    environment:
      - NEO4J_AUTH=${NEO4J_USER:-neo4j}/${NEO4J_SECRET:-bloodhoundcommunityedition}
      - NEO4J_dbms_allow__upgrade=${NEO4J_ALLOW_UPGRADE:-true}
    # Database ports are disabled by default. Please change your database password to something secure before uncommenting
    ports:
      - 127.0.0.1:${NEO4J_DB_PORT:-7687}:7687
      - 127.0.0.1:${NEO4J_WEB_PORT:-7474}:7474
    volumes:
      - ${NEO4J_DATA_MOUNT:-neo4j-data}:/data
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "wget -O /dev/null -q http://localhost:7474 || exit 1"
        ]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  bloodhound:
    image: docker.io/specterops/bloodhound:${BLOODHOUND_TAG:-latest}
    environment:
      - bhe_disable_cypher_complexity_limit=${bhe_disable_cypher_complexity_limit:-false}
      - bhe_enable_cypher_mutations=${bhe_enable_cypher_mutations:-false}
      - bhe_graph_query_memory_limit=${bhe_graph_query_memory_limit:-2}
      - bhe_database_connection=user=${POSTGRES_USER:-bloodhound} password=${POSTGRES_PASSWORD:-bloodhoundcommunityedition} dbname=${POSTGRES_DB:-bloodhound} host=app-db
      - bhe_neo4j_connection=neo4j://${NEO4J_USER:-neo4j}:${NEO4J_SECRET:-bloodhoundcommunityedition}@graph-db:7687/
      - bhe_recreate_default_admin=${bhe_recreate_default_admin:-false}
      - bhe_graph_driver=${GRAPH_DRIVER:-neo4j}
      ### Add additional environment variables you wish to use here.
      ### For common configuration options that you might want to use environment variables for, see `.env.example`
      ### example: bhe_database_connection=${bhe_database_connection}
      ### The left side is the environment variable you're setting for bloodhound, the variable on the right in `${}`
      ### is the variable available outside of Docker
    ports:
      ### Default to localhost to prevent accidental publishing of the service to your outer networks
      ### These can be modified by your .env file or by setting the environment variables in your Docker host OS
      ### - ${BLOODHOUND_HOST:-127.0.0.1}:${BLOODHOUND_PORT:-8080}:8080
      ### Change to 0.0.0.0
      - ${BLOODHOUND_HOST:-0.0.0.0}:${BLOODHOUND_PORT:-8080}:8080
    ### Uncomment to use your own bloodhound.config.json to configure the application
    # volumes:
    #   - ./bloodhound.config.json:/bloodhound.config.json:ro
    depends_on:
      app-db:
        condition: service_healthy
      graph-db:
        condition: service_healthy

volumes:
  neo4j-data:
  postgres-data:
```