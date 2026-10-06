

Readme · MD
🛡️ AI-Powered Scam & Phishing Detector
Detects scam messages and phishing URLs with classic NLP and machine learning, gives a 0-100 risk score, and explains why. Everything runs locally: URLs are analysed as text and never opened, and nothing is sent to any server.

Feature	How it works
Scam-message detection	TF-IDF (words + 2-word phrases) → Logistic Regression
URL / phishing detection	~30 lexical URL features → Random Forest
Explanations	Exact word contributions + plain-language rule flags + top model features
Risk score	Blend of ML probability, rule score and the riskiest link in a message
Dashboard	Streamlit app: analyzer, scan history, dataset stats, model evaluation
Evaluation	Accuracy, precision, recall, F1, ROC-AUC, confusion matrix, 5-fold CV, baseline comparison
Quick start
bash
git clone <your-repo-url> && cd scam-phishing-detector
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt

python -m src.train            # downloads public data, trains, evaluates (about 1 minute)
streamlit run app/streamlit_app.py
If you skip python -m src.train, the app trains the models automatically the first time it starts. No internet? Use python -m src.train --offline (synthetic data; see Datasets below).

Command-line use:

bash
python -m src.cli --url "paypal-secure-login.xyz/verify/account"
python -m src.cli --message "URGENT: your account is suspended. Verify now http://bank-verify.top"
Run the tests: pip install -r requirements-dev.txt && pytest -q

Project structure
scam-phishing-detector/
├── app/streamlit_app.py       # Streamlit UI (Analyze / Dashboard / Evaluation / About)
├── src/
│   ├── config.py              # paths, thresholds
│   ├── data_loader.py         # public dataset download + cache + offline fallback
│   ├── synthetic.py           # template-based data generators
│   ├── text_preprocessing.py  # message cleaning
│   ├── url_features.py        # URL feature extraction (no network calls)
│   ├── explain.py             # human-readable rule flags
│   ├── train.py               # training + evaluation + plots
│   ├── predictor.py           # inference, risk score, explanations
│   └── cli.py                 # command-line scanner
├── tests/                     # pytest suite
├── data/raw/                  # downloaded datasets (git-ignored)
├── models/                    # trained models (git-ignored)
├── reports/                   # metrics.json + plots (git-ignored)
└── .github/workflows/ci.yml   # runs tests on every push
How it works (step by step)
Data. data_loader.py fetches public datasets and caches them in data/raw/.
Message features. clean_text lowercases text and swaps links, long numbers and currency symbols for placeholder tokens, so the model learns patterns ("contains a link") instead of memorising values. TF-IDF then turns each message into weighted word and phrase counts.
Message model. Logistic Regression learns one weight per word/phrase. Because it is linear, the explanation is exact: contribution = weight × TF-IDF value. Naive Bayes is trained as a baseline for comparison.
URL features. url_features.py computes length, number of dots/hyphens/digits, IP-as-host, @ tricks, punycode, risky TLDs, shorteners, sensitive keywords, brand impersonation (e.g. paypal.com.evil.top) and more.
URL model. A Random Forest classifies those features. Logistic Regression is the baseline.
Rule engine. explain.py adds readable reasons (urgency, threats, OTP requests, prizes...) and a safety net for patterns the training data lacks. A genuine "do not share this code" OTP message is recognised as safer.
Risk score. URL: 0.75 × ML + 0.25 × rules. Message: 0.8 × ML + 0.2 × rules, or with links 0.55 × ML + 0.15 × rules + 0.30 × worst link. A clearly malicious link dominates the score. Levels: Low < 30, Medium 30-60, High ≥ 60 (edit in src/config.py).
Evaluation. A stratified 80/20 split gives held-out metrics; 5-fold cross-validation checks stability. Everything is written to reports/ and shown in the dashboard.
Datasets
Purpose	Source	Notes
Scam messages	SMS Spam Collection (UCI, CC BY 4.0; GitHub mirror as backup)	Real, ~5.5k messages (2011-era)
Phishing URLs	OpenPhish community feed	Real, current phishing URLs
Legitimate URLs	Tranco top sites	Real domains; typical page paths are attached
Modern-scam augmentation	src/synthetic.py	Template-based: bank KYC, delivery fees, OTP theft, crypto
Please check each source's terms of use before redistributing data or using it commercially.

Be honest about the numbers. If a download fails, the project falls back to synthetic data so it still runs. Synthetic data is template-based, so near-perfect scores on it say little about real-world performance. The dashboard's sidebar always shows which data each model was trained on.

Use your own URL dataset (recommended for serious use). Put a CSV named urls.csv in data/raw/ with columns url,label (labels 1/0 or phishing/legitimate, bad/good), then run python -m src.train. Public options include PhishTank and Kaggle phishing-URL datasets.

Limitations
An educational tool, not a replacement for security software; it can miss scams and flag legitimate messages.
It sees only text and URL structure: no domain age, reputation, DNS or page content.
The SMS data is old and mostly English. Scam styles change quickly, so retrain on fresh data regularly.
Lexical URL models can learn dataset quirks. Prefer data where legitimate and phishing URLs come from similar sources.
Ideas for next steps
Add multilingual support, a domain-reputation check (opt-in), email-header analysis, a browser extension, probability calibration, and a FastAPI endpoint.

License
MIT, see LICENSE.


