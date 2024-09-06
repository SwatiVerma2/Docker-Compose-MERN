# Docker-compose for MERN Application

## Using Dockerfile

1. Create a network for easy communication
   
    `docker network create mern`
   
2. Containerize the application by creating dockerfile for frontend.
   
   ```dockerfile

    FROM node:18.9.1
    
    WORKDIR /app
    
    COPY package.json .
    
    RUN npm install
    
    EXPOSE 5173
    
    COPY . .
  
    CMD ["npm", "run", "dev"]
  
   ```
    - Build the image: `docker build -t mern-frontend .`
    - Run the image: `docker run --name=frontend --network=mern -d -p 5173:5173 mern-frontend`
    - Verify that the container is running : Open your browser and type http://localhost:5173

4. Run the mongodb container
   
   `docker run --network=demo --name mongodb -d -p 27017:27017 -v ~/opt/data:/data/db mongo:latest`
   
   `-v ~/opt/data:/data/db` - This mounts a local directory (~/opt/data) to the MongoDB container’s /data/db directory. 

   This setup ensures that MongoDB’s data is stored persistently on the host machine, so the database data isn't lost when the container stops or restarts.

4. Backend  creating dockerfile for frontend.

```dockerfile

  FROM node:18.9.1
  
  WORKDIR /app
  
  COPY package.json .
  
  RUN npm install
  
  COPY . .
  
  EXPOSE 5050
  
  CMD ["npm", "start"]

```

 - Build the image: `docker build -t mern-backend .`
 - Run the container: `docker run --name=backend --network=mern -d -p 5050:5050 mern-backend`
 - Verify that the container is running : Open your browser and type http://localhost:5050


It has a particular order and multiple steps so to avoid this mannual activity we'll use docker-compose.

5. Remove the containers before creating docker compose.
   
   `docker rm -f 8772f8391955 40cfc6e1407c 289b57dcff65`

  
### Importance of creating a separate network

**Service Isolation and Scoped Communication:**

- By creating a custom network, you ensure that only the containers attached to that network can communicate with each other. 
- This provides better security, as it limits access to the containers to only those you explicitly allow.
- Using the default network (like the bridge network) means that all containers launched on the default bridge can communicate, potentially creating unwanted interactions between unrelated services.

## Using `docker-compose.yml`

```dockerfile
services:
  backend:
    build: ./mern/backend
    ports:
      - "5050:5050" 
    networks:
      - mern_network
    depends_on:
      - mongodb

  frontend:
    build: ./mern/frontend
    ports:
      - "5173:5173"  
    networks:
      - mern_network

  mongodb:
    image: mongo:latest  
    ports:
      - "27017:27017"  
    networks:
      - mern_network
    volumes:
      - mongo-data:/data/db  

networks:
  mern_network:
    driver: bridge 

volumes:
  mongo-data:
    driver: local  # Persist MongoDB data locally


```
`docker compose up -d`

![image](https://github.com/user-attachments/assets/e78606b4-70e8-4163-af14-cfee2d9e38f4)

The basic structure of docker-compose.yml inclues:

Key Sections:

1. **services** - Defines each service (container) in the application.
- build: Specifies where to find the Dockerfile.
- image: Defines the image to use for the service.
- ports: Maps ports between the container and the host.
- volumes: Mounts local directories to containers for data persistence.
- depends_on: Ensures service order (e.g., backend waits for MongoDB).
  
2. **networks** - Defines custom networks for communication between containers.

3. **volumes** - Sets up persistent storage for services (like MongoDB).

![image](https://github.com/user-attachments/assets/97772c0f-3538-4820-a10b-a47b9c0ec3fb)

Reference -  Abhishek.Veeramalla
