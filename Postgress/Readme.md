# PostgreSQL Setup on EC2 Instance (Ubuntu)
--------------------------------------------
## Step 1: Install PostgreSQL

1. **Update the Package List:**
   ```bash
   sudo apt update -y
   ```

2. **Install PostgreSQL:**
   ```bash
   sudo apt install postgresql postgresql-contrib -y
   ```

3. **Start PostgreSQL Service:**
   ```bash
   sudo systemctl start postgresql
   ```

4. **Enable PostgreSQL to Start on Boot:**
   ```bash
   sudo systemctl enable postgresql
   ```

## Step 2: Configure PostgreSQL

1. **Switch to the PostgreSQL User:**
   ```bash
   sudo -i -u postgres
   ```

2. **Access PostgreSQL Prompt:**
   ```bash
   psql
   ```

3. **Set a Password for the PostgreSQL User:**
   ```sql
   \password postgres
   ```

4. **Create a New Database and User (Optional):**
   ```sql
   CREATE DATABASE mydatabase;
   CREATE USER myuser WITH ENCRYPTED PASSWORD 'mypassword';
   GRANT ALL PRIVILEGES ON DATABASE mydatabase TO myuser;
   ```

5. **Exit the PostgreSQL Prompt:**
   ```sql
   \q
   ```

## Step 3: Configure Remote Access (Optional)

1. **Edit `pg_hba.conf`:**
   ```bash
   sudo nano /etc/postgresql/12/main/pg_hba.conf
   ```
   Add the following line at the end of the file to allow remote access:
   ```
   host    all             all             0.0.0.0/0               md5
   ```

2. **Edit `postgresql.conf`:**
   ```bash
   sudo nano /etc/postgresql/12/main/postgresql.conf
   ```
   Uncomment and set the following line to allow PostgreSQL to listen on all IP addresses:
   ```
   listen_addresses = '*'
   ```

3. **Restart PostgreSQL Service:**
   ```bash
   sudo systemctl restart postgresql
   ```

## Step 4: Connect PostgreSQL to a TypeScript Backend

1. **Install Dependencies:**
   Ensure you have `pg` installed in your TypeScript project:
   ```bash
   npm install pg
   ```

2. **Set Up Database Connection:**
   Create a `db.ts` file in your TypeScript project to handle the database connection.

   ```typescript
   import { Pool } from 'pg';

   const pool = new Pool({
     user: 'myuser',
     host: 'your-ec2-public-dns',
     database: 'mydatabase',
     password: 'mypassword',
     port: 5432, // default PostgreSQL port
   });

   export default pool;
   ```

3. **Using the Database Connection:**
   Use the connection pool in your backend to interact with the PostgreSQL database.

   ```typescript
   import pool from './db';

   const getUsers = async () => {
     const client = await pool.connect();
     try {
       const res = await client.query('SELECT * FROM users');
       return res.rows;
     } finally {
       client.release();
     }
   };

   export { getUsers };
   ```

4. **Ensure TypeScript Configuration:**
   Ensure your `tsconfig.json` includes the necessary configurations for TypeScript to handle ES modules and other settings:

   ```json
   {
     "compilerOptions": {
       "target": "ES6",
       "module": "commonjs",
       "strict": true,
       "esModuleInterop": true,
       "skipLibCheck": true,
       "forceConsistentCasingInFileNames": true
     }
   }
   ```

5. **Run Your Application:**
   Start your TypeScript application to ensure everything is set up correctly:

   ```bash
   npm run start
   ```
