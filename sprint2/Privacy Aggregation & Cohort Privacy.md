**1. Privacy-Preserving Aggregation Methods**

In the context of this project, a privacy-preserving aggregation strategy means converting individual health data into summaries at the group level in a way that makes it impossible to identify or draw conclusions about any one individual participant.

The system makes use of aggregation-based reporting techniques that are frequently used in public health settings to safeguard participant privacy. Researchers and medical professionals never have access to individual-level health records. Instead, aggregated statistics like counts, averages, percentages, and trends over time are used by the system to create reports. These privacy rules are enforced automatically by the system at report generation time and cannot be bypassed by users.

A minimum cohort size threshold based on k-anonymity is enforced for all aggregated outputs. Only when there are 10 or more participants in the chosen cohort are reports or comparisons produced. In order to avoid possible re-identification, the system suppresses the output whenever a cohort or sub-metric drops below this threshold. A cohort size of at least 10 participants is required to ensure meaningful aggregation while reducing the risk of identifying individuals within small groups.

Prior to aggregation, all direct identifiers are removed from the dataset, and only non-identifying demographic ranges and categorical indicators are used. Although more advanced privacy techniques, such as differential privacy, were considered, they deemed out of scope for this project as a rule-based aggregation approach that uses minimum cohort sizes and suppression provides a suitable balance between privacy protection, system complexity, and implementation feasibility.


**2. Cohort Privacy Rules**

In this system, cohort privacy rules define the conditions under which groups of participants may be created and analyzed while preventing the identification or interference of information about individual participants. These rules are enforced automatically by the system whenever cohorts are created or used for reporting.


**•	Minimum cohort size**

For a cohort to be considered valid, there must be at least ten participants. If a cohort does not meet this threshold, the system rejects the request or suppresses the resulting output. This number is chosen as it helps prevent re-identification through small-group interference.


**•	No direct identifiers**

Cohorts may not include or expose direct identifying information such as names, email addresses, exact dates of birth, physical addresses, or free-text responses. Only non-identifying attributes are permitted in cohort definitions.


**•	Restricted filter attributes**

Only a predetermined set of authorized, non-identifying filters, such as age ranges, date ranges, and high-level categorical indications (such as condition flags), may be used to create cohorts. Filters that could combine identifying characteristics or isolate specific people are prohibited.


**•	Dynamic cohort membership**

Static participant lists are not kept in cohorts. Instead, cohort membership is determined dynamically at query time using the specified criteria. This keeps people from being tracked over an extended period of time and guarantees that cohort composition shifts naturally as data changes. 


**•	Automatic enforcement and rejection**

All cohort definitions and cohort-based report requests are validated by the system prior to execution. If any privacy rule is violated, including minimum size requirements or restricted filter usage, the system automatically blocks the request and prevents report generation.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Reasoning**

These cohort privacy regulations guarantee that population-level analysis can be carried out securely without disclosing private data about vulnerable people. The approach mirrors established practices in public-health dashboards, where small counts are suppressed and cohort definitions are tightly constrained to prevent re-identification.


**References Used:**

**Khaled El Emam, PhD, Fida Kamal Dankar, MSc (2008)
Protecting Privacy Using k-Anonymity**

https://pmc.ncbi.nlm.nih.gov/articles/PMC2528029/

**Utrecht University
K-anonymity, l-diversity and t-closeness**

https://utrechtuniversity.github.io/dataprivacyhandbook/k-l-t-anonymity.html






