# Dependency-Track Quick Patch

Erzeugt ein gepatchtes [Dependency-Track](https://dependencytrack.org/) API-Server-Image, das gezielt einzelne Class-Dateien aus einem Build ersetzt – ohne Dependency-Track selbst neu zu bauen.

## Funktionsweise

Das `containerfile` unter `nb-patch/` baut in drei Stufen:

1. **dependencytrack** – zieht das offizielle `dependencytrack/apiserver:4.14.4` Image.
2. **patch** – kopiert das darin enthaltene `dependency-track-apiserver.jar` heraus und ersetzt mit `jar --update` Dateien aus `nb-patch/patch/` in der JAR. Die Pfade in `nb-patch/patch/` entsprechen der JAR-Struktur, z. B. `WEB-INF/classes/org/dependencytrack/...`.
3. **final** – übernimmt das ungepatchte Basis-Image und legt nur die gepatchte JAR wieder zurück (`/opt/owasp/dependency-track/dependency-track-apiserver.jar`).

## Patch hinzufügen

Class-Datei unter dem passenden Pfad in `nb-patch/patch/` ablegen, z. B.:

```
nb-patch/patch/WEB-INF/classes/org/dependencytrack/resources/v1/TeamResource.class
```

Anschließend das Image bauen (Beispiel mit Podman, Docker analog):

```bash
cd nb-patch
podman build -t test/dtrack:latest -f containerfile .
```

## Lokal starten

`docker-compose.yml` startet API-Server, Frontend und PostgreSQL:

```bash
docker compose up -d
```

- API-Server: `http://localhost:8081`
- Frontend:    `http://localhost:8080`
- PostgreSQL-Daten liegen in `./postgres-data`.

## Hinweise

- Nur einzelne Class-Patches gedacht, kein Ersatz für einen sauberen Build und kein Merge-Upstream.
- Basis-Image-Version im `containerfile` (`4.14.4`) muss zum eingesetzten Dependency-Track Release passen.
