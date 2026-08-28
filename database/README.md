# Database

**Responsibility**: Persistent storage for missions, experiments, astronauts, events, and audit logs.

---

## Overview

PostgreSQL database for EKA. Stores:

- Mission metadata and timelines
- Experiment definitions and instances
- Astronaut and mission control users
- All system events (vision, activity, alerts)
- Immutable audit logs
- User sessions and authentication

**Technology**: PostgreSQL 12+  
**ORM**: SQLAlchemy  
**Migrations**: Alembic  

---

## Schema Overview

### Core Tables

**missions**
```sql
id, name, description, astronauts, start_time, end_time, status
```

**experiments**
```sql
id, definition_id, mission_id, astronaut_id, status, current_step, started_at
```

**astronauts**
```sql
id, name, email, password_hash, role
```

**events**
```sql
id, mission_id, type, timestamp, data, source_module
```

**audit_logs**
```sql
id, user_id, action, resource_id, input, output, timestamp, ip_address
```

---

## Setup

### Create Database
```bash
createdb eka_db
createuser eka_user --password
```

### Run Migrations
```bash
alembic upgrade head
```

### Seed Demo Data
```bash
python database/scripts/seed_demo.py
```

---

## Queries

### Recent Events
```sql
SELECT * FROM events 
WHERE mission_id = 'mission_001' 
ORDER BY timestamp DESC 
LIMIT 100;
```

### Experiment Progress
```sql
SELECT e.id, COUNT(s.id) as completed_steps
FROM experiments e
LEFT JOIN experiment_steps s ON s.experiment_id = e.id AND s.status = 'completed'
WHERE e.mission_id = 'mission_001'
GROUP BY e.id;
```

### Audit Trail
```sql
SELECT * FROM audit_logs
WHERE user_id = 'ast_001' AND timestamp > NOW() - INTERVAL '7 days'
ORDER BY timestamp DESC;
```

---

## Backups

```bash
# Backup
pg_dump eka_db > backup.sql

# Restore
psql eka_db < backup.sql
```

---

## Performance

- Indexes on frequently queried columns (mission_id, timestamp)
- Partitioning for large tables (events, audit_logs)
- Connection pooling via SQLAlchemy

---

**For detailed setup**: See `database/README.md` after implementation

**Status**: Ready for implementation
