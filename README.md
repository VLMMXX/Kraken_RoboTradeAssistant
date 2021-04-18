# Kraken_RoboTradeAssistant

Lambda Code (just in case)

### SIREN ###
### Required Libraries ###
### PART 1 
### NOTHING CHANGED HERE!
#------------------------------------------------------------------------
from datetime import datetime
import json
from dateutil.relativedelta import relativedelta
from botocore.vendored import requests

### Functionality Helper Functions ###
def parse_int(n):
    """
    Securely converts a non-integer value to integer.
    """
    try:
        return int(n)
    except ValueError:
        return float("nan")


def build_validation_result(is_valid, violated_slot, message_content):
    """
    Define a result message structured as Lex response.
    """
    if message_content is None:
        return {"isValid": is_valid, "violatedSlot": violated_slot}

    return {
        "isValid": is_valid,
        "violatedSlot": violated_slot,
        "message": {"contentType": "PlainText", "content": message_content},
    }


### Dialog Actions Helper Functions ###
def get_slots(intent_request):
    """
    Fetch all the slots and their values from the current intent.
    """
    return intent_request["currentIntent"]["slots"]


def elicit_slot(session_attributes, intent_name, slots, slot_to_elicit, message):
    """
    Defines an elicit slot type response.
    """

    return {
        "sessionAttributes": session_attributes,
        "dialogAction": {
            "type": "ElicitSlot",
            "intentName": intent_name,
            "slots": slots,
            "slotToElicit": slot_to_elicit,
            "message": message,
        },
    }


def delegate(session_attributes, slots):
    """
    Defines a delegate slot type response.
    """

    return {
        "sessionAttributes": session_attributes,
        "dialogAction": {"type": "Delegate", "slots": slots},
    }


def close(session_attributes, fulfillment_state, message):
    """
    Defines a close slot type response.
    """

    response = {
        "sessionAttributes": session_attributes,
        "dialogAction": {
            "type": "Close",
            "fulfillmentState": fulfillment_state,
            "message": message,
        },
    }

    return response


#-------------------------------------------------------------------------------------
### PART 2 
### RETRIEVING PRICE OF CRYPTO FROM KRAKEN.COM!


""" Retrieves the current price of cryto in US Dollars from the kraken.com API."""
crypto_usd = 'adaxbt'
def price_usd(crypto_usd):
    crypto_map = {
        'xbtusd': 'XXBTZUSD',
        'adausd': 'ADAUSD',
        'ethusd': 'XETHZUSD',
        'xrpusd': 'XXRPZUSD',
        'dotusd': 'DOTUSD',
        'filusd': 'FILUSD',
        'linkusd': 'LINKUSD',
        'ltcusd': 'XLTCZUSD', 
    }
    kraken_url = f'https://api.kraken.com/0/public/Ticker?pair={crypto_map[crypto_usd]}'
    response = requests.get(kraken_url)
    response_json = response.json()
    price = response_json['result'][crypto_map[crypto_usd]]['c'][0]
    return price
    
""" Retrieves price of crypto bought using Bitcoin from the kraken.com API."""
crypto_xbt = 'adaxbt'
def price_xbt(crypto_xbt):
    crypto_xbt_map = {
        'adaxbt': 'ADAXBT',
        'ethxbt': 'XETHXXBT',
        'xrpxbt': 'XXRPXXBT',
        'dotxbt': 'DOTXBT',
        'filxbt': 'FILXBT',
        'linkxbt': 'LINKXBT',
        'ltcxbt': 'XLTCXXBT',
    }
    kraken_url = f'https://api.kraken.com/0/public/Ticker?pair={crypto_xbt}'
    response = requests.get(kraken_url)
    response_json = response.json()
    response_json
    price = response_json['result'][crypto_xbt_map[crypto_xbt]]['c'][0]
    return price
  
"""Retrieves price of crypto bought using Ethereum from the kraken.com API."""    
crypto_eth = 'adaxbt'
def price_eth(crypto_eth):
    crypto_eth_map = {
        'adaeth': 'ADAETH',
        'xrpeth': 'XRPETH',
        'doteth': 'DOTETH',
        'fileth': 'FILETH',
        'linketh': 'LINKETH',
        'ltceth': 'LTCETH',
    }
    kraken_url = f'https://api.kraken.com/0/public/Ticker?pair={crypto_eth}'
    response = requests.get(kraken_url)
    response_json = response.json()
    response_json
    price = response_json['result'][crypto_eth_map[crypto_eth]]['c'][0]
    return price

