# System Integration Testing

Dana Kripto Indonesia Versi 1.0

| Disiapkan Oleh | : | {{metadata.reporter}} |
| :---- | :---- | :---- |
| Tanggal | : | {{metadata.updated_at}} |
| Nomor Dokumen | : | {{report_id}} |

## Daftar Isi

1. LATAR BELAKANG PROYEK
2. TUJUAN PROYEK
3. SOLUTION OVERVIEW PROYEK
   - A. Ruang Lingkup
   - B. Batasan Dan Asumsi
4. EXECUTIVE SUMMARY
5. LINGKUNGAN PENGUJIAN
6. SKENARIO DAN HASIL PENGUJIAN
7. KRITERIA KEBERHASILAN
8. RENCANA TINDAK LANJUT
9. LAMPIRAN

# 1. LATAR BELAKANG PROYEK

{{sit_document.custom_fields.project_background}}

Sumber requirement/report:
{{#source_refs}}
- {{source}} / {{role}}: {{title}} {{url}}
{{/source_refs}}

# 2. TUJUAN PROYEK

{{sit_document.objective}}

# 3. SOLUTION OVERVIEW PROYEK

## A. Ruang Lingkup

Cakupan sistem yang dikembangkan dan diuji:
{{#sit_document.scope}}
- {{.}}
{{/sit_document.scope}}

## B. Batasan Dan Asumsi

{{#sit_document.known_limitations}}
- {{.}}
{{/sit_document.known_limitations}}

# 4. EXECUTIVE SUMMARY

Fokus pengujian pada fitur ini yaitu {{metadata.feature}}.

| No | Fitur | Jumlah Skenario SIT | Hasil Pengujian |
| :---: | ----- | -----: | ----- |
| 1 | {{metadata.feature}} | {{execution_metrics.selected_test_cases}} | {{sit_document.conclusion}} |

Ringkasan eksekusi:
- Passed: {{execution_metrics.passed}}
- Failed: {{execution_metrics.failed}}
- Blocked: {{execution_metrics.blocked}}
- Untested: {{execution_metrics.untested}}
- Retest: {{execution_metrics.retest}}
- Pass rate: {{execution_metrics.pass_rate}}%
- Completion rate: {{execution_metrics.execution_completion_rate}}%

Rekomendasi sign-off: `{{signoff_recommendation.recommendation}}`

Alasan: {{signoff_recommendation.reason}}

# 5. LINGKUNGAN PENGUJIAN

Berikut adalah spesifikasi lingkungan pengujian SIT terkait fitur yang ada dalam dokumen ini yakni {{sit_document.environment}}.

| Nama Sistem | Spesifikasi OS | Spesifikasi Aplikasi | Spesifikasi server | Spesifikasi Security |
| ----- | ----- | ----- | ----- | ----- |
| {{sit_document.custom_fields.system_name}} | {{sit_document.custom_fields.os_spec}} | {{sit_document.custom_fields.application_spec}} | {{sit_document.custom_fields.server_spec}} | {{sit_document.custom_fields.security_spec}} |

# 6. SKENARIO DAN HASIL PENGUJIAN

{{metadata.project}}
Modul: {{metadata.feature}}
Tanggal Pengujian: {{metadata.updated_at}}

| No. | Skenario | Langkah-Langkah | Hasil yang diharapkan | Hasil yang dicapai | Hasil |
| :---: | :--- | :--- | :--- | :--- | :---: |
{{#custom_fields.test_scenarios}}
| {{no}} | {{scenario}} | {{steps}} | {{expected_result}} | {{actual_result}} | {{status}} |
{{/custom_fields.test_scenarios}}

# 7. KRITERIA KEBERHASILAN

| No. | Kriteria Keberhasilan | Metrik Pengukuran | Target |
| :---: | :--- | :--- | :--- |
| 1 | Semua skenario prioritas kritikal tervalidasi | Critical coverage status | {{coverage.critical_coverage_status}} |
| 2 | Defect kritikal/high terkendali | Critical/High issue count | {{execution_metrics.critical_high_issue_count}} |
| 3 | Eksekusi test selesai sesuai target | Execution completion rate | {{execution_metrics.execution_completion_rate}}% |

# 8. RENCANA TINDAK LANJUT

{{#risk.open_risks}}
- Risiko: {{.}}
{{/risk.open_risks}}

{{#risk.mitigations}}
- Mitigasi: {{.}}
{{/risk.mitigations}}

{{#issue_package.issues}}
- Issue {{issue_id}}: {{title}} (`{{severity}}`, {{submission_status}})
{{/issue_package.issues}}

# 9. LAMPIRAN

Evidence references:
{{#summary.evidence_refs}}
- {{.}}
{{/summary.evidence_refs}}

Published artifacts:
{{#publication_history}}
- {{artifact_type}}: {{url}}
{{/publication_history}}

Report gaps:
{{#summary.report_gaps}}
- {{.}}
{{/summary.report_gaps}}

## Approval

| Disiapkan Oleh: |  |  |
| :---- | :---- | :---- |
| Jabatan | : | QA Engineer |
| Nama | : | {{metadata.reporter}} |
| Tanggal | : | {{metadata.updated_at}} |

| Diperiksa Oleh: |  | Disetujui Oleh: |  |
| :---- | :---- | :---- | :---- |
| Jabatan | : | Jabatan | : |
| Nama | : | Nama | : |
| Tanggal | : | Tanggal | : |
