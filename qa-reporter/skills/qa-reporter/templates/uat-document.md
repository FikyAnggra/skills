# User Acceptance Testing

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
4. TUJUAN USER ACCEPTANCE TEST
5. LINGKUNGAN USER ACCEPTANCE TEST
6. PERSIAPAN PENGUJIAN
7. SKENARIO DAN HASIL PENGUJIAN
8. KRITERIA KEBERHASILAN
9. PENCATATAN MASALAH DAN TINDAK LANJUT
10. EVALUASI DAN PENILAIAN
11. LAMPIRAN

# 1. LATAR BELAKANG PROYEK

{{uat_document.custom_fields.project_background}}

Sumber requirement/report:
{{#source_refs}}
- {{source}} / {{role}}: {{title}} {{url}}
{{/source_refs}}

# 2. TUJUAN PROYEK

{{uat_document.objective}}

# 3. SOLUTION OVERVIEW PROYEK

## A. Ruang Lingkup

{{#uat_document.scope}}
- {{.}}
{{/uat_document.scope}}

## B. Batasan Dan Asumsi

{{#uat_document.known_limitations}}
- {{.}}
{{/uat_document.known_limitations}}

# 4. TUJUAN USER ACCEPTANCE TEST

User Acceptance Test bertujuan memverifikasi bahwa sistem telah memenuhi persyaratan pengguna, memvalidasi kelayakan sistem pada lingkungan yang disepakati, dan mendapatkan persetujuan pengguna atas kecocokan serta kinerja sistem.

Rekomendasi sign-off: `{{signoff_recommendation.recommendation}}`

Alasan: {{signoff_recommendation.reason}}

# 5. LINGKUNGAN USER ACCEPTANCE TEST

Environment testing: {{uat_document.environment}}

| Nama Sistem | Environment | IP/URL | Catatan |
| ----- | ----- | ----- | ----- |
| {{uat_document.custom_fields.system_name}} | {{uat_document.environment}} | {{uat_document.custom_fields.environment_url}} | {{uat_document.custom_fields.environment_notes}} |

# 6. PERSIAPAN PENGUJIAN

{{#uat_document.custom_fields.test_preparation}}
- {{.}}
{{/uat_document.custom_fields.test_preparation}}

# 7. SKENARIO DAN HASIL PENGUJIAN

{{metadata.project}}
Modul: {{metadata.feature}}
Tanggal Pengujian: {{metadata.updated_at}}

| No. | Skenario | Langkah-Langkah | Hasil yang diharapkan | Hasil yang dicapai | Keterangan |
| :---: | :--- | :--- | :--- | :--- | :--- |
{{#custom_fields.test_scenarios}}
| {{no}} | {{scenario}} | {{steps}} | {{expected_result}} | {{actual_result}} | {{status}} |
{{/custom_fields.test_scenarios}}

# 8. KRITERIA KEBERHASILAN

| No. | Kriteria Keberhasilan | Metrik Pengukuran | Target |
| :---: | :--- | :--- | :--- |
| 1 | Skenario UAT dieksekusi sesuai cakupan | Execution completion rate | {{execution_metrics.execution_completion_rate}}% |
| 2 | Defect kritikal/high terkendali | Critical/High issue count | {{execution_metrics.critical_high_issue_count}} |
| 3 | Coverage kritikal sesuai kebutuhan pengguna | Critical coverage status | {{coverage.critical_coverage_status}} |

# 9. PENCATATAN MASALAH DAN TINDAK LANJUT

| No. | Deskripsi Masalah | Prioritas | Tindak Lanjut | Tingkat Kesulitan | Status | Catatan Developer |
| :---: | :--- | :---: | :--- | :---: | :---: | :--- |
{{#issue_package.issues}}
| {{issue_id}} | {{title}} | {{priority}} | {{severity_normalization.reason}} | {{custom_fields.difficulty}} | {{submission_status}} | {{custom_fields.developer_note}} |
{{/issue_package.issues}}

# 10. EVALUASI DAN PENILAIAN

{{uat_document.conclusion}}

Ringkasan:
- Passed: {{execution_metrics.passed}}
- Failed: {{execution_metrics.failed}}
- Blocked: {{execution_metrics.blocked}}
- Untested: {{execution_metrics.untested}}
- Retest: {{execution_metrics.retest}}
- Pass rate: {{execution_metrics.pass_rate}}%
- Completion rate: {{execution_metrics.execution_completion_rate}}%

Open risks:
{{#uat_document.open_risks}}
- {{.}}
{{/uat_document.open_risks}}

# 11. LAMPIRAN

Evidence references:
{{#uat_document.evidence_refs}}
- {{.}}
{{/uat_document.evidence_refs}}

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
