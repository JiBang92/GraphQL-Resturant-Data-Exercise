# **_<span style="color: #ff5733">GraphQL Restaurant Data Exercise</span>_**

<img href="./images/Node_Logo.png" alt="Node Logo"/>
<img href="./images/Express_Logo.png" alt="Express Logo"/>
<img href="./images/GraphQL_Logo.png" alt="GraphQL Logo"/>

## **_<span style="color: yellowgreen">Setup</span>_**

### 1. **_index.js_** file should contain:

```js
var { graphqlHTTP } = require("express-graphql");
var { buildSchema, assertInputType } = require("graphql");
var express = require("express");

// Construct a schema, using GraphQL schema language
var restaurants = [
  {
    id: 1,
    name: "WoodsHill ",
    description:
      "American cuisine, farm to table, with fresh produce every day",
    dishes: [
      {
        name: "Swordfish grill",
        price: 27,
      },
      {
        name: "Roasted Broccily ",
        price: 11,
      },
    ],
  },
  {
    id: 2,
    name: "Fiorellas",
    description:
      "Italian-American home cooked food with fresh pasta and sauces",
    dishes: [
      {
        name: "Flatbread",
        price: 14,
      },
      {
        name: "Carbonara",
        price: 18,
      },
      {
        name: "Spaghetti",
        price: 19,
      },
    ],
  },
  {
    id: 3,
    name: "Karma",
    description:
      "Malaysian-Chinese-Japanese fusion, with great bar and bartenders",
    dishes: [
      {
        name: "Dragon Roll",
        price: 12,
      },
      {
        name: "Pancake roll ",
        price: 11,
      },
      {
        name: "Cod cakes",
        price: 13,
      },
    ],
  },
];
var schema = buildSchema(`
type Query{
  restaurant(id: Int): restaurant
  restaurants: [restaurant]
},
type restaurant {
  id: Int
  name: String
  description: String
  dishes:[Dish]
}
type Dish{
  name: String
  price: Int
}
input restaurantInput{
  name: String
  description: String
}
type DeleteResponse{
  ok: Boolean!
}
type Mutation{
  setrestaurant(input: restaurantInput): restaurant
  deleterestaurant(id: Int!): DeleteResponse
  editrestaurant(id: Int!, name: String!): restaurant
}
`);
// The root provides a resolver function for each API endpoint

var root = {
  restaurant: (arg) => {
    // Your code goes here
  },
  restaurants: () => {
    // Your code goes here
  },
  setrestaurant: ({ input }) => {
    // Your code goes here
  },
  deleterestaurant: ({ id }) => {
    // Your code goes here
  },
  editrestaurant: ({ id, ...restaurant }) => {
    // Your code goes here
  },
};
var app = express();
app.use(
  "/graphql",
  graphqlHTTP({
    schema: schema,
    rootValue: root,
    graphiql: true,
  })
);
var port = 5500;
app.listen(5500, () => console.log("Running Graphql on Port:" + port));

export default root;
```

<br/>

### 2. **_package.json_** file should contain:

```json
{
  "name": "expressgraphql01",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "jrw@mit.edu",
  "license": "MIT",
  "dependencies": {
    "express": "^4.17.1",
    "express-graphql": "^0.11.0",
    "graphql": "^15.3.0"
  }
}
```

<br/>

### 3. Install dependencies:

```bash
npm install

# OR
# If you're starting with a fresh package.json,

npm install express express-graphql graphql --save
```

<br/>

### 4. Start the server at **_[localhost:5500/graphql](localhost:5500/graphql)_**

> NOTE: At this point, if you were to test any queries, you would receive **_null_** data because **_root_** hasn't been defined:
>
> ```js
> var root = {
>   restaurant: (arg) => {
>     // Your code goes here
>   },
>   restaurants: () => {
>     // Your code goes here
>   },
>   setrestaurant: ({ input }) => {
>     // Your code goes here
>   },
>   deleterestaurant: ({ id }) => {
>     // Your code goes here
>   },
>   editrestaurant: ({ id, ...restaurant }) => {
>     // Your code goes here
>   },
> };
> ```

<br/>

### 5. Define **_root_**

```js
var root = {
  restaurant: (arg) => restaurants[arg.id],
  restaurants: () => restaurants,
  setrestaurant: ({ input }) => {
    restaurants.push({ name: input.name, description: input.description });
    return input;
  },
  deleterestaurant: ({ id }) => {
    const exists = Boolean(restaurants[id]);
    restaurants = restaurants.filter((item) => item.id !== id);
    return { exists };
  },
  editrestaurant: ({ id, ...restaurant }) => {
    if (!restaurants[id]) {
      throw new Error("Contact Does Not Exist");
    }

    restaurants[id] = {
      ...restaurants[id],
      ...restaurant,
    };

    return restaurants[id];
  },
};
```

> ## **_<span style="color: skyblue">What Does This Do?</span>_**
>
> - **_restaurant_**: takes in an argument, in this case **_id_**, and returns the restaurant at the index matching **_id_**.
>
> - **_restaurants_**: simply returns all restaurants
>
> - **_setrestaurant_**: takes in an object as an argument. In this case, **_{ input }_** will be extracted from that object and used to:
>
>   - Create a new restaurant with a **_name_** and **_description_**
>   - Push the new restaurant into the array of existing restaurants
>   - Return the **_input_**
>
> - **_deleterestaurant_**: takes in an object as an argument. In this case, **_{ id }_** will be extracted from that object and used to:
>
>   - Search whether the contact exists or not
>   - Filter through the restaurants to return all restaurants that **_do not_** match the **_id_**
>   - Return a **_boolean_**
>
> - **_editrestaurant_**: takes in an object as an argument. In this case, **_id_** and a specific **_restaurant_** are extracted from that object and used to:
>   - Check if the restaurant exists
>   - If the restaurant exists, finds the restaurant and manipulates the restaurant's **_name_** and **_description_** through **_restaurantInput_**
>   - Return the edited restaurant

