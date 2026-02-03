# DevOps delavnica

Vsebina repozitorija:
- Quarkus backend 
- React frontent
- CI/CD cevovod:

## Naloge
### 1. Naloga
V mapi .github/workflows prilagodite datoteko workflow.yml tako, da bo izvedla gradnjo Quarkus backend (in hkrati tudi teste enot). Pred začetkom ne pozabite omogočiti GitHub Actions v repozitoriju na zavihku Actions.
Pri tem si lahko pomagate s spodnjim odsekom kode:
```
BE-build-test-package:
    name: BE Build & Test & Package
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: measdb
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 21
        uses: actions/setup-java@v3
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: maven

      - name: Build and test with Maven
        run: mvn -B package --file pom.xml -Dquarkus.datasource.reactive.url=vertx-reactive:postgresql://localhost:5432/measdb
        working-directory: backend
```

### 2. Naloga
Obstoječ workflow dopolnite tako, da se rezultat gradnje backend shrani kot artefakt. Poskrbite, da se naloga začne izvajati šele po koncu prejšnje (NAMIG: needs[ime prejšnjega job-a]).
Pri tem si lahko pomagate s spodnjim odsekom kode:
```
- name: Upload resulting Build package
  uses: actions/upload-artifact@v4
  with:
    name: Package
    path: backend/target
```

### 3. Naloga 
Obstoječ workflow dopolnite tako, da zgradite Docker sliko za backend in jo naložite na Dockerhub. Za to potrebujete račun na Dockerhub ter dodani skrivnosti DOCKER_HUB_USERNAME (username) in DOCKER_HUB_PASS (token) na GitHub. Poskrbite, da se naloga začne izvajati šele po koncu prejšnje.
Pri tem si lahko pomagate s spodnjim odsekom kode:
```
BE-docker-image-delivery:
    name: BE Docker Image Delivery
    runs-on: ubuntu-latest
    needs: [<TODO>]

    steps:
      - uses: actions/checkout@v3

      - uses: actions/download-artifact@v4
        with:
          name: Package
          path: backend/target/

      - name: Docker build
        run: docker build -t <vas dockerhub repo>:latest -f backend/src/main/docker/Dockerfile.jvm ./backend

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_PASS }}

      - name: Docker push
        run: docker push <vas dockerhub repo>:latest
```
