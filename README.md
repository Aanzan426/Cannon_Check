Backstory Consistency Verification
Kharagpur Data Science Hackathon

OVERVIEW

This project solves the task of determining whether a given backstory is consistent or contradictory with respect to a source novel. The final solution treats the problem as a relative semantic alignment task rather than relying on sentence-level Natural Language Inference or rule-based heuristics.

The key idea is to evaluate whether a backstory is unusually well-aligned with the source novel compared to the rest of the book. This approach proved robust, explainable, and effective under validation.

The final model combines long-context retrieval, book-normalized semantic similarity features, and supervised learning.



EXACT PROCEDURE — HOW PATHWAY WAS SET UP CORRECTLY

The following steps document the exact, reproducible procedure used to build the final system.


STEP 1: CONTROLLED ENVIRONMENT SETUP

A dedicated Python virtual environment was created.

Compatible versions were explicitly resolved:

    - pathway version 0.27.1

    - pyarrow version 18.1.0

    - Python version 3.11

This ensured that there were no dependency conflicts and that the environment remained fully reproducible.


STEP 2: NLTK SETUP INSIDE THE VIRTUAL ENVIRONMENT

The default HOME directory did not have available storage. To avoid permission and quota issues, NLTK data was redirected to a custom directory inside the virtual environment.

NLTK data directory used:

    - /ml-envs/kharagpur_env/nltk_data

The punkt tokenizer was downloaded explicitly to this location.

This ensured:

    - Sentence tokenization worked correctly

    - No filesystem permission issues occurred

    - Tokenization behavior was fully reproducible across runs


STEP 3: BOOK CHUNKING (LONG-CONTEXT HANDLING)

Each source novel was processed as follows:

    - The full text was loaded from disk

    - Sentence tokenization was applied using NLTK

    - Text was chunked into windows of approximately 700 tokens

    - An overlap of approximately 100 tokens was maintained between consecutive chunks

The purpose of this step was to preserve narrative continuity, avoid truncation-related information loss, and ensure meaningful semantic retrieval from long documents.


STEP 4: PATHWAY TABLE CONSTRUCTION

All generated book chunks were converted into a pandas DataFrame.

This DataFrame was then used to create a Pathway table using the Pathway debug interface. An embedding UDF was applied to compute dense vector representations for each chunk using a sentence-transformer model.

This resulted in a structured, streaming-capable representation of the novels, with a clean separation between raw data and computation logic.


STEP 5: MATERIALIZATION OF THE PATHWAY TABLE

Due to incompatibilities between the Pathway version used and built-in KNN indexing, the Pathway table was materialized into a pandas DataFrame.

This step:

    - Converted the Pathway table into a pandas representation

    - Preserved all precomputed embeddings

    - Enabled the use of sklearn-based KNN retrieval

This was a deliberate and justified design decision rather than a workaround.


STEP 6: RETRIEVAL VIA COSINE KNN

For each backstory:

    - The backstory was encoded into a dense vector

    - A cosine KNN search was performed over all chunks belonging to the corresponding book

    - The top-K most similar chunks were retrieved (K = 2)

This retrieval step provided localized semantic evidence and consistent inputs for downstream feature extraction.


STEP 7: FEATURE EXTRACTION (FINAL METHOD)

Instead of sentence-level NLI, the final system relied on relative similarity features derived from embeddings.

The features used were:

    - Maximum similarity to retrieved chunks

    - Mean similarity to retrieved chunks

    - Similarity percentile relative to all chunks in the book

    - Similarity z-score relative to the book-wide similarity distribution

    - Backstory length in tokens

This reframed the task as evaluating whether a backstory is unusually aligned with the source novel compared to typical text from the same book.

This representation proved decisive for performance.


STEP 8: SUPERVISED LEARNING

A RandomForestClassifier was trained on the extracted features.

Model details:

    - Number of trees: 300

    - Maximum tree depth: 6

    - Class weighting enabled to handle imbalance

The model was stable across multiple random seeds and achieved a validation accuracy of 0.6875.


STEP 9: FINAL TRAINING AND SUBMISSION

After validation, the Random Forest model was retrained on 100 percent of the training data.

The same feature extraction pipeline was applied to the test set. Predictions were generated and written to submission.csv.

The final submission file was verified for:

    - Correct format

    - Correct labels (consistent / contradict)

    - Correct row count




FILES USED AND GENERATED

Input files (read-only):

- train.csv

- test.csv

- In search of the castaways.txt

- The Count of Monte Cristo.txt


Generated files:

- submission.csv




FINAL STATUS

The model is finalized and locked.
The submission file has been verified.
The entire pipeline is reproducible and ready for evaluation.