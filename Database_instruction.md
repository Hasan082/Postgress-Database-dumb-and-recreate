# ============================================================
# STEP 1: BACKUP DATABASE (as regular user, NOT root)
# ============================================================
pg_dump -U primeacademy -d primeacademydb -F c -f ~/primeacademydb_backup_$(date +%Y%m%d_%H%M%S).dump

# Verify backup was created
ls -lh ~/primeacademydb_backup_*.dump


# ============================================================
# STEP 2: DROP AND RECREATE DATABASE
# ============================================================
# Connect to PostgreSQL
psql -U primeacademy -d postgres

# Inside psql, run these commands:
DROP DATABASE primeacademydb;
CREATE DATABASE primeacademydb OWNER primeacademy;
\q


# ============================================================
# STEP 3: RUN FRESH MIGRATIONS
# ============================================================
cd /var/www/backend/api

# Run all migrations from scratch
python manage.py migrate


# ============================================================
# STEP 4: RESTORE DATA (IMPORTANT: This will restore all your old data)
# ============================================================
# Find your backup file first
ls -lh ~/primeacademydb_backup_*.dump

# Restore (replace YYYYMMDD_HHMMSS with your actual backup filename)
pg_restore -U primeacademy -d primeacademydb ~/primeacademydb_backup_YYYYMMDD_HHMMSS.dump


# ============================================================
# STEP 5: RESTART DJANGO SERVER
# ============================================================
# If using systemd/supervisor
sudo systemctl restart your-django-service

# Or if running manually
python manage.py runserver


# ============================================================
# STEP 6: VERIFY
# ============================================================
# Check tables exist
psql -U primeacademy -d primeacademydb
\dt api_expense*
\q

# Test Django admin
# Visit: http://45.85.250.92:8080/admin/api/expense/
