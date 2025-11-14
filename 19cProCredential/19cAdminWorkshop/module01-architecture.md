# Module 1 – Database Architecture

## Concepts
- Oracle instance = background processes + memory structures
- Database = physical files (control, redo, data)
- Key components: SGA, PGA, PMON, SMON, DBWR, LGWR

## Commands & Examples
```sql
SHOW PARAMETER sga_target;
SHOW PARAMETER pga_aggregate_target;
```

## Notes
- Keep SGA and PGA sizing proportional to workload.
- Review `v$sga_dynamic_components` for actual allocations.
