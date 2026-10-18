# SmartLoan Predictor

A Flask web app that assesses loan eligibility using a transparent, explainable scorecard. An optional scikit-learn model adds an approval probability as a secondary signal. The scorecard makes the decision, so every result can be explained to the applicant factor by factor.

## Features

- 100-point scorecard with a per-factor breakdown
- EMI calculation (reducing balance) and FOIR (EMI as a share of monthly income)
- Decision outcomes: approved, review, or declined
- Suggestions to improve an application, including the loan size that keeps EMI at 40% of income
- Optional approval probability from a trained model (Logistic Regression or Random Forest)
- Rupee formatting with Indian digit grouping (₹12,34,567)
- Input validation, security headers, and a `/health` endpoint
- Docker support with Gunicorn

## Fairness

Gender, marital status and property area are deliberately not used, in either the scorecard or the optional model.

## How the scoring works

| Factor | Max points |
|---|---|
| Credit history | 35 |
| Affordability (FOIR) | 35 |
| Employment | 10 |
| Dependents | 8 |
| Education | 6 |
| Second income (co-applicant) | 6 |

- Score of 70 or more: approved
- Score of 50 to 69: review
- Score below 50: declined

Overrides apply regardless of score:

- No usable credit history: capped at review
- EMI above 50% of monthly income: capped at review
- EMI above 60% of monthly income: declined

The default annual interest rate is 9%.

## Project structure

```
loan_predictor/
├── app.py               # Flask app, form validation, routes
├── scoring.py           # Scorecard, EMI and FOIR logic
├── ml.py                # Optional model loading and prediction
├── train_model.py       # Trains and evaluates the optional model
├── templates/           # HTML templates
├── static/              # Static assets
├── tests/               # Test suite
├── Screenshots/         # App screenshots
├── Dockerfile
├── requirements.txt
└── requirements-dev.txt
```

## Getting started

Requires Python 3.11 or newer.

```bash
git clone https://github.com/Chowdri-Furkhan07/loan_predictor.git
cd loan_predictor
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
flask --app app run --debug
```

Open http://127.0.0.1:5000. If no trained model is found, the app runs on the scorecard alone.

## Training the optional model

The model uses the public "Loan Prediction" dataset (with a `Loan_Status` column of Y/N). Download the CSV yourself, then run:

```bash
python train_model.py --data data/train.csv
```

This compares Logistic Regression and Random Forest using 5-fold cross-validated ROC-AUC, trains the better one on an 80/20 stratified split, and writes:

- `models/loan_model.joblib`
- `models/loan_model.metrics.json`

The script also checks how strongly the model reacts to credit history and prints a warning if the effect is too small. If the installed scikit-learn version differs from the one used for training, the app ignores the saved model, so retrain in that case.

## Configuration

| Variable | Default | Purpose |
|---|---|---|
| `ANNUAL_INTEREST_RATE` | `9.0` | Interest rate (%) used for EMI |
| `MODEL_PATH` | `models/loan_model.joblib` | Location of the trained model |
| `LOG_LEVEL` | `INFO` | Logging level |
| `FLASK_DEBUG` | unset | Set to `1` to enable debug mode |

## Routes

| Route | Method | Description |
|---|---|---|
| `/` | GET | Application form |
| `/predict` | POST | Validates input and shows the assessment |
| `/health` | GET | Returns status and whether a model is loaded |

## Docker

```bash
docker build -t loan-predictor .
docker run -p 8000:8000 loan-predictor
```

The app is served by Gunicorn at http://localhost:8000.

## Tests

```bash
pip install -r requirements-dev.txt
pytest
```

## Screenshots

<!-- Add your images from the Screenshots folder, for example: -->
<!-- ![Application form](Screenshots/your-file-name.png) -->

## Tech stack

Python, Flask, Gunicorn, pandas, NumPy, scikit-learn, joblib, pytest, Docker

## Privacy

Application logs record only the decision, score and FOIR. No personal data is logged.

## Disclaimer

This is an educational project. Its output is not financial advice and should not be used for real lending decisions.

## Author

Chowdri Furkhan ([@Chowdri-Furkhan07](https://github.com/Chowdri-Furkhan07))
