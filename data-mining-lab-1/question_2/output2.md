SECTION A(A)
===
DuckDB v1.5.5 (Variegata)
Enter ".help" for usage hints.
memory D SELECT
             COUNT(*) AS notices,
             ROUND(AVG(LENGTH(body)), 2) AS avg_body_chars,
             MIN(LENGTH(body)) AS min_body_chars,
             MAX(LENGTH(body)) AS max_body_chars,
             ROUND(AVG(LENGTH(title)), 2) AS avg_title_chars
         FROM read_csv_auto(
             'C:/Users/ub02-glab-061/Downloads/data_2/data_2/notices/part-*.csv'
         );
┌─────────┬────────────────┬────────────────┬────────────────┬─────────────────┐
│ notices │ avg_body_chars │ min_body_chars │ max_body_chars │ avg_title_chars │
│  int64  │     double     │     int64      │     int64      │     double      │
├─────────┼────────────────┼────────────────┼────────────────┼─────────────────┤
│   12000 │         4527.3 │           1500 │           8127 │           80.57 │
└─────────┴────────────────┴────────────────┴────────────────┴─────────────────┘
memory D DESCRIBE
         SELECT *
         FROM read_csv_auto(
             'C:/Users/ub02-glab-061/Downloads/data_2/data_2/notices/part-*.csv'
         );
┌─────────────────────────┐
│        Describe         │
│                         │
│ notice_id       varchar │
│ portal_id       varchar │
│ published_at    date    │
│ title           varchar │
│ body            varchar │
│ estimated_value bigint  │
│ closing_date    date    │
└─────────────────────────┘
memory D

ons import defaultdict; base=r'C:\Users\ub02-glab-061\Downloads\data_2\data_2'; notices={}; [notices.update({r['notice_id']:r['body']}) for f in glob.glob(base+r'\notices\*.csv') for r in csv.DictReader(open(f,encoding='utf-8-sig',newline=''))]; labels=list(csv.DictReader(open(base+r'\labelled_pairs.csv',encoding='utf-8-sig',newline=''))); ids={x for r in labels for x in (r['notice_id_a'],r['notice_id_b'])}; cache={n:set(' '.join(w[i:i+3]) for i in range(len(w)-2)) for n in ids for w in [re.sub(r'[^a-z0-9 ]+',' ',notices[n].lower()).split()]}; vals=[]; [(lambda A,B: vals.append((len(A&B)/len(A|B) if A|B else 0,r['label'])))(cache[r['notice_id_a']],cache[r['notice_id_b']]) for r in labels]; d=defaultdict(list); [d[x[1]].append(x[0]) for x in vals]; print('Same average:',round(sum(d['same'])/len(d['same']),4)); print('Different average:',round(sum(d['different'])/len(d['different']),4)); print('Same min/max:',round(min(d['same']),4),round(max(d['same']),4)); print('Different min/max:',round(min(d['different']),4),round(max(d['different']),4))"


Same average: 0.6499
Different average: 0.2558
Same min/max: 0.1977 0.9933
Different min/max: 0.0951 0.5304
PS C:\Users\ub02-glab-061>
=================================================================================================================================
A(b)

Different min/max: 0.0951 0.5304
PS C:\Users\ub02-glab-061> python -c "import csv,glob,re,hashlib,math; base=r'C:\Users\ub02-glab-061\Downloads\data_2\data_2'; notices={}; [notices.update({r['notice_id']:r['body']}) for f in glob.glob(base+r'\notices\*.csv') for r in csv.DictReader(open(f,encoding='utf-8-sig',newline=''))]; labels=list(csv.DictReader(open(base+r'\labelled_pairs.csv',encoding='utf-8-sig',newline=''))); ids={x for r in labels for x in (r['notice_id_a'],r['notice_id_b'])}; sets={}; [sets.update({n:set(' '.join(w[i:i+3]) for i in range(len(w)-2))}) for n in ids for w in [re.sub(r'[^a-z0-9 ]+',' ',notices[n].lower()).split()]]; K=256; seeds=list(range(K)); sig={}; [sig.update({n:[min(int.from_bytes(hashlib.blake2b((str(k)+'|'+s).encode(),digest_size=8).digest(),'big') for s in sets[n]) if sets[n] else 0 for k in seeds]}) for n in ids]; errs=[]; [(errs.append(abs((sum(x==y for x,y in zip(sig[r['notice_id_a']],sig[r['notice_id_b']]))/K)-(len(sets[r['notice_id_a']]&sets[r['notice_id_b']])/len(sets[r['notice_id_a']]|sets[r['notice_id_b']]) if sets[r['notice_id_a']]|sets[r['notice_id_b']] else 0)))) for r in labels]; print('K:',K); print('MAE:',round(sum(errs)/len(errs),4)); print('RMSE:',round(math.sqrt(sum(x*x for x in errs)/len(errs)),4)); print('P95 error:',round(sorted(errs)[int(.95*len(errs))-1],4)); print('Max error:',round(max(errs),4)); print('Theoretical worst-case SE:',round(1/(2*math.sqrt(K)),4))"


