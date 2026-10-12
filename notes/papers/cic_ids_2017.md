# CIC-IDS2017

## Paper Information

- **Title:** *Toward Generating a New Intrusion Detection Dataset and Intrusion Traffic Characterization*
- **Authors:** Iman Sharafaldin, Arash Habibi Lashkari, and Ali A. Ghorbani
- **Affiliation:** Canadian Institute for Cybersecurity (CIC), University of New Brunswick (UNB), Canada
- **Publication:** ICISSP 2018, pages 108–116
- **Year distinction:** The traffic was collected in 2017; the paper was published in 2018.
- **Source:** [Publisher's paper page](https://www.scitepress.org/Papers/2018/66398/), DOI: 10.5220/0006639801080116

## One-Sentence Summary

CIC-IDS2017 combines generated benign activity with scheduled attacks in a controlled network, provides labeled flow features, and evaluates compact feature sets using seven machine learning algorithms.

## Research Problem

Intrusion Detection Systems (IDSs) need suitable data for training and evaluation. The paper reviews eleven earlier datasets released between 1998 and 2016 and identifies recurring problems: outdated attacks, limited traffic diversity, incomplete metadata, and anonymization that removes useful traffic information.

The authors therefore build CIC-IDS2017 using a new testbed and an existing framework of eleven dataset-quality criteria. Their claim of improved quality should be read as a comparison under that framework; it does not establish that laboratory traffic fully represents production networks.

## Data-Generation Method

### Testbed

The testbed separates an **attacker network** from a **victim network**. The victim side contains servers and workstations running Windows, Linux, and macOS, together with network infrastructure such as a firewall and switches. A mirror port captures traffic entering and leaving the monitored network.

### Benign Profile Agent

A B-Profile agent generates benign activity from user-behavior profiles. This aims to produce more realistic background traffic than a simple repetition of fixed requests. The profiles cover 25 users and activities over HTTP, HTTPS, FTP, SSH, and email. This is generated activity informed by user behavior, so its realism still requires evaluation against independently collected traffic. [Official dataset description](https://www.unb.ca/cic/datasets/ids-2017.html).

### Capture Period and Attack Schedule

The capture spans July 3–7, 2017. Monday contains only benign activity; later days combine benign traffic with scheduled attacks.

| Day | Date | Attack scenarios |
| --- | --- | --- |
| Monday | July 3 | None; benign activity only |
| Tuesday | July 4 | FTP-Patator and SSH-Patator brute force |
| Wednesday | July 5 | DoS: Slowloris, Slowhttptest, Hulk, GoldenEye; Heartbleed |
| Thursday | July 6 | Web brute force, XSS, SQL injection; infiltration |
| Friday | July 7 | Botnet, port scanning, and DDoS |

The Tuesday scenario is FTP brute force: the paper's Table 2 says “SFTP,” but its scenario description and the official documentation identify FTP-Patator. [Official attack schedule](https://www.unb.ca/cic/datasets/ids-2017.html).

## Processing and Labeling Pipeline

```text
B-Profile benign activity + scheduled attacks
                    |
             Network capture
                    |
                PCAP files
                    |
               CICFlowMeter
                    |
          Flow identifiers and features
                    |
   Matching attack timing and endpoint information
                    |
             Labeled flow CSVs
```

Labels use attack timing and connection information, including IP addresses, ports, and protocol. Being inside an attack window alone should not be treated as proof that every concurrent flow is malicious. [Official labeling description](https://www.unb.ca/cic/datasets/ids-2017.html).

## Input and Labels

Each CSV row represents a **network flow**, rather than an individual packet. CICFlowMeter generates bidirectional flows; the first packet establishes the forward direction, and traffic in the reverse direction contributes backward statistics. [CICFlowMeter documentation](https://www.unb.ca/cic/research/applications.html).

For modeling, use the flow's feature vector as input and its `Label` as the target. Two possible tasks are:

- **Binary classification:** Map `BENIGN` to normal and all attack labels to malicious.
- **Multi-class classification:** Predict the benign class or a specific attack category.

The target label must be excluded from model inputs. Treat flow identifiers, IP addresses, timestamps, and file names as metadata when designing the main experiment: they may identify the collection environment or attack schedule. Ports can carry useful service information, but their value should also be checked across environments.

## Feature Structure

The paper describes 80 features in its analysis and more than 80 elsewhere. The exact input dimension should be checked in the downloaded CSV: identifiers, metadata, and the target are not equivalent to predictive traffic statistics.

| Feature group | Examples or meaning |
| --- | --- |
| Volume | Packet counts and byte totals in each direction |
| Packet size | Statistics of packet lengths |
| Timing | Flow duration and inter-arrival time (IAT) |
| Direction | Separate forward and backward measurements |

CICFlowMeter computes statistical features from bidirectional traffic and exports CSV records with flow identifiers. Record the extractor version and actual column names when reproducing experiments. [CICFlowMeter documentation](https://www.unb.ca/cic/research/applications.html).

## Feature Selection and Experimental Results

The authors use `RandomForestRegressor` for feature ranking, then evaluate seven classifiers. Section 5 combines feature importance with class-specific standardized means to obtain attack-specific rankings.

Table 3 highlights flow timing for DoS, packet-length and byte statistics for Heartbleed, and initial TCP windows and flags for FTP/SSH brute force. These rankings describe this dataset and need validation under new traffic conditions.

Table 4 reports **weighted** metrics and combined training/testing execution time:

| Algorithm | Precision | Recall | F1 | Time (seconds) |
| --- | ---: | ---: | ---: | ---: |
| KNN | 0.96 | 0.96 | 0.96 | 1908.23 |
| Random Forest | 0.98 | 0.97 | 0.97 | 74.39 |
| ID3 | 0.98 | 0.98 | 0.98 | 235.02 |
| AdaBoost | 0.77 | 0.84 | 0.77 | 1126.24 |
| MLP | 0.77 | 0.83 | 0.76 | 575.73 |
| Naive Bayes | 0.88 | 0.04 | 0.04 | 14.77 |
| QDA | 0.97 | 0.88 | 0.92 | 18.79 |

ID3 has the highest reported F1. RF offers strong performance with less time than ID3 or KNN; it is not the fastest overall. These results do not measure generalization under distribution shift. [Original paper, Section 5 and Table 4](https://pdfs.semanticscholar.org/2342/9b5be933e16e6988da9a322ad95dfdc8c4b0.pdf).

## Output Files

The release includes PCAP captures and two CSV archives: `GeneratedLabelledFlows.zip` and `MachineLearningCSV.zip`. Check which archive is used before assuming that identifiers or timestamps are available for splitting. [Official release description](https://www.unb.ca/cic/datasets/ids-2017.html).

## Main Contributions

1. A public IDS dataset built from a documented network topology, generated benign behavior, and multiple attack scenarios.
2. Compact feature sets intended to help distinguish different attacks.
3. Baseline comparisons of seven common machine learning algorithms.

## Limitations and Threats to Validity

The following points are reading reflections and implications for further experiments:

- **Benign-traffic realism:** B-Profile generation may not reproduce the full variation in request frequency, packet sizes, or changing user behavior found in production networks. This is a concern to investigate, rather than a discrepancy quantified by this paper.
- **Testbed scale:** A small controlled environment limits host and application diversity. Expanding the environment would make it possible to test whether learned patterns transfer to unfamiliar systems.
- **Scheduled attacks:** Time, host, and service information may correlate with labels. A classifier could exploit these associations without learning attack behavior that transfers elsewhere.
- **Aggregate metrics:** Weighted scores may hide weak detection of rare attacks. Per-class recall, macro F1, false-positive rate, and confusion matrices would make evaluation more informative.
- **Feature-selection leakage:** Section 5 describes computing importance over the whole dataset. Reproduction should establish whether selection precedes splitting; fit feature selection and all learned preprocessing on training data only.
- **Evaluation scope:** Strong results within one dataset leave performance on future traffic and different networks unresolved.

## Relevance to Distribution-Shift Research

For the question **“Does high performance under random splitting reflect reliable generalization?”**, CIC-IDS2017 offers a useful contrast to UNSW-NB15 because its attacks are organized by day and scenario.

Compare random splitting with chronological splitting and grouping by attack session or host. Keep related flows in the same partition, and report each partition's class distribution. A cross-day split can change both traffic conditions and the available attack categories, so a performance drop cannot automatically be attributed to time alone.

Monday can provide benign data for a normal-behavior model, but it cannot by itself train a supervised classifier requiring both benign and attack examples. Holding out an attack family instead evaluates detection of unseen attacks and should be stated as a separate experimental question.

For comparison with UNSW-NB15, begin with feature groups shared by both datasets. Align feature definitions, units, labels, and extraction assumptions before interpreting a cross-dataset result as evidence of generalization.
