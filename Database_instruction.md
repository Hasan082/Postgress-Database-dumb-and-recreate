# ============================================================
# STEP 1: BACKUP DATABASE (add -h localhost to use password auth)
# ============================================================
pg_dump -U primeacademy -h localhost -d primeacademydb -F c -f ~/primeacademydb_backup_$(date +%Y%m%d_%H%M%S).dump

# It will ask for password - enter the primeacademy user password
# Verify backup was created
ls -lh ~/primeacademydb_backup_*.dump


# ============================================================
# STEP 2: DROP AND RECREATE DATABASE
# ============================================================
psql -U primeacademy -h localhost -d postgres

# Inside psql, run these commands:
DROP DATABASE primeacademydb;
CREATE DATABASE primeacademydb OWNER primeacademy;
\q


# ============================================================
# STEP 3: RUN FRESH MIGRATIONS
# ============================================================
cd /var/www/backend/api

# Activate virtual environment if not already active
source venv/bin/activate

# Run all migrations from scratch
python manage.py migrate


# ============================================================
# STEP 4: RESTORE DATA
# ============================================================
# Find your backup file first
ls -lh ~/primeacademydb_backup_*.dump

# Restore (replace YYYYMMDD_HHMMSS with your actual backup filename)
pg_restore -U primeacademy -h localhost -d primeacademydb ~/primeacademydb_backup_YYYYMMDD_HHMMSS.dump


# ============================================================
# STEP 5: RESTART DJANGO SERVER
# ============================================================
# Find your service name first
systemctl list-units | grep django
# Or
systemctl list-units | grep gunicorn

# Then restart (replace with your actual service name)
sudo systemctl restart gunicorn
# Or whatever your service is called


# ============================================================
# STEP 6: VERIFY
# ============================================================
psql -U primeacademy -h localhost -d primeacademydb
\dt api_expense*
\q
