# REST API Design Best Practices

This is an effort to summarize the best practices to follow in designing the RESTful APIs. 
The list is not comprehensive, most of the items are stemmed from experience and industry acceptance. 

Examples in this document, uses `json` as the content type, but they also applies to other content types. 

## Terminology

Before we start talking about the best practices, lets try to understand some of the basics of REST and the Http.
+ **Resource** - Is an object or representation which has some associated data with it and there can be set of methods
  to operate on it.
  
  Example: `Student` resource, `Employee` resource and operations like `Create`, `Get`, `Delete`
  
+ **Collection** - Is a set of resources.
  
  Example: `Students` collection, `Employees` collection
  
+ **Resource URL** - Uniform Resource Locator. A path through which a resource(s) can be located and actions can be performed.

   A URL is a specific type of Uniform Resource Identifier (URI). A generic URI is of the form:
   ```text
   scheme:[//[user[:password]@host[:port]][/path][?query][#fragment]
   ```
   
   It comprises:
   
   - _scheme:_ Like http(s), ftp, mailto, file, data, irc
   - _Two Slashes:_ 
   - _authority section:_ An optional username:password followed by @ symbol
   - _host:_ Consisting of a registered hostname or IP address 
   - _port:_ An optional port number, separated from hostname with colon.
   - _path:_ which contains the data or resource path
   - _query:_ An optional query paramerts in the form key=value separated by &
   - _fragment:_ 
   
+ **Actions** - A standard HTTP Verb, which indicates a type of action that can be performed on a resource.

   - _GET_: Requests data from a resource. Should not produce any side effects.
   - _POST_: Requests to create a resource or a collection.
   - _PUT_: Request to update a resource or create the resource if it does not exist
   - _DELETE_: Request to delete a resource.
   - _HEAD_: Request to get metadata of a resource. Resources that support GET may support HEAD as well.
   - _PATCH_: Request to apply a partial update to a resource
   - _OPTIONS_: Request to retrieve information about a resource, at a minimum returning valid methods for this resource.

+ **Path Parameters** -
    
    Path parameters are part of the URL, which can be varied and represents resource hierarchy.
    
+ **Query Parameters** - 

   Query parameters are added to the url after the ? mark, while a path parameter is part of the regular URL.

+ **Http Headers** - 

   HTTP headers allow the client and the server to pass additional information with the request or the response.
   A request header consists of its case-insensitive name followed by a colon ' : ', then by its value

+ **Http Body** - 

   Is the data that is sent along with a request.


## Resource Representation (Resource URLs)

1. __Make sure to have a unique identifier for each `Resource`__

   For example: `Student ID` for `Student` resource. This will help locate a specific resource using its ID.
   
2. __Endpoint URL should only contain resource/collection names (nouns) not actions/verbs__

   There is no need to have the 'action' in the resource URL, the action that can be performed on a resource/collection 
   can be expressed through Http Verb.    
   **Examples:**
   
   ```text
   * Not Recommended:
   
   /addEmployee
   /deleteEmployee
   /updateStudent 
   
   * Recommended
   
   /employees
   /employees/{empid}
   ```
   
3. __Use collection names to represent resources__

   To narrow down to a specific resource in a collection, use the unique id of the resource.

   **Examples:**
   
   ```text
   - /students     - To represent Student resource
   - /employees    - To represent Employee resource
   
   - /students/1000    - To locate Student resource with id 1000
   - /employees/123    - To locate Employee resource with id 123
   
   - /students/1000/courses    - Course resource nested under Student resource
   - /students/1000/courses/101    - Specific Course resource with id 101 for Student resource with id 1000
   
   - /courses/
   - /courses/101/students
   - /courses/101/students/1000
   ```
4. __Use HTTP Verbs to represent an action on a resource. Do not use action names in URLs__

    POST - create a resource
    GET - retrieve a resource
    PUT - update a resource (by replacing it with a new version)*
    PATCH - update part of a resource (if available and appropriate)*
    DELETE - remove a resource
    
   **Examples:**
   ```text
    - POST /students   -> to Create one/more Student resource in collection.
    - GET /students    -> to Get Students collection
    - GET /students/1000   -> to Get Student resource with id 1000
    - DELETE /students/1000    -> Delete a Student resource with id 1000
    ```
