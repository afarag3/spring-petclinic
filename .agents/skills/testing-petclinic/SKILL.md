---
name: testing-petclinic
description: Run and test the PetClinic application end-to-end locally. Use when verifying UI, JPA, validation, or API changes.
---

# Testing PetClinic Locally

## Prerequisites

- Java 21 (set `JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64` if needed)
- No external database required — app uses embedded HSQLDB by default
- No authentication or credentials needed

## Devin Secrets Needed

None. The app has no auth and uses an embedded database.

## Starting the App

```bash
cd /home/ubuntu/repos/spring-petclinic
JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64 ./mvnw spring-boot:run
```

- App starts on **port 8080** (http://localhost:8080)
- Startup takes ~3-5 seconds
- Look for `Started PetClinicApplication` in the logs to confirm it's ready
- Seeded data is loaded automatically from `src/main/resources/db/hsqldb/data.sql`

## Seeded Test Data

The embedded HSQLDB is populated with:
- **10 owners** (George Franklin, Betty Davis, Eduardo Rodriquez, Harold Davis, Peter McTavish, Jean Coleman, Jeff Black, Maria Escobito, David Schroeder, Carlos Estaban)
- **13 pets** across 6 types (cat, dog, lizard, snake, bird, hamster)
- **6 vets** with specialties: James Carter (none), Helen Leary (radiology), Linda Douglas (dentistry+surgery), Rafael Ortega (surgery), Henry Stevens (radiology), Sharon Jenkins (none)
- **4 visits** on pets 7 and 8

Useful for assertions: searching "Davis" returns 2 owners (Betty and Harold). Betty Davis (id=2) has pet Basil (hamster, 2012-08-06).

## Key UI Routes

| Route | What it tests |
|---|---|
| `/` | Homepage — Thymeleaf rendering, static resources |
| `/owners/find` | Owner search form |
| `/owners?lastName=Davis` | JPA query (`findByLastName`) |
| `/owners/{id}` | Owner detail with pets and visits (`@OneToMany`) |
| `/owners/new` | Owner creation with `jakarta.validation` (`@NotEmpty`) |
| `/owners/{id}/pets/new` | Pet creation with `PetTypeFormatter` |
| `/owners/{id}/pets/{petId}/visits/new` | Visit creation |
| `/vets.html` | Vet list — JPA `@ManyToMany` with specialties |
| `/vets.xml` | XML serialization (Jackson XML annotations) |
| `/vets.json` | JSON serialization |
| `/oups` | Error handling (throws RuntimeException) |

## Recommended Test Flow

1. **Homepage** — Verify renders without errors, nav bar has 4 links
2. **Veterinarians** — Verify 6 vets with correct specialties (proves JPA + seeded data)
3. **Vets XML** — Verify `/vets.xml` returns valid XML with `<vets>` root (proves serialization)
4. **Find Owner** — Search "Davis", verify 2 results, click through to Betty Davis details
5. **Create Owner** — Submit blank form first to verify validation errors ("must not be empty"), then fill and submit
6. **Add Pet** — Add a pet to the new owner, verify pet type dropdown loads all 6 types
7. **Add Visit** — Add a visit to the new pet, verify it appears in visit history

## What to Watch For

- **Validation messages**: Should say "must not be empty" (Jakarta Validation / Hibernate Validator 6+). If it says "may not be empty", the old javax.validation is still being used.
- **Vets XML endpoint**: If `/vets.xml` returns 406 Not Acceptable or an error, the Jackson XML annotations might be misconfigured. Check `@JacksonXmlRootElement` on `Vets.java` and `@JacksonXmlElementWrapper`/`@JacksonXmlProperty` on `Vet.java`.
- **Empty vet table**: If the vets page shows no data, `spring.sql.init` properties might be wrong or the schema/data SQL files aren't being loaded.
- **Pet type dropdown empty**: If the pet type dropdown has no options, `PetTypeFormatter` or the `PetType` JPA entity might have issues.
- **500 errors on form submission**: Likely a JPA entity mapping issue — check that all entities use `jakarta.persistence` annotations.

## Running Unit Tests

```bash
JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64 ./mvnw test
```

Expected: 41 tests pass, 1 skip (`CrashControllerTests` is `@Disabled`).

## Building

```bash
JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64 ./mvnw package -DskipTests
```
