TASK-A
========================

========== ANNAPURNA Q(a) EVIDENCE ==========
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab>
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> Write-Host "`n--- 1. DOCKER PLATFORM ---" -ForegroundColor Yellow

--- 1. DOCKER PLATFORM ---
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
NAMES                STATUS          PORTS
annapurna_postgres   Up 24 minutes   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp
annapurna_minio      Up 24 minutes   0.0.0.0:9000-9001->9000-9001/tcp, [::]:9000-9001->9000-9001/tcp
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab>
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> Write-Host "`n--- 2. MINIO OBJECT COUNT ---" -ForegroundColor Yellow

--- 2. MINIO OBJECT COUNT ---
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> docker run --rm --network container:annapurna_minio -v "$HOME\Desktop\annapurna-lab\mc-config:/root/.mc" quay.io/minio/mc find local/annapurna-sales/sales | Measure-Object


Count    : 4457
Average  :
Sum      :
Maximum  :
Minimum  :
Property :



PS C:\Users\ub02-glab-061\Desktop\annapurna-lab>
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> Write-Host "`n--- 3. S03 OCTOBER 2024 PARTITION ---" -ForegroundColor Yellow

--- 3. S03 OCTOBER 2024 PARTITION ---
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> Get-ChildItem "$HOME\Desktop\annapurna-lab\landing\sales\store=S03\year=2024\month=10" -File | Measure-Object -Property Length -Sum


Count    : 31
Average  :
Sum      : 872458
Maximum  :
Minimum  :
Property : Length



PS C:\Users\ub02-glab-061\Desktop\annapurna-lab>
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> Write-Host "`n--- 4. ORIGINAL FLAT SALES FOLDER ---" -ForegroundColor Yellow

--- 4. ORIGINAL FLAT SALES FOLDER ---
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> Get-ChildItem "$HOME\Downloads\data_2\data\sales" -File | Measure-Object -Property Length -Sum


Count    : 4457
Average  :
Sum      : 68706877
Maximum  :
Minimum  :
Property : Length



PS C:\Users\ub02-glab-061\Desktop\annapurna-lab>
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> Write-Host "`n--- 5. POSTGRESQL MASTER DATA ---" -ForegroundColor Yellow

--- 5. POSTGRESQL MASTER DATA ---
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> docker exec annapurna_postgres psql -U admin -d salesdb -c "SELECT 'stores' AS table_name, COUNT(*) AS rows FROM stores UNION ALL SELECT 'product_categories', COUNT(*) FROM product_categories UNION ALL SELECT 'products', COUNT(*) FROM products UNION ALL SELECT 'price_revisions', COUNT(*) FROM price_revisions;"
     table_name     | rows
--------------------+------
 stores             |   12
 product_categories |   14
 products           | 1224
 price_revisions    | 4320
(4 rows)

PS C:\Users\ub02-glab-061\Desktop\annapurna-lab>
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> Write-Host "`n--- 6. DUCKDB -> MINIO TEST ---" -ForegroundColor Yellow

--- 6. DUCKDB -> MINIO TEST ---
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> duckdb -c "INSTALL httpfs; SET s3_endpoint='localhost:9000'; SET s3_access_key_id='minioadmin'; SET s3_secret_access_key='minioadmin123'; SET s3_use_ssl=false; SET s3_url_style='path'; SELECT * FROM read_csv_auto('s3://annapurna-sales/sales/sales/store=S03/year=2024/month=10/SALES_S03_20241001.csv') LIMIT 5;"
┌────────────────────┬─────────┬──────────────┬───────┬───┬───────┬─────────┬───────┐
│      bill_no       │ line_no │ product_code │  qty  │ … │ month │  store  │ year  │
│      varchar       │  int64  │   varchar    │ int64 │ … │ int64 │ varchar │ int64 │
├────────────────────┼─────────┼──────────────┼───────┼───┼───────┼─────────┼───────┤
│ S03/20241001/00001 │       1 │ P107996      │     1 │ … │    10 │ S03     │  2024 │
│ S03/20241001/00001 │       2 │ P104334      │     2 │ … │    10 │ S03     │  2024 │
│ S03/20241001/00001 │       3 │ DISC         │     1 │ … │    10 │ S03     │  2024 │
│ S03/20241001/00001 │       4 │ TAX          │     1 │ … │    10 │ S03     │  2024 │
│ S03/20241001/00001 │       5 │ TENDER       │     1 │ … │    10 │ S03     │  2024 │
└────────────────────┴─────────┴──────────────┴───────┴───┴───────┴─────────┴───────┘
  5 rows            use .last to show entire result            10 columns (7 shown)
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab>
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> Write-Host "`n========== Q(a) EVIDENCE COMPLETE ==========" -ForegroundColor Green



