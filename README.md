# MZinga Decoupled Email Worker Lab

Welcome to the MZinga Email Worker Lab! This project demonstrates a decoupled architecture using Payload CMS (Node.js) and a background worker (Python), connected by a MongoDB replica set.

## 🛠️ Prerequisites
Before you begin, make sure you have the following installed on your PC:
- [Git](https://git-scm.com/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Make sure it is running!)
- [Node.js](https://nodejs.org/) (v18 or higher)
- [Python](https://www.python.org/downloads/) (v3.10 or higher)

---

## 🚀 Step 0: Clone the Repository
First, clone this repository to your local machine and open the folder:
```bash
git clone https://github.com/SwDA-Team-15/lab1.git
cd lab1
```

---

## 🗄️ Step 1: Start the Databases
We use Docker to run MongoDB, RabbitMQ, and Redis.

1. Navigate to the CMS folder:
```bash
cd mzinga-apps
```
2. Start the infrastructure in the background:
```bash
docker compose up database messagebus cache -d
```
3. **Initialize the Database:** Wait about 5-10 seconds for the database to boot, then run these two commands to set up the replica set and the master admin user:

   *Initialize the replica set:*
   ```bash
   docker exec mzinga-apps-database-1 mongosh --eval "rs.initiate()"
   ```
   *Wait 5 seconds, then create the admin user:*
   ```bash
   docker exec mzinga-apps-database-1 mongosh --eval "db.getSiblingDB('admin').createUser({user: 'admin', pwd: 'admin', roles: [{role: 'root', db: 'admin'}]})"
   ```

---

## 🌐 Step 2: Start Payload CMS
Now, let's start the main web application. 

1. Ensure you are still inside the `mzinga-apps` directory.
2. Open the `.env` file and verify this line is at the bottom (this enables the worker flow):
   ```env
   COMMUNICATIONS_EXTERNAL_WORKER=true
   ```
3. Install dependencies and start the server:
   ```bash
   npm install
   npm run dev
   ```
*Leave this terminal open! You should see a "Server listening on port 3000" message.*

---

## 🐍 Step 3: Start the Python Worker
The Python worker is an independent microservice that processes emails in the background.

1. Open a **new, separate terminal window** and navigate to the worker folder:
   ```bash
   cd lab1-worker
   ```
2. Create and activate a virtual environment:
   - **Windows:**
     ```bash
     python -m venv .venv
     .\.venv\Scripts\activate
     ```
   - **Mac/Linux:**
     ```bash
     python3 -m venv .venv
     source .venv/bin/activate
     ```
3. Install the required Python libraries:
   ```bash
   pip install -r requirements.txt
   ```
4. Start the worker:
   ```bash
   python worker.py
   ```
*Leave this terminal open! It will print out a message when it connects to MongoDB.*

---

## 🧪 Step 4: Test the Flow!
Let's watch the decoupled systems work together:

1. Open your browser and go to **http://localhost:3000/admin**.
2. Log in using the admin account we created in Step 1 (`admin` / `admin`).
3. Navigate to **Users** and create a dummy user with a fake email address.
4. Navigate to **Communications** and click **Create New**.
5. Fill out the Subject, Body, and select your dummy user in the "To" field.
6. Click **Save**.

**The Result:** Watch your Python worker terminal! Within 5 seconds, it will detect the new email, print the subject, and mark it as sent. If you refresh the page in your browser, you will see the status dropdown has automatically changed from **Pending** to **Sent**.
