# PostgreSQL: Parent-Child PID

## 1. Provision an Azure PostgreSQL Instance

### Step 1.1: Create a PostgreSQL Flexible Server on Azure

- **Log into the Azure Portal:**
   Visit [portal.azure.com](https://portal.azure.com/) and sign in.
- Create a Resource:
  - Click **“Create a resource”**.
  - Search for **“Azure Database for PostgreSQL Flexible Server”** and select it.
- Configure Server Settings:
  - **Subscription & Resource Group:** Choose an existing resource group or create a new one.
  - **Server Name:** Enter a unique server name.
  - **Region:** Select the region closest to your users.
  - **Workload/Compute:** Choose an appropriate compute size and storage configuration based on your demo needs.
  - **Administrator Credentials:** Set an admin username and a strong password.
- Networking & Security:
  - For demo purposes, you can select “Public access” and add your client IP address to the firewall rules.
- **Review and Create:**
   Verify your settings and click **“Create”** to provision the server.

### Step 1.2: Configure Connection Security

- Firewall Rules:
  - In the PostgreSQL server’s settings, navigate to **“Networking”**.
  - Ensure your IP (or a range of IPs you plan to use) is added to the allowed list.
- **SSL Requirement:**
   Note that Azure PostgreSQL may require SSL connections. Make sure your client or application is configured accordingly.

------

## 2. Connect to Your PostgreSQL Instance

### Step 2.1: Retrieve Connection Information

- In your PostgreSQL server dashboard on Azure, locate the **“Connection strings”** section.
- Note the host name, port (default 5432), admin username, and database name (default is often **postgres**).

### Step 2.2: Connect Using a Client

- Using psql:

  Open a terminal and run:

  ```bash
  psql "host=<your_server_name>.postgres.database.azure.com port=5432 dbname=postgres user=<admin_username>@<your_server_name> sslmode=require"
  ```

- Using pgAdmin:

  - Open pgAdmin and create a new server connection using the retrieved details.
  - Ensure the connection is set to use SSL.

------

## 3. Set Up Sample Data and Simulate Process Activity

For this demo, we will create sample functions and tables that simulate long-running or parallel queries. These will later help in generating entries in the system view `pg_stat_activity` that you can query to identify “parent” and “child” sessions.

### Step 3.1: Create a Demo Database and Table

- Create a New Database:

   (Optional, if you want to isolate your demo)

  ```sql
  CREATE DATABASE demo_db;
  \connect demo_db
  ```

- Create a Sample Table:

  ```sql
  CREATE TABLE sample_data (
      id SERIAL PRIMARY KEY,
      description TEXT,
      created_at TIMESTAMPTZ DEFAULT NOW()
  );
  ```

- Insert Sample Data:

  ```sql
  INSERT INTO sample_data (description)
  VALUES ('Demo entry 1'), ('Demo entry 2'), ('Demo entry 3');
  ```

### Step 3.2: Create a Sample Stored Procedure to Simulate a Long-Running Query

- Create a Function with pg_sleep:

  This simulates a process that holds a connection for a period.

  ```sql
  CREATE OR REPLACE FUNCTION simulate_long_query()
  RETURNS VOID AS $$
  BEGIN
      -- Simulate work by sleeping for 30 seconds
      PERFORM pg_sleep(30);
      -- Optionally, do some DML work
      INSERT INTO sample_data (description) VALUES ('Long query complete at ' || NOW());
  END;
  $$ LANGUAGE plpgsql;
  ```

- Invoke the Function in Parallel:

  Open multiple sessions (or use background jobs) to execute this function concurrently. For example, in psql:

  ```sql
  SELECT simulate_long_query();
  ```

  Open another terminal window and run the same query. This will help generate multiple active sessions.

### Step 3.3: Query pg_stat_activity for Process Details

- In a new session, run:

  ```sql
  SELECT pid, usename, application_name, client_addr, backend_start, state, query
  FROM pg_stat_activity;
  ```

- **Note:**
   While PostgreSQL doesn’t always show explicit “parent” and “child” PIDs like an operating system process tree, you can use this view to identify long-running sessions, and—if you structure your application appropriately—you might simulate relationships (e.g., by tagging sessions with an application name or using session variables).

------

## 4. Build a Practice Application to Demonstrate PID Identification

We will create a simple Python Flask application that connects to your Azure PostgreSQL instance, queries `pg_stat_activity`, and displays session details.

### Step 4.1: Set Up Your Python Environment

- Install Dependencies:

  Ensure you have Python 3 installed, then install Flask and psycopg2:

  ```bash
  pip install Flask psycopg2-binary
  ```

### Step 4.2: Write a Sample Flask Application

- Create an Application File (e.g., `app.py`):

  ```python
  from flask import Flask, render_template_string
  import psycopg2
  import os
  
  app = Flask(__name__)
  
  # Update these with your Azure PostgreSQL connection details
  DB_HOST = "<your_server_name>.postgres.database.azure.com"
  DB_NAME = "demo_db"
  DB_USER = "<admin_username>@<your_server_name>"
  DB_PASSWORD = os.environ.get("DB_PASSWORD", "your_password")
  DB_PORT = 5432
  
  def get_db_connection():
      conn = psycopg2.connect(
          host=DB_HOST,
          database=DB_NAME,
          user=DB_USER,
          password=DB_PASSWORD,
          port=DB_PORT,
          sslmode="require"
      )
      return conn
  
  @app.route('/')
  def index():
      conn = get_db_connection()
      cur = conn.cursor()
      cur.execute("""
          SELECT pid, usename, application_name, client_addr, backend_start, state, query
          FROM pg_stat_activity
          ORDER BY backend_start DESC;
      """)
      sessions = cur.fetchall()
      cur.close()
      conn.close()
      
      # Simple HTML template to display the sessions
      html = """
      <!DOCTYPE html>
      <html>
      <head>
          <title>PostgreSQL Sessions</title>
          <style>
              table, th, td { border: 1px solid black; border-collapse: collapse; padding: 5px; }
          </style>
      </head>
      <body>
          <h1>Active PostgreSQL Sessions</h1>
          <table>
              <tr>
                  <th>PID</th>
                  <th>User</th>
                  <th>Application</th>
                  <th>Client Address</th>
                  <th>Start Time</th>
                  <th>State</th>
                  <th>Query</th>
              </tr>
              {% for row in sessions %}
              <tr>
                  <td>{{ row[0] }}</td>
                  <td>{{ row[1] }}</td>
                  <td>{{ row[2] }}</td>
                  <td>{{ row[3] }}</td>
                  <td>{{ row[4] }}</td>
                  <td>{{ row[5] }}</td>
                  <td>{{ row[6] }}</td>
              </tr>
              {% endfor %}
          </table>
      </body>
      </html>
      """
      return render_template_string(html, sessions=sessions)
  
  if __name__ == '__main__':
      app.run(debug=True, host='0.0.0.0', port=5000)
  ```

- Run the Application:

  ```bash
  python app.py
  ```

  Open a web browser and navigate to 

  ```
  http://localhost:5000
  ```

   to view active PostgreSQL sessions.

### Step 4.3: Extend the Application (Optional)

- **Tag Sessions:**
   You might consider modifying your stored procedures to set an `application_name` so that you can visually group or identify sessions that belong together.
- **Simulate Parent–Child Relationships:**
   While PostgreSQL’s process model isn’t inherently hierarchical at the SQL level, you can simulate a parent process by having one session spawn sub-tasks (for example, via background jobs) and then log a common identifier in a custom table. Your application can then join this custom data with `pg_stat_activity` to “visualize” relationships.

------

## 5. Testing and Demo

- **Simulate Load:**
   Run multiple instances of the `simulate_long_query()` function from different sessions to generate several entries in `pg_stat_activity`.
- **Observe in the Application:**
   Refresh your Flask app page to see the active sessions.
- **Discuss Findings:**
   Use the displayed session information (PID, start time, query text) to explain how you would identify problematic processes and correlate them with your simulated parent–child activity (via custom tagging or additional logging).

------

## 6. Next Steps

- **Enhance Safety Checks:**
   Before implementing any PID cleanup or process termination, add robust checks and ensure the actions are only performed in non-production environments.
- **Integrate with Monitoring Tools:**
   Consider integrating the solution with Azure Monitor or a similar tool for real-time alerts.
- **Iterate Based on Feedback:**
   After your demo, collect feedback from your team to refine the data extraction, visualization, and any automated remediation steps.