# Current priorities

We are using **Python 3.6**.  

Priority #1 is writing webscrapers for each county, so we can collect daily jail data and dump to database.
We can always work on ETL and visualization later, once we have a couple months of data.

# Project timeline

ACLU said they will start lobbying at the next GA state legislative session starting Jan 2018. Ideally we would have something useful for them at that time.
But their bail reform project will be ongoing beyond then.

# CSV file format for webscrapers **(TENTATIVE - I welcome suggestions. Please submit pull request with your changes to this format)**

* Each webscraper should output a CSV file.
* **CSV name**: ```lowercase-county-name_yyyy_mm_dd_hh_mm_ss.csv```
* Use [```csv.writer```](https://docs.python.org/3/library/csv.html#csv.writer) with the [default parameters](https://docs.python.org/3/library/csv.html#csv-fmt-params) so all scrapers handle commas within fields the same way.
* Semicolons ```';'``` are separators within a field, like if an inmate has multiple charges. **To prevent a bug**, webscraper should replace text field ```';'``` with ```':'```.
* One column header row, then one row per inmate. Include **all** inmates available when the site was scraped. Even if the same inmates were there yesterday - we'll handle that later.
* Uppercase/lowercase is irrelevant, ETL code will probably lowercase everything.
* We prefer as many columns scraped as possible. Who knows what :gem::gem::gem: we could mine? But if pressed for time, scrape the columns critical to [project goals](https://github.com/lahoffm/aclu-bail-reform/raw/master/docs/ACLU-Bail-Reform-One-pager.pdf).

# CSV columns, in order
*If your county doesn't provide the data, leave the column blank.*  

Column name | Column description
------------ | -------------
county_name | County name
timestamp | [Postgres timestamp format](https://www.postgresql.org/docs/9.1/static/datatype-datetime.html): ```'2004-10-19 10:23:54 EST'``` - when row was scraped. Can have same stamp for all the rows if more convenient.
url | URL the row was scraped from
inmate_id | Inmate ID number if county posts it. Maybe useful later to look up inmates' eventual outcome.
inmate_lastname | Last name
inmate_firstname | First name
inmate_middlename | Middle name or initial, if any
inmate_sex	| ```'m'/'f'```
inmate_race	| ```'black'/'white'/'hispanic'``` etc.
inmate_age | Age in years
inmate_dob	| Date of birth, [Postgres timestamp format](https://www.postgresql.org/docs/9.1/static/datatype-datetime.html), ```'2004-10-19'```. Some counties only post year of birth.
inmate_address | Address, including if they list no address. Useful later to see where arrests are clustering.
booking_timestamp | [Postgres timestamp format](https://www.postgresql.org/docs/9.1/static/datatype-datetime.html), ```'2004-10-19 10:23:54 EST'``` - if county doesn't provide time, just insert date. If they just post arrest time, insert that, because booking would occur soon after that.
release_timestamp | [Postgres timestamp format](https://www.postgresql.org/docs/9.1/static/datatype-datetime.html), ```'2004-10-19 10:23:54 EST'``` - only fill out if they list an inmate as released. This data could be in separate release records, listed side-by-side with the booking/roster/inmate-name records, or not listed at all. ETL code will decide for each county how to interpret absence of release date. If release date is posted as ```Estimated```, append ```'estimated'``` to timestamp.
processing_numbers | Arrest/booking/case/docket IDs if county posts them. Prefix with brief description like ```'Police case #12345'``` or ```'Docket ID for charge 3: 12345'```. If multiple IDs available, separate with semicolons like with ```charge```. Maybe useful later to look up inmates' eventual outcome.
agency | Arresting/booking agency, only if specifically listed. Could be useful to target policy advocacy if county has multiple agencies
facility | Facility at which inmate is held. Leave blank if they don't post it, maybe county only has one jail. Could be useful for determining specific jails of interest.
charges	| Charges, same text as on website. ETL code can reconcile the county-specific formats that refer to the same charge. If multiple charges are listed, separate them with semicolons like ```'marijuana poss.;resisting arrest;driving under inf'```. If the charge refers to a legal code, list the code first, before the charge's descriptive text, like ```'OCGA16-13-30(a) Poss of Meth'```
severity | ```'misdemeanor'/'felony'``` - if not available, ETL code can determine this. If multiple charges, separate with semicolons like ```'misdemeanor;misdemeanor;felony'``` - in the same order as the charges.
bond_amount | Bond/bail amount in dollars, followed by space, then any other bond-related text, - like ```'$1000.00 state bond, type = surety, cleared'```. ```$``` prefix tells ETL code it's a number. Alternatively, if there was no bond set (such as a violent offender) just insert what it says like ```'held without bond'```. If different bonds listed for each charge, separate with semicolons in the same order as the charges.
current_status | Current status for each charge. For example, Columbia county posts ```AWAITING TRIAL/SENTENCED``` and some counties post disposition such as ```time served```. If multiple charges, separate with semicolons in the same order as the charges, like ```'awaiting trial;time served;unknown status'```
court_dates | Next court dates, [Postgres timestamp format](https://www.postgresql.org/docs/9.1/static/datatype-datetime.html), ```'2004-10-19'```. If multiple dates, separate with semicolons and add descriptive text like ```'2004-10-19, charge 1;2004-10-22, charge 2'```. Eventually this can help determine how long someone sat in jail before court appearance, if roster data shows they were never released on bail.
days_jailed | Days in custody as of ```timestamp```. Some counties explicitly post this. Otherwise, keep blank and let ETL code try to infer it from other information.
other | Any other data you feel is potentially useful. Please let us know, maybe we'll add it to the CSV format in the future.