### RETRIEVING PRICE OF CRYPTO FROM KRAKEN.COM!
### FIRST SLOT TYPES - BUTTONS - WHERE THEY SHOULD LEAD?  
def coin_category(coin_type):
    coin_type = {
        "XRP": "You have selected Ripple, current price  ",
        "ETH": "You have selected Ethereum, current price  ",
        "DOT": "You have selected Polkadot, current price  ",
        "FIL": "You have selected Filecoin, current price ",
        "LINK": "You have selected Chainlink, current price  ",
        "USDT": "You have selected Tether, current price  ",
        "LTC": "You have selected Litecoin, current price  ",
        "XBT": "You have selected Bitcoin, current price  ",
        "ADA": "You have selected Cardano, current price  ",
    }    
    return coin_type[coin_type.lower()]
# price (coin_type)

#-------------------------------------------------------------------------------
### PART 3 Validating data provided by the user.NOT NESSESARY NOW """


# def validate_data(buy_coin, quantity_buy, buy_order_type, buy_order_funding, buy_confirmation_total, intent_request):
#     # Validate the that number is not negative or <0.
#     if quantity_buy is not None:
#         quantity_buy  = parse_int(
#             quantity_buy 
#         )  # Since parameters are strings it's important to cast values
#         if quantity_buy <= 0:
#             return build_validation_result(
#                 False,
#                 "quantity_buy",
#                 " ""The minimum amount shold be more than 0, Please, enter amount of coin greater than 0",
#             )
    
    # Validate the investment amount, it should be >= 5000
    # if investment_amount is not None:
    #     investment_amount = parse_int(investment_amount)
    #     if investment_amount < 5000:
    #         return build_validation_result(
    #             False,
    #             "investmentAmount",
    #             "The minimum investment amount is $5,000 USD, "
    #             "could you please provide a greater amount?",
    #         )
    # return build_validation_result(True, None, None)


### PART 3 Intents Handlers ###

def Buy_Order(intent_request):
    """
    Performs dialog management and fulfillment for buying order.
    """
    buy_coin = get_slots(intent_request)["BuyCoin"]
    quantity_buy = get_slots(intent_request)["QuantityBuy"]
    buy_order_type = get_slots(intent_request)["BuyOrderType"]
    buy_order_funding = get_slots(intent_request)["BuyOrderFunding"]
    # if source == "DialogCodeHook":
        # Perform basic validation on the supplied input slots.
        # Use the elicitSlot dialog action to re-prompt
        # for the first violation detected.
        # slots = get_slots(intent_request)
        # validation_result = validate_data(buy_coin, quantity_buy, buy_order_type, buy_order_funding, buy_confirmation_total, intent_request)
        # if not validation_result["isValid"]:
            # slots[validation_result["violatedSlot"]] = None  # Cleans invalid slot
            # Returns an elicitSlot dialog to request new data for the invalid slot
            # return elicit_slot(
            #     intent_request["sessionAttributes"],
            #     intent_request["currentIntent"]["name"],
            #     slots,
            #     validation_result["violatedSlot"],
            #     validation_result["message"],
            # )
        # Fetch current session attibutes
        # output_session_attributes = intent_request["sessionAttributes"]
        # return delegate(output_session_attributes, get_slots(intent_request))
    order_price = 0
    if buy_order_funding == "ETH":
        pair = (buy_coin + buy_order_funding).lower()
        order_price = price_eth(pair)
        total_cost = round(float(quantity_buy) * float(order_price) * 1.0026,8)
    elif buy_order_funding == "XBT":
        pair = (buy_coin + buy_order_funding).lower()
        order_price = price_xbt(pair)
        total_cost = round(float(quantity_buy) * float(order_price) * 1.0026,8)
    else:
        pair = (buy_coin + buy_order_funding).lower()
        order_price = price_usd(pair)
        total_cost = round(float(quantity_buy) * float(order_price) * 1.0026,2)
    return close(
        intent_request["sessionAttributes"],
        "Fulfilled",
        {
            "contentType": "PlainText",
            "content": """Great news, your buy order has been filled for {} {} for a total amount {} {}!
            """.format(
                 quantity_buy, buy_coin, buy_order_funding, total_cost
            ),
        },
    )
    
    """ Performs dialog management and fulfillment for sell order. """
    