=========================================================================================================================================

TASK B

========== Q(a) EVIDENCE COMPLETE ==========
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> ^C
PS C:\Users\ub02-glab-061\Desktop\annapurna-lab> docker exec -it annapurna_postgres psql -U admin -d salesdb
psql (16.15 (Debian 16.15-1.pgdg13+2))
Type "help" for help.

salesdb=# CREATE TABLE IF NOT EXISTS ingestion_manifest (
    source_file TEXT PRIMARY KEY,
    store_id TEXT NOT NULL,
    business_date DATE NOT NULL,
    status TEXT NOT NULL,
    loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS sales_lines (
    bill_no TEXT NOT NULL,
    line_no INTEGER NOT NULL,
    store_id TEXT NOT NULL,
    business_date DATE NOT NULL,
    product_code TEXT,
    qty INTEGER,
    unit_price NUMERIC(12,2),
    line_type TEXT,
    PRIMARY KEY (bill_no, line_no)
);
CREATE TABLE
CREATE TABLE
salesdb=# INSERT INTO ingestion_manifest
(source_file, store_id, business_date, status)
VALUES
('SALES_S03_20241001.csv', 'S03', '2024-10-01', 'LOADED')
ON CONFLICT (source_file) DO NOTHING;
INSERT 0 1
salesdb=# INSERT INTO ingestion_manifest
(source_file, store_id, business_date, status)
VALUES
('SALES_S03_20241001.csv', 'S03', '2024-10-01', 'LOADED')
ON CONFLICT (source_file) DO NOTHING;
INSERT 0 0
salesdb=# SELECT source_file, store_id, business_date, status, COUNT(*) OVER () AS total_records
FROM ingestion_manifest;
      source_file       | store_id | business_date | status | total_records
------------------------+----------+---------------+--------+---------------
 SALES_S03_20241001.csv | S03      | 2024-10-01    | LOADED |             1
(1 row)

DROP TABLE
CREATE TABLE
NOTICE:  RUN 1 | row_count = 1 | checksum = 21d3e51db8b527b65e5aea0ca8d2a80f
NOTICE:  RUN 2 | row_count = 1 | checksum = 21d3e51db8b527b65e5aea0ca8d2a80f
NOTICE:  RUN 3 | row_count = 1 | checksum = 21d3e51db8b527b65e5aea0ca8d2a80f
DO
 final_row_count |          final_checksum
-----------------+----------------------------------
               1 | 21d3e51db8b527b65e5aea0ca8d2a80f
(1 row)

=========================================================================================================================================

TASK C

 D SELECT
             s.bill_no,
             s.line_no,
             s.product_code,
             s.qty,
             s.unit_price,
             s.line_type,
             p.product_sk,
             p.product_name,
             p.category_id,
             pr.selling_price AS historical_selling_price
         FROM read_csv_auto(
             's3://annapurna-sales/sales/sales/store=S03/year=2024/month=10/SALES_S03_20241001.csv'
         ) s
         LEFT JOIN pg.public.products p
             ON p.product_code = s.product_code
            AND DATE '2024-10-01' BETWEEN p.valid_from AND p.valid_to
         LEFT JOIN pg.public.price_revisions pr
             ON pr.product_sk = p.product_sk
            AND DATE '2024-10-01' BETWEEN pr.effective_from AND pr.effective_to
         WHERE s.line_type = 'SALE'
         LIMIT 10;
