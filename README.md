An Enhanced Phishing Detection System Using An Integrated Behavioural-Linguistic Machine Learning Approach  

FINAL YEAR PROJECT



Joseph Oluwatimilehin Ambali  (2023/B/CYB/0113) - 30023809

&#x20;

DEPARTMENT OF CYBERSECURITY

SCHOOL OF COMPUTING

MIVA OPEN UNIVERSITY, ABUJA, NIGERIA.



SUPERVISED BY:

Dr David Ijegwa Acheme

FEBRUARY, 2026









\### A complete guide to reproduce the proposed Multimodal Phishing Detection System



\---



**## Table of Contents**



**1. What This Project Does**

**2. How It Works — System Overview**

**3. The Four Models Explained**

**4. Datasets You Need**

**5. Setting Up Google Colab**

**6. Step-by-Step: Running the Training Code**

**7. What Each Training Cell Does**

**8. Understanding the Output Plots**

**9. Step-by-Step: Running the Flask Web App**

**10. Using the Web App](#10-using-the-web-app)**

**11. Understanding SHAP**

**12. Files Saved After Training**

**13. Common Errors and Fixes**

**14. Key Design Decisions**



\---



\## 1. What This Project Does



This system detects phishing emails and URLs using fusion of Structural + Linguistic + Behavioural signals:



\- \*\*The structure of a URL\*\* — analysed by a neural network trained on URL features

\- \*\*The words in the email\*\* — analysed by a machine learning LSTM model

\- \*\*Behavioural How the user moves their mouse/trackpad\*\* — unusual hesitation patterns can signal a suspicious email



The system combines all three signals into a single risk score and shows you \_exactly why\_ it made its decision using SHAP explanations.



\---



\## 2. How It Works — System Overview



```

User inputs email text + URL + mouse movements

&#x20;             │

&#x20;   ┌─────────┼──────────┐

&#x20;   ▼         ▼          ▼

┌────────┐ ┌──────┐ ┌─────────────┐

│  LSTM  │ │ URL  │ │   Mouse     │

│  Text  │ │Dense │ │  RF Model   │

│ Model  │ │Model │ │             │

└───┬────┘ └──┬───┘ └──────┬──────┘

&#x20;   │         │             │

&#x20;   └─────────┼─────────────┘

&#x20;             ▼

&#x20;   Weighted Fusion Score

&#x20;   (55% URL + 30% Text + 15% Mouse)

&#x20;             │

&#x20;             ▼

&#x20;   SHAP Explanations

&#x20;             │

&#x20;             ▼

&#x20;    Final Verdict + Risk %

```



There are also \*\*3 comparison models\*\* (Naive Bayes, SVM, Random Forest) trained on the URL dataset so you can benchmark all approaches side by side.



\---



\## 3. The Four Models Explained



\### Model 1 — LSTM Text Model



\- \*\*What it learns:\*\* The language patterns of phishing emails (urgency words, fake sender names, suspicious phrasing)

\- \*\*Input:\*\* Raw email text + 4 numerical features (URL length, domain age, mouse speed variance, click hesitation)

\- \*\*Architecture:\*\* Embedding layer → LSTM (64 units) → Dense layers → sigmoid output

\- \*\*Output:\*\* Probability 0–1 that the email is phishing



\### Model 2 — URL Dense Neural Network



\- \*\*What it learns:\*\* Structural patterns in URLs (number of dots, hyphens, length, special characters, whether it uses a known shortener, etc.)

\- \*\*Input:\*\* 87 hand-crafted URL features extracted automatically

\- \*\*Architecture:\*\* Dense(128) → Dropout → Dense(64) → Dropout → sigmoid output

\- \*\*Output:\*\* Probability 0–1 that the URL is phishing



\### Model 3 — Mouse Behaviour Model (Random Forest)



\- \*\*What it learns:\*\* Whether the user's mouse movement looks like someone reading a suspicious email (slow, hesitant, erratic movement)

\- \*\*Input:\*\* Mouse speed variance + click hesitation (plus polynomial interaction features)

\- \*\*Architecture:\*\* Random Forest with 300 trees, polynomial feature expansion (degree 2)

\- \*\*Output:\*\* Probability 0–1 that the behaviour is suspicious



\### Model 4 — Multimodal Fusion



\- \*\*What it does:\*\* Combines the LSTM text model (70% weight) and the mouse model (30% weight) into a single score

\- \*\*This is the main detection model for email analysis\*\*



\### Comparison Models (URL dataset only)



| Model                         | Why it is included                  |

| ----------------------------- | ---------------------------------- |

| Naive Bayes                   | Fast probabilistic baseline        |

| SVM (LinearSVC + Calibration) | Strong linear classifier benchmark |

| Random Forest                 | Best tree-based benchmark          |



\---



\## 4. Datasets You Need



You need \*\*three datasets\*\* placed in your Google Drive before running anything.



\### Dataset 1 — Phishing Email Dataset



\- \*\*File:\*\* `phishing\_email.csv`

\- \*\*Required columns:\*\*

&#x20; - `text\_combined` — the full email text

&#x20; - `label` — `1` for phishing, `0` for legitimate

\- \*\*Optional columns\*\* (will be created as zeros if missing):

&#x20; - `url\_length`, `domain\_age`, `mouse\_speed\_variance`, `click\_hesitation\_ms`

\- \*\*Where to get it:\*\* Kaggle — Phishing Email Dataset](https://www.kaggle.com/datasets/naserabdullahalam/phishing-email-dataset)



\### Dataset 2 — URL Features Dataset



\- \*\*File:\*\* `dataset\_phishing.csv`

\- \*\*Required columns:\*\*

&#x20; - `url` — the raw URL string

&#x20; - `status` — `"phishing"` or `"legitimate"`

&#x20; - All other columns should be numeric URL features

\- \*\*Where to get it:\*\* Kaggle — Web Page Phishing Detection](https://www.kaggle.com/datasets/shashwatwork/web-page-phishing-detection-dataset)



\### Dataset 3 — Mouse Dynamics Dataset



\- \*\*Folder:\*\* `Mouse-Dynamics-Challenge-master`

\- \*\*Expected subfolders:\*\*

&#x20; - `training\_files/` — CSV files of mouse movement recordings

&#x20; - `test\_files/` — additional CSV files

\- \*\*Required columns in each CSV:\*\* `x`, `y`, `client\_timestamp`, `state`

\- \*\*Where to get it:\*\* GitHub — Mouse Dynamics Challenge](https://github.com/balabit/Mouse-Dynamics-Challenge)



\### Checking Your Drive Structure



Before running, your Google Drive directory setup should look like this:



```

MyDrive/

├── phishing\_email/

│   └── phishing\_email.csv

├── dataset\_phishing.csv

└── Mouse-Dynamics-Challenge-master/

&#x20;   ├── training\_files/

&#x20;   │   ├── user1\_session1.csv

&#x20;   │   └── ...

&#x20;   └── test\_files/

&#x20;       ├── user1\_session2.csv

&#x20;       └── ...

```



\---



\## 5. Setting Up Google Colab



\### Step 1 — Open a New Colab Notebook



Go to colab.research.google.com](https://colab.research.google.com) and click \*\*New Notebook\*\*.



\### Step 2 — Change Runtime to GPU



1\. Click \*\*Runtime\*\* in the top menu

2\. Click \*\*Change runtime type\*\*

3\. Set \*\*Hardware accelerator\*\* to `GPU` (T4 GPU is fine)

4\. Click \*\*Save\*\*



> ⚠️ GPU is needed for the LSTM and URL Dense models to train in reasonable time. Without GPU, training will take much longer.



\### Step 3 — Copy the Code



The project has \*\*two separate code files:\*\*



\- `training\_cells.py` — run this first in Colab to train and save all models

\- `flask\_app.py` — run this second in Colab to launch the web app



Copy each file's contents into separate cells in your Colab notebook.



\---



\## 6. Step-by-Step: Running the Training Code



Follow these steps \*\*in order\*\*. Do not skip any cell.



\### Step 1 — Mount Google Drive (Cell 1)



```python

from google.colab import drive

drive.mount('/content/drive')

```



\- A popup will appear asking you to authorise Google Drive access

\- Click \*\*Connect to Google Drive\*\* and sign in

\- You will see `Mounted at /content/drive` when it works



\### Step 2 — Install Libraries (Cell 1 continued)



```

!pip install shap seaborn joblib -q

```



\- This installs SHAP (explainability), Seaborn (plots), and Joblib (saving models)

\- The `-q` flag keeps output quiet. It takes about 1–2 minutes.



\### Step 3 — Run Imports (Cell 2)



\- This imports every library the project needs

\- You will see `✅ All imports successful` if everything is working

\- \*\*Critical:\*\* This cell uses `%matplotlib inline` — this is what makes plots show inside the notebook instead of only saving as files



\### Step 4 — Define Plot Helper Functions (Cell 3)



\- This cell defines three reusable functions: `plot\_confusion\_matrix`, `plot\_classification\_report\_heatmap`, and `plot\_roc\_curve`

\- No output is produced — these are just function definitions



\### Step 5 — Load the Email Dataset (Cell 4)



\- Reads `phishing\_email.csv` from Drive

\- Fills missing mouse feature columns with 0 (this is intentional — see Key Design Decisions](#14-key-design-decisions))

\- Expected output example: `Email dataset: 18000 rows, {0: 9000, 1: 9000}`



\### Step 6 — Load the URL Dataset (Cell 5)



\- Reads `dataset\_phishing.csv` from Drive

\- Converts the `status` column to a binary `label` column

\- Drops the raw `url` string and `status` columns (keeps only numeric features)

\- Expected output example: `URL dataset: 11430 rows, {0: 5715, 1: 5715}`



\### Step 7 — Load Mouse Data (Cell 6)



\- Walks through every CSV file in the mouse training and test folders

\- Extracts 4 features per file: average speed, speed variance, number of clicks, average hesitation

\- Saves the population statistics to `mouse\_stats.json` (used later in the web app)

\- Expected output: `Mouse stats saved: {'avg\_speed\_mean': ..., 'avg\_speed\_std': ..., ...}`



\### Step 8 — Train Mouse Model (Cell 7)



\- Trains a Random Forest on `mouse\_speed\_variance` and `click\_hesitation\_ms` from the email dataset

\- Adds polynomial features (degree 2) to capture interaction patterns

\- Plots confusion matrix, classification heatmap, and ROC curve

\- \*\*Note:\*\* Because both mouse columns in the email dataset are zero (synthetic), this model learns the baseline distribution — it will improve when real mouse data is fed at inference time



\### Step 9 — Prepare Text Data (Cell 8)



\- Splits the email dataset 80/20 into training and test sets

\- Tokenises the `text\_combined` column (converts words to integer sequences)

\- Pads all sequences to length 200

\- Scales the 4 numerical features with `StandardScaler`



\### Step 10 — Train LSTM Text Model (Cell 9)



\- Builds a dual-input Keras model: sequence input + numerical input

\- Trains for 6 epochs with batch size 32

\- Validation split is 10% of the training data

\- You will see epoch-by-epoch accuracy and loss printed



\### Step 11 — Evaluate LSTM Model (Cell 10)



\- Generates predictions on the test set

\- Prints the classification report

\- Plots and saves confusion matrix, heatmap, and ROC curve



\### Step 12 — Multimodal Fusion (Cell 11)



\- Combines LSTM text predictions (70%) with mouse model predictions (30%)

\- Uses the same test set split (same `random\_state=42`) to ensure alignment

\- Plots all three evaluation charts for the fused model



\### Step 13 — Train URL Models (Cells 12–15)



Run each cell in order:



\- \*\*Cell 12:\*\* URL Dense Neural Network (8 epochs)

\- \*\*Cell 13:\*\* Naive Bayes

\- \*\*Cell 14:\*\* SVM with calibration (3-fold cross-validation during calibration)

\- \*\*Cell 15:\*\* Random Forest (200 trees)



Each cell prints its own classification report and generates 3 plots.



\### Step 14 — Summary \& All-Model Comparison (Cell 16)



\- Creates a summary table of all 7 models

\- Generates a heatmap comparing all models across all metrics

\- Generates a bar chart for each metric

\- Generates a single ROC curve plot with all 7 models overlaid



\### Step 15 — SHAP Explanations (Cell 17)



\- Generates SHAP beeswarm plots and bar charts for:

&#x20; - URL Dense Neural Network (uses `GradientExplainer`)

&#x20; - Random Forest (uses `TreeExplainer`)

\- If this cell throws an error, it prints the full traceback but \*\*does not stop the rest of training\*\*



\### Step 16 — Save All Models (Cell 19)



\- This is the most important cell — everything needed by the web app is saved here

\- Do not close Colab or let the runtime disconnect before this cell finishes



\---



\## 7. What Each Training Cell Does



| Cell | Purpose                         | Output                      |

| ---- | ------------------------------- | --------------------------- |

| 1    | Mount Drive + install packages  | Drive access                |

| 2    | Import all libraries            | `✅ All imports successful` |

| 3    | Define plot helper functions    | (no output)                 |

| 4    | Load email dataset              | Row/label count             |

| 5    | Load URL dataset                | Row/label count             |

| 6    | Load mouse data + compute stats | `mouse\_stats.json`          |

| 7    | Train Mouse RF model            | 3 plots + report            |

| 8    | Tokenise + split email data     | (no output)                 |

| 9    | Build + train LSTM model        | Epoch logs                  |

| 10   | Evaluate LSTM                   | 3 plots + report            |

| 11   | Multimodal fusion               | 3 plots + report            |

| 12   | Train URL Dense model           | 3 plots + report            |

| 13   | Train Naive Bayes               | 3 plots + report            |

| 14   | Train SVM                       | 3 plots + report            |

| 15   | Train Random Forest             | 3 plots + report            |

| 16   | All-model summary               | 3 summary plots             |

| 17   | SHAP explanations               | 4 SHAP plots                |

| 18   | Save all artefacts              | 12 model files              |



\---



\## 8. Understanding the Output Plots



\### Confusion Matrix (two panels)



\- \*\*Left panel — Counts:\*\* Shows exact numbers of correct and incorrect predictions

&#x20; - Top-left = True Negatives (correctly called Legitimate)

&#x20; - Top-right = False Positives (Legitimate incorrectly called Phishing)

&#x20; - Bottom-left = False Negatives (Phishing missed)

&#x20; - Bottom-right = True Positives (correctly caught Phishing)

\- \*\*Right panel — Normalised:\*\* Shows the same as percentages of each true class. Values on the diagonal close to 100% mean the model is performing well.



\### Classification Report Heatmap



A colour-coded table showing three metrics for each class and their averages:



\- \*\*Precision:\*\* Of everything the model called Phishing, how many actually were? (avoids false alarms)

\- \*\*Recall:\*\* Of all actual Phishing emails, how many did the model catch? (avoids misses)

\- \*\*F1-Score:\*\* The harmonic mean of precision and recall — the single best summary metric



\### ROC Curve



\- The curve shows the trade-off between catching real phishing (True Positive Rate) and falsely flagging legitimate emails (False Positive Rate)

\- \*\*AUC (Area Under Curve):\*\* A perfect model scores 1.0. Random guessing scores 0.5. Higher is better.

\- \*\*Red dot:\*\* The optimal threshold point where the gap between TPR and FPR is largest



\### All-Models ROC Comparison



\- All 7 models plotted on one chart — lets you visually compare which model has the best AUC

\- Models using the URL dataset and the email dataset are both shown (they use different test sets, so treat AUC comparisons across those groups as indicative only)



\---



\## 9. Step-by-Step: Running the Flask Web App



The Flask app runs \*\*in the same Colab session\*\* after training. It starts a local web server and exposes it publicly using ngrok.



\### Step 1 — Install Flask and ngrok (first line of Flask cell)



```

!pip install flask pyngrok shap -q

```



\### Step 2 — Set Your ngrok Token



In the Flask cell, find this line:



```python

ngrok.set\_auth\_token("YOUR\_TOKEN\_HERE")

```



\- Go to dashboard.ngrok.com](https://dashboard.ngrok.com) and sign up for a free account

\- Copy your \*\*Authtoken\*\* from the dashboard

\- Replace `"YOUR\_TOKEN\_HERE"` with your actual token



\### Step 3 — Run the Flask Cell



\- Run the entire Flask cell as one block

\- Wait for this line to appear in the output:



```

✅ App live at: NgrokTunnel: "https://xxxx.ngrok.io"

```



\- Click that URL to open the web app in your browser



\### Step 4 — Keep the Colab Tab Open



\- The Flask server runs inside Colab — if you close the tab or the runtime times out, the app stops

\- Colab free tier disconnects after \~90 minutes of inactivity



\---



\## 10. Using the Web App



\### Entering an Email



1\. Paste the email text into the large textarea

2\. \*\*Move your mouse slowly over the text\*\* as if you are reading it — this captures your mouse behaviour pattern

3\. The status bar below the text area shows how many movement points have been captured



\### Entering a URL (Optional but Recommended)



\- Paste a URL from the email into the \*\*URL to check\*\* field

\- If you leave this blank, only the text and mouse models will run (SHAP will not appear)

\- Entering a URL unlocks the full three-model fusion and XAI explanations



\### Domain Age (Optional)



\- If you know how old the domain is (in days), enter it

\- A domain registered last week (7 days) is far more suspicious than one registered 10 years ago (3650 days)

\- Leave blank if unknown — it defaults to 0



\### Reading the Results



The result card shows four scores:



| Score                   | What it means                                          |

| ----------------------- | ------------------------------------------------------ |

| \*\*Overall risk %\*\*      | The final fused probability — above 50% = phishing     |

| \*\*Text / linguistic %\*\* | What the LSTM thought of the email wording alone       |

| \*\*URL structural %\*\*    | What the URL dense model thought of the link structure |

| \*\*Behavioural %\*\*       | How suspicious your mouse movement pattern appeared    |



\### Colour coding



\- 🔴 \*\*Red card with ⚠️\*\* — Model predicts phishing

\- 🟢 \*\*Green card with ✅\*\* — Model predicts legitimate



\### Progress bars



Each score has a bar — red bars indicate high risk, green bars indicate low risk.



\### Whitelisted domains



If the URL belongs to a known-safe domain (Google, PayPal, Microsoft, etc.), you will see an information note and the URL score is capped at 15% to avoid false positives on these trusted domains.



\---



\## 11. Understanding SHAP



Both explanations appear under \*\*Toggle XAI explanation\*\* after you submit a URL.



\### SHAP (SHapley Additive exPlanations)



\- Shows the \*\*global importance\*\* of each URL feature for this specific prediction

\- \*\*Red bars\*\* push the model toward predicting Phishing

\- \*\*Green bars\*\* push the model toward predicting Legitimate

\- \*\*Longer bar = stronger influence\*\* on the decision

\- Example: A bar for `nb\_hyphens` pointing red and long means "this URL has many hyphens, which strongly suggests phishing"



\### When XAI is not available



If you did not enter a URL, SHAP will not appear because they work on URL structural features — there is nothing to explain without a URL input.



\---



\## 12. Files Saved After Training



All files are saved to the \*\*Colab runtime's local disk\*\* (`/content/`). After training completes, download them or copy to Drive before the session ends.



| File                     | Size (approx) | Purpose                                    |

| ------------------------ | ------------- | ------------------------------------------ |

| `text\_model.keras`       | \~15 MB        | LSTM text model weights                    |

| `url\_model.keras`        | \~2 MB         | URL dense model weights                    |

| `mouse\_model.joblib`     | \~20 MB        | Mouse Random Forest                        |

| `nb\_model.joblib`        | <1 MB         | Naive Bayes                                |

| `svm\_model.joblib`       | \~5 MB         | SVM with calibration                       |

| `rf\_model.joblib`        | \~50 MB        | Random Forest (URL)                        |

| `url\_scaler.joblib`      | <1 MB         | StandardScaler for URL features            |

| `text\_num\_scaler.joblib` | <1 MB         | StandardScaler for text numerical features |

| `mouse\_scaler.joblib`    | <1 MB         | StandardScaler for mouse features          |

| `mouse\_poly.joblib`      | <1 MB         | Polynomial feature transformer             |

| `url\_background.joblib`  | \~1 MB         | Background data for SHAP                   |

| `tokenizer.json`         | \~2 MB         | Keras tokenizer vocabulary                 |

| `url\_columns.json`       | <1 MB         | List of URL feature column names           |

| `mouse\_stats.json`       | <1 KB         | Population mouse statistics                |

| `multimodal\_summary.csv` | <1 KB         | All model metrics table                    |



\### Plot Files



```

mouse\_confusion.png          mouse\_roc\_curve.png

mouse\_classification\_report.png

text\_confusion.png           text\_roc\_curve.png

text\_classification\_report.png

fusion\_confusion.png         fusion\_roc\_curve.png

fusion\_classification\_report.png

url\_confusion.png            url\_roc\_curve.png

url\_classification\_report.png

nb\_confusion.png             nb\_roc\_curve.png

svm\_confusion.png            svm\_roc\_curve.png

rf\_confusion.png             rf\_roc\_curve.png

multimodal\_summary.png

multimodal\_bar\_comparison.png

all\_models\_roc.png

shap\_url\_summary.png         shap\_url\_bar.png

shap\_rf\_summary.png          shap\_rf\_bar.png

```



\---



\## 13. Common Errors and Fixes



\### ❌ `FileNotFoundError: phishing\_email.csv`



\*\*Cause:\*\* The CSV file is not at the expected Drive path.

\*\*Fix:\*\* Double-check your Drive folder structure matches exactly what is shown in Section 4](#4-datasets-you-need). Path is case-sensitive.



\### ❌ Plots only saving as `.png` files, not showing in notebook



\*\*Cause:\*\* `matplotlib.use('Agg')` was called somewhere before `%matplotlib inline`.

\*\*Fix:\*\* Make sure `%matplotlib inline` is the \*\*first matplotlib line\*\* in Cell 2, and that `matplotlib.use('Agg')` does not appear in any training cell (it belongs only in the Flask cell).



\### ❌ Web app says phishing for legitimate emails



\*\*Cause:\*\* Mouse feature values at inference don't match what the scaler saw during training (all zeros).

\*\*Fix:\*\* In the Flask route, always pass `0.0` for `mouse\_speed\_variance` and `click\_hesitation\_ms` in the numerical features array sent to the text model. The mouse model has its own separate score computed independently.



\### ❌ `NgrokAuthError` or `Invalid authtoken`



\*\*Cause:\*\* Your ngrok token is missing or wrong.

\*\*Fix:\*\* Log in to dashboard.ngrok.com](https://dashboard.ngrok.com), copy your Authtoken, and replace the token string in the Flask cell.



\### ❌ `SHAP error: could not broadcast input array`



\*\*Cause:\*\* SHAP version mismatch with Keras output shape.

\*\*Fix:\*\* The code already handles this with `sv.squeeze(-1)` for 3D arrays. If it still fails, try installing a specific version: `!pip install shap==0.43.0 -q`



\### ❌ Colab runtime crashes during Random Forest training



\*\*Cause:\*\* 200 trees with no `max\_depth` limit can run out of RAM on large datasets.

\*\*Fix:\*\* Reduce `n\_estimators=100` and add `max\_depth=20` temporarily.



\### ❌ `ValueError: X has 2 features but PolynomialFeatures is expecting N features`



\*\*Cause:\*\* The poly transformer was fitted on a different number of features than it's receiving.

\*\*Fix:\*\* Make sure you always pass exactly `mouse\_speed\_variance, click\_hesitation\_ms]` to the mouse model pipeline — in the same order as during training.



\---



\## 14. Key Design Decisions



Understanding \_why\_ certain choices were made helps if you want to modify the code.



\### Why are `mouse\_speed\_variance` and `click\_hesitation\_ms` set to 0 in the email dataset?



The phishing email dataset does not contain real mouse movement recordings. Rather than impute random values (which would teach the model misleading patterns), we fill them with 0. This means the text model's numerical branch effectively ignores these two features during training. At inference time, we also pass 0 for these fields in the text model — keeping the distribution consistent. The actual live mouse data goes to the \*\*separate mouse model\*\* only, which has its own scaler fitted on real mouse recordings.



\### Why is the mouse model's weight only 30% in the fusion?



The mouse model is trained on synthetic zero data from the email dataset — so its standalone accuracy on that data is essentially measuring noise. The 30% weight reflects this uncertainty. As you gather more real labelled mouse data, this weight can be increased.



\### Why does the URL model use hand-crafted features instead of the raw URL string?



A raw URL string would need a character-level model (like a CNN or transformer). Instead, 87 features are extracted programmatically (number of hyphens, URL length, whether it uses a known URL shortener, etc.). This makes the model faster, more interpretable via SHAP, and effective even on very short URLs.



\### Why is LinearSVC wrapped in `CalibratedClassifierCV`?



`LinearSVC` does not natively output probability scores — it only outputs class labels. `CalibratedClassifierCV` wraps it with Platt scaling (3-fold cross-validation) to produce probability estimates needed for the ROC curve and fusion.



\### Why is the fusion score formula different depending on whether a URL is present?



```python

\# With URL:    55% URL + 30% text + 15% mouse

\# Without URL: 80% text + 20% mouse

```



The URL model is the strongest signal when a URL exists. Without a URL, the text model must carry more weight, and the mouse score provides a small behavioural check.



\### Why are known-safe domains whitelisted and capped at 15%?



The URL model was trained on a dataset where `paypal.com`, `google.com`, etc. may appear as phishing targets (e.g. `login-paypal.com`). When the real domain IS `paypal.com`, the model may still flag structural features. The whitelist prevents false positives on globally trusted domains.



\---



\## Quick Reference Card



```

FULL PIPELINE IN ORDER:

━━━━━━━━━━━━━━━━━━━━━━

1\.  Open Colab → enable GPU runtime

2\.  Mount Drive → verify dataset paths

3\.  Run Cell 1  (install packages)

4\.  Run Cell 2  (imports + %matplotlib inline)

5\.  Run Cell 3  (plot helpers)

6\.  Run Cell 4  (load email data)

7\.  Run Cell 5  (load URL data)

8\.  Run Cell 6  (load mouse data)

9\.  Run Cell 7  (train mouse model)

10\. Run Cell 8  (prepare text data)

11\. Run Cell 9  (train LSTM)

12\. Run Cell 10 (evaluate LSTM)

13\. Run Cell 11 (multimodal fusion)

14\. Run Cell 12 (URL dense model)

15\. Run Cell 13 (Naive Bayes)

16\. Run Cell 14 (SVM)

17\. Run Cell 15 (Random Forest)

18\. Run Cell 16 (summary plots)

19\. Run Cell 17 (SHAP)

20. Run Cell 19 (SAVE EVERYTHING ← don't skip this)

21. Run Flask cell → click ngrok URL → use app

```



\---



\_Built with TensorFlow/Keras · scikit-learn · SHAP · Flask · ngrok\_



