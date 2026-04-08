## Q1

B) `app.use(express.json())` middleware is missing or registered after the route

## Q2

A 400 Bad Request means the client sent invalid or incomplete data. The server understands the request, but the input is wrong. In my server, this happens in routes like `POST /api/auth/register` if the user does not provide an email or password, or if the password is too short.

A 401 Unauthorized means the request requires authentication, but the user is either not logged in or provided invaloid credentials. In my server, this occurs in `POST /api/auth/login` when the email or password is incorrect, or in protected routes like `POST /api/scores` if the JWT is missing or invalid.

A 404 Not Found means the requested resource or route does not exist. For example if a user tries to access a route like `/api/user/123` that does not exist in my server, it would be appropriate to return a 404 error.

## Q3

The issue is that `Score.find()` is asynchronous, but the student is not waiting for it to finish. The query is started, but `res.json({ message: 'done' })` runs immediately before the database returns any results. As a result, no scores are ever sent back to the client. You need to wait for the databse query using `async/await` (or `.then()`), then send the results. 
```JS
app.get('/api/scores', async (req, res) => {
  try {
    const scores = await Score.find()
      .sort({ score: -1 })
      .limit(10)

    res.json(scores)
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch scores' })
  }
})
```

## Q4

B) A schema defines the shape and validation rules for documents; a model is the class you use to query and save documents based on that schema

## Q5

One advantage of storing the JWT in a cookie is that it can be automatically sent with every request by the browser, which simplifies the client-side code since you don't need to manually attach the token each time. This can make authentication feel more seamless in traditional web apps. 

One advantage of using the Authorization header is that it gives more explicit control over when and how the token is sent. it also works consistently across different clients like mobile apps or APIs and avoids some security concerns like CSRF that can affect cookies.

For a mobile-accessible game like QuizBlitz, the Authorization header approach is more appropriate. It is more flexible across different plaforms (browser, mobile, etc.) and fits well with a frontend like Vue that manually manages requests. It also keeps authentication logic clearer and more cnotrolled in the client code.