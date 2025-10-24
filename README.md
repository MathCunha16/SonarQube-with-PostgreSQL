# SonarQube with PostgreSQL (Docker Compose)

This repository provides an evergreen, persistent Docker Compose setup to run the **SonarQube Community Edition** with a dedicated **PostgreSQL** database.

This configuration is designed for developers who want a reliable, local SonarQube instance. It is "evergreen," meaning it uses floating tags (like `community` and `latest`) to pull the newest compatible images on first setup.

> **💡 Need Multi-Branch Analysis?**  
> If you need to analyze multiple branches and pull requests, check out the [SonarQube with Branch Plugin](https://github.com/MathCunha16/docker-compose-sonarqube-branch-plugin-with-postgres) version of this setup, which includes the Community Branch Plugin for free multi-branch support.

## Features

* **Stable Database:** Uses a dedicated PostgreSQL database, as recommended by SonarSource for persistent data storage.
* **Persistent Storage:** All data (database, SonarQube data, extensions, logs) is stored in named Docker volumes, so your data survives container restarts and image updates.
* **Isolated Networking:** Both services run on a custom bridge network, allowing them to communicate securely by name.

## Prerequisites

* [Docker](https://docs.docker.com/get-docker/)
* [Docker Compose](https://docs.docker.com/compose/install/)

---

## How to Use

1.  **Download:**
    Clone this repository or download the `docker-compose.yml` file into a new directory (for example, `~/docker/sonarqube`).

2.  **Start the Services:**
    From inside that directory, run:
    ```bash
    docker-compose up -d
    ```
    This will pull the latest images (if you don't have them) and start the containers.

3.  **Access SonarQube:**
    Wait 1-2 minutes for the server to initialize. You can check the logs with `docker-compose logs -f sonarqube`.

    Once it's running, access the dashboard in your browser at:
    **[http://localhost:9000](http://localhost:9000)**

4.  **Login:**
    * **Default Username:** `admin`
    * **Default Password:** `admin`
    (You will be prompted to change the password on first login).

---

## Customization & Versioning

This `docker-compose.yml` is designed to be a flexible template. Here is how you can customize it.

### 1. `sonarqube-db` (PostgreSQL Service)

* **`image: postgres:latest`**
    This file uses `postgres:latest` to always get the newest stable PostgreSQL version. If you prefer to lock into a specific major version, you can change this.
    * **Example:** `image: postgres:16` or `image: postgres:15`

* **`environment:`**
    You can change the `POSTGRES_USER`, `POSTGRES_PASSWORD`, or `POSTGRES_DB` values.
    **If you change these**, you **must** update the `SONAR_JDBC_...` variables in the `sonarqube` service to match.

* **`ports: - "5433:5432"`**
    This line is **optional**. It exposes the database to your host PC on port `5433`. This is only useful if you want to connect to the database with an external tool (like DBeaver or pgAdmin). SonarQube does *not* need this to function, so you can safely remove it for better security.

### 2. `sonarqube` (SonarQube Application)

* **`image: sonarqube:community`**
    This is the most important setting. The tag determines your SonarQube version.
    * **`sonarqube:community` (Recommended Default):** This tag points to the **latest Community Edition**. It is updated frequently and avoids the "version is no longer active" error.
    * **`sonarqube:lts-community`:** Use this tag if you specifically want the **Long-Term Support (LTS)** version, which is more stable and updated less frequently.
    * **`sonarqube:X.X.X-community`:** You can "pin" a specific version (for example, `sonarqube:25.10.0.114319-community`) if you need to guarantee a specific release.

* **`ports: - "9000:9000"`**
    If you already have a service on your host using port `9000`, you can change the *host* port (the left side).
    * **Example:** `ports: - "9001:9000"` will make SonarQube available at `http://localhost:9001`.

---

## Updating Your Stack

Because this setup uses floating tags, you can update your SonarQube and PostgreSQL images by running:

```bash
# 1. Pull the newest images defined in your compose file
docker-compose pull

# 2. Re-create the containers with the new images
docker-compose up -d
```
The new containers will start and connect to your existing data volumes.

## Troubleshooting

### "The version of SonarQube you are trying to upgrade from is too old"

This error can happen if you `docker-compose pull` after a long time. SonarQube cannot always jump from a very old version (like 9.x) directly to a very new one (like 11.x) while migrating the database.

**Fix (This will delete all your SonarQube analysis data):**
1.  Stop and remove all containers **AND** volumes.
    ```bash
    docker-compose down -v
    ```
2.  Start fresh. This will create a new, empty database for the new version.
    ```bash
    docker-compose up -d
    ```

---

## Related Projects

### SonarQube with Community Branch Plugin
If you need to analyze multiple branches and pull requests (features typically only available in commercial editions), check out this enhanced version:

**[docker-compose-sonarqube-branch-plugin-with-postgres](https://github.com/MathCunha16/docker-compose-sonarqube-branch-plugin-with-postgres)**

This setup includes the [SonarQube Community Branch Plugin](https://github.com/mc1arke/sonarqube-community-branch-plugin), which adds:
- ✅ Multi-branch analysis
- ✅ Pull request decoration
- ✅ Branch comparison
- ✅ CI/CD integration for branches

Perfect for teams working with feature branches and pull request workflows!

---

## Useful Links & Documentation

* **SonarQube Downloads:** [sonarsource.com/products/sonarqube/downloads/](https://www.sonarsource.com/products/sonarqube/downloads/)
    * (Good for seeing what the current LTS and Latest versions are)
* **SonarQube on Docker Hub:** [hub.docker.com/_/sonarqube](https://hub.docker.com/_/sonarqube)
    * **SonarQube Tags:** [hub.docker.com/_/sonarqube/tags](https://hub.docker.com/_/sonarqube/tags) (See all available versions)
* **PostgreSQL on Docker Hub:** [hub.docker.com/_/postgres](https://hub.docker.com/_/postgres)
    * **PostgreSQL Tags:** [hub.docker.com/_/postgres/tags](https://hub.docker.com/_/postgres/tags)
* **SonarQube Installation Docs (Docker):**
  *  [docs.sonarsource.com/sonarqube-server/latest/server-installation/from-docker-image](https://docs.sonarsource.com/sonarqube-server/latest/server-installation/from-docker-image/)
* **Docker Compose File Reference:** [docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)

## License

This project is licensed under the **MIT License**.
