# Serverless CRUD API using AWS Lambda, API Gateway, DynamoDB

![AWS](https://img.shields.io/badge/AWS-Lambda-orange?logo=amazon-aws)
![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python)
![Status](https://img.shields.io/badge/Status-Live-brightgreen)


## ➤ Project Overview
This project is a **simple REST API** built using AWS serverless services.  
It allows you to **Create, Read, Update, and Delete (CRUD)** user data.

No servers are managed manually — everything runs on AWS.

---

## ➤ How It Works

1. User sends request (Postman)
2. API Gateway receives request
3. Lambda function runs Python code
4. DynamoDB stores/retrieves data
5. Response is sent back

---

## ➤ Architecture Diagram

![Architecture Diagram](./pictures/architecture.png)

---

## ➤ Technologies Used

- AWS Lambda (Python)
- API Gateway
- DynamoDB
- IAM (for permissions)
- Postman (for testing)

---

## ➤ Features

- ➕ Create user  
- 📖 Get all users  
- 🔍 Get single user  
- ✏️ Update user  
- ❌ Delete user  

---

## ➤ API Endpoints

| Method | Endpoint | Description |
|--------|---------|------------|
| POST | /users | Create user |
| GET | /users | Get all users |
| GET | /users/{id} | Get one user |
| PUT | /users/{id} | Update user |
| DELETE | /users/{id} | Delete user |

---

# ➤ Step-by-Step Setup (Beginner Friendly)

## 1️⃣ Create DynamoDB Table

- Go to AWS → DynamoDB  
- Click **Create Table**  
- Table name: `Users`  
- Partition key: `user_id` (String)  
- Click Create  

![DynamoDB](./pictures/DynamoDB.png)

---

## 2️⃣ Create IAM Role

- Go to IAM → Roles → Create Role  
- Select **Lambda**  
- Attach policy: `AmazonDynamoDBFullAccess`  
- Name: `LambdaDynamoDBRole`

![IAM](./pictures/IAM.png)

---

## 3️⃣ Create Lambda Function

- Go to AWS Lambda  
- Click **Create Function**  
- Name: `createUser`  
- Runtime: Python  
![Lambda-1](./pictures/Lambda-1.png)
- Select Custom execution role and IAM role created above 
![Lambda-2](./pictures/Lambda-2.png)

---

## 4️⃣ Add Lambda Code

Paste this code:

```python
import json
import boto3
import uuid

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

def lambda_handler(event, context):
    try:
        http_method = event['httpMethod']
        path_params = event.get('pathParameters')
        body = json.loads(event.get('body') or '{}')

        # CREATE USER
        if http_method == 'POST':
            user_id = str(uuid.uuid4())

            item = {
                'user_id': user_id,
                'name': body.get('name', ''),
                'email': body.get('email', '')
            }

            table.put_item(Item=item)

            return response(200, {'message': 'User created', 'user_id': user_id})

        # GET ALL USERS
        elif http_method == 'GET' and not path_params:
            result = table.scan()
            return response(200, result.get('Items', []))

        # GET SINGLE USER
        elif http_method == 'GET' and path_params:
            user_id = path_params['id']
            result = table.get_item(Key={'user_id': user_id})
            return response(200, result.get('Item', {}))

        # UPDATE USER
        elif http_method == 'PUT':
            user_id = path_params['id']

            table.update_item(
                Key={'user_id': user_id},
                UpdateExpression="set #n=:n, email=:e",
                ExpressionAttributeNames={'#n': 'name'},
                ExpressionAttributeValues={
                    ':n': body.get('name', ''),
                    ':e': body.get('email', '')
                }
            )

            return response(200, {'message': 'User updated'})

        # DELETE USER
        elif http_method == 'DELETE':
            user_id = path_params['id']

            table.delete_item(Key={'user_id': user_id})

            return response(200, {'message': 'User deleted'})

        else:
            return response(400, {'message': 'Invalid request'})

    except Exception as e:
        return response(500, {'error': str(e)})


def response(status, body):
    return {
        'statusCode': status,
        'headers': {
            'Content-Type': 'application/json'
        },
        'body': json.dumps(body)
    }

```

## 5️⃣ Create API Gateway

- Go to AWS Console → API Gateway  
- Click **Create API** → Select **REST API**  
- Click **Build**
![API](./pictures/API.png)

#### ➤ Create Resource
- Click **Create Resource**
- Resource name: `users`
- Resource path: `/users`

#### ➤ Add Methods to `/users`
- Add **POST** method  
- Add **GET** method  

---

## 6️⃣ Create Dynamic Resource `{id}`

- Click on `/users`
- Click **Create Resource**
- Resource name: `{id}` (important: include curly brackets)

This will create:
```
/users/{id}
```


#### ➤ Add Methods to `/users/{id}`
- Add **GET** method  
- Add **PUT** method  
- Add **DELETE** method  

![API-GETandPOST](./pictures/API-GETandPOST.png)

---

## 7️⃣ Connect Lambda Function

For each method:

- Integration type: **Lambda**
- Select your Lambda function (`createUser`)
- Enable **Lambda Proxy Integration**
- Click **Save**
- Click **Allow** when prompted
![lambda-integration](./pictures/lambda-integration.png)

---

## 8️⃣ Deploy API

- Click **Deploy API**
- Create new stage:
  - Stage name: `dev`
- Click **Deploy**
![deploy-api](./pictures/deploy-api.png)

### ➤ API URL Format:
```
https://<api-id>.execute-api.<region>.amazonaws.com/dev
```


---

## ➤ Testing Using Postman

### ➤ 1. Create User (POST)

**Method:** POST  
```
/dev/users
```

**Body:**
```json
{
  "name": "Prutha",
  "email": "prutha@gmail.com"
}
```
![test-1](./pictures/test-1.png)

### ➤ 2. Get All Users (GET)

**Method:** GET  

```
/dev/users
```
![test-2](./pictures/test-2.png)

---

### ➤ 3. Get Single User (GET)

**Method:** GET  
```
/dev/users/{id}
```


Replace `{id}` with actual `user_id`

---

### ➤ 4. Update User (PUT)

**Method:** PUT  
```
/dev/users/{id}
```

**Body:**
```json
{
  "name": "Updated Name",
  "email": "updated@gmail.com"
}
```
![test-3](./pictures/test-3.png)

### ➤ 5. Delete User (DELETE)

**Method:** DELETE
```
/dev/users/{id}
```
![test-4](./pictures/test-4.png)

### ➤ 6. Check entries in DynamoDB
![entries in DynamoDB](./pictures/table-entries.png)


## ➤ Learning Outcome
- Learned AWS serverless architecture
- Built REST API from scratch
- Connected multiple AWS services
- Tested APIs using Postman

---

## ➤ Conclusion

This project demonstrates how to build a complete **serverless REST API** using AWS services without managing any servers.  

By integrating **API Gateway, Lambda, and DynamoDB**, we successfully implemented full **CRUD operations** in a scalable and cost-efficient way.

Through this project, I gained hands-on experience in:
- Designing REST APIs  
- Working with AWS serverless architecture  
- Handling real-time data operations  
- Testing APIs using Postman  

---
