# Genesys: A Product Management System

![logo](https://user-images.githubusercontent.com/29889429/77192438-96a93680-6b02-11ea-978a-d71a8d15523e.png)

An expandable REST API driven Product Management System to facilitate product management based on the concept of an Epic and story Backlog as well as a product catalogue.

The API specification was made on Swaggerhub following the OAS 2.0 guidelines. 

## SWAGGER DOCUMENTATION
Link to swagger documentation: [GENESYS API](https://app.swaggerhub.com/apis/nikjohn7/Genesys/1.0.0) 

## Features

### Security - Authentication and Authorisation


There are three roles in this system:
+ Product Manager - is responsible for setup and administration of the system entities like ```Clients```, ```Products```, ```Categories```, ```Epics```, ```User Stories``` and ```Utilities```
+ Employees - report to the Product Manager, and have delegated authority to perform restricted administration
+ Clients - primarily use the system and also suggest new features through Epics and Stories

The authentication process is as follows:

- The Product Manager and Employees are authenticated using API keys - we let the option to authenticate using a header or query param
- Clients are authenticated basic username:password
- All autheticated users are issued a basic JWT token to access API endpoints
- All users are authorised based on their roles

**NOTE**:  The "OAuth2" declaration has been used here but a real
    OAuth2 workflow isn't implemented: the intention here is just to be able to extract scopes from a passed JWT token 

```
securityDefinitions:
  isRegistered:
    # This scheme uses the header: "Authorization: Basic {base64 encoded string defined by username:password}"
    # Scopes are not supported by this type of authorization.
    type: basic

  isProductTeam:
    # This scheme uses the header: "X-Custom-Key: {base64 encoded string}"
    # Scopes are not supported by this type of authorization.
    type: apiKey
    in: header
    name: X-Custom-Key
  hasRole:
    # This scheme uses the header: "Authorization: Bearer {base64 encoded string representing a JWT}"
    # Alternatively, the query param: "access_token" may be used.
    #
    # In this system, we must use the query param: "access_token" version in order to avoid passing several headers with key 'Authorization'
    type: oauth2
    # The flow and URLs in spec are for documentary purpose only
    flow: accessCode
    authorizationUrl: 'https://dummy.oauth.net/auth'
    tokenUrl: 'https://dumy.oauth.net/token'
    # Required scopes are passed by the runtime to the authorizer
    scopes:
      ProductManager: scope of Product Manager
      Employee: scope of employees working under the Product Manager
      Client: scope of added clients

# Default Security requirements for all operations
security:
  - isRegistered: []
```
.
### Backlog Details
 A Product Backlog feature has been implemented in this system, which is a prioritized features list. This contains short descriptions of all functionality desired in the product, called Epics. 

#### Backlog

| EpicID| Description |
| --- | --- |
| EP001 | Revamp customer interactive chat |
| EP002 | Add PayPal as payment option |
.

Each Epic contains User Stories associated with it, which are more detailed descriptions of the feature vaguely described in the Epic, and are assigned to a module.

#### User Stories for a specific Epic

| storyID| Description | parentEpicID | status | module |
| --- | --- | --- | --- | ---|
| US001 | Add option to upload files through the chat | EP001 | In Progress | UI/UX |
| US002 | Check functioning of auto backup feature | EP001 | Assigned | Testing |

.

- The Product Manager can POST/GET/PUT/DELETE any Epic or User Story into the Backlog
- Employees can PUT/GET from the Backlog
- Clients can POST into the Backlog any feature they want to be implemented

.

### General Overview  of Remaining Modules
+ The system stores item data as a ```Product```. Products have fields to describe different aspects of the item and these fields can be used to filter through the data
+ All Products have a ```basePrice``` and are assigned to a ```taxCategory``` which defines the GST rate applied on it. There is also a provision to give some Products a ```discount```. 
+ Products are assigned to a certain ```Category```. The system supports categories at two levels: one top level of categories and one level of sub-categories. An example is **Electronics** is a parent category for **Smart Phone** and **Laptop**
+ External users are added as ```Clients```. 
+ Clients can view various products and categories, while the Product Manager and employees can add/modify categories and can add new Epics and User Stories to the Product Backlog

.
### Data Handling
+ ```Products``` can be loaded into the system as a JSON object (via ```POST /product```) or as form data (via ```POST /product/{productSKU}```)
+ Categories and Clients can be added to the system in a similar manner via ```POST /category``` and ```POST /clients```(JSON object) and ```POST /category/{categoryID}``` (form data)
+ Categories, Clients and Products can be modified via ```PUT /category```, ```PUT /clients/{clientID}``` and ```PUT /product``` respectively. The payload for these requests only need to contain the fields desired to be altered and the unique identifier (```productSKU ```, ```clientID``` or ```categoryID```).
+ Likewise ```DELETE /category/{categoryID}```, ```DELETE /clients/{clientID}``` and ```DELETE /product/{productSKU}``` is used to remove products and categories from the database. Removing a category will remove *all child categories* (and recursively, their child categories and so on), all products associated with these categories and any other metadata linked to them.

.
### Searching
+ Fetching all subcategories of a category is as simple as calling ```GET /category/{categoryID}/getSubCategories``` and getting all products associated with a category is achieved by ```GET /category/{categoryID}/getProducts```
+ The system also supports advanced search feature on products via ```POST/catalog/searchProducts```. This search feature allows users to search all various product attributes and any combination of them. This allows a single route to serve various search requirements

.
### Price Utilities
+ The tax rate can be adjusted for each category and viewed via the ```/utils/taxRates``` set of endpoints
+ Items can be put on discount, discounts can be altered and removed via tha ```/utils/discounts``` family of endpoints
+ A detailed split-up and calculation for the final price is obtained via ```POST /utils/calculateFinalPrice```

.
### Miscellaneous
+ endpoints with semantic significance and no functionality
.
## Key Contributions
This was my first experience working with REST APIs, and it has been a great journey of learning. Some of my key contributions in this project are:
+ Providing a Product Backlog through which clients can suggest various features or changes to the Product. I was able to communicate with a couple of Product Managers in order to understand the significance of a Backlog.
+ At first, I took the standard approach of providing multiple routes for various search filters. But after a lot of writing and rewriting, I was able to implement a single route for Searching, through which all kinds of combinations of filters can be applied
+ Rather than providing separate routes for Authorisation and Authentication, my entire project performs the same implicitly with the use of Roles.