┌────────────────────┬─────────┬──────────────┬───┬─────────────┬──────────────────────────┐
│      bill_no       │ line_no │ product_code │ … │ category_id │ historical_selling_price │
│      varchar       │  int64  │   varchar    │ … │   varchar   │      decimal(12,2)       │
├────────────────────┼─────────┼──────────────┼───┼─────────────┼──────────────────────────┤
│ S03/20241001/00004 │       7 │ P100033      │ … │ C05         │                   828.62 │
│ S03/20241001/00034 │       6 │ P100074      │ … │ C06         │                    57.78 │
│ S03/20241001/00008 │       2 │ P100077      │ … │ C06         │                   173.95 │
│ S03/20241001/00026 │       4 │ P100121      │ … │ C11         │                   107.67 │
│ S03/20241001/00020 │       6 │ P100183      │ … │ C09         │                  1493.10 │
│ S03/20241001/00020 │       7 │ P100194      │ … │ C13         │                    46.79 │
│ S03/20241001/00003 │       2 │ P100203      │ … │ C06         │                   272.10 │
│ S03/20241001/00012 │       4 │ P100210      │ … │ C09         │                  1073.90 │
│ S03/20241001/00032 │       3 │ P100221      │ … │ C09         │                   139.88 │
│ S03/20241001/00023 │       1 │ P100255      │ … │ C05         │                  1250.31 │
└────────────────────┴─────────┴──────────────┴───┴─────────────┴──────────────────────────┘
  10 rows               use .last to show entire result               10 columns (5 shown)
memory D SELECT '1. DIM_STORE' AS section;
┌──────────────┐
│   section    │
│   varchar    │
├──────────────┤
│ 1. DIM_STORE │
└──────────────┘
memory D SELECT COUNT(*) AS rows_in_dim_store
         FROM pg.public.dim_store;
┌───────────────────┐
│ rows_in_dim_store │
│       int64       │
├───────────────────┤
│                12 │
└───────────────────┘
memory D
memory D SELECT '2. DIM_PRODUCT' AS section;
┌────────────────┐
│    section     │
│    varchar     │
├────────────────┤
│ 2. DIM_PRODUCT │
└────────────────┘
memory D SELECT COUNT(*) AS rows_in_dim_product
         FROM pg.public.dim_product;
┌─────────────────────┐
│ rows_in_dim_product │
│        int64        │
├─────────────────────┤
│                1224 │
└─────────────────────┘
memory D
memory D SELECT '3. DIM_DATE' AS section;
┌─────────────┐
│   section   │
│   varchar   │
├─────────────┤
│ 3. DIM_DATE │
└─────────────┘
memory D SELECT COUNT(*) AS rows_in_dim_date
         FROM pg.public.dim_date;
┌──────────────────┐
│ rows_in_dim_date │
│      int64       │
├──────────────────┤
│              366 │
└──────────────────┘
memory D
memory D SELECT '4. FACT_SALES TABLE' AS section;
┌─────────────────────┐
│       section       │
│       varchar       │
├─────────────────────┤
│ 4. FACT_SALES TABLE │
└─────────────────────┘
memory D SELECT COUNT(*) AS rows_in_fact_sales
         FROM pg.public.fact_sales;
┌────────────────────┐
│ rows_in_fact_sales │
│       int64        │
├────────────────────┤
│                  0 │
└────────────────────┘
memory D
memory D SELECT '5. PRODUCTS MASTER' AS section;
┌────────────────────┐
│      section       │
│      varchar       │
├────────────────────┤
│ 5. PRODUCTS MASTER │
└────────────────────┘
memory D SELECT COUNT(*) AS products
         FROM pg.public.products;
┌──────────┐
│ products │
│  int64   │
├──────────┤
│     1224 │
└──────────┘
memory D
memory D SELECT '6. PRICE REVISIONS' AS section;
┌────────────────────┐
│      section       │
│      varchar       │
├────────────────────┤
│ 6. PRICE REVISIONS │
└────────────────────┘
memory D SELECT COUNT(*) AS price_revisions
         FROM pg.public.price_revisions;
