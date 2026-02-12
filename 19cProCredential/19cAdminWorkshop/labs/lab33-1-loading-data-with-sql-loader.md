# Lab 33-1 - Loading Data with SQL*Loader (Flat Files Meet Reality)

Practice 2-1 loads SH data into `ORCLPDB1` using three SQL*Loader modes:

- Express
- Conventional
- Direct

---

## 1. Goal

You will:

1. load `products.dat` into `SH.PRODUCTS` (Express mode)
2. load `dp_inventories.dat` into `SH.INVENTORIES` (Conventional mode)
3. run the inventory load in Direct mode and compare behavior

Assumption:

- logged in as `oracle` OS user

---

## 2. Setup

1. Open terminal.
2. Set environment:

```bash
. oraenv
# accept default SID (orclcdb)
```

3. Start listener/database if needed:

```bash
dbstart.sh
```

4. Run lab setup:

```bash
dpsetup.sh
```

---

## 3. Express Mode (`SH.PRODUCTS`)

1. Inspect source file:

```bash
cat products.dat
```

2. Check starting row count:

```bash
sqlplus sh/cloud_4U@orclpdb1
SELECT COUNT(*) FROM products;
-- expected: 7
EXIT;
```

3. Run Express mode load:

```bash
sqlldr sh/cloud_4U@orclpdb1 table=products data=products.dat
```

Expected:

- `8 Rows successfully loaded`

4. Recheck:

```bash
sqlplus sh/cloud_4U@orclpdb1
SELECT * FROM products;
-- expected total: 15
EXIT;
```

5. Inspect logs:

```bash
cat products.log
ls -l products_*.log_xt
```

What happened behind the curtain:

- Express mode generated temporary external-table logic, loaded rows, then cleaned up.

---

## 4. Conventional Mode (`SH.INVENTORIES`) - `APPEND`

1. Baseline row count:

```bash
sqlplus sh/cloud_4U@orclpdb1
SELECT COUNT(*) FROM inventories;
-- expected baseline: 476
EXIT;
```

2. Load in conventional mode:

```bash
sqlldr sh/cloud_4U@orclpdb1 control=dp_inventories.ctl log=inventories.log data=dp_inventories.dat
```

Expected:

- `Path used: Conventional`
- `83 Rows successfully loaded`

3. Verify new count:

```bash
sqlplus sh/cloud_4U@orclpdb1
SELECT COUNT(*) FROM inventories;
-- expected: 559
EXIT;
```

4. Inspect log/control:

```bash
cat inventories.log
gedit dp_inventories.ctl
```

You should see `APPEND` behavior.

---

## 5. Conventional Mode with `TRUNCATE`

1. Change control file action from `APPEND` to `TRUNCATE`.
2. Rerun with row checkpoint parameter:

```bash
sqlldr sh/cloud_4U@orclpdb1 control=dp_inventories.ctl log=inventories.log data=dp_inventories.dat rows=10
```

3. Verify:

```bash
sqlplus sh/cloud_4U@orclpdb1
SELECT COUNT(*) FROM inventories;
-- expected: 83
EXIT;
```

Meaning:

- previous rows were cleared and replaced by the current load file set.

---

## 6. Constraint Scenario (Partial Load)

1. Run provided reset/check script:

```bash
dp_check.sql
```

2. Reload:

```bash
sqlldr sh/cloud_4U@orclpdb1 control=dp_inventories.ctl log=inventories.log data=dp_inventories.dat rows=10
```

Expected:

- `20 Rows successfully loaded`
- additional rows rejected due to constraint violations
- loader stops after exceeding default error tolerance (`50`)

3. Confirm:

```bash
cat inventories.log
```

---

## 7. Direct Mode

Run direct path load:

```bash
sqlldr sh/cloud_4U@orclpdb1 control=dp_inventories.ctl log=inventories.log data=dp_inventories.dat direct=true
```

Expected:

- `Path used: Direct`
- `83 Rows successfully loaded`

Observed behavior notes:

- enforces `PRIMARY KEY`, `UNIQUE`, `NOT NULL`
- does not behave like full conventional mode for all constraints/triggers
- log reflects direct-path save behavior rather than row-by-row commit semantics

---

## 8. Result

You validated mode differences in real output, not marketing slides:

- Express mode: fast and low-friction for aligned file/table loads
- Conventional mode: row-semantic behavior (`APPEND` vs `TRUNCATE` matters)
- Direct mode: faster path with different enforcement/runtime rules