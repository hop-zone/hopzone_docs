# Hopzone - Installation Manual

## About
This document describes the process of setting up and running the Hopzone multiplayer platformer game. For all information about the project itself, please visit the **[project documentation](https://github.com/hop-zone/hopzone_docs)**.

## :1234: Setting up the backend
### Step 1: Cloning the github repository
The source code of the backend project is located on github and can be found [here](https://github.com/hop-zone/hopzone_backend).
Make sure you have **git** installed and clone the repository.
```
git clone https://github.com/hop-zone/hopzone_backend
```

### Step 2: Set up a Google Firebase project

The project uses Google's Firebase for authentication and managing users. Head over to the **[Firebase Console](https://console.firebase.google.com)**, create a new project and follow the steps. You can choose to use Google analytics, but this is not required.


### Step 3: Enabling authentication
Once the project is created, you will be reacted to the firebase console of your newly created project. Head over to the authentication section of your project and click on **Get Started**.

![](https://i.imgur.com/tnMU62r.png)
![](https://i.imgur.com/z74AtgP.png)


In the sign-in providers section, enable the **Email/Password** and **Anonymous** providers.

### Step 4: Creating a service account
The backend needs credentials to communicate with the Firebase project we just created. To make this possible, we need to create a **service account** for our backend to use.
- Click on project settings next to your project overview.
- Head over to the **Service accounts** section.
- Click Generate new private key

![](https://i.imgur.com/64lwRuD.png)
![](https://i.imgur.com/Ujw5yEY.png)

:warning: **NEVER** share these credentials with anyone. Anyone with these credentials can access and modify your application.


This will download a file containing your service account credentials. Be sure to **NEVER** share this file with anyone, or make it publicly available. 

### Step 5: Moving the google credentials.
Open the directory where you have cloned the **github**.
- Copy the google credentials file into `server/auth`
- Rename the file to `application-credentials.json`
- Ensure the path to this file is correct, as the application will not recognize these credentials otherwise: `server/auth/application-credentials.json`

### Step 6: Changing the database credentials (optional)
This step is not required, but **highly recommended** for security reasons. If you want to set up your own admin username and password for the database, head over to the `docker-compose.yml` file. and change the following lines with your own username and password:
- `MONGO_INITDB_ROOT_USERNAME`
- `MONGO_INITDB_ROOT_PASSWORD`

```yaml    
version: '3.4'

services:
  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
      - 27018:27018
    environment:
      MONGO_INITDB_ROOT_USERNAME: <your_username>
      MONGO_INITDB_ROOT_PASSWORD: <your_password>
    volumes:
      - hopzone_db:/data/db
  api:
    image: hopzone.azurecr.io/backend
    depends_on:
      - mongo
    build:
      context: .
      dockerfile: ./Dockerfile
    environment:
      NODE_ENV: production
    env_file:
      - docker.env
    ports:
      - 3001:3001

volumes:
  hopzone_db:
```

### Step 7: Creating an environment file.
For the application to connect to our database, we have to specify 2 environment variables. As we will user **Docker** for running the application, create an environment file in the root directory of the repository. Change the username and password values with your own from the previous step.

- Create a `docker.env` file

```
DB_CONNECTION_URL=mongodb://<your_username>:<your_password>@mongo:27017/
ENTITIES_DIR=/entities/**/*{.ts,.js}
```

### Step 8: Running the application
Make sure you have **Docker** installed on your system. If not, please head over to the [official docker website](https://docs.docker.com/get-docker/) and follow the instructions

- Open a terminal within your project folder and run the following command:
```
docker compose up -d --build
```

This will start up the backend. To validate that both services is running, run the following command:
```
docker ps
```

To validate that the backend started up correctly, head over to the logs of the api with the following command:
```
docker logs hopzone_backend_api_1
```
![](https://i.imgur.com/yksXr1Z.png)
:information_source: Following messages should be visible. If not, something went wrong in the installation process.



## :computer: Setting up the frontend

### Step 1: Cloning the repository
The source code of the frontend project is located on github and can be found [here](https://github.com/hop-zone/hopzone_frontend).
Go ahead and clone the github repository

```
git clone https://github.com/hop-zone/hopzone_frontend
```

### Step 2: Linking your firebase app

The frontend application also communicates directly via our Firebase Project. To make this possible, we also need to provide some credentials.
- Head over to the project settings of our existing **Firebase project**
- In the **General**, create a new app to link.
- Follow the steps (don't set up firebase hosting)

![](https://i.imgur.com/64lwRuD.png)
![](https://i.imgur.com/cuHyM5y.png)

The following credentials will appear:

```  javascript
const firebaseConfig = {
  apiKey: "****",
  authDomain: "***",
  projectId: "***",
  storageBucket: "***",
  messagingSenderId: "***",
  appId: "***"
};
```
:information_source: Hold on to these credentials for now.


### Step 3: Creating an environment file

- Create a `.env` file within the root directory of the git repository
- Fill in the correct values from your firebase app from the following config
- for the `NEXT_PUBLIC_BACKEND` field, replace `<your_url>` with the ip address of wherever your backend image is running on. If you are running the backend locally, you can replace this with `localhost`

```
NEXT_PUBLIC_FB_APIKEY=***
NEXT_PUBLIC_FB_AUTHDOMAIN=***
NEXT_PUBLIC_FB_PROJECTID=***
NEXT_PUBLIC_FB_STORAGEBUCKET=***
NEXT_PUBLIC_FB_MESSAGINGSENDERID=***
NEXT_PUBLIC_FB_APPID=***
NEXT_PUBLIC_BACKEND=<your_url>:3001
```

### Step 4: Building the docker image

The project is provided with the necessary dockerfile to build our image. Open a terminal within your project folder and run the following command.

```
docker build -t hopzone_frontend
```

Running this command will start building the docker image. This might take a while.

### Step 5: Running the docker image
After building the image, we can run it and pass our environment file in our run command.
- Run the image with the following command:
```
docker run --env-file .env -d -p 3000:3000 hopzone_frontend 
```

### Step 6: Test the application

Your application should now be fully operational. You can test the application by heading over to the following url to play around with your application! http://localhost:3000







