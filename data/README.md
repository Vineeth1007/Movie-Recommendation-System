# Supporting datasets

Datasets behind the movie table and the sentiment model of the [Movie Recommendation System](../README.md). None of the files in this folder is read by `main.py`; the table the app uses at run time is [`../main_data.csv`](../main_data.csv), which stays in the repository root because the app opens it by relative path.

## Files

| File | Size (rows x columns) | What it is |
|---|---|---|
| [movie_metadata.csv](movie_metadata.csv) | 5,043 x 28 | Table in the schema of the IMDB 5000 Movie Dataset (`color`, `director_name`, `num_critic_for_reviews`, ..., `imdb_score`, `movie_facebook_likes`) |
| [data.csv](data.csv) | 5,043 x 6 | Reduced table: `director_name`, `actor_1_name`, `actor_2_name`, `actor_3_name`, `genres`, `movie_title` |
| [new_data.csv](new_data.csv) | 5,364 x 7 | The same six columns plus `comb` |
| [final_data.csv](final_data.csv) | 5,869 x 7 | The same seven columns, with more rows |
| [reviews.txt](reviews.txt) | 7,086 lines | Tab-separated labelled sentences: a label (`1` or `0`), a tab, then the text |
| [`../main_data.csv`](../main_data.csv) | 6,010 x 7 | Final table used by the app (same columns as `final_data.csv`) |

`comb` is the text field the recommender works on: actor 1, actor 2, actor 3, director and genres joined by single spaces (checked for all 6,010 rows of `main_data.csv`).

The row counts grow from `data.csv` (5,043) through `new_data.csv` (5,364) and `final_data.csv` (5,869) to `main_data.csv` (6,010), which looks like successive stages of building the table. The scripts or notebooks that produced these files are not in the repository, so this lineage is an inference from the files themselves.

## About reviews.txt

- 7,086 lines: 3,995 labelled `1` and 3,091 labelled `0`. The sentences read as positive for `1` and negative for `0`.
- Only 1,411 of the lines are distinct.
- Every line mentions one of four titles: The Da Vinci Code, Harry Potter, Brokeback Mountain or Mission: Impossible.
- It is probably the data the sentiment classifier in `../nlp_model.pkl` was trained on, but the training code is not included, so treat that as unconfirmed.

## Sources

As listed by the upstream project (see the root README for the full list and links): the IMDB 5000 Movie Dataset, The Movies Dataset (Kaggle) and Wikipedia lists of American films of 2018, 2019 and 2020. Terms of use are set by the respective providers. The source of `reviews.txt` is not stated in the repository.

## Duplicate set aside

The original layout contained a second, byte-identical copy of `main_data.csv` in this folder (then named `datasets/`). One copy is kept, in the repository root, because the code reads it from there.
