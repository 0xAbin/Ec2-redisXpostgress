# Redis Installation and Configuration on AWS EC2 with TypeScript

This guide provides detailed steps to install, configure, and secure Redis on an AWS EC2 instance, as well as accessing Redis from the frontend using TypeScript. Additionally, it covers writing data from your database to Redis.

## Prerequisites
- Basic knowledge of vim and launching EC2 instances via AWS console.
- Familiarity with TypeScript and Node.js.

## Launch EC2 Instance
For this tutorial, a t2.micro (free tier) AWS EC2 instance is used. For step-by-step details, refer to the [AWS EC2 Getting Started Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html).

## Step 1 — Installing and Configuring Redis

1. **Update the Package Cache**

    ```sh
    sudo apt update
    ```

2. **Install Redis**

    ```sh
    sudo apt install redis-server
    ```

3. **Configure Redis**

    Open the Redis configuration file:

    ```sh
    sudo vim /etc/redis/redis.conf
    ```

    Find the `supervised` directive and set it to `systemd`:

    ```conf
    supervised systemd
    ```

4. **Restart Redis Service**

    ```sh
    sudo systemctl restart redis
    ```

## Step 2 — Testing Redis

1. **Check Redis Service Status**

    ```sh
    sudo systemctl status redis
    ```

2. **Connect to Redis CLI**

    ```sh
    redis-cli
    ```

    Run a simple command inside Redis CLI to test:

    ```sh
    ping
    ```

    You should get a response: `PONG`.

## Step 3 — Create a User in Redis

1. **Open Redis Configuration File**

    ```sh
    sudo vim /etc/redis/redis.conf
    ```

2. **Set Password for Default User**

    Scroll to the Security section and update the password:

    ```conf
    requirepass your_secure_password
    ```

3. **Create a New User**

    Connect to Redis CLI and run:

    ```sh
    ACL setuser myuser allcommands allkeys on >mypassword
    ```

    Add this user to the configuration file:

    ```conf
    user myuser on >mypassword ~* &* +@all
    ```

4. **Restart Redis Server**

    ```sh
    sudo systemctl restart redis
    ```

5. **Authenticate with New User**

    ```sh
    redis-cli
    auth myuser mypassword
    ```

## Step 4 — Expose Redis to Public IP

1. **Modify Bind Address**

    Open the Redis configuration file:

    ```sh
    sudo vim /etc/redis/redis.conf
    ```

    Update the bind address to your EC2 public IP:

    ```conf
    bind 0.0.0.0
    ```

2. **Restart Redis Server**

    ```sh
    sudo systemctl restart redis
    ```

## Step 5 — Writing Data from Database to Redis with TypeScript

To write data from your database to Redis using TypeScript, follow these steps:

1. **Initialize Your TypeScript Project**

    ```sh
    mkdir my-redis-project
    cd my-redis-project
    npm init -y
    npm install typescript ts-node @types/node --save-dev
    npx tsc --init
    ```

2. **Install Redis Client Library**

    ```sh
    npm install redis
    npm install @types/redis --save-dev
    ```

3. **Create a Redis Client in TypeScript**

    Create a file named `index.ts` and add the following code:

    ```typescript
    import * as redis from 'redis';

    const client = redis.createClient({
        host: 'your_ec2_public_ip',
        port: 6379,
        password: 'your_secure_password'
    });

    client.on('connect', () => {
        console.log('Connected to Redis');
    });

    client.on('error', (err) => {
        console.error('Redis error', err);
    });

    interface UserData {
        id: string;
        name: string;
    }

    // Function to write data to Redis
    const writeDataToRedis = (data: UserData[]) => {
        data.forEach(user => {
            client.set(user.id, JSON.stringify(user), (err, reply) => {
                if (err) {
                    console.error('Error setting data in Redis', err);
                } else {
                    console.log('Set data in Redis', reply);
                }
            });
        });
    };

    // Example data
    const data: UserData[] = [
        { id: 'user:1', name: 'John Doe' },
        { id: 'user:2', name: 'Jane Doe' }
    ];

    writeDataToRedis(data);

    // To ensure the program runs long enough to complete the operations
    setTimeout(() => client.quit(), 1000);
    ```

4. **Run Your TypeScript Code**

    ```sh
    npx ts-node index.ts
    ```

## Accessing Redis from Frontend with TypeScript and Express

To access Redis from the frontend, you can set up an API server using Express and TypeScript.

1. **Install Express and Types**

    ```sh
    npm install express
    npm install @types/express --save-dev
    ```

2. **Create an Express Server**

    Create a file named `server.ts` and add the following code:

    ```typescript
    import * as express from 'express';
    import * as redis from 'redis';

    const app = express();
    const client = redis.createClient({
        host: 'your_ec2_public_ip',
        port: 6379,
        password: 'your_secure_password'
    });

    app.get('/data/:key', (req, res) => {
        const key = req.params.key;
        client.get(key, (err, reply) => {
            if (err) {
                res.status(500).send(err);
            } else {
                res.send(reply ? JSON.parse(reply) : 'Key not found');
            }
        });
    });

    const PORT = 3000;
    app.listen(PORT, () => {
        console.log(`Server is running on port ${PORT}`);
    });
    ```

3. **Run Your Express Server**

    ```sh
    npx ts-node server.ts
    ```

    You can now access data stored in Redis via your Express API, e.g., `http://your_ec2_public_ip:3000/data/user:1`.

