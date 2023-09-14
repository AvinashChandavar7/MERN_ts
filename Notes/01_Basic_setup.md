# 1. Initial StepUp

```npm
npm init -y
npm install -D typescript
npm install -D ts-node
npm install -D nodemon
npm install express http cors body-parser cookie-parser compression
npm install -D @types/express @types/cors @types/body-parser @types/cookie-parser @types/compression

npm install mongoose colors

npm i lodash
npm i -D  @types/lodash
```

```js
{
  "name": "ts_node_mongo_rest_api",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "nodemon",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "Avinash",
  "license": "ISC",
  "devDependencies": {
    "@types/body-parser": "^1.19.2",
    "@types/colors": "^1.2.1",
    "@types/compression": "^1.7.3",
    "@types/cookie-parser": "^1.4.4",
    "@types/cors": "^2.8.14",
    "@types/express": "^4.17.17",
    "@types/lodash": "^4.14.198",
    "nodemon": "^3.0.1",
    "ts-node": "^10.9.1",
    "typescript": "^5.2.2"
  },
  "dependencies": {
    "body-parser": "^1.20.2",
    "colors": "^1.4.0",
    "compression": "^1.7.4",
    "cookie-parser": "^1.4.6",
    "cors": "^2.8.5",
    "express": "^4.18.2",
    "http": "^0.0.1-security",
    "lodash": "^4.17.21",
    "mongoose": "^7.5.1"
  }
}

```

# 2. tsconfig.json

```ts
{
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "node",
    "baseUrl": "src",
    "outDir": "dist",
    "sourceMap": true,
    "noImplicitAny": true,
  },
  "include": ["src/**/*"],
}
```

# 3. nodemon.json

```ts
{
  "watch": ["src"],
  "ext": ".ts,.js",
  "exec": "ts-node ./src/index.ts"
}
```

# 4. src/index.ts

```ts
import express from 'express';
import cors from 'cors';
import http from 'http';
import mongoose from 'mongoose';
import bodyParser from 'body-parser';
import compression from 'compression';
import cookieParser from 'cookie-parser';

import router from 'router';


const app = express();

app.use(cors({ credentials: true, }));
app.use(compression());
app.use(cookieParser());
app.use(bodyParser.json());

const server = http.createServer(app);


const MONGO_URL = 

mongoose.Promise = Promise;
mongoose.connect(MONGO_URL);
mongoose.connection.on('error', (error: Error) => console.log(error));

const PORT = 8080;
server.listen(PORT, () => {
  console.log(`Server Running on port ${PORT}`);
})

app.use('/', router())
```

# 5. src/db/user.ts

```ts
import mongoose from "mongoose";

const UserSchema = new mongoose.Schema({
  username: { type: String, required: true },
  email: { type: String, required: true },
  authentication: {
    password: { type: String, required: true, select: false },
    salt: { type: String, select: false },
    sessionToken: { type: String, select: false },
  },
}, { timestamps: true }
);

export const UserModel = mongoose.model('User', UserSchema);

// Controller
export const getUsers = () => UserModel.find();
export const getUserById = (id: string) => UserModel.findById(id);
export const getUserByEmail = (email: string) => UserModel.findOne({ email });
export const getUserBySessionToken = (sessionToken: string) => UserModel.findOne({
  'authentication.sessionToken': sessionToken,
});

export const createUser = (values: Record<string, any>) => new UserModel(values).save().then((user) => user.toObject());
export const updateUserById = (id: string, values: Record<string, any>) => UserModel.findOneAndUpdate({ id, values });
export const deleteUserById = (id: string) => UserModel.findOneAndDelete({ _id: id });
```

# 6. src/helpers/index.ts

```ts
import crypto from 'crypto';

const SECRET = `ZORO-REST-API`

export const random = () => crypto.randomBytes(128).toString('base64');
export const authentication = (salt: string, password: string) => {
  return crypto.createHmac('sha256', [salt, password].join('/')).update(SECRET).digest('hex')
}
```

# 7. src/controllers/authentication.ts

```ts
import express from 'express';

import { getUserByEmail, createUser } from '../db/users';
import { authentication, random } from '../helpers';

export const register = async (req: express.Request, res: express.Response) => {
  try {
    const { email, password, username } = req.body;

    if (!email || !password || !username) {
      return res.sendStatus(400);
    }

    const existingUser = await getUserByEmail(email)

    if (existingUser) {
      return res.sendStatus(400);
    }

    const salt = random();

    const user = await createUser({
      email,
      username,
      authentication: {
        salt,
        password: authentication(salt, password)
      },
    })

    return res.status(200).json(user).end();

  } catch (error) {
    console.log(error);
    return res.sendStatus(400);
  }
}
```

# 8. src/router/index.ts

```ts
import express from 'express';

import authentication from './authentication';

const router = express.Router();

export default (): express.Router => {
  authentication(router)
  return router;
}
```

# 9. src/router/authentication.ts

