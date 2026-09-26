# Dataset Release for "The Case of the Missing Cases: Inferring Sealed U.S. Federal Cases"

This directory is the public artifact for the IMC 2026 paper "The Case of the Missing Cases: Inferring Sealed U.S. Federal Cases".  
Paper DOI: [10.1145/3777912.3839786](https://doi.org/10.1145/3777912.3839786)  
The PDF can also be found at: https://shuye.dev/papers/IMC26-Sealed.pdf

The dataset file is at [`federal_sealed_case_measurement_2020_2025.jsonl.gz`](federal_sealed_case_measurement_2020_2025.jsonl.gz), which contains 612,001 federal district-court cases from 20 districts, filed in years 2020 to 2025, and relevant information about their observed public and sealed status.
The dataset combines our observations and query results across CM/ECF, CourtListener, and FJC Integrated Database. 
The dataset is redacted to remove sensitive information and to preserve privacy, while retaining sufficient information to conduct analysis over the sealing status and reproduce the results in the paper.
See the paper's Ethics section for more details and rationales behind the redaction.

There is also a smaller uncompressed 120-row sample at [`federal_sealed_case_measurement_2020_2025_sample.jsonl`](federal_sealed_case_measurement_2020_2025_sample.jsonl) for quick inspection of the data and schema.

This dataset is released under the [CC BY 4.0 license](https://creativecommons.org/licenses/by/4.0/).

## Schema

### Case Number and Identifiers

| Column                      | Type            | Meaning |
| --------------------------- | --------------- | ------- |
| `district_code`             | string          | Federal district abbreviation, e.g. `nysd` for Southern District of New York. |
| `file_year`                 | integer         | Four-digit filing year in the case number. |
| `office`                    | integer (nullable) | Divisional office when known. Null means the source did not identify one or the district shares numbering across offices. |
| `case_type`                 | string          | Case type in the case number, e.g., `cr`, `cv`, and `mj`. |
| `case_number`               | string          | Normalized identifier `OFFICE:YY-TYPE-NNNNN`; `X` denotes unknown/shared office. Unique within `district_code`. |
| `inferred_segment_position` | integer (nullable) | Position within the inferred segment in the district/year/case type (see paper for detailed methodology). Null for cases not captured by the inferred segment. |
| `inferred_segment_start`    | integer (nullable) | First sequence number of the inferred segment containing the case. |
| `inferred_segment_end`      | integer (nullable) | Last sequence number of the inferred segment containing the case. |
| `office_separate_numbers`   | boolean         | Whether the district is deemed as having separate number sequences by office ("Unified"/"Separate" in the paper). |
| `is_source_only`            | boolean         | True for a case observed only by CM/ECF or IDB, and are absent from the inferred segment using CourtListener data. |
| `known_alternate_case_type` | string (nullable)  | Cases in the 23-cv numbering space but belongs to a different case type (`pq` in S.D. Illinois or `mc` in E.D. Louisiana). See paper for details. |

### CourtListener Data

| Column                            | Type              | Meaning |
| --------------------------------- | ----------------- | ------- |
| `observed_in_courtlistener`       | boolean           | At least one matching CourtListener docket record was observed. The case is unsealed and available to public. |
| `courtlistener_docket_ids`        | array (nullable)  | Unique CourtListener docket IDs associated with the case. |
| `courtlistener_first_created_at`  | timestamp (nullable) | Earliest CourtListener record-creation timestamp across all records. |
| `courtlistener_earliest_filed_at` | timestamp (nullable) | Earliest filing date across all CourtListener records. |

### CM/ECF Data

| Column                            | Type              | Meaning |
| --------------------------------- | ----------------- | ------- |
| `in_ecf`                          | boolean           | True when the case appeared in a CM/ECF report as public. It does not include endpoint-only observations. |
| `in_ecf_sealed`                   | boolean           | True when the case, or a redacted number of it (e.g. 24-1234), appeared in a CM/ECF report as sealed. See the paper for details on the matching methodology for redacted case numbers. |
| `ecf_case_number_endpoint_status` | string (nullable) | The hidden possible-case-number endpoint result: `sealed`, `unsealed`, `not_found`, if a check was performed. |
| `cmecf_sealed_title_pattern`      | string (nullable) | Coded categories of sealing-related case name titles: `sealed_v_sealed`, `us_v_sealed`, `other_sealed`, or null if the case title is unsealed. |
| `cmecf_filed_date`                | string (nullable) | Filing date reported by CM/ECF. Note: Different districts use different formats. |
| `ecf_notes`                       | object (nullable) | Redacted CM/ECF notes. Only `Office`, `Division`, `Cause`, `NOS`, `Jurisdiction`, `Jury demand`, and `Case flags` fields are preserved. |

### FJC Integrated Database

| Column                        | Type             | Meaning |
| ----------------------------- | ---------------- | ------- |
| `matched_to_idb`              | boolean          | At least one IDB records was found and linked to the case. |
| `idb_case_numbers`            | array (nullable) | All IDB case numbers identified. |
| `idb_file_date`               | date (nullable)  | Earliest filing date among linked IDB records. |
| `idb_historical_sealing_flag` | boolean (nullable) | Historical-sealing indicator from IDB. For civil cases, when both party names were `SEALED`; for criminal cases, a sealing-related `PROCCD` was observed. |
| `idb_linkage_method`          | string (nullable) | Linkage method, generally `exact` case-number matching or unique logical-number matching. |
| `idb_proccd`                  | array (nullable) | Unique list of criminal proceeding codes among linked IDB records. |
| `idb_nos`                     | array (nullable) | Unique list of civil nature-of-suit codes among linked IDB records. |

## Reproduce release-derived results

See the [`reproduce_from_release.ipynb`](reproduce_from_release.ipynb) notebook for a demonstration of how to reproduce the paper's results from the released dataset.

The notebook reads the `federal_sealed_case_measurement_2020_2025.jsonl.gz` file and reproduces cases sealed after 2 years (Table 3), the historically sealed rate (Figure 1), and the sealing duration survival curves (Figure 2).


Note that, due to the different aggregation methods in creating the release dataset, the produced plots are extremely close but have slight differences with the paper's plots, which are derived directly from the original source data.
For the historically sealed plot, 319 of 360 data points are identical, with 41 data points differing by a mean of 0.028 percentage points.
The survival plot has 12,312 rather than 12,319 observations.
