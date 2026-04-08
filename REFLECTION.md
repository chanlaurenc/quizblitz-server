## Q1 - Middleware

Middleware in Express is code that runs between receiving a request and sending a reponse. It can modify the request, check permissions, or prepare data before the route handler runs. In my server, I use `app.use(express.json())` to parse incoming JSON so I can access values like `req.body` in my routes, and `app.use(cors({ origin: allowedOrigins }))` to allow my frontend to communicate with the backend.

The order middleware is registered matters because Express runs it top to bottom. For example, `express.json()` must come before routes like `/api/auth/register` and `api/auth/login`, otherwise `req.body` would be undefined. Similarly, CORS needs to run before routes so requests from my frontend are allowed.

GLobal middleware, like `app.use(...)`, applies to all routes. In contrast, route-level middleware is only applied to specific routes. In my code, `verifyToken` is used as route-level middlware on `POST /api/scores`, so only authenticated users can submit scores, while other routes like fetching quesitons remain public.

## Q2 - Password Security

Passwords should never be stored in plain text because if the databse is ever leaked or accessed by an attacker, all user passwords would be immediately exposed. This is especially dangerous because many people reuse passwords across different sites. Storing passwords securely helps protext users even if the database is compromised.

In my server, I use `bcrypt.has(password, 10)` when creating a new account. Bcrypt hashes the password into a secure, irreversible string. The number 10 refers to the "salt rounds", which controls how many times the hashing algorithm runs. A higher number makes the hash more secure but also slower to compute. 

When logging in, I use `bcrypt.compare()`, which checks if a given password matches the stored hash. It does this by hashing the input password using the same salt and comparing the result. Since hashing is one-way, bcrypt never needs to reverse the hash, it just checks if the hashes match.

## Q3 - JWT FLow

From the user's perspective, the flow starts with registration. The client sends a `POST /api/auth/register` request with and email and password in the request body. The server first checks that both fields exist and that the password is at least 6 characters. It then checks whether a user with that email already exists. If not, it hashes the password with bcrypt and stores the new user in MongoDB. The server returns a success response with a message plus the user's id and email.

Next is logn. The client sends a `POST /api/auth/login` request with the same email and password. The server looks up the user by email, compares the submitted password with the sotred hashed password using `bcrypt.compare()`, and if they match it creates a JWT. In my code the token includes the user's userID and email, along with an expiration time from `JWT_EXPIRES_IN`. The server returns the token, plus the user id and email.

When submitting a score, the client sends a `POST /api/scores` request with the score data and includes the JWT, typically in the Authorization header. The `verifyToken` middleware checks that token before the route runs. If it is valid, the middlware decodes the payload and attaches the user information to `req.user`. Then the route uses `req.user.userId` and `req.user.email` when saving the score. The server returns the newly created score document.

The reason the server does not need to query the database just to verify the token is that JWTs are self-contained. The token already includes the user identity information the server needs, and it is signed using the server’s secret key. When the token comes back, the server can verify the signature and trust that the embedded data has not been changed. That means it can confirm who the user is without doing a database lookup on every protected request.

## Q4 - In-Memory vs Database

With the in-memory approach using a plain JavaScript array (`let scores = []`), one major problem is that the data is not persistent. Every time the server restarts, all scores are lost because they only exist in memory. This makes it impossible to maintain a real leaderboard over time. A second issue is scalability. If multiple users are accessing the app or if the server is redeployed, each instance would have its own separate array, leading to inconsistent data across users.

MongoDB solves both of these problems. First, it stores data persistently in a database, so scores remain even if the server restarts. Second, it provides a centralized data store that all users and server instances can access consistently. 

If I redeploy the server while using MongoDB, the data will still be there because it is stored externally in MongoDB Atlas, not inside the serve itself. This is different from the in-memory array, which would be completely reset on restart because it only exists temporarily while the server is running.

## Q5 - Route Design Decision

Having `GET /api/scores` public and `POST /api/scores` protected makes sense because the two routes do different jobs. Reading the leaderboard is a public feature of the game, so anyone should be able to view high scores without needing an account. In contrast, posting a score changes data in the database, so it should require authentication to make sure the score is tied to a real logged-in user. In my server, protecting `POST /api/scores` with `verifyToken` helps prevent anonymous users from submitting scores under any name they want. 

If `GET /api/scores` also required authentication, it owuld make the app less uable. New users or casual visitors would not be able to view the leaderboard unless they first registered and logged in, which adds friction to a feature that is usually meant to be easy to access. It would also complicate the frontend, since even simply displaying leaderboard data would require storing and sending a token. FOr a leaderboard app, that would be unnecessary and would make the public-facing experience worse.