┌─────────────────┐
│ price_revisions │
│      int64      │
├─────────────────┤
│            4320 │
└─────────────────┘
memory D
memory D SELECT '7. HISTORICAL PRICE + CROSS-SYSTEM QUERY' AS section;
┌──────────────────────────────────────────┐
│                 section                  │
│                 varchar                  │
├──────────────────────────────────────────┤
│ 7. HISTORICAL PRICE + CROSS-SYSTEM QUERY │
└──────────────────────────────────────────┘
memory D SELECT
             s.bill_no,
             s.line_no,
             s.product_code,
             s.qty,
             s.unit_price,
             s.line_type,
             p.product_sk,
             p.product_name,
             p.category_id,
             pr.selling_price AS historical_selling_price
         FROM read_csv_auto(
             's3://annapurna-sales/sales/sales/store=S03/year=2024/month=10/SALES_S03_20241001.csv'
         ) s
         LEFT JOIN pg.public.products p
             ON p.product_code = s.product_code
            AND DATE '2024-10-01' BETWEEN p.valid_from AND p.valid_to
         LEFT JOIN pg.public.price_revisions pr
             ON pr.product_sk = p.product_sk
            AND DATE '2024-10-01' BETWEEN pr.effective_from AND pr.effective_to
         WHERE s.line_type = 'SALE'
         LIMIT 10;
┌────────────────────┬─────────┬──────────────┬───┬─────────────┬──────────────────────────┐
│      bill_no       │ line_no │ product_code │ … │ category_id │ historical_selling_price │
│      varchar       │  int64  │   varchar    │ … │   varchar   │      decimal(12,2)       │
├────────────────────┼─────────┼──────────────┼───┼─────────────┼──────────────────────────┤
│ S03/20241001/00004 │       7 │ P100033      │ … │ C05         │                   828.62 │
│ S03/20241001/00034 │       6 │ P100074      │ … │ C06         │                    57.78 │
│ S03/20241001/00008 │       2 │ P100077      │ … │ C06         │                   173.95 │
│ S03/20241001/00026 │       4 │ P100121      │ … │ C11         │                   107.67 │
│ S03/20241001/00020 │       6 │ P100183      │ … │ C09         │                  1493.10 │
│ S03/20241001/00020 │       7 │ P100194      │ … │ C13         │                    46.79 │
│ S03/20241001/00003 │       2 │ P100203      │ … │ C06         │                   272.10 │
│ S03/20241001/00012 │       4 │ P100210      │ … │ C09         │                  1073.90 │
│ S03/20241001/00032 │       3 │ P100221      │ … │ C09         │                   139.88 │
│ S03/20241001/00023 │       1 │ P100255      │ … │ C05         │                  1250.31 │
└────────────────────┴─────────┴──────────────┴───┴─────────────┴──────────────────────────┘
  10 rows               use .last to show entire result               10 columns (5 shown)
memory D

=========================================================================================================================================

TASK D 

───────────┴────────────┘
memory D SELECT
             'March 2024' AS month,
             41971649.09 AS sales_folder_revenue,
             486250.00 AS institutional_bulk_invoice,
             41971649.09 + 486250.00 AS reconciled_revenue,
             42457899 AS finance_signed_off,
             (41971649.09 + 486250.00) - 42457899 AS difference;
┌────────────┬──────────────────────┬────────────────────────┬───┬────────────────────┬───────────────┐
│   month    │ sales_folder_revenue │ institutional_bulk_in… │ … │ finance_signed_off │  difference   │
│  varchar   │    decimal(10,2)     │      decimal(8,2)      │ … │       int32        │ decimal(13,2) │
├────────────┼──────────────────────┼────────────────────────┼───┼────────────────────┼───────────────┤
│ March 2024 │          41971649.09 │              486250.00 │ … │           42457899 │          0.09 │
└────────────┴──────────────────────┴────────────────────────┴───┴────────────────────┴───────────────┘
  1 rows                     use .last to show entire result                      6 columns (5 shown)
memory D
========================================================================================================================================

TASK E

