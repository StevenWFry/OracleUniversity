# Lab Log – Database Initialization

## Environment
Host: imsdb1.rmc.ca  
OS: Oracle Linux 9  
DB Version: 19.28  
Container: CDAF_CDB  
PDB: CISQA  

## Steps
1. Initialized instance with DBCA silent mode:
   ```bash
   dbca -silent -createDatabase -gdbName CDAF_CDB -createAsContainerDatabase true
   ```

2. Verified creation:
   ```sql
   SELECT name, open_mode FROM v$pdbs;
   ```

## Results
| Step | Status | Notes |
|------|--------|--------|
| DBCA Create | ✅ | Successful |
| PDB Open | ✅ | Confirmed |
