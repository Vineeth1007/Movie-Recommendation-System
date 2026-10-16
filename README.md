# Movie Recommendation System with Sentiment Analysis

A Flask backend that recommends the ten movies most similar to a chosen title and labels IMDB user reviews of a movie as Good or Bad with a pickled TF-IDF and Naive Bayes classifier.

![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-000000?logo=flask&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?logo=scikitlearn&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-150458?logo=pandas&logoColor=white)
![Beautiful Soup](https://img.shields.io/badge/BeautifulSoup-scraping-4B8BBE)

> Type: web app (backend and data only - the browser front end is not included, see [Limitations](#limitations-and-known-issues))
>
> Based on [kishan0725/Movie-Recommendation-System-with-Sentiment-Analysis](https://github.com/kishan0725/Movie-Recommendation-System-with-Sentiment-Analysis). See [Credits](#credits).

## Overview

The project combines two ideas in one small web service:

1. **Content-based recommendation.** Each of the 6,010 movies in `main_data.csv` is described by one text field (`comb`): its top three actors, its director and its genres. The text is turned into word counts, cosine similarity is computed between all movies, and the ten titles most similar to the requested movie are returned.
2. **Review sentiment.** For a selected movie the server downloads the IMDB user-review page, and a pickled TF-IDF vectorizer plus Multinomial Naive Bayes classifier labels each review `Good` or `Bad`.

The Python side (`main.py`) is complete. The HTML templates and static files that the upstream project uses to present the results, and to fetch movie details from TMDB in the browser, are not part of this repository. As committed, the `/similarity` endpoint works on its own, while the pages `/` and `/recommend` fail because their templates are missing.

## Contents

| Path | What it is |
|---|---|
| [main.py](main.py) | Flask application: routes, similarity search, IMDB review scraping and sentiment labelling |
| [main_data.csv](main_data.csv) | Movie table read by the app at run time (6,010 rows) |
| [nlp_model.pkl](nlp_model.pkl) | Pickled scikit-learn `MultinomialNB` sentiment classifier |
| [tranform.pkl](tranform.pkl) | Pickled scikit-learn `TfidfVectorizer` (the file name is spelled this way in `main.py`, so it is kept) |
| [Procfile](Procfile) | Process definition for gunicorn hosting: `web: gunicorn main:app` |
| [requirements.txt](requirements.txt) | Python packages imported by `main.py`, plus the server and parser it relies on |
| [data/](data/) | Supporting datasets used to build the table and the sentiment model; see [data/README.md](data/README.md) |

## Data

- **Used at run time:** `main_data.csv` in the repository root, read with the relative path `main_data.csv`, so the app must be started from the repository root. Columns: `director_name`, `actor_1_name`, `actor_2_name`, `actor_3_name`, `genres`, `movie_title`, `comb`. It is committed.
- **Sources, as listed in this repository's original README:** the [IMDB 5000 Movie Dataset](https://www.kaggle.com/carolzhangdc/imdb-5000-movie-dataset), [The Movies Dataset](https://www.kaggle.com/rounakbanik/the-movies-dataset), and the Wikipedia lists of American films for [2018](https://en.wikipedia.org/wiki/List_of_American_films_of_2018), [2019](https://en.wikipedia.org/wiki/List_of_American_films_of_2019) and [2020](https://en.wikipedia.org/wiki/List_of_American_films_of_2020).
- **Supporting files:** the intermediate tables and the labelled review sentences are in [data/](data/); none of them is read by `main.py`.
- **Licences:** terms are set by the respective data providers.

## Approach

1. **Similarity matrix.** `create_similarity()` reads `main_data.csv`, converts the `comb` column to word counts with scikit-learn's `CountVectorizer`, and computes pairwise cosine similarity (a 6,010 x 6,010 matrix).
2. **Recommendations.** `rcmd(title)` lower-cases the title, looks it up in `movie_title`, sorts all movies by similarity, drops the movie itself and returns the next ten titles. If the title is unknown it returns a "not in our database" message.
3. **Reviews.** `/recommend` receives the movie details posted by the front end (title, cast, poster, `imdb_id` and so on), downloads `https://www.imdb.com/title/<imdb_id>/reviews` with `urllib`, and extracts the review text from `div` elements of class `text show-more__control` using BeautifulSoup with the `lxml` parser.
4. **Sentiment.** Each review is transformed by the TF-IDF vectorizer in `tranform.pkl` and classified by the Naive Bayes model in `nlp_model.pkl`; a prediction of class 1 is shown as `Good` and class 0 as `Bad`.

### HTTP routes

| Route | Method | Behaviour |
|---|---|---|
| `/` and `/home` | GET | Renders `home.html` with the list of titles for autocomplete (template not included) |
| `/similarity` | POST, form field `name` | Returns the ten similar titles joined with `---`, or the "not in our database" message |
| `/recommend` | POST, movie details as form fields | Scrapes and labels the reviews, then renders `recommend.html` (template not included) |

## Results

No evaluation results (accuracy, precision, user studies and so on) are committed for either component, so none are reported here. Facts that can be checked in the repository:

- `main_data.csv` has 6,010 movies (5,978 distinct titles).
- The scikit-learn objects in both pickle files record version 0.23.1 as the version that saved them.
- Example `/similarity` output for the title `avatar` begins with `aliens`, `mystery men`, `deep rising`, `dragonball: evolution` and `terminator 2: judgment day` (observed by running `main.py` from this layout with the Flask test client).

## How to run

1. Create a Python environment and install the dependencies:
   ```bash
   pip install -r requirements.txt
   ```
   The versions are not pinned. The pickles were saved with scikit-learn 0.23.1; use that version (or re-create the pickles) for the sentiment part. With scikit-learn 1.9.1 the pickles load, but transforming text with the vectorizer fails with a `NotFittedError`.
2. Start the server from the repository root, so that the relative file names resolve:
   ```bash
   python main.py
   ```
   Flask starts in debug mode on <http://127.0.0.1:5000/>.
3. Try the self-contained endpoint:
   ```bash
   curl -X POST -d "name=avatar" http://127.0.0.1:5000/similarity
   ```
4. The browser interface needs the missing `templates/` (`home.html`, `recommend.html`) and `static/` folders next to `main.py`. In the upstream project the movie details are fetched from [TMDB](https://www.themoviedb.org/documentation/api) by JavaScript in `static/recommend.js`, which needs a TMDB API key; never commit a real key.
5. For hosting on a platform that reads a `Procfile`, `web: gunicorn main:app` expects `main.py` in the repository root, which is where it now is.

## Project structure

```text
.
├── README.md             # this file
├── Procfile              # gunicorn entry point
├── requirements.txt      # Python dependencies (unpinned)
├── .gitignore
├── main.py               # Flask app
├── main_data.csv         # movie table used at run time
├── nlp_model.pkl         # MultinomialNB sentiment classifier
├── tranform.pkl          # TfidfVectorizer
└── data/                 # supporting datasets (see data/README.md)
    ├── README.md
    ├── movie_metadata.csv
    ├── data.csv
    ├── new_data.csv
    ├── final_data.csv
    └── reviews.txt
```

## Limitations and known issues

- **Front end missing.** `templates/` and `static/` are not in the repository, so the pages `/` and `/recommend` return an error and movie details from TMDB are not available.
- **Sentiment model provenance.** The notebooks or scripts that trained the model and built the datasets are not included. `data/reviews.txt` looks like the training data, but this is an inference. All 7,086 sentences in it mention one of four titles (The Da Vinci Code, Harry Potter, Brokeback Mountain, Mission: Impossible) and only 1,411 of the lines are distinct, so the model's coverage of other movies is unknown.
- **Scraping fragility.** The IMDB URL pattern and the CSS class used to find reviews reflect the page layout when the code was written; they were not re-tested against the live site and may no longer work. Reviews whose text contains nested markup are skipped.
- **Performance.** The cache check at the top of `rcmd()` never succeeds (it reads local variables before they are assigned), so the similarity matrix is rebuilt on every `/similarity` request.
- **Dependencies.** `requirements.txt` is unpinned and the pickles depend on scikit-learn 0.23.1 (see [How to run](#how-to-run)).

## Credits

- **Upstream project:** this repository is based on [Movie-Recommendation-System-with-Sentiment-Analysis](https://github.com/kishan0725/Movie-Recommendation-System-with-Sentiment-Analysis) by **kishan0725**. The attribution rests on the original README of this repository, which links its `requirements.txt` to that repository; this repository carries none of the upstream history or licence, and the upstream repository could not be re-checked when this README was written. The application code, the model files, the datasets and the original description, including the similarity-score explanation, are presumed to come from that project and are credited to it. The restructuring of this repository was limited to the folder layout (files moved to the repository root, `datasets/` renamed to `data/`, a duplicate copy of `main_data.csv` and the duplicate README set aside), the rewritten documentation, a trimmed `requirements.txt` and a `.gitignore`; `main.py` was not edited.
- **Datasets:** the IMDB 5000 Movie Dataset and The Movies Dataset (Kaggle), and Wikipedia film lists, as listed under [Data](#data).
- **Libraries:** Flask, pandas, NumPy, scikit-learn, Beautiful Soup and lxml.

## License

No license file - all rights reserved by the respective authors. Refer to the upstream project for its terms.
