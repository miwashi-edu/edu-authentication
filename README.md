# edu-authentication - Basic Auth

## Instructions

### Project

```bash
#Create directory for server
mkdir basic-auth && cd basic-auth

mkdir -p {src/routes,src/middleware}
touch ./src/service.js
touch ./src/server.js
touch ./src/routes/auth_routes.js
touch ./src/middleware/basic_auth.js
touch ./src/middleware/users.js

# Initialize a new Node.js project
npm init -y

# Install packages
npm install express
npm install basic-auth
npm install -D nodemon jest

# Set up scripts in package.json
npm pkg set main="./src/service.js"
npm pkg set scripts.start="node ./src/service.js"
npm pkg set scripts.dev="nodemon ./src/service.js"
npm pkg set scripts.test="jest"
```

## ./src/service.js
```bash
cat > src/service.js << 'EOF'
const app = require('./server');
const PORT = 3000;

// Server
app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});
EOF
```

## ./src/server.js
```bash
cat > src/server.js << 'EOF'
const express = require('express');
const authRoutes = require('./src/routes/auth_routes');

const app = express();

// Use auth routes
app.use(authRoutes);

module.exports = app;  // Exporting app to be used in service.js
EOF
```

## ./src/routes/auth_routes.js
```bash
cat > src/routes/auth_routes.js << 'EOF'
const express = require('express');
const basicAuthMiddleware = require('../middleware/basicAuthMiddleware');

const router = express.Router();

router.get('/open', (req, res) => {
    res.send('This is an open page, no authentication required!');
});

router.use('/protected', basicAuthMiddleware);

router.get('/protected', (req, res) => {
    res.send('This is a protected route, authentication required!');
});

router.get('/protected/another', (req, res) => {
    res.send('This is another protected route, authentication required!');
});

router.get('/safe',basicAuthMiddleware, (req, res) => {
    res.send('This is another protected route, authentication required!');
});

module.exports = router;
EOF
```

## ./src/middleware/basic_auth.js
```bash
cat > src/middleware/basic_auth.js << 'EOF'
const basicAuth = require('basic-auth');
const { users } = require('./users');  // Assume you have a users module

const basicAuthMiddleware = (req, res, next) => {
    const userCredentials = basicAuth(req);

    if (!userCredentials || !isValidUser(userCredentials.name, userCredentials.pass)) {
        res.setHeader('WWW-Authenticate', 'Basic realm="Example"');
        return res.status(401).send('Authentication required');
    }

    next();
};

const isValidUser = (username, password) => {
    const user = users.find(u => u.username === username);
    return user && user.password === password;
};

module.exports = basicAuthMiddleware;
EOF
```

## ./src/middleware/users.js
```bash
cat > src/middleware/users.js << 'EOF'
exports.users = [
    { username: 'user1', password: 'password1' },
    { username: 'user2', password: 'password2' }
];
EOF
```
