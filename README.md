# Digitalt Overlapsystem – Bostedet Slottet & Skoven

Digitalt vagtoverlap-system der erstatter et papir- og Excel-baseret overlapsskema på et bosted med dag-, aften- og nattevagter.

Udviklet som 3. semesters projekt på UCL Online Datamatiker, forår 2026.

## Teknisk stack

- **Frontend:** Blazor WebAssembly (.NET 10)
- **Backend:** ASP.NET Core API
- **Database:** Microsoft SQL Server 2022
- **ORM:** Entity Framework Core
- **Realtime:** SignalR
- **Containerisering:** Docker Compose
- **Tests:** MSTest + Moq + EF Core InMemory

## Arkitektur

Systemet følger Clean Architecture med 5 projekter:

```
OverlapSystem.Domain          -> Entities, enums, interfaces
OverlapSystem.Application     -> Services, DTOs, interfaces
OverlapSystem.Infrastructure  -> EF Core DbContext, repositories
OverlapSystem.API             -> ASP.NET Core controllers, SignalR hub
OverlapSystem.Client          -> Blazor WASM frontend
tests/OverlapSystem.Tests     -> 69 unit + integration tests
```

## Opsætning

### Forudsætninger

- Docker Desktop installeret (https://docker.com)
- Mindst 4 GB ledig RAM
- Mindst 20 GB ledig diskplads

### Trin 1 – Hent koden

```bash
git clone https://github.com/[INDSAET-USERNAME]/OverlapSystem.git
cd OverlapSystem
```

### Trin 2 – Start systemet

```bash
docker compose up -d --build
```

Første gang tager det 5-10 minutter at bygge alle containere.

### Trin 3 – Åbn systemet

Åbn en browser og gå til:

```
http://localhost:5001
```

### Trin 4 – Log ind

Forhåndsoprettede testbrugere:

| Email                  | Kodeord    | Rolle         |
|------------------------|------------|---------------|
| admin@slottet.dk       | Admin123!  | Administrator |
| anne@slottet.dk        | Staff1!    | ShiftLeader   |
| jesper@slottet.dk      | Staff1!    | Staff         |
| amir@slottet.dk        | Staff1!    | Staff         |
| ayman@slottet.dk       | Staff1!    | Staff         |
| hassan@slottet.dk      | Staff1!    | Staff         |
| lotte@slottet.dk       | Staff1!    | Staff         |
| mette@slottet.dk       | Staff1!    | Staff         |
| brian@slottet.dk       | Staff1!    | Staff         |
| sofie@slottet.dk       | Staff1!    | Staff         |

Skift kodeord ved første login i produktion.

## Brug af systemet

### Første gang

1. Vælg afdeling: **Slottet** eller **Skoven**
2. Vælg enhedstype: **Skærm** (kun visning) eller **Fælles PC** (login muligt)
3. Valget gemmes i browseren – ryd cache for at ændre

### Som personale

1. Log ind med email og kodeord
2. Tryk **Start vagt** og vælg vagttype (dag/aften/nat)
3. Skriv evt. dit arbejdstelefonnummer
4. Opdater borgerstatus, medicin, opgaver i løbet af vagten
5. Tryk **Afslut vagt** når du går hjem

### Som administrator

Gå til `/admin` for at få adgang til:
- Brugerstyring (opret, deaktiver, anonymiser)
- Borgerstyring (opret, rediger, slet)
- Vagtstatus (se aktive medarbejdere)
- Auditlog (alle ændringer i systemet)
- Rapport-eksport (PDF over hændelser og historik)

## Funktioner

### Borgerfelt
- Anonymt ID på skærmen (fx S-01)
- Trafiklysmodel: Rød / Gul / Grøn
- Medicintidspunkter med afkrydsning
- PN-medicin registrering
- 1-2 personale pr. borger (ansigtsskift midt i vagt)
- Habitusfelt med tidsstempel
- Opgaver med CarryOver til næste vagt
- Beskeder til næste vagthold
- Særlige hændelser med opfølgning

### Fælles rubrikker
- 7 arbejdstelefoner (41522-41529)
- 8 ansvarsopgaver: Medicinansvarlig, Omsorgsperson, Aftensmad, Hygiejne, Kaffe, Skraldespand, FMK, Afdelingsmobil

### Vagtoverlap
- To vagter kan være aktive samtidigt
- Forrige vagts data vises dæmpet ved siden af aktive vagt
- Farvekodning: Dag=blå, Aften=orange, Nat=lilla

### Historik
- Alle vagter gemmes med ændringslog
- Hver ændring viser: tidspunkt, handling, borger, medarbejder, gammel→ny værdi
- Administrator kan eksportere som PDF

## GDPR

- Fulde navne vises aldrig på 55" skærmen
- Alias og pårørendekontakt kun synlig ved email/kode-login
- Administrator kan anonymisere medarbejdere i auditlog
- Administrator kan slette borgerdata (GDPR-endpoint)
- Alle ændringer logges med medarbejder og tidspunkt

## Test

Kør alle 69 tests:

```bash
cd tests/OverlapSystem.Tests
dotnet test
```

| Test-fil                          | Antal | Type             |
|-----------------------------------|-------|------------------|
| ShiftServiceTests                 | 15    | Integration      |
| ResidentServiceTests              | 16    | Unit (Moq)       |
| ShiftOperationsServiceTests       | 8     | Integration      |
| ShiftEmployeeRepositoryTests      | 10    | Integration      |
| ResidentStatusRepositoryTests     | 7     | Integration      |
| RiskLevelTests                    | 8     | Domain unit      |
| ResidentEntityTests               | 5     | Domain unit      |

## Vedligeholdelse

### Opdater systemet
```bash
git pull
docker compose down
docker compose build --no-cache
docker compose up -d
```

### Backup af database
```bash
docker exec -t overlapsystem-db-1 /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U sa -P "$DB_PASSWORD" \
  -Q "BACKUP DATABASE OverlapSystem TO DISK = '/var/opt/mssql/backup.bak'"
```

### Stop systemet
```bash
docker compose down
```

### Slet alt (incl. data)
```bash
docker compose down -v
```

## Fejlsøgning

**Kan ikke logge ind:**
- Tjek at API'et kører: `docker compose logs api`
- Tjek at databasen er klar: `docker compose logs db`

**Skærmen viser ikke aktiv vagt:**
- Tjek at seeden har kørt: `docker compose logs api | grep -i seed`
- Genstart med `docker compose down -v && docker compose up -d`

**Realtime virker ikke:**
- Tjek at SignalR-forbindelsen er oprettet i browser-konsollen (F12)
- Tjek at WebSockets er enabled i miljøet

## Licens

Projektet er udviklet i undervisningsregi på UCL Online og er ikke licenseret til kommerciel brug.

## Bidragsydere

- 3. semester datamatiker-studerende, UCL Online, forår 2026
- Projektejer: Bostedet Slottet & Skoven (case)
- Vejleder: [INDSAET-NAVN]