5. __Use query parameters to further narrow down or filter or sort on a resource or collection__

   _Searching_, _Filtering_, _Sorting_ and _Pagination_ are some common actions that we perform on a resource. 
   All of these actions are simply a GET on a resource or collection with additional conditions. 
   There is no need for special endpoints to handle these actions, instead can be achieved through query parameters.
   
   **Sorting**:
   ```text       
   GET /employees?sort=rank_asc
   GET /employees?sortOn=rank&sortOrder=asc
  
   GET /employees?sort=-rank,+age  -- sort by age ascending and rand descending
   ```
   
   **Filtering**:
   ```text
    GET /employees?country=US
    GET /employees?country=US&dept=IT
    GET /employees?country=US&dept=IT&sortOn=firstName&sortOrder=asc
   ```
   **Searching**:
   ```text
   GET /employees?search=James
   GET /students?search=Algorithms
   ```
   **Pagination**
   ```text
   GET /students?page=21
   ```
   **Limit the response fields**
   ```text
   Use `fields` params to let client decide which fields should be returned in response.

   GET /students?fields=id,firstName   
   ```
   
6. __Use `Path Aliases` to group multiple query parameters__ 
   
   Consider packaging multiple params into easily navigate-able paths. As in below example `/students/enrolled` can be a
   _path alias_ to get all enrolled students, which otherwise will be represented with several query parameters like
   `/students?active=true&enrolled=true&year=2017`
   
   ```text
   GET /students/enrolled
   
   GET /students/graduated
   ```
7. __How about actions that does not fit into simple CRUD operations__

    Some operations like below can not be mapped to world of CRUD and to a particular resource.
        - login
        - logout
        - print
        - transfer funds
        - cancel a booking
       
    So, how to represent the endpoints for these operations? Recommended:
    
    - First, try to see if the action can fit on to an existing resource or there is a need for new resource.
         
    For example, `cancel a booking` can be mapped to a `booking` resource and `transfer funds` can be mapped on to a `accounts` resource.
         
     ```text
       -- to cancel a booking
       DELETE /bookings/101  
       PUT /bookings/101
       
       -- to transfer funds from one account to another, and also gives options to extend with other services
       POST /accounts/100?service=transfer
       POST /accounts/100?service=statements    
    
     ```
    - If not, try to designate a 'controller' resource for each operation.
      
      For example `login` and `logout` can be mapped to a URI like below
        
      ```text
        // we are trying to group login and logout on to a `auth`.
        POST /auth/login
        POST /auth/logout
    
        or
     
        POST /auth?action=login
        POST /auth?action=logout
    
      ```
      A `transfer of funds` and `print` can be grouped in to a `servces` endpoint.
        
      ```text
        POST /services/print
        POST /services/transfer
      ```
        
8. __How about retrieving multiple resources in single GET__

   A multi-resource search does not really make sense to be applied to a specific resource's endpoint.
   In this case, `GET /search` would make the most sense even though it isn't a resource.
   
   In special cases like above, it is fine to have a special endpoints like `/search`.

9. __Versioning__

   Use major version in the URL. Optionally represent minor version using headers.
   ```text
   POST /api/v1/students/
   POST /api/v1.0/students/

   POST /api/v2.1/students/
   ```

   If your API changes frequently, use an HTTP header to represent the minor version. 
   Other techniques to consider would be to use a date based minor version as header values.

   ```text
   "X-API-VERSION" : "1.1"
   "X-VERSION" : "20171228"
   ```
10. __When to use `Path Params` vs `Request Params`__

   Path params are used to identify a specific resource or resources, while query parameters are used to sort/filter those resources. 
   And also paths tend to be cached, parameters tend to not be.
   
   It is recommended to use any required parameters in the path, and any optional parameters should certainly be query string parameters. 
   Putting optional parameters in the URL will end up getting really messy when trying to write URL handlers that match different combinations.

## Request

1. __Make use of standard request headers, only introduce custom headers if required__

   Using HTTP standard headers leads less confusion and also the API will comply to standards.
   [List of headers HTTP 1.1](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)

2. __Try to use `commonly` used non-standard headers when standard headers does not fit your requirement__

   This Wikipedia page will be a good start: https://en.wikipedia.org/wiki/List_of_HTTP_header_fields
   
