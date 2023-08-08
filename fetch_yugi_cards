import csv
import requests
import json
from datetime import datetime, timedelta, date
import mysql.connector
from urllib.parse import quote

# Constants for configuration
YGO_API_ENDPOINT = 'https://db.ygoprodeck.com/api/v7/cardinfo.php'
NEW_CARDS_WEEKS = 16

class YugiohApi:
    """
    A class to interact with the Yugioh API.
    Fetches and processes card data, and can output to CSV.
    """
    
    def fetch_card(self, card_name: str, set_name: str) -> dict:
        """
        Fetch a specific card by name and set name from the Yugioh API.
        
        Args:
        - card_name (str): The name of the card.
        - set_name (str): The name of the card set.
        
        Returns:
        - dict: Card data.
        """
        card_name = quote(card_name)
        set_name = quote(set_name)
        response = requests.get(f'{YGO_API_ENDPOINT}?name={card_name}&cardset={set_name}')
        
        if response.status_code != 200:
            print(f"Error fetching card data for {card_name} in set {set_name}: {response.text}")
            return None

        yugi_card = response.json()['data'][0]
        return yugi_card

    def get_all_cards(self) -> dict:
        """
        Fetch all cards from the Yugioh API.
        
        Returns:
        - dict: All card data.
        """
        response = requests.get(YGO_API_ENDPOINT)
        
        if response.status_code != 200:
            print(f"Error fetching all cards: {response.text}")
            return {}
        
        all_yugioh_cards = response.json()
        with open('yugi_all_cards.json', 'w') as f:
            json.dump(all_yugioh_cards, f)
        return all_yugioh_cards

    def get_all_new_cards(self) -> dict:
        """
        Fetch all new cards from the past NEW_CARDS_WEEKS weeks from the Yugioh API.
        
        Returns:
        - dict: All new card data.
        """
        end_date = datetime.now()
        start_date = end_date - timedelta(weeks=NEW_CARDS_WEEKS)
        url = f"{YGO_API_ENDPOINT}?&startdate={start_date.strftime('%m/%d/%Y')}&enddate={end_date.strftime('%m/%d/%Y')}&dateregion=tcg_date"
        
        response = requests.get(url)
        
        if response.status_code != 200:
            print(f"Error fetching new cards: {response.text}")
            return {}

        all_yugioh_cards = response.json()
        with open(f'yugi_all_cards{datetime.now().strftime("-%m-%d-%Y")}.json', 'w') as f:
            json.dump(all_yugioh_cards, f)
        return all_yugioh_cards

    def process_card_data(self, all_yugioh_cards: dict) -> list:
        """
        Process card data to prepare it for CSV export.
        
        Args:
        - all_yugioh_cards (dict): Raw card data.
        
        Returns:
        - list: Processed card data.
        """
        items_list = []
        for card in all_yugioh_cards['data']:
            try:
                card_data = self.extract_card_details(card)
                items_list.extend(card_data)
            except KeyError as e:
                print(f"Error processing card {card.get('name', 'Unknown')}: {e}")
        return items_list

    def extract_card_details(self, card: dict) -> list:
        """
        Extract relevant details from a single card's data.
        
        Args:
        - card (dict): Raw card data for a single card.
        
        Returns:
        - list: Extracted card details.
        """
        details_list = []

        card_id = card['id']
        card_name = card['name']
        card_type = card['type']
        card_description = card['desc']

        large_pic_url = card['card_images'][0]['image_url']
        small_pic_url = card['card_images'][0]['image_url_small']

        for i, card_set in enumerate(card['card_sets']):
            set_name = card_set['set_name']
            card_number = card_set['set_code']
            price_regular = card_set['set_price']

            if price_regular == '0' and i < len(card.get('card_prices', [])):
                price_regular = card['card_prices'][i]['tcgplayer_price']

            rarity = card_set['set_rarity']
            scryfall_id = f'{card_id}-{card_number}-{rarity.lower().replace(" ", "")}'

            details_list.append([card_name, set_name, "", scryfall_id, "", small_pic_url, large_pic_url,
                                 card_number, price_regular, 0, rarity, card_type, "", 3, card_description])
        
        return details_list

    def create_csv_from_processed_data(self, processed_data: list):
        """
        Write the processed card data to a CSV file.
        
        Args:
        - processed_data (list): Processed card data.
        """
        with open(f'all_yugi_cards{date.today().strftime("-%d-%m-%Y")}.csv', 'w', newline='', encoding='utf-8') as file:
            writer = csv.writer(file, delimiter=',', quoting=csv.QUOTE_MINIMAL)
            writer.writerow(["card_name", "set_name", "set_code", "scryfall_id", "mkm_id", "small_pic_url",
                             "large_pic_url", "card_number", "price_regular", "price_foil", "rarity", "card_type", "color",
                             "game_type_id", "oracle_text"])
            for row in processed_data:
                writer.writerow(row)

    def yugi_cards_to_db(self, file_to_upload="all_yugi_cards-some_date.csv"):  # added this here for showing parts of working code.
        mydb = mysql.connector.connect(host='some_host.us-east-1.rds.amazonaws.com',
                                       user='user', passwd='pass', database="db")
        cursor = mydb.cursor()

        file = open(file_to_upload, "r", encoding='utf-8')
        data = list(csv.reader(file, delimiter=","))
        file.close()
        data.pop(0)
        sql_command = f"INSERT IGNORE INTO some_table (card_name, set_name, set_code, scryfall_id, mkm_id, small_pic_url, " \
                      f"large_pic_url, card_number, price_regular, price_foil, rarity, card_type, color, game_type_id, oracle_text) VALUES " \
                      f"(%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)"
        cursor.executemany(sql_command, data)
        mydb.commit()


def run():
    app = YugiohApi()
    all_new_cards = app.get_all_new_cards()
    processed_data = app.process_card_data(all_new_cards)
    app.create_csv_from_processed_data(processed_data)
    print('yugi csv file created.')

if __name__ == '__main__':
    run()
