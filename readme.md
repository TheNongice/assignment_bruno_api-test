# Auntie Som Lab - API Testing Report
> [!NOTE]
> This forked repository is contains API test cases and backend code for Auntie Som's Noodle Stall assignment.

Author: Wasawat Junnasaksri, Chulalongkorn University.

## What This Lab Covers
The tests focus on:
1. Authentication behavior
2. Order input validation
3. Stock handling
4. Price calculation
5. Error status code correctness

## Bug List
1. The total price isn't expected with actual value.
    - `POST: /orders` Given the product 'A' is 30 baht and buy with 2 quantity. The total price expected to show 60 baht but show 55 baht.
    - For example:
    ```json
    { "orderId": "ORD-YYYYY", "status": "created", "totalPrice": 55 }
    ``` 
2. Wrong HTTP return status code for get the order that's not created before.
    - `GET: /orders/<Order_ID>` Given "ORD-YYYYY" is order id that has been assigned but user type wrong at last character into "ORD-YYYYX". The result and status code should show as a not found meaning.
    - The result is correct but for HTTP status code is show as `200`. So that's is wrong.
3. User can order with excessed than max of stock.
    - `POST: /orders` It uses arithmetic to decrease stock quantity by without checking stock remaining.
    - Your code at line 80 that shown `item["stock"] -= quantity` is explicitly to decrese without checking
4. User can order with minus quantity
    - `POST: /orders` The API doesn't verify what user type. Therefore, user can enter quantity with the value less than zero.
    - Example Input:
    ```json
    { "itemId": 2, "quantity": -1 }
    ```
    - Example Output:
    ```json
    { "orderId": "ORD-YYYYY", "status": "created", "totalPrice": -50 }
    ```
    - As you can see. Server is still accept the quantity with minus quantity.
      - Developer should fix by always verify `quantity` value before created order.
5. User can order with zero quantity
    - `POST: /orders` The API doesn't verify what user type. Therefore, user can enter quantity with the value less than or equl with zero.
    - Example Input:
    ```json
    { "itemId": 2, "quantity": 0 }
    ```
    - Example Output:
    ```json
    { "orderId": "ORD-YYYYY", "status": "created", "totalPrice": -5 }
    ```
    - As you can see. Server is still accept the quantity with minus quantity.
      - Developer should fix by always verify `quantity` value before created order.
6. Missing itemId is treated like not-found item
    - When `itemId` is missing, API falls through to `Item not found (404)`.
Semantically better behavior is `400 Bad Request`.
    - For example:
    ```json
    {"error": "itemId is required"}
    ```

---

# 🍜 API Testing Lab: Auntie Som’s Noodle Stall

> **Objective:** Prove that Nephew Lek’s "it works on my machine" attitude is a recipe for disaster.

---

## 📖 The Backstory

Auntie Som runs a **legendary noodle stall**. Her Tom Yum is world-class, her customers are loyal, and her business is booming.  

Recently, her nephew **Lek** (who just finished a 2-week coding bootcamp) decided to "digitize" the business. He built a backend API to handle online orders. Auntie Som is thrilled, but the code… well, Lek says:  

> *"It’s fine lah Auntie! I tested it once with my own phone. No need for professional testing!"*

**You are the Last Line of Defense.**  
Before Auntie Som launches this app to 1,000+ hungry customers, you must audit the system, find the logic flaws, and automate the validation using **Bruno**.

---

## 🎯 Learning Objectives

By the end of this lab, you should be able to:
- 🖋️ Write **Bruno test scripts** for automated validation.
- 📦 Extract values from API responses using **Post-response scripts**.
- 🔐 Manage and use **Environment Variables**.
- 🧠 Verify **Business Rules**, not just HTTP status codes.
- 🚀 Catch regressions automatically before they hit production.

---

## 🛠️ Setup Instructions

### 1️⃣ Prepare the Project
- **Fork this repository** to your own GitHub account.
- **Clone it** locally to your machine.

### 2️⃣ Start the API Server
The backend is a lightweight Flask application.
- **Requirements:** Python 3.10+ installed.
- **Navigate** to the `auntie-som-noodle-api` folder.
- **Install dependencies:**  
  ```bash
  pip install flask
  ```
- **Run the server:**  
  ```bash
  python app.py
  ```
- The server will run at: `http://localhost:5000`

### 3️⃣ Setup Bruno
- Download and install [Bruno](https://www.usebruno.com).
- Create a **New Collection** named `Auntie Som Lab`.
- Create an **Environment** (e.g., `local`) and set a variable `baseUrl` to `http://localhost:5000`.

---

## 📑 API Reference

### 🔐 Authentication
`POST /auth/login` - Get an access token.
- **Body (JSON):**
  ```json
  {
    "username": "student",
    "password": "hungry"
  }
  ```
- **Response (200 OK):**
  ```json
  {
    "accessToken": "uuid-token-string",
    "expiresIn": 3600
  }
  ```

---

### 🍜 Menu
`GET /menu` - View available items and stock levels.
- **Response (200 OK):**
  ```json
  [
    { "id": 1, "name": "Tom Yum Noodles", "price": 50, "stock": 2 },
    { "id": 2, "name": "Fried Rice", "price": 45, "stock": 1 }
  ]
  ```

---

### 🛒 Orders
`POST /orders` - Place a new order.
- **Headers:** `Authorization: Bearer <token>`
- **Body (JSON):**
  ```json
  {
    "itemId": 1,
    "quantity": 1
  }
  ```
- **Response (200 OK):**
  ```json
  {
    "orderId": "ORD-abc123",
    "status": "created",
    "totalPrice": 45
  }
  ```

`GET /orders/<orderId>` - Retrieve order details.
- **Response (200 OK):**
  ```json
  {
    "orderId": "ORD-abc123",
    "status": "created",
    "totalPrice": 45
  }
  ```

---

## 🕵️ Your Mission: The Silent Auditor

Your task is to create a comprehensive Bruno collection that validates the entire workflow.  

⚠️ **The Catch:** Nephew Lek left several **logic bugs** in the system. Some endpoints might return `200 OK` even when the business logic is completely broken.

**Your collection must include:**
1.  **Auth Flow**: Automatically store the `accessToken` from login and use it for subsequent requests.
2.  **Order Validation**: Ensure orders can only be placed with valid auth, valid items, and available stock.
3.  **Data Integrity**: Check that the `totalPrice` calculated by the API matches your expectations.
4.  **Edge Cases**: What happens if you order more than what's in stock? What happens if you request a non-existent order?

> [!TIP]
> **Clicking "Send" is not testing.** Your tests must contain scripts (Assertions or Javascript) that **FAIL** when the system behaves incorrectly.

---

## 🧠 Rules of the Lab
- ❌ **Do NOT** modify the `app.py` backend code.
- ❌ **Do NOT** rely on manual checking.
- ✅ **Use environment variables** for the Base URL and Tokens.
- ✅ **Use Scripts/Assertions** for all validations.

---

## 📦 What to Submit

1.  **Push your Bruno collection folder** to your GitHub repository.
2.  **Update your README.md** (the one in your fork) with:
    - A summary of the bugs you discovered.
    - How your tests detect these bugs.
4.  **Send the GitHub link** to your instructor via Discord.

---
*Good luck. Auntie Som is counting on you!* 🍜🔥
