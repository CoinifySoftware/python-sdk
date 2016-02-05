# python-sdk
A Python wrapper for Coinify merchant API and callbacks

This Python SDK consists of two classes, `CoinifyAPI` and `CoinifyCallback`, which are designed to make it easier for you,
the developer, to utilize the [Coinify API](https://coinify.com/docs/api) and validate [IPN callbacks](https://coinify.com/docs/api/#callbacks) from Coinify, respectively. 

## CoinifyAPI

### Creating an instance
The `CoinifyAPI` class is instantiated as follows:

```python
api_key = "<my_api_key>"
api_secret = "<my_api_secret>"
api = CoinifyAPI(api_key, api_secret)
```

### Response format
The `CoinifyAPI` returns responses as they are described in [Response format](https://coinify.com/docs/api/#response-format) in the API documentation.

The JSON responses are automatically decoded into Python dict's, so you can for example do the following:

```python
response = api.invoices_list()

if not response['success']:
    api_error = response['error']
    return "API error: %s (%s)" % (api_error['message'], api_error['code'] )

invoices = response['data']
```

### Rates
With the [Coinify rates API](https://coinify.com/docs/api/#rates) you can *list* the current exchange rates for all supported currencies. 
Returned rates will define the exchange rate for the number of fiat currency units equivalent to one BTC.

This end-point is public and no API key/secret is needed.

`buy` is the rate for buying BTC from Coinify.

`sell` is the rate for selling BTC to Coinify.

#### Listing rates for all currencies
```python
response = api.rates_get()
```

#### Listing rates for a specific currency
```python
// f.e. currency = 'USD'
response = api.rates_get(currency)
```

#### Listing rates for all supported altcoins
```python
response = api.altrates_get()
```

#### Rate for a specific altcoin
```python
// f.e. altcoin = 'LTC'
response = api.altrates_get(altcoin)
```

### Account
With the [Coinify account API](https://coinify.com/docs/api/#account) you can execute operations or get data regarding your merchant account.

#### Check account balance
```python
response = api.balance_get()
```

### Invoices
With the [Coinify invoice API](https://coinify.com/docs/api/#invoices), you can *list* all your invoices, *create* new invoices, *get* a specific invoice and *update* an existing invoice as follows:

#### Listing all invoices
```python
response = api.invoices_list()
```

The interface for the `invoiceList` method is the following:

```python
invoices_list()
```

#### Creating a new invoice
**Example:** Create an invoice for 20 USD.

```python
plugin_name = 'MyPlugin'
plugin_version = '1'

response = api.invoice_create(20.0, "USD", plugin_name, plugin_version)
```

The interface for the `invoice_create` method is the following:

```python
invoice_create(amount, currency, 
    plugin_name, plugin_version,
    description=None, custom=None, 
    callback_url=None, callback_email=None, 
    return_url=None, cancel_url=None
    input_currency=None, input_return_currency=None)
```

#### Get a specific invoice
```python
invoice_id = 12345
response = api.invoice_get(invoice_id)
```

The interface for the `invoice_get` method is the following:

```python
invoice_get(invoice_id)
```

#### Update an existing invoice
```python
invoice_id = 12345
response = api.invoice_update(invoice_id, 'Updated description')
```

The interface for the `invoice_update` method is the following:

```python
invoice_update(invoice_id, description=None, custom=None)
```

#### Pay with another input currency
```python
invoice_id = 12345
currency = 'LTC'
return_address = 'Ler4HNAEfwYhBmGXcFP2Po1NpRUEiK8km2'
response = api.invoice_input_create(invoice_id, currency, return_address)
```

The interface for the `invoice_input_create` method is the following:

```python
invoice_input_create(invoice_id, currency, return_address)
```


### Buy orders
With the [Coinify Buy order API](https://coinify.com/docs/api/#buy-orders), *preapproved* merchants
can use their fiat account balance to buy bitcoins. The API exposes methods
for *listing* all buy orders, *getting* a specific buy order, and *create* and *confirm*
new buy orders:


#### Listing all buy orders
```python
response = api.buy_orders_list()
```

The interface for the `buy_orders_list` method is the following:

```python
buy_orders_list()
```

#### Get a specific buy order
```python
buy_order_id = 12345
response = api.buy_order_get(buy_order_id)
```

The interface for the `buy_order_get` method is the following:
```python
buy_order_get(buy_order_id)
```

#### Creating a new buy order
**Example:** Buy bitcoins for 100 USD.

```python
amount = 100
currency = 'USD'
btc_address = '<my_bitcoin_address>'

response = api.buy_order_create( amount, currency, btc_address )
```

The interface for the `buy_order_create` method is the following:
```python
buy_order_create( amount, currency, btc_address, 
    instant_order=None, callback_url=None, callback_email=None )
```

#### Confirming a buy order
```python
buy_order_id = 12345
response = api.buy_order_confirm(buy_order_id)
```

The interface for the `buy_order_confirm` method is the following:
```python
public function buy_order_confirm(buy_order_id)
```

### Input currencies
Apart from receiving payments in Bitcoin (`BTC`), we also support a range of other input currencies such as Litecoin (`LTC`), Ether (`ETH`), and Dogecoin (`DOGE`).

#### Supported input currencies
```python
response = api.input_currencies_list()
```


### Catching errors
The `CoinifyAPI` internally uses the Python `requests` module for
communicating with the API, so you might want to watch out for the exceptions described
on the (requests errors and exceptions documentation)[http://docs.python-requests.org/en/latest/user/quickstart/#errors-and-exceptions]:

```python
try:
    response = api.invoices_list()
except requests.exceptions.ConnectionError as e:
    # Handle exception
```

If no exception is thrown, the result (`response`) is a dict, and the [response format](https://coinify.com/docs/api/#response-format) from the API documentation is used, which can communicate an error (if `response['success']` is `False`) or a successful API call (if `response['success']` is `True`).


## Validating callbacks
If you choose to receive HTTP callbacks for when your invoice state changes and handle them with Python code, you can use the `CoinifyCallback` class to validate the callback - i.e. to make sure that the callback came from Coinify, and not some malicious entity:

```python
ipn_secret = '<my_ipn_secret>'
callback_validator = CoinifyCallback(ipn_secret)

postdata_raw = "" # Get the raw HTTP data from your web framework, a JSON string
signature = "" # Extract the contents of the 'X-Coinify-Callback-Signature' HTTP header

is_valid = callback_validator.validate_callback(postdata_raw, signature)
```