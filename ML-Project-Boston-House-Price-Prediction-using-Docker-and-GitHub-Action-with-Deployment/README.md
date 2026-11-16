# Boston House Price Prediction

[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-Web%20App-000000?logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![scikit--learn](https://img.shields.io/badge/scikit--learn-Linear%20Regression-F7931E?logo=scikitlearn&logoColor=white)](https://scikit-learn.org/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

![Capture](https://github.com/user-attachments/assets/caa74f88-82ca-4d12-94d3-1b90213aa62b)

A small end-to-end machine learning application that estimates the median value of a Boston-area home from 13 supplied features. It combines a scikit-learn regression model with a Flask web interface and JSON API, and includes Docker, GitHub Actions, and Render configuration.

> **Important:** This is an educational demonstration using a historical dataset. Its predictions are not current market valuations and should not be used for financial, housing, lending, or policy decisions. The underlying Boston Housing dataset has known limitations and problematic social variables; see [Dataset notes](#dataset-notes).

## Contents

- [Project overview](#project-overview)
- [Features](#features)
- [Repository layout](#repository-layout)
- [How prediction works](#how-prediction-works)
- [Run locally](#run-locally)
- [Use Docker](#use-docker)
- [HTTP API](#http-api)
- [Model training notebook](#model-training-notebook)
- [Continuous integration](#continuous-integration)
- [Deploy to Render](#deploy-to-render)
- [Dataset notes](#dataset-notes)
- [Limitations](#limitations)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)
- [Connect with me](#connect-with-me)

## Project overview

The application loads two serialized scikit-learn artifacts at startup: a StandardScaler and a fitted LinearRegression estimator. Input features are scaled with the saved scaler, then passed to the regression model. The web form displays the estimate, while POST /predict_api provides a JSON response for programmatic use.

The model was trained in the Jupyter notebook using the CSV in data/. The notebook splits the rows into training and test sets with test_size=0.2 and random_state=42, fits the scaler on the training features, and saves the trained estimator and scaler under models/.

## Features

- Browser-based house-price prediction form.
- JSON prediction endpoint for integrations.
- Saved preprocessing and model artifacts, so the web application does not retrain at startup.
- Docker image for a reproducible container environment.
- GitHub Actions workflow that builds the image and runs the repository's pytest suite inside it on pushes and pull requests to main.
- Render service configuration using Gunicorn.
- Jupyter notebook documenting data exploration, preprocessing, model training, and evaluation.

## Repository layout

~~~
.
├── .github/workflows/ci-cd.yml  # Docker build and test workflow
├── data/
│   └── boston_house_prices.csv  # Dataset used by the notebook
├── jupyter/
│   └── boston_house_price.ipynb # Exploration and model training
├── models/
│   ├── regmodel.pkl             # Fitted linear regression model
│   └── scaling.pkl              # Fitted StandardScaler
├── static/
│   └── style.css                # Web interface styles
├── templates/
│   └── home.html                # Prediction form
├── tests/                       # Pytest suite
├── app.py                       # Flask application and endpoints
├── Dockerfile                   # Container build instructions
├── render.yaml                  # Render service configuration
├── requirements.txt             # Python dependencies
└── README.md
~~~

## How prediction works

1. The user supplies all 13 input features in the form or API request.
2. The application transforms the feature vector with the saved StandardScaler.
3. The fitted linear regression model estimates price.
4. The web page displays the estimate to two decimal places. The API returns the numerical prediction as JSON.

The dataset's target column is price, representing median owner-occupied home value in **thousands of US dollars**. For example, an estimate of 24.50 corresponds to approximately $24,500 in the dataset's original units; it is not inflation-adjusted.

### Input features

Supply values in this order. The JSON API reads values in the order they appear in the data object, so preserve this order when calling it.

| # | Feature | Description |
|---:|---|---|
| 1 | CRIM | Per-capita crime rate by town |
| 2 | ZN | Share of residential land zoned for lots over 25,000 sq. ft. |
| 3 | INDUS | Share of non-retail business acres per town |
| 4 | CHAS | Charles River boundary indicator (1 if tract bounds the river; otherwise 0) |
| 5 | NOX | Nitric oxide concentration (parts per 10 million in the source dataset) |
| 6 | RM | Average number of rooms per dwelling |
| 7 | Age | Share of owner-occupied units built before 1940 |
| 8 | DIS | Weighted distance to five Boston employment centers |
| 9 | RAD | Accessibility index for radial highways |
| 10 | TAX | Property-tax rate per $10,000 of full-value property |
| 11 | PTRATIO | Pupil-teacher ratio by town |
| 12 | B | Transformed demographic feature in the original dataset |
| 13 | LSTAT | Percentage of the population recorded as lower status in the original dataset |

## Run locally

### Requirements

- Python 3.9 or newer
- pip

### Setup

~~~
git clone https://github.com/Md-Emon-Hasan/Ml-Project-Boston-House-Price-Prediction-using-Docker-and-GitHub-Action-with-Deployment.git
cd Ml-Project-Boston-House-Price-Prediction-using-Docker-and-GitHub-Action-with-Deployment

python -m venv .venv
~~~

Activate the environment, then install dependencies:

~~~
# macOS / Linux
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1

pip install -r requirements.txt
~~~

Start the Flask development server:

~~~
python app.py
~~~

Open [http://127.0.0.1:5000](http://127.0.0.1:5000) in a browser. The model files in models/ must be present in the repository checkout.

## Use Docker

Build and start the container from the repository root:

~~~
docker build -t boston-house-price .
docker run --rm -p 5000:5000 boston-house-price
~~~

Then open [http://127.0.0.1:5000](http://127.0.0.1:5000). Stop the container with Ctrl+C.

## HTTP API

### POST /predict_api

Send a JSON object with a data property containing the 13 named feature/value pairs in the order listed above. All values should be numeric.

~~~
curl -X POST http://127.0.0.1:5000/predict_api \
  -H "Content-Type: application/json" \
  -d '{"data":{"CRIM":0.00632,"ZN":18,"INDUS":2.31,"CHAS":0,"NOX":0.538,"RM":6.575,"Age":65.2,"DIS":4.09,"RAD":1,"TAX":296,"PTRATIO":15.3,"B":396.9,"LSTAT":4.98}}'
~~~

The response is a JSON number, for example:

~~~
24.12
~~~

The example output is illustrative; the actual result depends on the checked-in model artifact and installed scikit-learn version. The web form is available at GET /; submitting it posts the same 13 features to POST /predict and returns the result in the page.

## Model training notebook

Open jupyter/boston_house_price.ipynb in Jupyter Notebook or JupyterLab. The notebook covers:

- Loading and inspecting the provided CSV.
- Exploring feature distributions and correlations.
- Separating the 13 predictors from the price target.
- Creating a reproducible 80/20 train/test split.
- Standardizing features using a scaler fit only on the training split.
- Fitting a linear regression estimator and calculating regression metrics, including MAE, MSE, RMSE, and R².
- Saving the trained estimator and scaler in models/ for use by Flask.

Run the notebook from its own directory if using the relative dataset path as written. Re-training will replace the model artifacts and may change predictions.

## Continuous integration

The workflow at .github/workflows/ci-cd.yml runs for pushes and pull requests targeting main. It checks out the repository, builds the Docker image, and runs pytest tests/ inside the image. The workflow validates the container build and tests; it does **not** publish an image or deploy the service.

## Dataset notes

This project uses the Boston Housing dataset, a historical dataset collected from the Boston area and commonly distributed with the MEDV/price target representing median owner-occupied home values in thousands of dollars. The CSV's B feature is a transformation of a racial demographic variable; LSTAT is also a socially sensitive proxy. These variables reflect the dataset's historical design and must not be treated as neutral or appropriate inputs for present-day housing decisions.

## Limitations

- The model reflects a small, historical dataset and does not represent current Boston-area prices.
- Predictions depend on the quality and range of the supplied input values; extrapolation can be unreliable.
- The application does not provide prediction intervals or uncertainty estimates.
- The serialized pickle files must come from a trusted source. Loading untrusted pickle files can execute arbitrary code.
- Reproducing serialized model behavior may require compatible Python and scikit-learn versions; the project currently declares dependencies without version pins.

## Contributing

Issues and pull requests are welcome. For a change:

1. Fork the repository and create a focused branch.
2. Make the change and add or update relevant automated tests.
3. Run the checks locally where applicable.
4. Open a pull request with a clear summary and any relevant screenshots or output.

## License

Distributed under the MIT License. See [LICENSE](LICENSE).

## Connect with me

- **Portfolio:** [emonlabs-ai.hitechparks.com](https://emonlabs-ai.hitechparks.com)
- **GitHub:** [Md-Emon-Hasan](https://github.com/Md-Emon-Hasan)
- **LinkedIn:** [Md Emon Hasan](https://www.linkedin.com/in/md-emon-hasan-695483237/)
- **Hugging Face:** [emonHasan01](https://huggingface.co/emonHasan01)
- **Email:** [emon.mlengineer@gmail.com](mailto:emon.mlengineer@gmail.com)
- **WhatsApp:** [+880 1834 363533](https://wa.me/8801834363533)
