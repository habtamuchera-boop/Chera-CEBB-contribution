# Chera CEBB Contributions
# Chera Habtamu — CEBB database and storage contribution

This repository documents Chera Habtamu's contribution to **Cyber Emergency Black Box (CEBB)**, a final project developed for the INSA Cyber Talent Center 5th Round Summer Camp 2026.

- **Name:** Chera Habtamu
- **CTC ID:** CTC-2240-26
- **Role:** Backend, with a focus on the database and storage layer
- **Main project:** [Cyber Emergency Black Box](https://github.com/shalomtesfaye22-design/Cyber-Emergency-Black-Box)

## Project overview

CEBB is a local, Windows-first endpoint monitoring and digital-forensics prototype. It collects selected security-related endpoint metadata, checks event sequences for suspicious activity and prepares incident timelines and evidence packages for review.

The project uses Python, FastAPI, SQLite and SQLAlchemy, with a local web dashboard. It was built for an academic demonstration. Its alerts are investigative leads, not proof that an attack has taken place.

## My contribution

My contribution focused on the database and storage side of the backend. The database needs to keep collected events in a form that the rest of the application can query during correlation and incident investigation.

### Database models

The application uses SQLAlchemy declarative models to define its database tables. The main models are:

- `EventModel` stores normalized endpoint events, including their timestamp, type, source, user, process, file or network details, severity and hash fields.
- `IncidentModel` stores the incidents created by detection rules, including the rule, severity, status, summary and evidence path.
- `IncidentEventModel` links an incident to the events associated with it. Its foreign keys connect records in the incidents and events tables.
- `SystemSnapshotModel` stores host information captured by the application.
- `SettingsModel` stores application settings as key-value records.

The event table has indexes on fields used for event queries, including timestamp, event type and severity. There is also a combined timestamp/event-type index.

Source: [backend/database/models.py](https://github.com/shalomtesfaye22-design/Cyber-Emergency-Black-Box/blob/main/Cyber%20Emergency%20Black%20Box/backend/database/models.py)

### SQLite and SQLAlchemy configuration

CEBB uses SQLite for its local database and SQLAlchemy's asynchronous engine and sessions to interact with it. The database engine applies several SQLite settings when a connection is opened:

```sql
PRAGMA journal_mode=WAL;
PRAGMA synchronous=NORMAL;
PRAGMA foreign_keys=ON;
PRAGMA busy_timeout=5000;
```

WAL mode allows readers and a writer to work with less blocking than SQLite's default journal mode. `synchronous=NORMAL` sets the durability/synchronization behavior used by this prototype. Foreign-key enforcement makes SQLite check the declared relationships, and the 5000 ms busy timeout lets a connection wait briefly when the database is locked.

The engine creates asynchronous database sessions for application operations. During initialization, SQLAlchemy creates the tables from the model metadata and the application seeds its default settings.

Source: [backend/database/engine.py](https://github.com/shalomtesfaye22-design/Cyber-Emergency-Black-Box/blob/main/Cyber%20Emergency%20Black%20Box/backend/database/engine.py)

### How events reach storage

When an event arrives, the event service normalizes it before inserting it into the database. It also retrieves the previous event hash and supplies it to the normalizer, which allows the new event to be linked to the preceding record. Once the event has been stored, the service can trigger the correlation engine.

This means the database is used throughout the investigation flow: collectors submit events, the application stores them, and the correlation engine queries them to look for suspicious sequences. Incident records can then be linked back to the events that caused a rule to fire.

Source: [backend/services/event_service.py](https://github.com/shalomtesfaye22-design/Cyber-Emergency-Black-Box/blob/main/Cyber%20Emergency%20Black%20Box/backend/services/event_service.py)

### Why the storage layer matters

A detection rule is only useful when the relevant events have been stored and can be queried in the right order. The database models keep event and incident information structured, while the relationship table allows an incident to refer to its supporting events. This is what lets the application build an incident timeline and prepare evidence for review.

Hash chaining and evidence manifests help detect some changes to stored event sequences and exported files. They do not prove that the original endpoint data was truthful or authentic.

## Presentation video

[Open the Google Drive folder for the contribution video](https://drive.google.com/drive/folders/1dBXsqwx5dvRvO61Jqu9jyknNn8chhAK1?usp=sharing)

## See also

For the complete application, installation instructions, system architecture, detection rules and test information, visit the [main Cyber Emergency Black Box repository](https://github.com/shalomtesfaye22-design/Cyber-Emergency-Black-Box).

---

This README describes the database and storage work in the context of the integrated team project. The main repository contains the current source code and complete project documentation.
