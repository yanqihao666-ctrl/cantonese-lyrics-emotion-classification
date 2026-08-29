# Emotion Classification in Cantonese Lyrics

MSc Computer Science research project comparing classical machine-learning and Transformer-based models for multi-class emotion classification of Cantonese song lyrics.

## Emotion classes

- joy
- sadness
- anger
- calmness

The locked four-class taxonomy is used throughout the completed development
study. Labels are AI-assisted silver labels, not independent human-gold labels.

## Research questions

1. How do classical machine-learning and Transformer-based models compare on Cantonese lyric emotion classification?
2. How do character-level and word-level representations affect classical-model performance?
3. How does adding the expansion training corpus affect model performance under a fixed-test design?
4. What class-level and label-evidence patterns characterise model errors?

## Repository structure

```text
configs/       Experiment configuration
data/          Non-text metadata and dataset schema
docs/          Research plan, decisions and dissertation material
scripts/       Reproducible artifact builders
src/           Reusable implementation
tests/         Automated tests
results/       Final machine-readable metrics, predictions and figures
```

Raw copyrighted lyrics must not be committed or redistributed. Only metadata, labels, derived statistics and acquisition instructions should be shared unless licensing explicitly permits the text.

`data/cantopop_seed_metadata.csv` contains public bibliographic metadata imported from the Cantopop Corpus Project. It contains no lyric text and no emotion labels.

## Status

The expanded development project is complete. The frozen local dataset contains
6,000 lyric sections from 3,246 songs and 257 artists. It retains the original
680 research-assisted labels and adds 5,320 GitHub-sourced sections labelled by
a disclosed three-method AI consensus. Raw and derived lyric text remains under
git-ignored `data/research_only/` and must not be redistributed.

The final evaluation contains 15 shared artist-exclusive partitions (three
seeds by five folds), complete Classical ML baselines and imbalance treatments,
and 60 Transformer runs. The validation-selected Classical ML baseline achieved
mean test Macro-F1 0.352 (SD 0.015), XLM-R 0.368 (SD 0.029), and Chinese MacBERT
0.558 (SD 0.022). Balanced-loss MacBERT was the best imbalance variant at 0.579
(SD 0.013). In a separate 15-pair fixed-test comparison, expanding MacBERT's
training data increased Macro-F1 from 0.254 to 0.457 (mean delta +0.203; all 15
pairs improved). The final automated project audit passes every data, leakage,
coverage and artifact check. These are development results on AI-assisted
labels, not performance claims against an independent human-gold standard. See
the [final methodology](docs/final_project_methodology.md) and the
[final result tables](results/consensus_final_6000/report_tables/results_tables.md).

A post-completion robustness analysis now evaluates every primary model on five
predefined label-evidence subsets. MacBERT remains first after excluding all
418 three-way-tie labels, on the original-plus-unanimous subset, on unanimous
expansion labels alone, and on the 680-item original reviewed cohort. This
supports the internal model-family ordering but does not remove the absence of
human validation. See the
[silver-label robustness report](results/consensus_final_6000/silver_label_robustness/SILVER_LABEL_ROBUSTNESS_REPORT.md)
and the [final methodology](docs/final_project_methodology.md).

## Data availability and repository scope

This submission repository is a copyright-safe research snapshot. It contains
the implementation, configuration, tests, non-text metadata, final case-level
prediction outputs, aggregate statistics and figures needed to inspect the
reported experiments. It intentionally excludes raw and transformed lyric
text, annotation workbooks, virtual environments, downloaded model weights and
temporary build artefacts. Row-level qualitative review queues are also excluded
because free-text evidence fields may contain short restricted excerpts. Dataset reconstruction therefore requires lawful
access to the documented sources and local placement under the ignored data
directories.

## Verification status

The copyright-safe submission snapshot passes `127` automated tests. Two
integration tests are skipped because they require the deliberately excluded
restricted lyric datasets. On the complete local research environment, those
audits were run against the frozen 6,000-section dataset before export.