```ts
import express from 'express';

import { register } from '../controllers/authentication';

export default (router: express.Router) => {
  router.post('/auth/register', register)
}
```

# 10. Testing in post (Register)

### 1. `Post : http://localhost:8080/auth/register`
### 2. Body -> raw -> json
```ts
  {
   "email":"zoro@codes.com",
   "password":"123",
   "username":"Zoro"
  }
```
### 3. if Successfully registered

```ts
{
    "username": "Zoro",
    "email": "zoro@codes.com",
    "authentication": {
        "password": "f0dfddbed81acdb91345baf2dceaf1f04b9e7ffd2bfd37ad0a19f6ff30015a8a",
        "salt": "QfU+7l49U8AQIJ1lZ5mK4OklTuP6ghchksqJoUnZe8QJIeaAIn/+YAiDpP7lFMHH/bwyqLHDMjEmELS0Y1wLKl9w8bKLSMuhbKmSuqI2iHaSC6ouO5RdpajpY7K4Vo4540V+L3ANswfK9S4F2R86MH87GuaGHrbC47K2kzkn/Aw="
    },
    "_id": "6501b0a59ccc6b629219572e",
    "__v": 0
} 
```

# 11. src/controllers/authentication.ts (login)

```ts
import express from 'express';

import { getUserByEmail, createUser } from '../db/users';
import { authentication, random } from '../helpers';

export const login = async (req: express.Request, res: express.Response) => {
  try {

    const { email, password } = req.body;

    if (!email || !password) {
      return res.sendStatus(400);
    }

    const user = await getUserByEmail(email).select('+authentication.salt +authentication.password');

    if (!user) {
      return res.sendStatus(400);
    }

    const expectedHash = authentication(user.authentication.salt, password);

    if (user.authentication.password !== expectedHash) {
      return res.sendStatus(403);
    }

    const salt = random();
    user.authentication.sessionToken = authentication(salt, user._id.toString());
    await user.save();

    res.cookie(
      'Zoro-Auth',
      user.authentication.sessionToken,
      { domain: 'localhost', path: '/', }
    )

    return res.status(200).json(user).end();

  } catch (error) {
    console.log(error);
    return res.sendStatus(400);
  }
}
```

# 12. src/router/authentication.ts

```ts
import express from 'express';

import { login, register } from '../controllers/authentication';

export default (router: express.Router) => {
  router.post('/auth/register', register)
  router.post('/auth/login', login)
}
```

# 13. Testing in post (Login)

### 1. `Post : http://localhost:8080/auth/login`
### 2. Body -> raw -> json
```ts
  {
   "email":"zoro@codes.com",
   "password":"123",
  }
```
### 3. if Successfully Login

```ts
{
    "authentication": {
        "password": "f0dfddbed81acdb91345baf2dceaf1f04b9e7ffd2bfd37ad0a19f6ff30015a8a",
        "salt": "QfU+7l49U8AQIJ1lZ5mK4OklTuP6ghchksqJoUnZe8QJIeaAIn/+YAiDpP7lFMHH/bwyqLHDMjEmELS0Y1wLKl9w8bKLSMuhbKmSuqI2iHaSC6ouO5RdpajpY7K4Vo4540V+L3ANswfK9S4F2R86MH87GuaGHrbC47K2kzkn/Aw=",
        "sessionToken": "b72a60fbc700308363170f36d707de166db754df3b78a6f78183ab1d2d972802"
    },
    "_id": "6501b0a59ccc6b629219572e",
    "username": "Zoro",
    "email": "zoro@codes.com",
    "__v": 0,
    "updatedAt": "2023-09-13T13:22:53.875Z"
}
```

#### 4. cookie is also created successfully
```ts
Zoro-Auth= 08b2a61f7a195eef574fffd2710fb5b3e3330c0a76ebaacaf0cdb3218f95031b; Path=/; Domain=localhost;
```

# 14. src/middleware/index.ts

```ts
import express from 'express';
import { get, merge } from 'lodash';

import { getUserBySessionToken } from '../db/users';

export const isAuthenticated = async (req: express.Request, res: express.Response, next: express.NextFunction) => {
  try {
    const sessionToken = req.cookies['Zoro-Auth'];

    if (!sessionToken) {
      return res.sendStatus(403);
    }

    const existingUser = await getUserBySessionToken(sessionToken);

    if (!existingUser) {
      return res.sendStatus(403);
    }

    merge(req, { identity: existingUser });

    return next();

  } catch (error) {
    console.log(error);
    return res.sendStatus(400);
  }
}
```

# 15. src/controllers/users.ts

```ts
import express from 'express';

import { getUsers } from '../db/users';

export const getAllUsers = async (req: express.Request, res: express.Response) => {
  try {
    const users = await getUsers();

    return res.status(200).json(users);

  } catch (error) {
    console.log(error);
    return res.sendStatus(400);
  }

}
```

