<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Dockerized Fullstack Webapp</title>
</head>
<body style="font-family: Arial, sans-serif; line-height: 1.6;">

  <h1><code>docker-compose.yml</code> File</h1>
  <p>This <code>docker-compose.yml</code> defines a multi-container web application using Docker Compose v3.8. The stack includes a frontend, backend, and MySQL database.</p>

  <h2>DATABASE (MySQL)</h2>
  <img src="https://github.com/user-attachments/assets/99b2bf7c-a840-42c2-bb01-be9e571de6b0" alt="MySQL image">

  <h3>webapp-db (MySQL)</h3>
  <ul>
    <li><strong>Image:</strong> Uses <code>mysql:5.7</code>, a stable version.</li>
    <li><strong>Environment:</strong> Sets variables such as root password (<code>1234</code>) and database name (<code>webapp</code>).</li>
    <li><strong>Ports:</strong> Maps internal port <code>3306</code> to <code>3307</code> on host.</li>
    <li><strong>Networks:</strong> Connected to <code>mi_red</code>.</li>
  </ul>

  <h2>BACKEND (Node.js)</h2>
  <img src="https://github.com/user-attachments/assets/0f8682c6-9edc-4bf1-af5e-30d174b08498" alt="Backend image">

  <h3>backend (REST API)</h3>
  <ul>
    <li><strong>Build:</strong> From <code>./backend</code> directory (Node.js with Express).</li>
    <li><strong>Ports:</strong> Exposes <code>3000</code> for API access.</li>
    <li><strong>Environment:</strong> Connects to MySQL with:
      <ul>
        <li>Host: <code>webapp-db</code></li>
        <li>Port: <code>3306</code></li>
        <li>User: <code>root</code></li>
        <li>Password: <code>1234</code></li>
        <li>Database: <code>webapp</code></li>
      </ul>
    </li>
    <li><strong>Depends_on:</strong> Waits for database.</li>
    <li><strong>Networks:</strong> Connected to <code>mi_red</code>.</li>
  </ul>

  <h2>FRONTEND (Nginx)</h2>
  <img src="https://github.com/user-attachments/assets/7737fbda-da2d-4dba-b01f-0a61014ba618" alt="Frontend image">

  <h3>frontend (Nginx)</h3>
  <ul>
    <li><strong>Image:</strong> <code>nginx:alpine</code> (lightweight and fast).</li>
    <li><strong>Build:</strong> From <code>./frontend</code> directory (HTML/CSS/JS).</li>
    <li><strong>Ports:</strong> Exposes port <code>80</code> for browser access.</li>
    <li><strong>Networks:</strong> Connected to <code>mi_red</code>.</li>
  </ul>

  <h2>NETWORK & VOLUMES</h2>
  <pre><code>volumes:
  db_data:

networks:
  mi_red:
    driver: bridge</code></pre>

  <ul>
    <li><strong>Networks:</strong> Custom bridge network for inter-service communication.</li>
    <li><strong>Volumes:</strong> Declares <code>db_data</code> (not used yet but available for data persistence).</li>
  </ul>

  <h2>HTML Pages</h2>
  <img src="https://github.com/user-attachments/assets/8d3cbbf4-02e3-49ff-8937-c938ec7a9f23" alt="Page overview">

  <h3>Registration Page</h3>
  <img src="https://github.com/user-attachments/assets/e2ca10ee-e366-4f8a-b15c-78fbb047bc27" alt="Registration">

  <p>When the form is submitted, the data is saved in the local Docker MySQL database:</p>
  <img src="https://github.com/user-attachments/assets/fecbe7d0-941d-4a0f-b604-3ebab6379eb6" alt="MySQL data saved">

  <p>After logging in with the same credentials, the user is redirected to a confirmation page:</p>
  <img src="https://github.com/user-attachments/assets/3ee2b7c4-8d00-4644-9474-aab1e7fc78b9" alt="Login success">

  <h3>Running the Project</h3>
  <p>Use the command:</p>
  <pre><code>docker-compose up --build -d</code></pre>
  <p>The <code>-d</code> flag runs containers in detached mode so your terminal remains free. Verify successful backend connection logs before doing this.</p>

  <h3>Database Table Schema</h3>
  <pre><code>CREATE TABLE registros (
  id INT AUTO_INCREMENT PRIMARY KEY,
  correo VARCHAR(50) NOT NULL,
  nickname VARCHAR(30) NOT NULL,
  password VARCHAR(20) NOT NULL,
  fecha_registro DATETIME DEFAULT CURRENT_TIMESTAMP
);</code></pre>

  <p>Although the setup is basic, it was a great learning experience working with Docker and understanding container-based deployments.</p>

  <h2>How to Deploy the Project</h2>
  <ol>
    <li>Download all project files and place them in a folder named <code>WEBAPP</code>.</li>
    <li>Install and launch Docker Desktop.</li>
    <li>Open a terminal and navigate to the directory containing <code>docker-compose.yml</code>.</li>
    <li>Run:
      <pre><code>docker-compose up -d</code></pre>
    </li>
  </ol>

  <p>The first build might take a few minutes.</p>
  <img src="https://github.com/user-attachments/assets/8262a438-68f7-447f-8cb0-40dd3a3dee15" alt="docker-compose running">

  <h3>Set Up the Databases</h3>

  <h4>Slaves:</h4>
  <pre><code>docker exec -it webapp-db-slave1 mysql -uroot -p1234</code></pre>
  <p>If the <code>webapp</code> database doesn’t exist, create it:</p>
  <pre><code>create database webapp;</code></pre>

  <pre><code>docker exec -it webapp-db-slave2 mysql -uroot -p1234</code></pre>
  <p>In both slaves, run:</p>
  <pre><code>START SLAVE;</code></pre>

  <h4>Master:</h4>
  <pre><code>docker exec -it webapp-db-master mysql -uroot -p1234</code></pre>
  <pre><code>use webapp;</code></pre>
  <pre><code>CREATE TABLE registros (
  id INT AUTO_INCREMENT PRIMARY KEY,
  correo VARCHAR(50) NOT NULL,
  nickname VARCHAR(30) NOT NULL,
  password VARCHAR(20) NOT NULL,
  fecha_registro DATETIME DEFAULT CURRENT_TIMESTAMP
);</code></pre>

  <p>The changes should automatically replicate to <code>webapp-db-slave1</code> and <code>webapp-db-slave2</code>.</p>

  <p>Visit the site at: <code>http://localhost</code></p>

</body>
</html>