## Reproducible workflow

Create the environment and run all automated tests:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pytest -q
```

Audit the literature matrix, search log, BibTeX metadata and author-year citations:

```powershell
.\.venv\Scripts\python.exe -m src.audit_references --output results\reference_audit.json
```

Audit which dissertation claims are already supported and which remain blocked:

```powershell
.\.venv\Scripts\python.exe -m src.audit_dissertation_evidence --output results\dissertation_evidence_audit.json
```

Audit canonical research questions, chapter numbering, terminology, model revisions, evaluation constants and result-placeholder scope:

```powershell
.\.venv\Scripts\python.exe -m src.audit_dissertation_consistency --output results\dissertation_consistency_audit.json
```

Reproduce the silver-label sensitivity analysis without retraining any model:

```powershell
.\.venv\Scripts\python.exe -m src.silver_label_robustness data\research_only\expanded_6000_consensus_final.csv results\consensus_final_6000\analysis\primary_test_predictions.csv --output-dir results\consensus_final_6000\silver_label_robustness
```

Audit the local licensed corpus and generate derived leakage statistics:

```powershell
.\.venv\Scripts\python.exe -m src.data_audit data\licensed_text\cantopop_corpus_sections.csv --output-dir results\section_data_audit
```

Generate independent, blinded annotation sheets. The researcher key contains `case_id`, parent-song and identifying metadata; annotator files expose only anonymous IDs, section text and annotation fields.

```powershell
.\.venv\Scripts\python.exe -m src.prepare_annotation data\licensed_text\cantopop_corpus_sections.csv --output-dir data\annotation_pilot
```

The local annotation package also contains formatted `annotator_1_blind.xlsx` and `annotator_2_blind.xlsx` workbooks with label/confidence validation, ambiguity flags and live completion summaries. Annotators should receive only their own workbook. The researcher key must remain separate.

Generate the bilingual supervisor-review pack:

```powershell
.\.venv\Scripts\python.exe scripts\build_annotator_pack.py
```

The output is `deliverables/Annotator_Pack_DRAFT.docx`. Do not distribute it until every placeholder is completed and the ethics route, consent wording and Traditional-Chinese translation have been approved. The full render and accessibility record is in [annotator pack validation](docs/annotator_pack_validation.md).

After both completed sheets are returned, calculate raw agreement, Cohen's kappa and the adjudication queue directly from Excel:

```powershell
.\.venv\Scripts\python.exe -m src.annotation_agreement data\annotation_pilot\annotator_1_blind.xlsx data\annotation_pilot\annotator_2_blind.xlsx --output-dir results\annotation_agreement
```

The queue includes every label disagreement, any confidence score of 1–2 and every ambiguity/exclusion flag. Complete all six decision fields in `adjudication_required.csv`: adjudicated label, include/exclude decision, exclusion reason where applicable, rationale, adjudicator code and date.

Freeze the gold dataset only after the queue is complete. This command preserves both original workbooks, writes a lyric-free decision audit and hashes every input and output:

```powershell
.\.venv\Scripts\python.exe -m src.build_gold_dataset data\licensed_text\cantopop_corpus_sections.csv data\annotation_pilot\researcher_key.csv data\annotation_pilot\annotator_1_blind.xlsx data\annotation_pilot\annotator_2_blind.xlsx results\annotation_agreement\adjudication_required.csv --output-dir data\processed --item-id-column case_id
```

Do not continue unless `gold_freeze_metadata.json` confirms all four labels and at least five distinct artist groups per label. The full rule set is in [docs/gold_label_freeze_protocol.md](docs/gold_label_freeze_protocol.md).

The classical runner must only be executed on the frozen labelled file whose label column contains the four documented classes:

```powershell
.\.venv\Scripts\python.exe -m src.classical_experiment data\processed\gold_labelled.csv --label-column emotion --id-column case_id --output-dir results\classical
```

The runner writes aggregate results, fold-level metrics, every out-of-fold prediction and environment metadata. These files - not expected values - are the only valid source for dissertation result tables.

Create a fixed 70/15/15-style split with exclusive artist groups before Transformer training:

```powershell
.\.venv\Scripts\python.exe -m src.make_splits data\processed\gold_labelled.csv data\processed\gold_labelled_split.csv --label-column emotion --group-column artist
```

The split optimiser records requested versus achieved sizes, class counts and artist-group counts. Run one auditable Transformer configuration at a time; repeat the command for every declared model and seed:

```powershell
.\.venv\Scripts\python.exe -m src.transformer_experiment data\processed\gold_labelled_split.csv --model FacebookAI/xlm-roberta-base --model-revision e73636d4f797dec63c3081bb6ed5c7b0bb3f2089 --seed 42 --output-dir results\transformer
```

The Transformer runner validates that every split contains all four classes and rejects artist leakage. It keeps validation metrics separate from the final test metrics and saves only metadata - not lyric text - in the test-prediction artifact.

For the formal comparison on a small corpus, generate repeated artist-exclusive partitions shared by every model family:

```powershell
.\.venv\Scripts\python.exe -m src.make_cv_manifest data\processed\gold_labelled.csv data\processed\cv_manifest.csv --id-column case_id --label-column emotion --group-column artist --seeds 13 42 97 --outer-folds 5 --inner-folds 5 --materialised-dir data\processed\cv_partitions
```

The command refuses to continue when a class occurs in too few distinct artist groups or when any train, validation or test role lacks a class. It writes one materialised input per seed and outer fold. Run the classical models on each file with the predeclared role column:

```powershell
.\.venv\Scripts\python.exe -m src.classical_experiment data\processed\cv_partitions\seed_13_fold_1.csv --fixed-split --seeds 13 --id-column case_id --output-dir results\classical\seed_13_fold_1
```

RQ3 uses seed 42's five outer folds, four nested artist-level fractions and three fixed models. Plan all 60 jobs (20 subsets × Character-SVM/XLM-R/MacBERT) while keeping validation/test cases unchanged:

```powershell
.\.venv\Scripts\python.exe -m src.run_learning_curve_matrix data\processed\cv_partitions --subsets-root data\processed\learning_curves --output-root results\formal\learning_curves --dry-run
```

Remove `--dry-run` after checking that the manifest reports 5 base partitions, 20 subsets and 60 unique jobs. The runner includes seed, fold and fraction in every output identity and can resume safely. Then validate identical evaluation coverage and generate the curve:

```powershell
.\.venv\Scripts\python.exe -m src.learning_curve_analysis results\formal\learning_curves\classical results\formal\learning_curves\transformer --output-dir results\formal\learning_curve_analysis
```

The curve reports individual fold points plus mean lines and actual training-section/source-song/artist ranges. See [docs/learning_curve_protocol.md](docs/learning_curve_protocol.md) for scope and infeasibility rules.

After checking the materialised manifest, plan all 45 formal jobs (15 classical-matrix runs and 30 required Transformer runs) without starting training:

```powershell
.\.venv\Scripts\python.exe -m src.run_experiment_matrix data\processed\cv_partitions --output-root results\formal --dry-run
```

Remove `--dry-run` to execute sequentially. The matrix persists status after every job and skips runs whose expected metadata already exists, so an interrupted experiment can resume without overwriting completed evidence.

Use the [formal experiment execution checklist](docs/formal_experiment_execution_checklist.md) as the required gate after annotation. The matrix design and its synthetic validation are documented in [formal experiment matrix orchestration](docs/experiment_matrix_orchestration.md).

Select one classical configuration separately inside every outer partition using validation macro-F1 only. A stable lexicographic key resolves exact validation ties; the selection audit shows that no test metric enters the choice:

```powershell
.\.venv\Scripts\python.exe -m src.select_classical_configuration results\formal\classical --output-dir results\formal\classical_validation_selection
```

Collect the validation-selected classical predictions and both required Transformers. The collector rejects duplicate predictions for the same model, section and partition:

```powershell
.\.venv\Scripts\python.exe -m src.collect_predictions results\formal\classical_validation_selection\selected_test_predictions.csv results\formal\transformer --output results\formal\primary_test_predictions.csv
```

Quantify partition variability, song-cluster bootstrap uncertainty and the three primary paired differences. The exact model scope is declared on the command line before analysis:

```powershell
.\.venv\Scripts\python.exe -m src.statistical_analysis results\formal\primary_test_predictions.csv --output-dir results\formal\primary_statistical_analysis --models classical::validation_selected transformer::FacebookAI/xlm-roberta-base transformer::hfl/chinese-macbert-base --analysis-label primary_family_comparison --bootstrap-iterations 2000 --permutations 5000
```

Generate the dissertation tables from those hash-matched artifacts. The generator recalculates the partition summary and refuses stale statistics or unequal model coverage:

```powershell
.\.venv\Scripts\python.exe -m src.generate_results_tables results\formal\primary_test_predictions.csv results\formal\primary_statistical_analysis --output-dir results\formal\report_tables
```

RQ2 is a separate predeclared multiplicity family. Collect all classical outputs, then compare character, word and combined TF-IDF using the same Linear SVM:

```powershell
.\.venv\Scripts\python.exe -m src.collect_predictions results\formal\classical --output results\formal\classical_test_predictions.csv
.\.venv\Scripts\python.exe -m src.statistical_analysis results\formal\classical_test_predictions.csv --output-dir results\formal\representation_statistical_analysis --models classical::character_tfidf::linear_svm classical::word_tfidf::linear_svm classical::combined_tfidf::linear_svm --analysis-label rq2_linear_svm_representations --bootstrap-iterations 2000 --permutations 5000
```

Holm correction is applied within each declared family, not across an opportunistic mixture of primary and exploratory questions. The procedure is documented in [docs/statistical_analysis_plan.md](docs/statistical_analysis_plan.md), and the full evidence flow in [docs/formal_reporting_pipeline.md](docs/formal_reporting_pipeline.md).

Generate consistent count/normalised confusion matrices and a fold-variability plot directly from the validated combined predictions:

```powershell
.\.venv\Scripts\python.exe -m src.evaluation_plots results\formal\primary_test_predictions.csv --output-dir results\formal\evaluation_figures
```

Create paired success/failure strata and a section-level qualitative review queue. Lyric text is intentionally excluded from the output:

```powershell
.\.venv\Scripts\python.exe -m src.error_analysis results\formal\primary_test_predictions.csv --metadata data\processed\gold_labelled.csv --output-dir results\formal\error_analysis
```

Generate publication-ready corpus-audit figures:

```powershell
.\.venv\Scripts\python.exe -m src.plot_data_audit results\section_data_audit\corpus_audit_rows.csv --output-dir results\section_data_audit\figures
```

Generate zero-shot candidate labels for review prioritisation. These outputs are weak/silver suggestions only and must remain hidden from independent human annotators:

```powershell
.\.venv\Scripts\python.exe -m src.silver_label_zero_shot data\licensed_text\cantopop_corpus_sections.csv data\interim\cantopop_candidate_labels_zero_shot.csv
.\.venv\Scripts\python.exe -m src.candidate_label_audit data\interim\cantopop_candidate_labels_zero_shot.csv --output-dir results\candidate_label_audit
```

The zero-shot runner checkpoints each hypothesis template so interrupted CPU runs can resume safely. The audit reports class collapse, prompt sensitivity, confidence margins and the cases that need human review first. Candidate distributions are not model-performance results.

Audit tokenizer coverage and 512-token truncation before formal fine-tuning:

```powershell
.\.venv\Scripts\python.exe -m src.tokenizer_audit data\licensed_text\cantopop_corpus_sections.csv --id-column case_id --output-dir results\section_tokenizer_audit --models FacebookAI/xlm-roberta-base hfl/chinese-macbert-base indiejoseph/bert-base-cantonese --revisions e73636d4f797dec63c3081bb6ed5c7b0bb3f2089 a986e004d2a7f2a1c2f5a3edef4e20604a974ed1 65442e1c2227c3d5394dfd44ea52400d0ec73679
```

Only derived counts and non-text metadata are saved. The section-level audit found no input over 512 tokens for any of the three audited tokenizers. Formal Transformer runs retain the tested head-tail safeguard, but it should not activate on the current 680 sections.

## Research controls

- [COMP5200M marking alignment](docs/marking_alignment.md)
- [Dissertation master plan and proportional word budget](docs/dissertation_master_plan.md)
- [Dissertation claim-to-artifact register](docs/dissertation_evidence_register.csv)
- [Dissertation terminology and consistency contract](docs/dissertation_consistency_contract.md)
- [Literature synthesis](docs/literature_synthesis.md)
- [Introduction chapter draft](docs/introduction_draft.md)
- [Emotion taxonomy rationale](docs/emotion_taxonomy_rationale.md)
- [Methodology and experimental-design draft](docs/methodology_draft.md)
- [Implementation and validation chapter draft](docs/implementation_chapter_draft.md)
- [Artifact-driven results chapter template](docs/results_chapter_template.md)
- [Formal reporting and multiplicity pipeline](docs/formal_reporting_pipeline.md)
- [RQ3 learning-curve protocol](docs/learning_curve_protocol.md)
- [Learning-curve synthetic validation](docs/learning_curve_smoke_validation.md)
- [Zero-shot candidate-label findings](docs/candidate_label_findings.md)
- [Transformer model and revision audit](docs/transformer_model_audit.md)
- [Transformer GPU smoke validation](docs/transformer_smoke_validation.md)
- [GPU setup and validation](docs/gpu_setup_and_validation.md)
- [Predeclared statistical analysis plan](docs/statistical_analysis_plan.md)
- [Predeclared hyperparameter protocol](docs/hyperparameter_protocol.md)
- [Formal experiment execution checklist](docs/formal_experiment_execution_checklist.md)
- [Formal experiment matrix orchestration](docs/experiment_matrix_orchestration.md)
- [Software architecture and validation inventory](docs/software_architecture_and_validation.md)
- [Project management and decision log](docs/project_log.md)
- [Legal, ethical and professional assessment](docs/legal_ethical_assessment.md)
- [Draft annotator information sheet](docs/annotator_information_sheet_draft.md)
- [Draft annotator consent form](docs/annotator_consent_form_draft.md)
- [Traditional-Chinese annotation-guide draft](docs/annotation_guide_zh_hant_draft.md)
- [Annotator pack generation and validation](docs/annotator_pack_validation.md)
- [Gold-label adjudication and freeze protocol](docs/gold_label_freeze_protocol.md)
- [Generative-AI and academic-integrity record](docs/ai_use_and_academic_integrity.md)
- [Data feasibility and expansion decisions](docs/data_feasibility_report.md)
- [Draft access request for the 827-song corpus](docs/corpus_access_request_draft.md)

## GPU note

The pinned `torch` dependency is platform neutral and may resolve to a CPU build. This machine has been validated with `torch 2.13.0+cu130` on an RTX 5060 Laptop GPU. Re-run the device operation check before formal training:

```powershell
.\.venv\Scripts\python.exe -m src.gpu_environment_check --require-cuda --matrix-size 2048 --output results\environment\gpu_validation.json
```

See [GPU setup and validation](docs/gpu_setup_and_validation.md) for the Windows path-length workaround and reproducibility notes.

Validate the declared full-length Transformer training shape with a real mixed-precision forward/backward pass:

```powershell
.\.venv\Scripts\python.exe -m src.transformer_memory_smoke --model FacebookAI/xlm-roberta-base --model-revision e73636d4f797dec63c3081bb6ed5c7b0bb3f2089 --batch-size 4 --sequence-length 512 --output results\environment\transformer_memory_smoke.json
```
