# edu-authentication - Basic Auth

## Sequence Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant C as Client
    participant S as Server

    U->>C: Enter Username & Password
    Note right of U: User provides<br/>credentials
    C->>S: Request with credentials
    Note right of C: Credentials are<br/>base64 encoded<br/>and sent in<br/>Authorization header
    S->>S: Validate credentials
    Note right of S: Server checks<br/>credentials against<br/>stored values
    alt Authentication Successful
        S->>C: 200 OK (with content)
        Note right of S: Server sends<br/>requested content
    else Authentication Failed
        S->>C: 401 Unauthorized
        Note right of S: Server requests<br/>authentication
    end
```

## Topology

```mermaid
graph TB
    A[Client] --> B[Express App]
    B --> C{Is Logged In?}
    C -->|Yes| D[Home - Logged In]
    C -->|No| E[Home - Not Logged In]
    B --> F[Login Page]
    B --> G{Valid Credentials?}
    G -->|Yes| H[Set Session & Redirect to /]
    G -->|No| I[Invalid Credentials]
    B --> J[Destroy Session & Redirect to /]

```


## Server.js

```js
const express = require('express');
const bodyParser = require('body-parser');
const session = require('express-session');

const app = express();
const PORT = 3000;

// Middleware
app.use(bodyParser.urlencoded({ extended: false }));
app.use(session({
    secret: 'your_secret_key',
    resave: false,
    saveUninitialized: true
}));

// Mock user
const user = {
    username: 'user',
    password: 'password'
};

// Routes
app.get('/', (req, res) => {
    const isLoggedIn = req.session.isLoggedIn;
    res.send(`
        <h1>Welcome</h1>
        <p>${isLoggedIn ? 'Logged in' : 'Not logged in'}</p>
        <a href="/login">Login</a> | <a href="/logout">Logout</a>
    `);
});

app.get('/login', (req, res) => {
    res.send(`
        <h1>Login</h1>
        <form method="post" action="/login">
            Username: <input type="text" name="username" required><br>
            Password: <input type="password" name="password" required><br>
            <input type="submit" value="Login">
        </form>
    `);
});

app.post('/login', (req, res) => {
    const { username, password } = req.body;
    if (username === user.username && password === user.password) {
        req.session.isLoggedIn = true;
        res.redirect('/');
    } else {
        res.send('Invalid credentials. <a href="/login">Try again</a>');
    }
});

app.get('/logout', (req, res) => {
    req.session.destroy(err => {
        if (err) {
            return res.redirect('/');
        }
        res.redirect('/');
    });
});

// Server
app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});
```
