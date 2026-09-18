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