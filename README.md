# python-copaco-connections
Easy python integrations for the Copaco Customer Connections

## Package use

This package has three main functions:

- Retrieve the pricelist of Copaco
- Send orders to Copaco
- Retrieve order responses

These three functions will be explained seperately below.

You need to requests a connection with Copaco first before you can use this package.
More info here: https://www.copaco.com/en-be/customer-service-e-commerce-fulfillment

## Getting started

### Install

Install with pip.

```python
pip install python-copaco-connections
```

## Pricelist

### Limitations

The pricelist is limited to the Copaco BE - Dutch Productlist in CSV format via FTP. Might be extended in the future.

### Import

Import the package and the CopacoConnectionBE object.

```python
from copaco.connection import CopacoConnectionBE
```

### Setup connection

Make the connection with your provided FTP credentials.

```python
conn = CopacoConnectionBE(FTP_HOST, FTP_LOGIN, FTP_PASSWD)
```

### Retrieve the pricelist

You can retrieve the pricelist as follows:

```python
priceList = conn.priceList.get()
```

This will return an ordinary list which contains PriceListItem objects.

You can find the attributes of this object and their use below:

**PriceListItem object**

| Attribute  | Contains | Type |
| ------------- | ------------- |-------------|
| article  | Article number  | string |
| vendorCode  | Unique vendor code  | string |
| description  | Short description  | string |
| price  | Price, excluding levies  | float |
| priceWithLevies  | Price, including levies | float |
| stock  | Amount of stock available  | integer |
| hierarchy  | Product hierarchy  | string |
| unspscCode  | UNSPSC code  | string |
| EAN  | EAN code  | string |
| statusCode  | Status code (0 - 12). Refer to docs.  | integer |
| status  | Human-readable status  | string |
| auvibel  | Price of Auvibel  | float |
| reprobel  | Price of Reprobel  | float |
| recupel  | Price of Recupel  | float |
| bebat  | Price of Bebat  | float |
| nextDelivery  | Next delivery date of this product | date |
| nextDeliveryAmount  | Amount that will be delivered on next delivery | integer |
| inventoryStatusCode  | ATP code | string |
| inventoryStatus  | Human-readable ATP code | string |


### Sample script

You can find a sample script at examples/get-pricelist.py


## Orders

### Limitations

Fields that are not supported at this moment: 

- end-user_information
- orderline_info
- registration_info
- notification

### Import

Import the package and the CopacoOrders object.

```python
from copaco.orders import CopacoOrders
```

### Setup connection

Make the connection with your Customer ID and provided Sender ID for orders.

```python
orders = CopacoOrders(CUSTOMER_ID, SENDER_ID)
```

### Create a new order

Before creating an order you have to understand that creating an order does not equal sending the order to Copaco for processing.
You need to create the order internally first, and then send the order over to Copaco.

```python
orders = CopacoOrders(CUSTOMER_ID, SENDER_ID)
order = orders.create(external_document_id, supplier, customer_ordernumber, completedelivery, requested_deliverydate=None, recipientsreference=None)
```

The above function is a simplified function that allows you to create a basic Order object, with no lines attached to it.

| Param  | Required? | Type | Format | Remarks
| ------------- | ------------- |-------------|-------------|-------------|
| external_document_id  | X  | string | Free text field | Contains an unique number or token from your system |
| supplier  | X  | string | 'COPACO'/'6010' | 'COPACO' = Copaco NL, '6010' = Copaco BE |
| customer_ordernumber  | X  | string | Free text field | Contains an unique purchase order number from your system |
| completedelivery  | X  | string | 'Y'/'N' | 'Y' = Complete delivery, 'N' = Partial delivery allowed |
| requested_deliverydate  |  | string | 'DD-MM-YYYY' | Date you would like the order to be delivered |
| recipientsreference  |  | string | Free text field | Reference for the goods receiver |


### Attach lines to an existing order

You can attach lines to an existing order.

```python
order.addOrderLine(item_id, tag, quantity, price=None, currency=None, deliverydate=None, textqualifier=None, text=None)
```

| Param  | Required? | Type | Format | Remarks
| ------------- | ------------- |-------------|-------------|-------------|
| item_id  | X  | string | Free text field | Contains the part number |
| tag  | X  | string | 'PN'/'MF'/'CU' | 'PN' = Supplier part number, 'MF' = Manufacturer part number, 'CU' = Customer part number |
| quantity  | X  | integer | integer | Only full numbers allowed |
| price  |  | float | float | Must match with the price in ERP system or within agreed margins |
| currency  |  | string | 'EUR' | At this moment only 'EUR' is allowed, might be extended later |
| deliverydate  |  | string | 'DD-MM-YYYY' | Date you would like the line to be delivered. Overrules delivery date on the order. |
| textqualifier  |  | string | '0001'/'BID' | '0001' = Instruction line, 'BID' = Line contains special BID |
| text  |  | string | Free text field | If qualifier '0001' = Free text field, If qualifier 'BID' = Special BID number |


### Set shipping address

You can set the shipping address of an order if you would like to ship to another address that is not your default.
If no shipping address is supplied, your default address will be used.

```python
order.setShippingAdress(firstname, lastname, street, postalcode, city, country)
```

Country has to be the country code.

### Send order to Copaco

Once the full order has been created, you can send the order to Copaco for processing.

```python
orders = CopacoOrders(CUSTOMER_ID, SENDER_ID)
order = orders.create(external_document_id, supplier, customer_ordernumber, completedelivery)
orders.sendToCopaco(order)
```

### Sample script

You can find a sample script at examples/create-order.py


## Order responses

### Limitations

Not all fields are supported at this moment. List of unsupported fields is not available at this moment.

### Import

Import the package and the CopacoOrders object.

```python
from copaco.orders import CopacoOrders
```

### Setup connection

Make the connection with your Customer ID and provided Sender ID for orders.

```python
orders = CopacoOrders(CUSTOMER_ID, SENDER_ID)
```

### Retrieve responses

Copaco has four different responses.

| Response type  | Contains | 
| ------------- | ------------- |
| INT  | Initial response. Indicates that the order is received. |
| OBV  | Order confirmation. Contains information on how the order is processed. |
| FAC  | Invoice in XML format. |
| PAK  |  Dispatch advice response. Generated after the goods are picked and ready for shipment. |

You can retrieve these responses seperately or all at once.

```python
orders = CopacoOrders(CUSTOMER_ID, SENDER_ID)
intResponses = orders.getResponses(type='INT')
obvResponses = orders.getResponses(type='OBV')
pakResponses = orders.getResponses(type='PAK')
facResponses = orders.getResponses(type='FAC')
allResponses = orders.getResponses(type='ALL')
```

Requesting a specific type will return an array that contains objects of the XML returned.
Requesting all types will return a dictionary with the following format: 

```python
{
  'INT' : [intResponses], 
  'OBV' : [obvResponses],
  'FAC' : [facResponses],
  'PAK' : [pakResponses]
}
```

### Structure

The structure of these responses are the same as the XML that is returned, and thus is defined by Copaco.
Be aware that there are a lot of inconsistencies in the XML format: camelCase, snake_case, PascalCase are all used.

Refer to the XML documentation to know the structure.

### Sample script

You can find example responses for each type at examples/xml-responses

You can find a sample script for each response at examples/get-response-XXX.py where XXX is the type.