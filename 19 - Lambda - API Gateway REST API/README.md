# Lab 19 — Lambda: API Gateway REST API

![Difficulty](https://img.shields.io/badge/Difficulty-Medium-orange)
![Time](https://img.shields.io/badge/Time-~40min-blue)
![Cost](https://img.shields.io/badge/Cost-<_%241-yellow)
![Service](https://img.shields.io/badge/Service-Lambda%20%2F%20API%20Gateway-blue)

> "API Gateway is the front door of your building — it checks IDs, routes visitors to the right room, and kicks out anyone who shouldn't be there." — Rithu

---

## 🎯 Objective

In this lab, you'll create a **REST API** using **Amazon API Gateway** that invokes a **Lambda function** for every HTTP request. You'll build a simple student management API with GET and POST endpoints — the foundation of modern serverless web applications.

**By the end of this lab you will be able to:**
- Create a Lambda function that handles HTTP requests
- Create and configure a REST API in API Gateway
- Create API resources (`/hello`, `/students`) and methods (GET, POST)
- Connect API Gateway to Lambda using Lambda Proxy Integration
- Deploy the API and test it with a browser, curl, or Postman
- Understand the request/response flow between client → API Gateway → Lambda

---

## 🧠 Prerequisites

- [ ] Completed [Lab 18 — Lambda: S3 Triggered Function](../18%20-%20Lambda%20-%20S3%20Triggered%20Function/README.md)
- [ ] AWS account with root or admin access
- [ ] A tool for testing HTTP requests: browser, `curl` (command line), or [Postman](https://www.postman.com/downloads/)
- [ ] Basic understanding of HTTP methods (GET, POST) and JSON

> 💡 **Rithu's Tip:** If you've never used curl or Postman before, don't worry — we'll cover the basics. A browser address bar is actually a GET request, so you can test GET endpoints just by visiting a URL!

---

## 💰 Cost Warning

**This lab costs less than $1!** 💸

API Gateway Free Tier (12 months):
- **1 million** REST API calls per month
- **750,000** HTTP API calls per month

Lambda Free Tier (always free):
- **1 million requests** per month
- **400,000 GB-seconds** of compute time

⚠️ **IMPORTANT:** After the lab, delete the API Gateway (API, stage, deployment), the Lambda function, and the IAM role. An exposed API endpoint can be exploited, and forgotten APIs can incur charges.

---

## 🏗️ Architecture

```
    Client (Browser/curl/Postman)
              │
              │  HTTPS Request
              ▼
    ┌─────────────────────┐
    │   API Gateway        │
    │   ravi-student-api   │
    │                      │
    │   /hello  → GET     │
    │   /students → GET   │
    │   /students → POST  │
    └──────────┬──────────┘
               │
               │  Lambda Proxy Integration
               ▼
    ┌─────────────────────┐
    │   Lambda Function    │
    │   ravi-rest-api      │
    │                      │
    │  Returns JSON based  │
    │  on path + method    │
    └─────────────────────┘
```

**Request Flow:**
```
1. Client sends: GET https://xxxxx.execute-api.us-east-1.amazonaws.com/prod/hello
2. API Gateway receives the request
3. API Gateway matches route: /hello + GET
4. API Gateway invokes Lambda with a JSON event payload
5. Lambda processes the request and returns a JSON response
6. API Gateway forwards the response back to the client
```

---

## 🛠️ Step-by-Step Instructions

### Step 1: Create Lambda Function

1. Sign in to the **AWS Management Console**.
2. In the search bar, type **Lambda** and click on **AWS Lambda**.
3. Click **Create function**.
4. **Author from scratch** (default).
5. **Function name:** `ravi-rest-api`
6. **Runtime:** Select **Python 3.12**.
7. **Architecture:** `x86_64`
8. **Execution role:**
   - Select: ⚫ **Create a new role with basic Lambda permissions**
   - **Role name:** `api-lambda-role`
9. Click **Create function**.

#### Write the Lambda Code:

1. Scroll down to **Code source** → `lambda_function.py`.
2. **Replace ALL the code** with:

```python
import json

def lambda_handler(event, context):
    http_method = event.get('httpMethod', 'GET')
    path = event.get('path', '/')

    if path == '/hello' and http_method == 'GET':
        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({
                'message': 'Hello from Ravi\'s API!',
                'method': http_method,
                'path': path
            })
        }
    elif path == '/students' and http_method == 'GET':
        students = [
            {'id': 'S001', 'name': 'Ravi', 'topic': 'Lambda'},
            {'id': 'S002', 'name': 'Rithu', 'topic': 'API Gateway'}
        ]
        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({'students': students})
        }
    elif path == '/students' and http_method == 'POST':
        body = json.loads(event.get('body', '{}'))
        return {
            'statusCode': 201,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps({
                'message': 'Student created!',
                'data': body
            })
        }
    else:
        return {
            'statusCode': 404,
            'body': json.dumps({'message': 'Not found'})
        }
```

3. Click **Deploy**.

> 📸 [Screenshot: The Lambda code editor with the REST API code]

**What does this code do?**

| Path | Method | Response |
|------|--------|----------|
| `/hello` | GET | Returns a greeting message with method and path info |
| `/students` | GET | Returns a list of students as JSON |
| `/students` | POST | Accepts a JSON body and returns confirmation |
| Any other path | Any method | Returns 404 Not Found |

> 💡 **Rithu's Tip:** Notice how Lambda receives the `event` parameter — when API Gateway invokes Lambda, it passes the entire HTTP request as a JSON object, including the method, path, headers, query parameters, and body. Your Lambda then routes the request based on these values. It's like a doorman checking your reservation before seating you!

---

### Step 2: Create API Gateway

1. In the search bar, type **API Gateway** and click on **API Gateway**.
2. You'll see different API types. Under **REST API**, click **Build**.
   - ⚠️ Make sure you select **REST API** (the one with the yellow/orange icon), NOT HTTP API. They're different!
3. **Choose an API type:** ⚫ REST
4. **Protocol:** REST
5. **Create new API:** ⚫ New API
6. **API name:** `ravi-student-api`
7. **Description:** `Student management REST API for Ravi's lab`
8. **Endpoint Type:** Regional
9. Click **Create API**.

> 📸 [Screenshot: The API Gateway creation form showing REST API type selected with the name ravi-student-api]

> 💡 **Rithu's Tip:** API Gateway offers three API types:
> - **REST API** — Full-featured, supports request validation, caching, API keys, etc. (This is what we're using.)
> - **HTTP API** — Simpler, cheaper, faster. Good for basic proxying to Lambda.
> - **WebSocket API** — For real-time bidirectional communication.
>
> For learning and production, REST API is the most commonly used.

---

### Step 3: Create Resources and Methods

#### Create /hello Resource:

1. In your new API's **Resources** tree, click on the root `/`.
2. Click **Actions → Create Resource**.
3. **Resource Name:** `hello`
4. **Resource Path:** `/hello` (should auto-fill)
5. Check: ☑ **Enable API Gateway CORS** (we'll need this for browser access)
6. Click **Create Resource**.

#### Create GET Method on /hello:

1. Click on the newly created `/hello` resource.
2. Click **Actions → Create Method**.
3. Select **GET** from the dropdown → Click the checkmark ✅.
4. **Integration type:** ⚫ Lambda Function
5. Check: ☑ **Use Lambda Proxy integration**
   - ⚠️ This is CRITICAL! Without this checkbox, Lambda won't receive the HTTP request details properly.
6. **Lambda Function:** Type `ravi-rest-api` (it should auto-suggest)
7. A warning appears: "You are about to give API Gateway permission to invoke your Lambda function." → Click **OK**.
8. Click **Save**.

> 📸 [Screenshot: The method configuration showing Lambda Proxy Integration enabled]

> 💡 **Rithu's Tip:** What's "Lambda Proxy Integration"? It means API Gateway passes the ENTIRE HTTP request to Lambda as-is (method, path, headers, body, everything). Without it, you'd have to manually map request/response templates — and that's a headache. Always use Lambda Proxy Integration unless you have a specific reason not to.

---

### Step 4: Create /students Resource

1. Click on the root `/` resource.
2. Click **Actions → Create Resource**.
3. **Resource Name:** `students`
4. **Resource Path:** `/students`
5. Check: ☑ **Enable API Gateway CORS**
6. Click **Create Resource**.

#### Create GET Method on /students:

1. Click on `/students` → **Actions → Create Method**.
2. Select **GET** → Checkmark ✅.
3. **Integration type:** ⚫ Lambda Function
4. Check: ☑ **Use Lambda Proxy integration**
5. **Lambda Function:** `ravi-rest-api`
6. Click **Save** → **OK** on the permission warning.

#### Create POST Method on /students:

1. Click on `/students` → **Actions → Create Method**.
2. Select **POST** → Checkmark ✅.
3. **Integration type:** ⚫ Lambda Function
4. Check: ☑ **Use Lambda Proxy integration**
5. **Lambda Function:** `ravi-rest-api`
6. Click **Save** → **OK** on the permission warning.

> 📸 [Screenshot: The Resources tree showing /hello (GET) and /students (GET, POST)]

**Your final API structure should look like:**
```
/
├── /hello
│   └── GET
└── /students
    ├── GET
    └── POST
```

---

### Step 5: Deploy API

Your API is configured but it's not live yet. You need to **deploy** it!

1. In the API Gateway console, click **Actions → Deploy API**.
2. **Deployment stage:** ⚫ [New Stage]
3. **Stage name:** `prod`
4. **Stage description:** `Production stage`
5. **Deployment description:** `Initial deployment`
6. Click **Deploy**.

> 📸 [Screenshot: The Deploy API dialog with prod stage selected]

After deployment, you'll be taken to the **Stage Editor**. At the top of the page, you'll see the **Invoke URL** — this is your live API endpoint!

```
Invoke URL: https://abc123def4.execute-api.us-east-1.amazonaws.com/prod
```

> ⚠️ **WRITE DOWN THIS URL!** You'll need it for testing. Copy it somewhere safe.

> 💡 **Rithu's Tip:** API Gateway uses stages to manage different environments. Common stages are:
> - `dev` — Development/testing
> - `staging` — Pre-production testing
> - `prod` — Live production
>
> Each stage has its own URL and can have different settings (throttling, caching, logging, etc.).

---

### Step 6: Test the API

Now let's test all three endpoints!

#### Test 1: GET /hello

**Using a browser:**
```
https://abc123def4.execute-api.us-east-1.amazonaws.com/prod/hello
```

**Using curl:**
```bash
curl https://abc123def4.execute-api.us-east-1.amazonaws.com/prod/hello
```

**Expected response:**
```json
{
  "message": "Hello from Ravi's API!",
  "method": "GET",
  "path": "/hello"
}
```

> 📸 [Screenshot: Browser showing the JSON response from the /hello endpoint]

#### Test 2: GET /students

**Using a browser:**
```
https://abc123def4.execute-api.us-east-1.amazonaws.com/prod/students
```

**Using curl:**
```bash
curl https://abc123def4.execute-api.us-east-1.amazonaws.com/prod/students
```

**Expected response:**
```json
{
  "students": [
    {"id": "S001", "name": "Ravi", "topic": "Lambda"},
    {"id": "S002", "name": "Rithu", "topic": "API Gateway"}
  ]
}
```

#### Test 3: POST /students

**Using curl:**
```bash
curl -X POST \
  https://abc123def4.execute-api.us-east-1.amazonaws.com/prod/students \
  -H "Content-Type: application/json" \
  -d '{"name": "Ravi", "topic": "API Gateway"}'
```

**Using Postman:**
1. Method: `POST`
2. URL: `https://abc123def4.execute-api.us-east-1.amazonaws.com/prod/students`
3. Headers: `Content-Type: application/json`
4. Body (raw JSON):
```json
{
  "name": "Ravi",
  "topic": "API Gateway"
}
```
5. Click **Send**.

**Expected response:**
```json
{
  "message": "Student created!",
  "data": {
    "name": "Ravi",
    "topic": "API Gateway"
  }
}
```

#### Test 4: Unknown Path (404)

**Using curl:**
```bash
curl https://abc123def4.execute-api.us-east-1.amazonaws.com/prod/anything
```

**Expected response:**
```json
{
  "message": "Not found"
}
```

> 📸 [Screenshot: Terminal showing all four curl commands and their responses]

🎉 **Your API is live and working!** Anyone in the world with this URL can now call your API!

> 💡 **Rithu's Tip:** In production, you would NEVER expose an API without authentication! You'd add:
> - **API Keys** — Simple key-based auth
> - **IAM Authorization** — AWS credentials
> - **Cognito Authorizer** — User pool-based JWT tokens
> - **Lambda Authorizer** — Custom auth logic
>
> We're keeping it open for learning purposes, but in production, always lock it down!

---

### Step 7: Enable CORS (if needed)

If you're getting CORS errors when testing from a web browser or a frontend app:

1. In API Gateway, click on the `/hello` resource.
2. Click **Actions → Enable CORS**.
3. **Access-Control-Allow-Origin:** `*` (or your specific domain)
4. **Access-Control-Allow-Headers:** `Content-Type,Authorization`
5. **Access-Control-Allow-Methods:** `GET,POST,OPTIONS`
6. Click **Enable CORS** → **Replace Access-Control-Allow-Origin header**.
7. **Redeploy the API:**
   - Click **Actions → Deploy API** → Select `prod` → **Deploy**.

Repeat the same for the `/students` resource.

> 💡 **Rithu's Tip:** CORS (Cross-Origin Resource Sharing) is a browser security feature. It prevents a web page on `example.com` from making API calls to `api.another.com` unless the API explicitly allows it. For curl and Postman, CORS doesn't apply — it's a browser-only restriction.

---

### Step 8: Verify Your Work ✅

1. **Lambda function exists:** Lambda → Functions → `ravi-rest-api`.
2. **API Gateway exists:** API Gateway → APIs → `ravi-student-api`.
3. **API has correct structure:** `/hello` (GET), `/students` (GET, POST).
4. **API is deployed:** Stage `prod` with an Invoke URL.
5. **GET /hello returns JSON:** Browser or curl shows the greeting.
6. **GET /students returns student list:** Browser or curl shows two students.
7. **POST /students returns 201:** curl with JSON body returns confirmation.

---

## ✅ Validation Checklist

| # | Check | Status |
|---|-------|--------|
| 1 | Lambda function `ravi-rest-api` created with correct code | ☐ |
| 2 | API Gateway `ravi-student-api` created (REST API) | ☐ |
| 3 | `/hello` resource with GET method exists | ☐ |
| 4 | `/students` resource with GET and POST methods exists | ☐ |
| 5 | All methods use Lambda Proxy Integration | ☐ |
| 6 | API deployed to `prod` stage | ☐ |
| 7 | GET /hello returns `{"message": "Hello from Ravi's API!"}` | ☐ |
| 8 | GET /students returns student list | ☐ |
| 9 | POST /students returns `{"message": "Student created!"}` with 201 | ☐ |
| 10 | Unknown paths return 404 | ☐ |

---

## 🧹 Cleanup (IMPORTANT!)

**Delete all resources to avoid charges and security risks!**

### Delete API Gateway:
1. Go to **API Gateway → APIs → ravi-student-api**.
2. Click **Actions → Delete**.
3. Confirm by typing `ravi-student-api` → **Delete API**.
   - This automatically deletes the stage, deployment, resources, and methods.

### Delete Lambda Function:
1. Go to **Lambda → Functions → ravi-rest-api**.
2. Click **Actions → Delete**.
3. Type `delete` → **Delete**.

### Delete IAM Role:
1. Go to **IAM → Roles**.
2. Search for `api-lambda-role`.
3. Click on it → **Delete** → Type the role name → **Delete role**.

> 💡 **Rithu's Tip:** Deleting the API first is important — if you delete Lambda first, the API will return 500 errors until the Lambda is recreated. Not dangerous, but messy. Clean up top-down: API Gateway → Lambda → IAM.

---

## 🎓 What You Learned

In this lab, you learned:

1. **Amazon API Gateway** — Creating and managing REST APIs in AWS.
2. **REST API Concepts** — Resources, methods (GET, POST), stages, and deployments.
3. **Lambda Proxy Integration** — Passing the full HTTP request to Lambda and returning HTTP responses.
4. **API Stages** — Managing different environments (dev, staging, prod) with separate URLs.
5. **CORS** — Understanding Cross-Origin Resource Sharing and how to enable it.
6. **Request Routing** — How API Gateway routes requests to Lambda based on path and method.
7. **HTTP Status Codes** — 200 (OK), 201 (Created), 404 (Not Found).

---

## 🔗 What's Next?

You've built APIs, processed files, secured access with IAM, and sent messages with SNS/SQS. Now let's put it all together with **containerized workloads on ECS Fargate** — the future of cloud computing!

👉 **[Lab 20 — ECS: Deploy NGINX on Fargate](../20%20-%20ECS%20-%20Deploy%20NGINX%20on%20Fargate/README.md)**

In the next lab, you'll deploy a containerized NGINX web server on AWS ECS with Fargate — no servers to manage!

---

## ❓ Troubleshooting

### 502 Bad Gateway error
**Cause:** Lambda function returned an invalid response (not valid JSON, or missing `statusCode`).
**Fix:** Check Lambda → Code → Make sure `lambda_handler` returns a dictionary with `statusCode` and `body` keys. Also check CloudWatch Logs for errors.

### 500 Internal Server Error
**Cause:** Lambda function crashed (runtime error, syntax error, import error).
**Fix:** Go to Lambda → Monitor → Logs → Check the most recent log stream for the error message.

### CORS error in browser ("No 'Access-Control-Allow-Origin' header")
**Cause:** CORS is not enabled on the API Gateway resources.
**Fix:** Enable CORS on each resource (Step 7), then REDEPLOY the API to the `prod` stage. Don't forget to redeploy!

### "Missing Authentication Token" error
**Cause:** The method doesn't exist at that path, or the API isn't deployed.
**Fix:** Make sure the method exists in API Gateway (check the Resources tree). Deploy the API again after making changes.

### POST request returns "Unsupported Media Type"
**Cause:** The `Content-Type` header is missing or wrong.
**Fix:** Make sure you're sending `Content-Type: application/json` in your request headers.

### POST body is empty in Lambda
**Cause:** The body isn't being parsed correctly.
**Fix:** Make sure Lambda Proxy Integration is enabled. The body comes as a string in `event['body']`, so you need to parse it with `json.loads(event['body'])`.

### API returns "Endpoint request timed out"
**Cause:** Lambda function took more than 29 seconds (API Gateway timeout).
**Fix:** Optimize your Lambda code, or increase the API Gateway timeout (up to 29 minutes for REST APIs).

> 💡 **Rithu's Tip:** The most common mistake with API Gateway + Lambda is forgetting to **redeploy the API after making changes**. API Gateway doesn't automatically deploy updates — you have to manually deploy each time. I've been bitten by this so many times, Ravi. So many times. 😅