K: 256
MAE: 0.0244
RMSE: 0.0307
P95 error: 0.0602
Max error: 0.1189
Theoretical worst-case SE: 0.0312
PS C:\Users\ub02-glab-061>

=========================================================================================================================================

A(c)
MinHash values K = 128
LSH bands       = 32
Rows per band   = 4
b × r           = 128
Theoretical threshold ≈ 0.4204

Candidate survival probability:
Similarity 0.20 -> P(candidate) = 0.0500 (5.0%)
Similarity 0.30 -> P(candidate) = 0.2291 (22.9%)
Similarity 0.42 -> P(candidate) = 0.6364 (63.6%)
Similarity 0.55 -> P(candidate) = 0.9536 (95.4%)
Similarity 0.60 -> P(candidate) = 0.9882 (98.8%)
Similarity 0.65 -> P(candidate) = 0.9981 (99.8%)
Similarity 0.80 -> P(candidate) = 1.0000 (100.0%)
Similarity 0.95 -> P(candidate) = 1.0000 (100.0%)

Operating point:
At similarity 0.60: 0.9882 (98.8%)
At similarity 0.25: 0.1177 (11.8%)

False merge cost : False split cost = 100 : 1
False merge is 100.0 times more costly
Policy: prioritize high candidate recall, then use exact similarity verification to control false merges
PS C:\Users\ub02-glab-061>
=========================================================================================================================================
d 

Notices stored: 12,000
LSH bucket rows: 384,000

Before indexing:
SCAN b1
BLOOM FILTER ON b2 (bucket_hash=? AND band_id=?)
SEARCH b2 USING AUTOMATIC PARTIAL COVERING INDEX (bucket_hash=? AND band_id=?)
USE TEMP B-TREE FOR DISTINCT
5 lookups: 10.4251 seconds

Creating composite B-tree index...
Index creation time: 0.1727 seconds

After indexing:
SCAN b1
SEARCH b2 USING INDEX idx_band_bucket (band_id=? AND bucket_hash=?)
USE TEMP B-TREE FOR DISTINCT
5 indexed lookups: 0.1364 seconds
Measured speedup: 76.41x

Access method: Composite B-tree
Indexed columns: band_id, bucket_hash
Reason: candidate lookup searches directly for matching
band/bucket keys instead of scanning the entire bucket table.
========================================================================================================================================
e

Q2(e) - EMPIRICAL SKEW & MITIGATION

Largest single bucket size: 8,329 notices
Top 5 largest bucket sizes: [8329, 8329, 8329, 8329, 8329]

Uncapped candidate pair comparisons: 1,117,609,408
Capped candidate comparisons: 333,856
Candidate workload reduction: 99.97%

Retrieval Quality on labelled_pairs.csv:
True duplicate pairs: 279
Duplicates retrieved: 30
Recall after mitigation: 10.75%
Recall loss: 89.25%

Portal volume information from portal_profiles.md:
P094 = 1,426 notices
P002 = 800 notices
P006 = 792 notices
P001 = 778 notices
P003 = 772 notices
P005 = 768 notices

Mitigation:
Buckets containing more than 50 notices are excluded from pair generation.
=========================================================================================================================================