────────────────────────────────────┐
│┌───────────────────────────────────┐│
││    Query Profiling Information    ││
│└───────────────────────────────────┘│
└─────────────────────────────────────┘
EXPLAIN ANALYZE SELECT     s.bill_no,     s.line_no,     s.product_code,     s.qty,     s.unit_price,     s.line_type,     p.product_sk,     p.product_name,     pc.category_name,     st.store_name,     st.city,     st.region FROM read_csv_auto(     's3://annapurna-sales/sales/sales/store=S03/year=2024/month=03/SALES_S03_20240301.csv' ) s LEFT JOIN pg.public.products p     ON p.product_code = s.product_code    AND DATE '2024-03-01' BETWEEN p.valid_from AND p.valid_to LEFT JOIN pg.public.product_categories pc     ON p.category_id = pc.category_id LEFT JOIN pg.public.stores st     ON st.store_id = 'S03' WHERE s.line_type = 'SALE' LIMIT 10;
┌─────────────────────────────────────┐
│┌───────────────────────────────────┐│
││         HTTPFS HTTP Stats         ││
││                                   ││
││            in: 23.8 KiB           ││
││            out: 0 bytes           ││
││              #HEAD: 1             ││
││              #GET: 1              ││
││              #PUT: 0              ││
││              #POST: 0             ││
││             #DELETE: 0            ││
│└───────────────────────────────────┘│
└─────────────────────────────────────┘
┌────────────────────────────────────────────────┐
│┌──────────────────────────────────────────────┐│
││              Total Time: 0.0432s             ││
│└──────────────────────────────────────────────┘│
└────────────────────────────────────────────────┘
┌───────────────────────────┐
│           QUERY           │
└─────────────┬─────────────┘
┌─────────────┴─────────────┐
│      EXPLAIN_ANALYZE      │
│    ────────────────────   │
│                           │
│           0 rows          │
│           0.00s           │
└─────────────┬─────────────┘
┌─────────────┴─────────────┐
│         PROJECTION        │
│    ────────────────────   │
│          bill_no          │
│          line_no          │
│        product_code       │
│            qty            │
│         unit_price        │
│         line_type         │
│         product_sk        │
│        product_name       │
│       category_name       │
│         store_name        │
│            city           │
│           region          │
│                           │
│                           │
│                           │
│          10 rows          │
│           0.00s           │
└─────────────┬─────────────┘
┌─────────────┴─────────────┐
│      STREAMING_LIMIT      │
│    ────────────────────   │
│                           │
│          10 rows          │
│           0.00s           │
└─────────────┬─────────────┘
┌─────────────┴─────────────┐
│     BLOCKWISE_NL_JOIN     │
│    ────────────────────   │
│      Join Type: LEFT      │
│      Condition: true      │
│                           ├────────────────────────────────────────────────────────────────────────┐
│                           │                                                                        │
│                           │                                                                        │
│          264 rows         │                                                                        │
│           0.00s           │                                                                        │
└─────────────┬─────────────┘                                                                        │
┌─────────────┴─────────────┐                                                          ┌─────────────┴─────────────┐
│         HASH_JOIN         │                                                          │         TABLE_SCAN        │
│    ────────────────────   │                                                          │    ────────────────────   │
│      Join Type: LEFT      │                                                          │       Table: stores       │
│                           │                                                          │                           │
│        Conditions:        │                                                          │        Projections:       │
│ category_id = category_id │                                                          │          store_id         │
│                           │                                                          │         store_name        │
│                           │                                                          │            city           │
│                           ├───────────────────────────────────────────┐              │           region          │
│                           │                                           │              │                           │
│           0.00s           │                                           │              │          Filters:         │
│                           │                                           │              │       store_id='S03'      │
│                           │                                           │              │                           │
│                           │                                           │              │                           │
│                           │                                           │              │                           │
│          264 rows         │                                           │              │           1 row           │
│          (0.00s)          │                                           │              │           0.00s           │
└─────────────┬─────────────┘                                           │              └───────────────────────────┘
┌─────────────┴─────────────┐                             ┌─────────────┴─────────────┐
│         HASH_JOIN         │                             │         TABLE_SCAN        │
│    ────────────────────   │                             │    ────────────────────   │
│      Join Type: RIGHT     │                             │           Table:          │
│                           │                             │     product_categories    │
│        Conditions:        │                             │                           │
│product_code = product_code│                             │        Projections:       │
│                           ├──────────────┐              │        category_id        │
│                           │              │              │       category_name       │
│                           │              │              │                           │
│                           │              │              │                           │
│           0.00s           │              │              │                           │
│          264 rows         │              │              │          14 rows          │
│          (0.00s)          │              │              │           0.00s           │
└─────────────┬─────────────┘              │              └───────────────────────────┘
┌─────────────┴─────────────┐┌─────────────┴─────────────┐
│         TABLE_SCAN        ││           FILTER          │
│    ────────────────────   ││    ────────────────────   │
│      Table: products      ││    (line_type = 'SALE')   │
│                           ││                           │
│        Projections:       ││                           │
│        product_code       ││                           │
│         valid_from        ││                           │
│          valid_to         ││           0.00s           │
│        category_id        ││                           │
│         product_sk        ││                           │
│        product_name       ││                           │
│                           ││                           │
│          Filters:         ││                           │
│ valid_from<='2024-03-01': ││                           │
│           :DATE           ││                           │
│  valid_to>='2024-03-01':  ││                           │
│           :DATE           ││                           │
│                           ││                           │
│      Dynamic Filters:     ││                           │
│  optional: product_code>= ││                           │
│  'P100033' AND optional:  ││                           │
│   product_code<='P108389' ││                           │
│                           ││                           │
│                           ││                           │
│                           ││                           │
│         1,200 rows        ││          264 rows         │
│           0.00s           ││          (0.00s)          │
└───────────────────────────┘└─────────────┬─────────────┘
                             ┌─────────────┴─────────────┐
                             │         TABLE_SCAN        │
                             │    ────────────────────   │
                             │         Function:         │
                             │       READ_CSV_AUTO       │
                             │                           │
                             │        Projections:       │
                             │        product_code       │
                             │         line_type         │
                             │          bill_no          │
                             │          line_no          │
                             │            qty            │
                             │         unit_price        │
                             │                           │
                             │    Total Files Read: 1    │
                             │                           │
                             │        Filename(s):       │
                             │ s3://annapurna-sales/sales│
                             │ /sales/store=S03/year=2024│
                             │         /month=03         │
                             │  /SALES_S03_20240301.csv  │
                             │                           │
                             │                           │
                             │                           │
                             │          391 rows         │
                             │           0.00s           │
                             └───────────────────────────┘