3. __When to use `Query Params` Vs `Headers`__

   Usually query parameters will have a strong relationship to the resource under request. For example: 'student_id' 
   belongs to 'student' resource. If the parameter under consideration has a strong affinity to a resource, then using it
   as a header is not the right choice.

   Custom headers should only be used for things that don't involve the `name of the resource`, `the state of the resource`
   or `the parameters directly affecting the resource`. Http headers should be used to send contextual information about 
   the request to the resource.
   
   For example: 
   a `Supervisor` and an `Employee` both request `Timesheet` resource, but employee need to see only his/her timesheets 
   while supervisor should be able to see all his/her reportee's.

   The context employee/supervisor(auth) is a good candidate for header whereas querying for timesheets in a given month 
   (month=Jan) is a good candidate for query parameter.

   Header parameters are typically added out-of-band by applications like user agents and/or proxies. 
   If you need to allow message intermediaries to add the parameter to the message, then it should be located in the header.

   Query string is reserved for end users/applications that are invoking resource. 
   On the other hand headers are reserved for applications and intermediaries that are facilitating this invocation.
   
4. __Generate a unique `Request ID` for each received request and pass it along with response__



## Processing

1. __Always validate a request before start processing it__

   //TODO

2. __Always return meaningful error codes and/or messages__

   //TODO
   



## Response

1. __Use Http Status codes to send the request status__

   The HTTP standard provides over 70 status codes to describe the return values. Most of the APIs would need to dela with below: 
   
   - _200 (OK)_: To indicate that the request is succeeded.
   - _201 (Created)_: To indicate resource `created` successfully
   - _201 (Accepted)_: To indicate the request is accepted for processing, but the processing has not been completed. Its purpose is to allow a server to accept a request for some other process (perhaps a batch-oriented process)
   - _204 (No Content)_: The server has fulfilled the request but does not need to return an entity-body. Useful when a resource is `deleted`
   - _304 (Not Modified)_: To Indicate a resource is not modified. Client can use cached data.
   - _400 (Bad Request)_: To indicate the request was invalid or can not be served. Exact error should be explained in the payload.
   - _401 (Unauthorized)_: To indicate the request requires authentication.
   - _403 (Forbidden)_: To indicated the request is OK but is refusing or the access is not allowed
   - _404 (Not found)_: To indicate there is no resource behind the URI
   - _422 (Unprocessable Resource)_: 
   - _500 (Internal Server Error)_: 
   - _501 (Not Implemented)_: To indicate the server does not support the functionality required to fulfill the request. 
   
2. __Have all APIs to return consistent response format__

   This will enable the client to look up for information in the response in a uniform and consistent way. 
   
   You can come up with your own standard response format, below is an example. 
   
   ```json
    {
      "status": null,
      "errors": null,
      "meta": null,
      "data": null,
      "links": null
    }
   ```

2. __Wrap API response into a `data` container object__

   It is a good practice to wrap the response data into a container object like `data`
   ```json
    {
      "status": {
   
      },
      "errors": null,
      "data": {
        "id": 1234,
        "firstName": "James",
        "lastName": "Bond",
        "email": "james.bond@007.com"
      }
    }
   ```
   
3. __Include `status` section in the response__

   It is a good practice to include `status` object in the response body along with the standard http status codes.
   
   ```json
   {
     "status": {
       "code": 400,
       "message": "Bad Request"
     },
     "errors": [],
     "data": null
   }
   ```
      
4. __Include `errors` section in the response__

   It is a good practice to return the list of errors in the response body along with the http status codes. 
   Use an object like `errors` to pass on the errors.
   
   ```json
   {
    "status": {
      "code": 201,
      "message": "User Created Successfully"
    },
    "errors": [ 
       {
         "code": "ERR001",
         "description": "Validation Error",
         "target": "Email Address"
       }
     ],
     "data": null
    }
   ```

5. __Distinguish `Errors` and `Faults`__

   - _Errors_: Classified as client passing an invalid data and the API is _correctly_ rejecting that data.
   
      Errors do not contribute to the overall API availability. These are typically `4XX` errors.
      
      Examples include invalid format, missing required params, invalid path, method not supported, invalid credentials etc..
      
   - _Faults_: Classified as API failing to correctly return response to a valid request.
   
      Faults to contribute to the overall API availability. These are typically `5XX` errors.

6. __Use Pagination where required__

   As data grows, so do collections. Planning for pagination is important for all APIs. 
   
   APIs that return collections must support pagination. 

