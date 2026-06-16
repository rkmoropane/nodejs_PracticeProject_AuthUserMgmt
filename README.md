# coding-project-template

# 🚀 Practice Project: Friends List Application Using Express Server with JWT

## 📌 Overview

In this practice project, you will enhance a Friends List API by securing CRUD operations with **JWT (JSON Web Token)** and **Session Authentication** using **Express.js**.

Unlike the previous CRUD lab where data was publicly accessible, only authenticated users will be allowed to perform CRUD operations on the friends list.

The friends data is stored as a JavaScript object (dictionary) where:

* **Email** = Key
* **Friend Object** = Value

Each friend object contains:

* `firstName`
* `lastName`
* `DOB`

You will use data from the request **body** instead of URL parameters or query strings for creating and updating records.

---

# 🎯 Learning Objectives

By completing this project, you will learn how to:

* Register new users
* Authenticate users using JWT
* Manage sessions using Express Session
* Protect API endpoints with middleware
* Implement CRUD operations
* Test secured APIs using Postman

---

# 📂 Project Structure

```text
project/
│
├── index.js
├── router/
│   └── friends.js
│
└── package.json
```

---

# Exercise 1: Understanding User Authentication

## 1. Register New Users

Create a `/register` endpoint that:

* Accepts `username` and `password`
* Checks whether the user already exists
* Stores the new user if unique
* Returns appropriate success or error messages

### Registration Endpoint

```js
app.post('/register', (req, res) => {
const username = req.body.username;
const password = req.body.password;

if (username && password) {
    if (!doesExist(username)) {
        users.push({
            username,
            password
        });

        return res.status(200).json({
            message: 'User successfully registered. Now you can login'
        });
    }

    return res.status(404).json({
        message: 'User already exists!'
    });
}

return res.status(404).json({
    message: 'Unable to register user.'
});

});
```

---

## 2. Check if a User Already Exists

Create a utility function that verifies username uniqueness.

```js
const doesExist = (username) => {
let userswithsamename = users.filter((user) => {
return user.username === username;
});

return userswithsamename.length > 0;

}
```

---

## 3. Authenticate Existing Users

Create a utility function to verify username and password combinations.

```js
const authenticatedUser = (username, password) => {
let validusers = users.filter((user) => {
return (
user.username === username &&
user.password === password
);
});

return validusers.length > 0;

}
```

---

## 4. Configure Session Middleware

Use Express Session to store authentication information.

```js
app.use(
session({
secret: 'fingerpint',
resave: true,
saveUninitialized: true
})
);
```

---

## 5. Implement Login Endpoint

The login endpoint should:

* Validate username and password
* Generate a JWT token
* Store token in session
* Allow authenticated access for one hour

```js
app.post('/login', (req, res) => {

const username = req.body.username;
const password = req.body.password;

if (!username || !password) {
    return res.status(404).json({
        message: 'Error logging in'
    });
}

if (authenticatedUser(username, password)) {

    let accessToken = jwt.sign(
        {
            data: password
        },
        'access',
        {
            expiresIn: 60 * 60
        }
    );

    req.session.authorization = {
        accessToken,
        username
    };

    return res.status(200)
              .send('User successfully logged in');

} else {

    return res.status(208).json({
        message:
        'Invalid Login. Check username and password'
    });
}

});
```

---

## 6. Protect Routes Using Middleware

All `/friends` endpoints must pass through authentication middleware.

```js
app.use('/friends', function auth(req, res, next) {

if (req.session.authorization) {

    let token =
        req.session.authorization['accessToken'];

    jwt.verify(
        token,
        'access',
        (err, user) => {

            if (!err) {
                req.user = user;
                next();
            } else {
                return res.status(403).json({
                    message:
                    'User not authenticated'
                });
            }
        }
    );

} else {

    return res.status(403).json({
        message: 'User not logged in'
    });
}

});
```

---

# Exercise 2: Implement GET All Friends

Retrieve all friend records.

```js
router.get('/', (req, res) => {
res.send(
JSON.stringify(
friends,
null,
4
)
);
});
```

### Expected Result

Returns all friend data in JSON format.

---

# Exercise 3: Implement GET Friend by Email

Retrieve a single friend using email.

```js
router.get('/:email', (req, res) => {

const email = req.params.email;

res.send(
    friends[email]
);

});
```