memory D

========================================================================================================================================
TASK F 
────────┬───────────────┬───────────────┬───────────────┬──────────┐
│  month  │    s01_s05    │    s06_s09    │    s10_s12    │ finance  │
│ varchar │ decimal(10,2) │ decimal(10,2) │ decimal(10,2) │  int32   │
├─────────┼───────────────┼───────────────┼───────────────┼──────────┤
│ 2024-01 │   17207944.85 │   11437711.94 │   10113818.44 │ 38446071 │
│ 2024-02 │   15713253.81 │   10083292.87 │    9461521.15 │ 34887086 │
│ 2024-03 │   18928511.86 │   12202350.58 │   11062178.95 │ 42457899 │
│ 2024-04 │   17297296.19 │   11096403.17 │    9805485.67 │ 37958457 │
│ 2024-05 │   19251451.34 │   11943345.05 │   11134387.99 │ 41764716 │
│ 2024-06 │   17811462.16 │   11097765.02 │   10501673.29 │ 38987083 │
│ 2024-07 │   18907815.39 │   11731900.93 │   10526530.20 │ 40527292 │
│ 2024-08 │   20503354.45 │   13662743.68 │   11607272.99 │ 45252182 │
│ 2024-09 │   20402351.10 │   13088007.01 │   11695818.47 │ 44615037 │
│ 2024-10 │   26020424.96 │   16130067.48 │   14601355.49 │ 56359196 │
│ 2024-11 │   24358792.59 │   15121806.48 │   14610632.72 │ 51583838 │
│ 2024-12 │   23420997.41 │   15083012.84 │   13166539.35 │ 50745209 │
└─────────┴───────────────┴───────────────┴───────────────┴──────────┘
  12 rows                                                  5 columns

  ======================================================================================================================================