<br/>

### 6. Restart The Server And Test Your Queries

First, we will be setting up an **_$iid_** variable so we don't have to create new queries for each restaurant.

Under **_Query Variables_**, it should look something like this:

```
{
  "iid": 2
}
```

> NOTE: The **_Int = \<number\>_** value that you will see denotes a default value if a variable is not specified.

<br/>

**_<span id="top" style="color: skyblue">query Restaurant</span>_**

```
query getRestaurant ($iid: Int = 1) {
	restaurant(id: $iid) {
    id
    name
    description
    dishes {
      name
      price
    }
  }
}
```

> See the results [here](#getRestaurant)

<br/>

**_<span style="color: skyblue">query Restaurants</span>_**

```
query getRestaurants {
  restaurants {
    id
    name
    description
    dishes {
      name
      price
    }
  }
}
```

> See the results [here](#getRestaurants)

<br/>

**_<span style="color: skyblue">mutation setRestaurant</span>_**

```
mutation setRestaurants {
  setrestaurant(input: {
    name: "New Restaurant",
    description: "Opening Soon"
  }) {
    id
    name
    description
  }
}
```

> See the results [here](#setRestaurant)

<br/>

**_<span style="color: skyblue">mutation deleteRestaurant</span>_**

```
mutation deleteRestaurants($iid: Int = 1) {
  deleterestaurant(id: $iid) {
    exists
  }
}
```

> See the results [here](#deleteRestaurant)

<br/>

**_<span style="color: skyblue">mutation editRestaurant</span>_**

```
mutation editRestaurants ($iid: Int = 1, $name: String = "New Restaurant Name") {
  editrestaurant(id: $iid, name: $name) {
    id
    name
  }
}
```

> See the results [here](#editRestaurant)

<br/>

### Now we have full control over the data. To review, here are the things we covered:

- Setup project directory and install dependencies
- Define **_root_**
- Learn how to manipulate data using <span style="font-size: 18px; font-weight: bold">C</span>reate <span style="font-size: 18px; font-weight: bold">R</span>ead <span style="font-size: 18px; font-weight: bold">U</span>pdate <span style="font-size: 18px; font-weight: bold">D</span>elete:
  - Retrieving a specific restaurant (Read)
  - Retrieving all restaurants (Read)
  - Creating and submitting a new restaurant (Create)
  - Deleting a restaurant by id (Delete)
  - Editing a restaurant's name and description (Update)

<br/>
<br/>

> Link to [Github]().

<hr/>

<br/>
<br/>
<br/>
<br/>
<br/>

## **_<span style="color: yellowgreen">Query Results</span>_**

<hr/>
<br/>

<span id="getRestaurant" style="color: skyblue">query getRestaurant</span>

```json
{
  "data": {
    "restaurant": {
      "id": 3,
      "name": "Karma",
      "description": "Malaysian-Chinese-Japanese fusion, with great bar and bartenders",
      "dishes": [
        {
          "name": "Dragon Roll",
          "price": 12
        },
        {
          "name": "Pancake roll ",
          "price": 11
        },
        {
          "name": "Cod cakes",
          "price": 13
        }
      ]
    }
  }
}
```

<span>[Go Back](#top)</span>

<hr/>
<br/>

<span id="getRestaurants" style="color: skyblue">query getRestaurants</span>

```json
{
  "data": {
    "restaurants": [
      {
        "id": 1,
        "name": "WoodsHill ",
        "description": "American cuisine, farm to table, with fresh produce every day",
        "dishes": [
          {
            "name": "Swordfish grill",
            "price": 27
          },
          {
            "name": "Roasted Broccily ",
            "price": 11
          }
        ]
      },
      {
        "id": 2,
        "name": "Fiorellas",
        "description": "Italian-American home cooked food with fresh pasta and sauces",
        "dishes": [
          {
            "name": "Flatbread",
            "price": 14
          },
          {
            "name": "Carbonara",
            "price": 18
          },
          {
            "name": "Spaghetti",
            "price": 19
          }
        ]
      },
      {
        "id": 3,
        "name": "Karma",
        "description": "Malaysian-Chinese-Japanese fusion, with great bar and bartenders",
        "dishes": [
          {
            "name": "Dragon Roll",
            "price": 12
          },
          {
            "name": "Pancake roll ",
            "price": 11
          },
          {
            "name": "Cod cakes",
            "price": 13
          }
        ]
      }
    ]
  }
}
```

<span>[Go Back](#top)</span>

<hr/>
<br/>

<span id="setRestaurant" style="color: skyblue">mutation setRestaurant</span>

```json
{
  "data": {
    "setrestaurant": {
      "id": null,
      "name": "New Restaurant",
      "description": "Opening Soon"
    }
  }
}
```

<span>[Go Back](#top)</span>

<hr/>
<br/>

<span id="deleteRestaurant" style="color: skyblue">mutation deleteRestaurant</span>

```json
{
  "data": {
    "deleterestaurant": {
      "exists": true
    }
  }
}
```

<span>[Go Back](#top)</span>

<hr/>
<br/>

<span id="editRestaurant" style="color: skyblue">mutation editRestaurant</span>

```json
{
  "data": {
    "editrestaurant": {
      "id": null,
      "name": "New Restaurant Name"
    }
  }
}
```

<span>[Go Back](#top)</span>