### Expected Result

Returns friend details matching the email.

---

# Exercise 4: Implement POST Method

Add a new friend.

```js
router.post('/', (req, res) => {

if (req.body.email) {

    friends[req.body.email] = {

        firstName:
            req.body.firstName,

        lastName:
            req.body.lastName,

        DOB:
            req.body.DOB
    };
}

res.send(
    'The user ' +
    req.body.firstName +
    ' Has been added!'
);

});
```

### Expected Result

A new friend record is added to the friends object.

---

# Exercise 5: Implement PUT Method

Update an existing friend's information.

```js
router.put('/:email', (req, res) => {

const email = req.params.email;

let friend = friends[email];

if (friend) {

    let firstName =
        req.body.firstName;

    let lastName =
        req.body.lastName;

    let DOB =
        req.body.DOB;

    if (firstName) {
        friend.firstName = firstName;
    }

    if (lastName) {
        friend.lastName = lastName;
    }

    if (DOB) {
        friend.DOB = DOB;
    }

    friends[email] = friend;

    res.send(
        `Friend with the email ${email} updated.`
    );

} else {

    res.send(
        'Unable to find friend!'
    );
}

});
```

### Expected Result

Updates one or more fields:

* firstName
* lastName
* DOB

---

# Exercise 6: Implement DELETE Method

Delete a friend using email.

```js
router.delete('/:email', (req, res) => {


const email = req.params.email;

if (email) {
    delete friends[email];
}

res.send(
    `Friend with the email ${email} deleted.`
);


});
```

### Expected Result

Removes the friend from the object.

---

# Exercise 7: Test the API Using Postman

---

## Register User

### Endpoint

```text
POST /register
```

### Request Body

```json
{
'username': 'user2',
'password': 'password2'
}
```

### Expected Response

```json
{
'message':
'User successfully registered. Now you can login'
}
```

---

## Login User

### Endpoint

```text
POST /login
```

### Request Body

```json
{
'username': 'user2',
'password': 'password2'
}
```

### Expected Response

```text
User successfully logged in
```

---

## GET All Friends

### Endpoint

```text
GET /friends
```

Returns all friend records.

---

## GET Friend by Email

### Endpoint

```text
GET /friends/:email
```

Example:

```text
GET /friends/johnsmith@gamil.com
```

Returns information for the specified friend.

---

## Add New Friend

### Endpoint

```text
POST /friends
```

### Request Body

```json
{
'email': '[andysmith@gamil.com](mailto:andysmith@gamil.com)',
'firstName': 'Andy',
'lastName': 'Smith',
'DOB': '1/1/1987'
}
```

### Expected Result

Friend is added successfully.

---

## Update Friend

### Endpoint

```text
PUT /friends/:email
```

### Request Body Example

```json
{
'DOB': '1/1/1989'
}
```

### Expected Result

Friend information is updated.

---

## Delete Friend

### Endpoint

```text
DELETE /friends/:email
```

### Expected Result

Friend is deleted successfully.

---

# JWT Token Expiration Testing

Initially the JWT token is valid for **1 hour**.

```js
expiresIn: 60 * 60
```

Change the validity period to **60 seconds**.

```js
expiresIn: 60
```

### Testing

1. Register user
2. Login user
3. Access `/friends` within 60 seconds

Result:

* Access granted

Wait longer than 60 seconds and retry.

Result:

```json
{
'message': 'User not authenticated'
}
```

This confirms the JWT token has expired.

---

# 🏆 Project Completion Checklist

* [ ] Implement user registration
* [ ] Implement user login
* [ ] Configure JWT authentication
* [ ] Configure Express Session
* [ ] Protect `/friends` routes
* [ ] Implement GET all friends
* [ ] Implement GET friend by email
* [ ] Implement POST friend
* [ ] Implement PUT friend
* [ ] Implement DELETE friend
* [ ] Test all endpoints using Postman
* [ ] Verify JWT expiration functionality

---

# 📖 Summary

In this practice project, you built a secure Friends List API using:

* Express.js
* JWT Authentication
* Express Session
* CRUD Operations
* Middleware Protection
* Postman Testing

The application allows authenticated users to securely create, retrieve, update, and delete friend records while ensuring access control through JWT-based authentication.