## Resource Creation

1. __Use `POST` to create resources__

   When creating new resources, `POST` should be used.

2. __Allow to creating a collection of resources instead of one in a single request__

   **POST /students**
   ```json
   [
     {
       "firstName": "James",
       "lastName": "Bond",
       "email": "james.bond@007.com"
     },
     {
       "firstName": "James",
       "lastName": "Cooper",
       "email": "james.cooper@007.com"
     }
   ]
   ```
   **Instead Of:**
   ```json
   {
     "firstName": "James",
     "lastName": "Bond",
     "email": "james.bond@007.com"
   }
   ```

3. __Let your API generate the resource IDs at server side, Do not rely on client to send unique resource ID__

   `Do not support` accpeting the Ids of the resource being created in the request.
   ```json
   [
     {
       "Id": 1001,
       "firstName": "James",
       "lastName": "Bond",
       "email": "james.bond@007.com"
     },
     {
       "Id": 1002,
       "firstName": "James",
       "lastName": "Cooper",
       "email": "james.cooper@007.com"
     }
   ]
   ```

4. __Return the created / modified state as response of create or update response.__

   Its always useful and specific cases like the resource 'id' which the client need later to lookup the created resource.

   **POST /students**
   ```json
   [
     {
       "firstName": "James",
       "lastName": "Bond",
       "email": "james.bond@007.com"
     },
     {
       "firstName": "James",
       "lastName": "Cooper",
       "email": "james.cooper@007.com"
       }
   ]
   ```
   **Response:**
   ```json
   [
     {
       "id": 1234,
       "firstName": "James",
       "lastName": "Bond",
       "email": "james.bond@007.com"
     },
     {
       "id": 1235,
       "firstName": "James",
       "lastName": "Cooper",
       "email": "james.cooper@007.com"
     }
   ]
   ```

5. __Use HATEOS to return the newly created resource URL.__

   Which helps the clients to navigate and locate the resource easily.

   **Response:**
   ```json
   [
     {
       "id": 1234,
       "firstName": "James",
       "lastName": "Bond",
       "email": "james.bond@007.com",
       "links": [
         {
         "rel": "self",
         "href": "http://host:port/xyz/students/1234"
         }
       ]
     },
     {
       "id": 1235,
       "firstName": "James",
       "lastName": "Cooper",
       "email": "james.cooper@007.com",
       "links": [
         {
         "rel": "self",
         "href": "http://host:port/xyz/students/1235"
         }
       ]
     }
   ]
   ```

## Getting Resources

1. __When querying collection and applying filtering/sorting/pagination follow below order__

   - Filtering: filter the collection
   - Sorting: Sort the filtered collection
   - Pagination: sorted collection is paginated
   
2. __Use HATEOS to represent pagination data__

   Use HATEOS to pass along the links to the pages for the client to navigate the pages easily. 
   Send pre-built links to next, prev, current, first, last pages. 
   
   **Response:**
      ```json
      {
      "status": {
         "code": 200,
         "message": "Operation successful"
      },
      "errors": null,
      "data":[
        {
          "id": 1234,
          "firstName": "James",
          "lastName": "Bond",
          "email": "james.bond@007.com",
          "links": [
            {
            "rel": "self",
            "href": "http://host:port/xyz/students/1234"
            }
          ]
        },
        {
          "id": 1235,
          "firstName": "James",
          "lastName": "Cooper",
          "email": "james.cooper@007.com",
          "links": [
            {
            "rel": "self",
            "href": "http://host:port/xyz/students/1235"
            }
          ]
        }
      ],
      "links": [
         {
            "rel": "self",
            "href": "http://host:port/xyz/students?page=5&count=20"
         },
         {
            "rel": "prev",
            "href": "http://host:port/xyz/students?page=4&count=20"
         },
         {
            "rel": "next",
            "href": "http://host:port/xyz/students?page=6&count=20"
         },
         {
            "rel": "first",
            "href": "http://host:port/xyz/students?page=1&count=20"
         },
         {
            "rel": "last",
            "href": "http://host:port/xyz/students?page=121&count=20"
         }
       ]
      }
      ```

3. __Use `meta` fields to to indicate the information on the results__

   It is also a good practice to include the meta fields like `number of records` `page number` `offset` etc. in the response along with the results.


## Updating Resources



## Deleting Resources

