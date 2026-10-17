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

## ✅ Completed audit

The [Auntie Som Lab collection](auntie-som-noodle-api-test/) contains 13 ordered requests and 14 automated tests. Its `local` environment supplies `baseUrl`; the login post-response script stores `accessToken` as a secret environment variable, and the create-order script stores `orderId` as a runtime variable. The tests also save menu prices and starting stock as runtime variables, then compare API responses against those values. Bruno never writes the token value into the collection file.

### Bugs found

| Business rule | Observed behavior | Bruno test that catches it |
| --- | --- | --- |
| An order for one 50-baht noodle should cost 50 baht; no discount rule is specified. | `POST /orders` returns 45 baht. `GET /orders/<orderId>` repeats the incorrect total. | `07-create-order.bru`, `08-get-order.bru` |
| An unknown order should return HTTP 404. | `GET /orders/ORD-does-not-exist` returns HTTP 200 with an error body. | `09-unknown-order.bru` |
| A customer cannot buy more portions than are in stock; a rejected order must not change stock. | Ordering two rice portions when only one is available returns HTTP 200 and changes stock from 1 to -1. | `10-too-many.bru`, `11-stock-after-rejection.bru` |
| Order quantity must be positive. | Zero and negative quantities both create orders; a negative quantity also increases stock. | `12-zero-quantity.bru`, `13-negative-quantity.bru` |

The collection also checks menu data, successful and failed login, token reuse, missing and invalid bearer tokens, unknown items, and retrieval of a created order. Those checks pass on the supplied backend.

### Run the audit

Start with a **fresh API process** for each full collection run because the backend keeps stock and orders in memory. Leave the process running in one terminal:

```bash
python3 -m pip install flask
python3 auntie-som-noodle-api/app.py
```

In another terminal, install the [Bruno CLI](https://docs.usebruno.com/bru-cli/installation), then run from the collection folder:

```bash
cd auntie-som-noodle-api-test
bru run --env local --noproxy
```

The local environment uses `http://127.0.0.1:5000` to reach Flask reliably on macOS. Change `baseUrl` in `environments/local.bru` if the server uses another address. Run requests in order; a full run changes stock. The CLI should exit with failure while the backend bugs remain. On the supplied backend, the verified result is **7 passing and 7 failing tests**. These failures are the audit findings, not a reason to change `app.py` for this assignment.

---
*Good luck. Auntie Som is counting on you!* 🍜🔥
