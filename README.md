# book_harvest_project
# BOOKHARVEST WEB SCRAPING & DATA PIPELINE
## Project Report
*Web Data Extraction, Transformation and PostgreSQL Loading*
## 1. Executive Summary
The BookHarvest project is a Python-based web scraping and data-loading pipeline designed to extract structured book information from Books to Scrape, transform the extracted data, and load it into PostgreSQL. The pipeline uses Requests for HTTP retrieval, BeautifulSoup for HTML parsing, Pandas for data organisation, and SQLAlchemy with psycopg2 for database integration.
The catalogue was designed to be scraped across 50 pages, with 20 books per page, giving an expected dataset of approximately 1,000 books. The project successfully demonstrated extraction of title, price, rating, availability and genre, transformation of price and rating values, and loading of 1,000 records into the PostgreSQL table.
## 2. Project Objectives
Extract book records from the website across multiple catalogue pages.
Transform scraped values into analysis-ready data types.
Convert textual star ratings into numeric ratings.
Organise the extracted records into a Pandas DataFrame.
Securely connect Python to PostgreSQL using environment variables.
Load the DataFrame into an existing PostgreSQL table.
## 3. Technologies Used
Python for programming
Requests for HTTP requests
BeautifulSoup for HTML parsing
Pandas for Data manipulation
python-dotenv for configuration
SQLAlchemy for Database toolkit
psycopg2 for PostgreSQL driver
PostgreSQl for Database
VS Code/Jupyter for development
## 4. Data Extraction Process
The scraper first requests the catalogue page and checks the HTTP status code. BeautifulSoup then identifies each book using the article element with class product_pod. For every book card, the script extracts the title from the image alt attribute, the price from p.price_color, the rating from the star-rating class, and availability from p.instock.availability.
Inspection of the catalogue-card HTML showed that genre is not contained inside the article.product_pod element. The card does contain a relative link to the individual book page. Therefore, genre should be extracted by following that link and inspecting the detail-page HTML rather than assuming it exists in the catalogue card.
The next implementation step is to extract the book URL, request the individual book page, identify the actual HTML element containing the genre, and add the genre to each record before creating the DataFrame.
The current extracted fields are:
title
price_gdp (price in GBP)
rating
availability
genre
## 5. Rating Transformation
The website stores ratings as words such as One, Two, Three, Four and Five. A dictionary named rating_maps converts these textual values into numbers from 1 to 5. The .get() method uses 0 as a fallback for an unexpected value.
Example: "Three" → rating_maps.get("Three", 0) → 3
## 6. Pagination
The scraper loops from page 1 through page 50 and dynamically builds each catalogue URL. If a page does not return HTTP status 200, the script skips that page and continues. With 20 books per page, the intended collection size is 1,000 records.
## 7. DataFrame and Database Loading
Each book is stored as a dictionary in bookharvest_pages. The list can then be converted into a Pandas DataFrame. SQLAlchemy provides the engine used by Pandas to communicate with PostgreSQL.
The loading statement is:

```python
df.to_sql("books", engine, if_exists="append", index=False)
```
The table name is books. append means that records are added to an existing table rather than replacing it. index=False prevents the Pandas DataFrame index from being inserted as a database column. The successful return value of 1000 indicates that the operation wrote 1,000 rows.
## 8. Environment Variables and Database Security
Database credentials are stored in a .env file using variables such as DB_HOST, DB_PORT, DB_NAME, DB_USER and DB_PASSWORD. python-dotenv loads these values into the Python environment. This avoids hard-coding database credentials in the notebook.
A special-character issue was encountered because the database password contains an @ symbol. When a connection URL is manually constructed with an f-string, special characters can interfere with URL parsing.
## 10. Challenges Encountered
PostgreSQL authentication: The initial connection reported that no password was supplied; the connection was corrected to explicitly load DB_PASSWORD.
Password containing @: Manual connection-string construction can misinterpret @; URL.create() or URL encoding is recommended.
Incorrect host parsing: A host was interpreted as rd@localhost, demonstrating why password and host values must remain separate.
## 11. Recommendations
Use SQLAlchemy URL.create() for database connections containing special characters.
Keep .env outside .venv and add .env to .gitignore.
Validate DataFrame column names and data types against the PostgreSQL schema.
Add a unique book URL or stable identifier to prevent duplicate records when the scraper is rerun.
Use an UPSERT strategy if the pipeline will run repeatedly.
Add request timeouts and controlled delays for more robust scraping.
Handle errors when individual book-detail pages cannot be retrieved.
Verify row counts in PostgreSQL after each load.
Separate extraction, transformation and loading into reusable functions as the project matures.
## 12. Conclusion
The BookHarvest project demonstrates an end-to-end ETL workflow in Python. It retrieves web data, parses and transforms the data, organises it in a DataFrame, and loads the result into PostgreSQL. The project also demonstrates practical database credential management, pagination, data-type conversion and database integration.
The principal next enhancement is genre extraction from the individual book pages, followed by duplicate prevention and stronger validation. These improvements would make the pipeline more reliable for repeated execution and closer to a production-quality data pipeline.
