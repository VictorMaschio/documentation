# <center> Express </center> 

## Créer la base de l'application
1. Créer le dossier `src`.
2. Créer les ous-dossiers `controllers`, `middlewares` et `routers`.
3. Créer dans `src` les fichiers :

- `main.ts`
```ts
import { startServer } from './app';

startServer();
```

- `app.ts`
```ts
import express from 'express';
import compression from 'compression';
import { router } from './routers/router';
import { createServer, Server } from 'http';

/**
 * Initializes and starts the Express server, including WebSocket handling and database synchronization.
 *
 * This function performs the following tasks:
 * - Creates an Express application
 * - Sets up API and static file routes
 * - Starts the HTTP server on the specified port
 *
 * @category Server
 * @returns {Promise<{ server: Server}>} Resolves with the HTTP server and Sequelize instance.
 *
 * @throws {Error} If an error occurs during server startup.
 *
 * @example
 * startServer()
 *   .then(({ server }) => console.log('Server is running'))
 *   .catch(error => console.error('Failed to start server:', error));
 *
 * @see {@link router} for API routing logic.
 */
export async function startServer(
): Promise<{ server: Server }> {
    try {
        const PORT = 8081;

        const app = express();
        app.disable('x-powered-by');
        app.use(compression());

        app.use(express.json());

        app.use('/api', router());
        
        const server = createServer(app);
        server.listen(PORT, () => {
            console.log(`Server running at http://localhost:${PORT}`);
        });
        return { server };
    } catch (error) {
        console.error('Error starting server:', error);
        throw error;
    }
}
```

4. Créer les routers suivants :
- `router.ts`
```ts
import express from 'express';
import { authenticationRouter } from './authenticationRouter';
import { userRouter } from './userRouter';

/**
 * Combined Application Router.
 *
 * Binds all feature routers (auth, user) into a single router.
 *
 * @returns {express.Router} Configured Express router.
 *
 * @example
 * const app = express();
 * app.use('/api', router());
 */
export const router = (): express.Router => {
    const router = express.Router();

    authenticationRouter(router);
    userRouter(router);

    return router;
}
```

- `authenticationRouter.ts`
```ts
import express from 'express';
import { registerController, loginController } from '../controllers/authenticationController';

/**
 * Auth types/Router.
 *
 * Defines routes for authentication: register and login.
 *
 * @param {express.Router} router - Express router instance to bind the routes.
 * @returns {express.Router} Configured router with auth endpoints.
 *
 * @example
 * router.post('/auth/register', registerController);
 * router.post('/auth/login', loginController);
 */
export const authenticationRouter = (router: express.Router) => {
    router.post('/auth/register', registerController);
    router.post('/auth/login', loginController);
    return router;
}
```

- `userRouter.ts`
```ts
import express from 'express';
import { authenticate } from '../middlewares/authenticationMiddleware';
import { getUserController } from '../controllers/userController';

/**
 * User Router.
 *
 * Defines routes for user-related endpoints.
 *
 * @param {express.Router} router - Express router instance to bind the routes.
 * @returns {express.Router} Configured router with user endpoints.
 *
 * @example
 * router.get('/user/me', authenticate, getUserController);
 */
export const userRouter = (router: express.Router) => {
    router.get('/user/me', authenticate, getUserController);
    return router;
}
```

5. Créer dans `middlewares` le fichier `authenticationMiddleware.ts` :
```ts
import { Request, Response, NextFunction } from 'express';

declare module 'express-serve-static-core' {
  interface Request {
    user?: {
      id: number;
      username: string;
    };
  }
}

/**
 * Authentication Middleware.
 *
 * Checks for a valid token in the `Authorization` header.
 *
 * @param {Request} req - Incoming request.
 * @param {Response} res - Response object.
 * @param {NextFunction} next - Callback to pass control to next middleware.
 *
 * @returns {void} Either passes control or sends 401 Unauthorized.
 *
 * @example
 * Authorization: Bearer fake-jwt-token
 */
export const authenticate = (req: Request, res: Response, next: NextFunction) => {
    const token = req.headers.authorization;

    if (token === 'Bearer fake-jwt-token') {
        req.user = { id: 1, username: 'testuser' };
        next();
    } else {
        return res.status(401).json({ message: 'Unauthorized' });
    }
};
```


6. Créer dans `controllers` les fichier suivants :
- `authenticationController.ts`
```ts
import { Request, Response } from 'express';

/**
 * Controller to handle user registration.
 *
 * @param {Request} req - The request object containing `username` and `password`.
 * @param {Response} res - The response object used to return the result.
 *
 * @returns {Promise<void>} Responds with user info or an error message.
 *
 * @example
 * // POST /api/auth/register
 * {
 *   "username": "john",
 *   "password": "secret"
 * }
 */
export const registerController = async (req: Request, res: Response) => {
    const { username, password } = req.body;

    if (!username || !password) {
        return res.status(400).json({ message: 'Username and password required' });
    }

    const user = { id: 1, username };
    return res.status(201).json({ message: 'User registered', user });
};

/**
 * Controller to handle user login.
 *
 * @param {Request} req - The request object containing `username` and `password`.
 * @param {Response} res - The response object used to return a token or an error.
 *
 * @returns {Promise<void>} Responds with a token or an error.
 *
 * @example
 * // POST /api/auth/login
 * {
 *   "username": "john",
 *   "password": "secret"
 * }
 */
export const loginController = async (req: Request, res: Response) => {
    const { username, password } = req.body;

    if (username === 'test' && password === 'password') {
        return res.status(200).json({ token: 'fake-jwt-token' });
    }

    return res.status(401).json({ message: 'Invalid credentials' });
};
```

- `userController.ts`
```ts
import { Request, Response } from 'express';

/**
 * Controller to fetch current user info.
 *
 * @param {Request} req - The authenticated request object.
 * @param {Response} res - The response object returning user data.
 *
 * @returns {void} Sends a JSON object with user info.
 *
 * @example
 * // GET /api/user/me
 * Authorization: Bearer fake-jwt-token
 */
export const getUserController = (req: Request, res: Response) => {
    const user = { id: 1, username: 'testuser', email: 'test@example.com' };
    res.status(200).json({ user });
};
```

## Implementer les tests

Créer dans `src` le dossiers `__tests__` et les sous-dossiers `units/controllers` et `e2e`.
