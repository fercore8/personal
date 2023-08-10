#Yugioh API Data Fetcher
This script provides functionality to fetch data from the Yugioh API and process it into CSV format for further use. 
The data represents the details of Yugioh cards, including their names, images, descriptions, and other relevant attributes.

##Features:
* Fetch a specific card's details by card name and set name.
* Fetch details of all available cards.
* Fetch details of all cards released in the past 16 weeks.
* Process and format card data for CSV export.
* Export card data to CSV.
* Upload card data to a MySQL database.

##Why JSON and CSV?
###JSON: 
We use JSON because the Yugioh API provides data in this format. The script saves the fetched data in .json files for debugging or archival purposes.

###CSV:
Exporting the Yugioh card data to CSV allows for easy data analysis, sharing, and potential import into other systems or databases.

##Requirements:
Python libraries: csv, requests, json, datetime, mysql.connector, urllib.parse.
MySQL database credentials and setup if you wish to use the database upload functionality.
###How to use:
Run the script with the command: python yugioh_connect.py .
The script will fetch all new cards from the past 16 weeks and generate a CSV file named all_yugi_cards<current_date>.csv.
###MySQL Integration:
The script also contains a method (yugi_cards_to_db()) that allows for uploading the CSV data to a MySQL database if the unique identifier is not there. 
If you wish to use this functionality, ensure you have the mysql.connector library installed and provide the correct database credentials in the method.