# 16. src/router/users.ts
```ts
import express from 'express';

import { getAllUsers } from '../controllers/users';

export default (router: express.Router) => {
  router.get('/users', getAllUsers);
}
```

# 17. src/router/index.ts

```ts
import express from 'express';

import authentication from './authentication';
import users from './users';

const router = express.Router();

export default (): express.Router => {
  authentication(router);
  users(router);

  return router;
}
```

# 18. Testing in Postman in Getting All Users


### 1. `GET : http://localhost:8080/users`

### 2. getting all users 

```ts
[
    {
        "_id": "6501b0a59ccc6b629219572e",
        "username": "Zoro",
        "email": "zoro@codes.com",
        "__v": 0,
        "updatedAt": "2023-09-13T13:25:21.354Z"
    },
    {
        "_id": "6501bf6ca08a4daaaca75e32",
        "username": "luffy",
        "email": "luffy@codes.com",
        "createdAt": "2023-09-13T13:55:56.212Z",
        "updatedAt": "2023-09-13T13:55:56.212Z",
        "__v": 0
    }
]
```

# 19. src/router/index.js

```ts
import express from 'express';

import { getAllUsers } from '../controllers/users';
import { isAuthenticated } from '../middleware';

export default (router: express.Router) => {
  router.get('/users', isAuthenticated, getAllUsers);
}
```



# 20. Testing in Postman in Getting All Users with Middlware

### 1. `GET : http://localhost:8080/users`

### 2. getting all users 

```ts
'Forbidden'
```

### 3. if u login then 

```ts
[
    {
        "_id": "6501b0a59ccc6b629219572e",
        "username": "Zoro",
        "email": "zoro@codes.com",
        "__v": 0,
        "updatedAt": "2023-09-13T14:01:18.751Z"
    },
    {
        "_id": "6501bf6ca08a4daaaca75e32",
        "username": "luffy",
        "email": "luffy@codes.com",
        "createdAt": "2023-09-13T13:55:56.212Z",
        "updatedAt": "2023-09-13T13:55:56.212Z",
        "__v": 0
    }
]
```

# 21. src/controllers/users.ts

```ts
import express from 'express';

import { deleteUserById, getUsers } from '../db/users';

export const getAllUsers = async (req: express.Request, res: express.Response) => {
  try {
    const users = await getUsers();
    return res.status(200).json(users);
  } catch (error) {
    console.log(error);
    return res.sendStatus(400);
  }
}

export const deleteUsers = async (req: express.Request, res: express.Response) => {
  try {
    const { id } = req.params;

    const deleteUser = await deleteUserById(id);
    return res.json(deleteUser);

  } catch (error) {
    console.log(error);
    return res.sendStatus(400);
  }
}

export const updateUser = async (req: express.Request, res: express.Response) => {
  try {
    const { id } = req.params;
    const { username } = req.body;

    if (!username) {
      return res.sendStatus(400);
    }

    const user = await getUserById(id);
    user.username = username;
    await user.save();

    return res.status(200).json(user).end();
  } catch (error) {
    console.log(error);
    return res.sendStatus(400);
  }
}
```

# 22. src/router/users.ts

```ts
import express from 'express';

import { deleteUser, getAllUsers, updateUser } from '../controllers/users';
import { isAuthenticated, isOwner } from '../middleware';

export default (router: express.Router) => {
  router.get('/users', isAuthenticated, getAllUsers);
  router.delete('/users/:id', isAuthenticated, isOwner, deleteUser);
  router.delete('/users/:id', isAuthenticated, isOwner, updateUser);
}
```

# 23. src/middleware/index.ts

```ts
import express from 'express';
import { get, merge } from 'lodash';

import { getUserBySessionToken } from '../db/users';

export const isAuthenticated = async (req: express.Request, res: express.Response, next: express.NextFunction) => {
  try {
    const sessionToken = req.cookies['Zoro-Auth'];

    if (!sessionToken) {
      return res.sendStatus(403);
    }

    const existingUser = await getUserBySessionToken(sessionToken);

    if (!existingUser) {
      return res.sendStatus(403);
    }

    merge(req, { identity: existingUser });

    return next();

  } catch (error) {
    console.log(error);
    return res.sendStatus(400);
  }
}

export const isOwner = async (req: express.Request, res: express.Response, next: express.NextFunction) => {
  try {
    const { id } = req.params;
    const currentUserId = get(req, 'identity.id') as string;

    if (!currentUserId) {
      return res.sendStatus(403);
    }

    if (currentUserId.toString() !== id) {
      return res.sendStatus(403);
    }

    next();

  } catch (error) {
    console.log(error);
    return res.sendStatus(400);
  }
}
```

# 24. src/router/users.ts

```ts
import express from 'express';

import { deleteUser, getAllUsers } from '../controllers/users';
import { isAuthenticated, isOwner } from '../middleware';

export default (router: express.Router) => {
  router.get('/users', isAuthenticated, getAllUsers);
  router.delete('/users/:id',isAuthenticated, isOwner, deleteUser);
}
```