def Sell_Order(intent_request):
    sell_coin = get_slots(intent_request)["SellCoin"]
    quantity_sell = get_slots(intent_request)["QuantitySell"]
    sell_order_type = get_slots(intent_request)["SellOrderType"]
    sell_funding_type = get_slots(intent_request)["SellFundingType"]
    # if source == "DialogCodeHook":
        # Perform basic validation on the supplied input slots.
        # Use the elicitSlot dialog action to re-prompt
        # for the first violation detected.
        # slots = get_slots(intent_request)
        # validation_result = validate_data(buy_coin, quantity_buy, buy_order_type, buy_order_funding, buy_confirmation_total, intent_request)
        # if not validation_result["isValid"]:
            # slots[validation_result["violatedSlot"]] = None  # Cleans invalid slot
            # Returns an elicitSlot dialog to request new data for the invalid slot
            # return elicit_slot(
            #     intent_request["sessionAttributes"],
            #     intent_request["currentIntent"]["name"],
            #     slots,
            #     validation_result["violatedSlot"],
            #     validation_result["message"],
            # )
        # Fetch current session attibutes
        # output_session_attributes = intent_request["sessionAttributes"]
        # return delegate(output_session_attributes, get_slots(intent_request))
    order_price = 0
    if sell_funding_type == "ETH":
        pair = (sell_coin + sell_funding_type).lower()
        order_price = price_eth(pair)
        total_cost = round(float(quantity_sell) * float(order_price) * 1.0026,8)
    elif sell_funding_type == "XBT":
        pair = (sell_coin + sell_funding_type).lower()
        order_price = price_xbt(pair)
        total_cost = round(float(quantity_sell) * float(order_price) * 1.0026,8)
    else:
        pair = (sell_coin + sell_funding_type).lower()
        order_price = price_usd(pair)
        total_cost = round(float(quantity_sell) * float(order_price) * 1.0026,2)
    return close(
        intent_request["sessionAttributes"],
        "Fulfilled",
        {
            "contentType": "PlainText",
            "content": """Great news, your sell order has been filled for {} {} for a total amount {} {}!
            """.format(
                 quantity_sell, sell_coin, sell_funding_type, total_cost
            ),
        },
    )
    
def Fetch_Price(intent_request):
    fetch_map = {
        'ETH': 'ethusd',
        'ADA': 'adausd',
        'XBT': 'xbtusd',
        'XRP': 'xrpusd',
        'DOT': 'dotusd',
        'FIL': 'filusd',
        'LINK': 'linkusd',
        'LTC': 'ltcusd', 
    }
    fetch_input = get_slots(intent_request)["CoinFetch"]
    current_price = price_usd(fetch_map[fetch_input])
    return close(
        intent_request["sessionAttributes"],
        "Fulfilled",
        {
            "contentType": "PlainText",
            "content": """The current market price of {} is {}.
            """.format(fetch_input, current_price
                 
            ), 
        },
    )
    
def Stake(intent_request):
    stake_coin = get_slots(intent_request)["StakeCoin"]
    stake_amount = get_slots(intent_request)["StakeAmount"]
    return close(
        intent_request["sessionAttributes"],
        "Fulfilled",
        {
            "contentType": "PlainText",
            "content": """Congratulations, you have successfully staked {} {} at 12%, payed out twice weekly!
            """.format(
                 stake_amount, stake_coin
            ),
        },
    )    
    
    
### Intents Dispatcher ###
def dispatch(intent_request):
    """
    Called when the user specifies an intent for this bot.
    """

    intent_name = intent_request["currentIntent"]["name"]

    # Dispatch to bot's intent handlers
    if intent_name == "BuyOrder":
        return Buy_Order(intent_request)
    if intent_name == "SellOrder":
        return Sell_Order(intent_request)
    if intent_name == "FetchPrice":
        return Fetch_Price(intent_request)
    if intent_name == "Stake":
        return Stake(intent_request)

    raise Exception("Intent with name " + intent_name + " not supported")


### Main Handler ###
def lambda_handler(event, context):
    """
    Route the incoming request based on intent.
    The JSON body of the request is provided in the event slot.
    """

    return dispatch(event)
