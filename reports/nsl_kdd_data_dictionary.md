# Data Dictionary — NSL-KDD Intrusion Detection

Documents the real NSL-KDD dataset (Tavallaee et al., 2009) and the derived columns created during preprocessing.

## Raw files

| File | Records | Description |
|---|---|---|
| `KDDTrain+.txt` | 125,973 | Training set, 22 attack types + normal |
| `KDDTest+.txt` | 22,544 | Test set, 39 attack types + normal (17 never appear in training) |

Both are headerless CSVs — 41 features + `label` + `difficulty`, applied via the column list below.

## The 41 original features

| Column | Type | Description |
|---|---|---|
| duration | numeric | Length of the connection (seconds) |
| protocol_type | categorical | tcp, udp, or icmp |
| service | categorical | Network service on destination (http, ftp, smtp, etc. — 70 values) |
| flag | categorical | Status of the connection (e.g. SF = normal completion, S0 = no reply) |
| src_bytes | numeric | Bytes sent from source to destination |
| dst_bytes | numeric | Bytes sent from destination to source |
| land | binary | 1 if connection is from/to the same host/port, else 0 |
| wrong_fragment | numeric | Number of "wrong" fragments |
| urgent | numeric | Number of urgent packets |
| hot | numeric | Number of "hot" indicators (suspicious command sequences) |
| num_failed_logins | numeric | Count of failed login attempts |
| logged_in | binary | 1 if successfully logged in, else 0 |
| num_compromised | numeric | Number of "compromised" conditions |
| root_shell | binary | 1 if root shell obtained, else 0 |
| su_attempted | binary | 1 if "su root" command attempted, else 0 |
| num_root | numeric | Number of root accesses |
| num_file_creations | numeric | Number of file creation operations |
| num_shells | numeric | Number of shell prompts |
| num_access_files | numeric | Number of operations on access control files |
| num_outbound_cmds | numeric | Number of outbound commands in an ftp session |
| is_host_login | binary | 1 if login belongs to "host" list, else 0 |
| is_guest_login | binary | 1 if login is a "guest" login, else 0 |
| count | numeric | Connections to the same host in the past 2 seconds |
| srv_count | numeric | Connections to the same service in the past 2 seconds |
| serror_rate | numeric | % of connections with SYN errors (same host) |
| srv_serror_rate | numeric | % of connections with SYN errors (same service) |
| rerror_rate | numeric | % of connections with REJ errors (same host) |
| srv_rerror_rate | numeric | % of connections with REJ errors (same service) |
| same_srv_rate | numeric | % of connections to the same service |
| diff_srv_rate | numeric | % of connections to different services |
| srv_diff_host_rate | numeric | % of connections to different hosts (same service) |
| dst_host_count | numeric | Connections to the same destination host |
| dst_host_srv_count | numeric | Connections to the same destination host + service |
| dst_host_same_srv_rate | numeric | % of connections to the same service (dest host) |
| dst_host_diff_srv_rate | numeric | % of connections to different services (dest host) |
| dst_host_same_src_port_rate | numeric | % of connections from the same source port |
| dst_host_srv_diff_host_rate | numeric | % of connections to different hosts (dest host + service) |
| dst_host_serror_rate | numeric | % SYN errors (dest host) |
| dst_host_srv_serror_rate | numeric | % SYN errors (dest host + service) |
| dst_host_rerror_rate | numeric | % REJ errors (dest host) |
| dst_host_srv_rerror_rate | numeric | % REJ errors (dest host + service) |
| label | text | Specific attack name, or "normal" (39 distinct values across train+test) |
| difficulty | numeric | Difficulty score assigned by dataset creators (not used in this project) |

## Derived columns (created during preprocessing, notebook 2)

| Column | Description |
|---|---|
| attack_category | `label` grouped into 5 classes: normal, DoS, Probe, R2L, U2R |
| is_novel_attack_type | 1 if this row's `label` never appears in the training set (test set only) |
| protocol_type_enc, service_enc, flag_enc | Numeric encodings of the 3 categorical text columns |

## Attack category groupings

Standard groupings used across NSL-KDD literature:

| Category | Meaning | Example attack labels |
|---|---|---|
| DoS | Denial of Service | neptune, smurf, back, teardrop, apache2, worm |
| Probe | Surveillance/scanning | satan, ipsweep, nmap, portsweep, mscan, saint |
| R2L | Remote-to-Local (unauthorized access from remote) | guess_passwd, ftp_write, warezmaster, httptunnel |
| U2R | User-to-Root (privilege escalation) | buffer_overflow, rootkit, loadmodule, sqlattack |
| normal | Legitimate traffic | — |

## Known data characteristics

- **Severe class imbalance:** normal (67,343 training records) vs. U2R (52 training records) — a ~1,295:1 ratio.
- **Train/test distribution shift is intentional:** the test set includes 17 attack types absent from training, by design, to test genuine generalization rather than memorization.
- **No missing values** in either file — confirmed during notebook 1 data quality checks.
- **service** has more distinct values in some test rows than appear in training; these are mapped to a safe `"unseen"` category during encoding (notebook 2) rather than dropped or causing an error.
