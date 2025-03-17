# AI Dreamweaver Codebase

- **Mobile App**: React Native (for cross-platform ease) with Kotlin for any native Android functionality.
- **Backend**: Node.js with Express.js and GraphQL for API communication.
- **Database**: PostgreSQL as outlined.
- **Machine Learning**: TensorFlow/PyTorch for AI functionality based on DSM-5 dataset.
- **Security**: HIPAA and GDPR compliance baked into data handling and encryption.
- **Cloud Deployment**: Cheapest route, likely AWS free-tier or an alternative based on cost.
- **Third-party Integrations**: Bubble.io, Make.com, Voiceflow as specified.
- **CI/CD Pipeline**: GitHub Actions for automated deployment.

# AI Dreamweaver Codebase

The **AI Dreamweaver** project is structured as a full-stack application with mobile, backend, database, and machine learning components. Below is a breakdown of each part of the system, including technology choices, file structure, and configuration details. This codebase is designed for easy deployment and adherence to security standards.

## Mobile App (React Native & Kotlin)

The mobile app is developed using **React Native**, allowing a single JavaScript/TypeScript codebase to run on both iOS and Android devices ([
        What Is React Native? Complex Guide for 2024
      ](https://www.netguru.com/glossary/react-native#:~:text=framework%20that%20allows%20you%20to,using%20the%20same%20codebase)). React Native renders native UI components for smooth performance, and it supports platform-specific code when needed (for Android, we use Kotlin for any custom native modules). Key features of the app include:

- **Goal Tracking UI:** Users can create and monitor personal goals. This is implemented with React Native components (e.g., `FlatList` for goal lists, progress bars for goal completion).
- **Stress Detection Interface:** The app monitors user inputs and interactions to gauge stress levels. For example, it might track mood survey responses or use phone sensors (like heart rate via Bluetooth) if available. A native module in Kotlin could be used to access Android-specific APIs (like health sensors or device data) and expose them to React Native ([Creating a Native Module in React Native | by Mohamed Elsdody](https://medium.com/@mohamed.ma872/creating-a-native-module-in-react-native-df1a694c89a7#:~:text=Creating%20a%20Native%20Module%20in,cannot%20be%20done%20in)).
- **AI-Guided Recommendations:** A section of the app provides personalized tips or exercises. This can be a chatbot-like interface or a notification system that presents recommendations from the AI module.

**Project Structure (Mobile):**
```bash
/mobile
├── package.json             # React Native project dependencies (React, React Native, etc.)
├── App.js                   # Entry point of the app, sets up navigation and screens
├── ios/                     # iOS native project files (for Objective-C/Swift modules)
├── android/                 # Android native project files (for Kotlin modules)
│   └── app/src/main/java/.../DreamWeaverModule.kt   # Example custom native module in Kotlin
├── components/              # Reusable UI components (e.g., GoalCard, StressMeter)
├── screens/                 # Screen components for different app pages (Home, Goals, Profile, Recommendations)
└── utils/                   # Utility functions (e.g., formatting dates, computing stress score)
```

**Key Dependencies (Mobile):** The `package.json` includes `"react": "...", "react-native": "..."` along with UI libraries like React Navigation for navigation, and any sensors or charts library if needed for visualization. For native modules, React Native’s bridging is used to integrate Kotlin code when necessary (e.g., accessing hardware features).

**Kotlin Integration:** If the app needs to use an Android-specific feature (like a native stress detection API or advanced background service), a native module in Kotlin is created. React Native's bridge allows calling Kotlin/Java code from JavaScript ([Creating a Native Module in React Native | by Mohamed Elsdody](https://medium.com/@mohamed.ma872/creating-a-native-module-in-react-native-df1a694c89a7#:~:text=Creating%20a%20Native%20Module%20in,cannot%20be%20done%20in)). For instance, `DreamWeaverModule.kt` might expose a function `getHeartRate()` to JavaScript. The module is then registered in `MainApplication.java` so it can be invoked via `NativeModules` in React Native.

## Backend API (Node.js, Express & GraphQL)

The backend is a **Node.js** server using **Express.js** to handle HTTP requests. It exposes a **RESTful API** for standard endpoints and a **GraphQL** endpoint for optimized data querying. GraphQL allows clients (like the mobile app) to request exactly the data they need in a single request, avoiding over-fetching or multiple round trips ([Efficient Data Fetching: REST vs. GraphQL Explained](https://blog.pixelfreestudio.com/efficient-data-fetching-rest-vs-graphql-explained/#:~:text=GraphQL%20is%20a%20query%20language,endpoint%20that%20handles%20all%20requests)). This combination provides flexibility: REST endpoints are used for simple actions (e.g., health check, authentication callbacks), while GraphQL is used for complex data fetching (e.g., loading a user’s profile with goals and recommendations in one query).

**Project Structure (Backend):**
```bash
/backend
├── package.json           # Lists Node dependencies (Express, GraphQL, pg, redis, jsonwebtoken, etc.)
├── server.js              # Entry point: sets up Express app and GraphQL server
├── config/
│   ├── database.js        # Database connection (PostgreSQL client setup)
│   ├── redis.js           # Redis client configuration for caching
│   ├── auth.js            # OAuth2 and JWT configuration (keys, secrets)
├── routes/
│   ├── authRoutes.js      # REST endpoints for OAuth callback, login, etc.
│   └── apiRoutes.js       # REST endpoints for basic data (if any, e.g., health check)
├── graphql/
│   ├── schema.js          # GraphQL schema definitions (type definitions for User, Goal, etc.)
│   └── resolvers.js       # Resolver functions for GraphQL queries/mutations
├── middlewares/
│   ├── authMiddleware.js  # Express middleware for JWT verification on protected routes
│   └── errorHandler.js    # Error handling middleware
└── services/
    └── aiService.js       # Service that interfaces with the ML module (e.g., calls Python script or API)
```

**Key Features (Backend API):**

- **Express Server & GraphQL:** `server.js` initializes an Express app. It uses `apollo-server-express` (or a similar library) to mount a GraphQL endpoint at `/graphql`. The GraphQL schema might include types like `User`, `Goal`, `Insight`, with queries like `getUserProfile` and mutations like `addGoal`. This allows the mobile app to fetch nested data (e.g., user profile with all goals and latest AI insights) in one request ([Efficient Data Fetching: REST vs. GraphQL Explained](https://blog.pixelfreestudio.com/efficient-data-fetching-rest-vs-graphql-explained/#:~:text=GraphQL%20is%20a%20query%20language,endpoint%20that%20handles%20all%20requests)).
- **JWT Authentication:** Protected routes and GraphQL resolvers require a valid JWT. We issue JWTs upon login. A middleware (`authMiddleware.js`) uses a library like `express-jwt` or `jsonwebtoken` to verify tokens on incoming requests and attach the user info to the request context. For GraphQL, a context function will read the `Authorization` header (Bearer token) and verify the JWT, so resolvers know the authenticated user.
- **OAuth 2.0:** For social login or integration with other services, OAuth2 is set up (e.g., Google or Apple login for the app). The backend uses a library (like Passport.js or Grant) to handle OAuth flows. Routes in `authRoutes.js` redirect users to OAuth provider and handle the callback with an exchange of code for tokens. On success, the user is either created or fetched from the database and then a JWT is issued for the app to use.
- **Security Middleware:** The backend uses **Helmet** for setting secure HTTP headers and **cors** configured to allow the mobile app domain. All endpoints are served over HTTPS (enforced at the cloud level via TLS 1.3).

Example **GraphQL Schema** (snippet in `graphql/schema.js`):
```js
const typeDefs = `#graphql
  type User {
    id: ID!
    name: String!
    email: String!
    goals: [Goal!]!
    insights: [Insight!]!
  }
  type Goal {
    id: ID!
    userId: ID!
    title: String!
    progress: Int!
    targetDate: String
  }
  type Insight {
    id: ID!
    userId: ID!
    createdAt: String!
    recommendation: String!
    stressLevel: Int
  }
  type Query {
    getUserProfile(userId: ID!): User
    getGoals(userId: ID!): [Goal!]
  }
  type Mutation {
    addGoal(title: String!, targetDate: String): Goal
    recordInsight(userId: ID!, stressLevel: Int!, recommendation: String!): Insight
  }
`;
```

Resolvers in `graphql/resolvers.js` will fetch data from the database (or cache) and possibly call the AI service for generating recommendations when a new insight is needed.

## Database (PostgreSQL & Redis)

All persistent data is stored in a **PostgreSQL** database, chosen for its reliability and relational structure. The schema is designed to support user profiles, goal tracking data, and AI-generated insights or recommendations. For performance, **Redis** is used as an in-memory cache for frequently accessed data and session information.

**PostgreSQL Schema:** The database has tables such as:
- **users** – fields: `id (PK)`, `name`, `email`, `password_hash` (for authentication if not using external OAuth), etc.
- **goals** – fields: `id (PK)`, `user_id (FK to users)`, `title`, `description`, `progress` (e.g., percentage or completed steps), `target_date`, etc.
- **insights** – fields: `id (PK)`, `user_id (FK)`, `created_at`, `stress_level` (numeric score), `recommendation` (text from AI), etc.
- (Additional tables could include a log of user mood entries or messages if the AI does conversation.)

The project might use an **ORM** (like Sequelize or TypeORM) or query builder (like Knex) to manage schema and queries. For clarity, SQL migrations would be provided (e.g., in a `/backend/migrations` folder) to create these tables.

**Redis Caching:** A Redis instance is used to cache results and reduce load on the Postgres DB. For example, when the app requests the user’s dashboard data (goals and latest insights), the backend can cache this combined result in Redis with a key like `dashboard:{userId}`. Subsequent requests fetch from Redis (a cache hit) rather than recompute the queries, greatly improving response time. (In one scenario, caching yielded a **175x improvement** in response times for repeated requests ([Improving Node.js App Performance with Redis Caching | Better Stack Community](https://betterstack.com/community/guides/scaling-nodejs/nodejs-caching-redis/#:~:text=match%20at%20L462%20In%20my,time%20when%20using%20the%20cache)).) The `redis.js` config file connects using environment variables for host, port, and password (if any). We implement cache invalidation strategies (e.g., clear or update the cache when a new goal or insight is added).

**Example Redis Usage (in an Express middleware or resolver):** 
```js
// Pseudocode for caching a GraphQL resolver
const userId = args.userId;
const cacheKey = `dashboard:${userId}`;
const cached = await redisClient.get(cacheKey);
if (cached) {
    return JSON.parse(cached); // return cached response
}
// If not in cache, fetch fresh data
const goals = await db.Goal.findAll({ where: { userId } });
const insights = await db.Insight.findAll({ where: { userId }, order: [['created_at', 'DESC']], limit: 5 });
const responseData = { goals, insights };
await redisClient.set(cacheKey, JSON.stringify(responseData), { EX: 60 * 5 }); // cache for 5 minutes
return responseData;
```

**Connection Details:** Both Postgres and Redis connection strings/credentials are stored securely (not hard-coded). Locally, you might use a `.env` file (with entries like `DATABASE_URL` and `REDIS_URL`). In production, environment variables or a secrets manager (AWS Secrets Manager) provide these values. The Node server uses a pooling client for Postgres to efficiently handle connections.

## Machine Learning Module (TensorFlow & PyTorch AI Model)

AI Dreamweaver’s core intelligence comes from a machine learning module responsible for psychological analysis and personalized recommendations. This module is developed in **Python**, utilizing frameworks like **TensorFlow** and **PyTorch**. It is trained on mental health data aligned with **DSM-5** categories for conditions and symptoms, enabling it to profile mental state and suggest goals or coping strategies. (For example, a research dataset with DSM-5 based annotations of depressive symptoms is used to inform the model ([Are You Depressed? Analyze User Utterances to Detect Depressive Emotions Using DistilBERT](https://www.mdpi.com/2076-3417/13/10/6223#:~:text=expressed%20in%20user%20utterances,of%20depressive%20emotions%20in%20user)).)

**Functionality:**
- The AI model analyzes user inputs (such as journal entries, responses to in-app questionnaires, or usage patterns) to detect signs of stress, anxiety, or other mental health markers. It then generates **recommendations** (like suggesting a breathing exercise, reaching out to a friend, or professional help if needed).
- The model could be a text classification or sequence model that maps user text to psychological states, or a recommendation model that selects appropriate goals for the user.

**Tech Stack (ML):** During development, we use Python with TensorFlow/PyTorch for training. The trained model is saved (e.g., as a TensorFlow SavedModel or a PyTorch `.pt` file). For deployment, we have two options:
  - **Server-side Inference:** A small Python microservice (Flask or FastAPI) loads the trained model and exposes an endpoint (e.g., `/predict`) that the Node backend can call. This keeps heavy ML logic in Python, while Node just makes HTTP calls or uses a Python subprocess. The repository includes `ml_model/server.py` to start this service.
  - **On-device or TF.js:** (Optional) For certain real-time analyses (like simple sentiment detection), a lighter weight model could be converted to run on-device using TensorFlow Lite or TensorFlow.js. This would avoid network calls, but for our scope, server-side is easier to manage given the model complexity.

**Project Structure (ML Module):**
```bash
/ml_model
├── train.py            # Script to train or fine-tune the AI model on DSM-5 dataset
├── model.py            # Script defining the model architecture (if using PyTorch, e.g., a nn.Module class)
├── predict.py          # Script or module to load the trained model and run a prediction given new data
├── dsm5_dataset/       # (Not provided due to license) Placeholder for DSM-5 training data
├── model_weights.pt    # Trained PyTorch model weights (or model.h5 / saved_model directory for TensorFlow)
└── requirements.txt    # Python dependencies (TensorFlow, PyTorch, pandas, numpy, etc.)
```

**Example (Pseudo-code for inference):**
```python
# predict.py
import torch
from model import MentalHealthModel  # assume a PyTorch model class

# Load model and weights
model = MentalHealthModel()
model.load_state_dict(torch.load("model_weights.pt"))
model.eval()

def predict_profile(user_text: str) -> dict:
    """Analyze user input text and return stress level and recommendation."""
    # Preprocess the text (tokenization, etc.)
    inputs = preprocess_text(user_text)
    # Run through model
    with torch.no_grad():
        output = model(inputs)
    # Suppose output is a set of scores or categories
    stress_score = output["stress_level"].item()
    suggested_goal = derive_recommendation(output)  # some logic to pick a recommendation
    return {"stress_level": stress_score, "recommendation": suggested_goal}
```

If using TensorFlow, a similar `predict` function would load a saved Keras model and call `model.predict(...)`. The Node backend’s `aiService.js` can spawn this `predict.py` with user data or make an HTTP request to the ML service, then return the result up to the client.

**Fine-Tuning with DSM-5 Dataset:** The `train.py` script is prepared to fine-tune the model on the DSM-5 dataset or any new data. **DSM-5 dataset** in this context refers to a dataset labeled with categories of mental health symptoms consistent with the DSM-5 definitions (for example, levels of depression, anxiety, etc., for various inputs). The training script would:
1. Load and preprocess the dataset (which could be text transcripts, survey responses, etc., with labels).
2. Define a model (e.g., a text classifier or a recommendation model). We might start with a pre-trained transformer model (like BERT or DistilBERT) for text analysis, given the DSM-5 based labels ([Are You Depressed? Analyze User Utterances to Detect Depressive Emotions Using DistilBERT](https://www.mdpi.com/2076-3417/13/10/6223#:~:text=expressed%20in%20user%20utterances,of%20depressive%20emotions%20in%20user)).
3. Train the model (possibly fine-tuning a pre-trained model) and evaluate its accuracy in predicting mental health categories.
4. Save the updated model weights.

Instructions for retraining are provided in the documentation: for instance, *"Place your DSM-5 dataset files in `ml_model/dsm5_dataset/` (ensuring the format expected by `train.py`), then run `python train.py`. You may need to adjust hyperparameters or model architecture in `model.py` depending on the dataset."* After training, update the `model_weights.pt` file on the server or cloud storage so the app starts using the new model.

## Security & Compliance (HIPAA & GDPR)

Security is paramount for AI Dreamweaver, as it handles sensitive personal and health-related data. The entire system is engineered to be compliant with **HIPAA** (for health data privacy in the US) and **GDPR** (for user data protection in the EU). Key security measures include:

- **Data Encryption at Rest:** All sensitive data stored in the database is encrypted using strong algorithms (PostgreSQL can employ TDE or column-level encryption for certain fields). For instance, any field containing personal health information (PHI) can be additionally encrypted with **AES-256** before storing ([HIPAA Encryption: Protect ePHI Protected Health Information](https://www.kiteworks.com/hipaa-compliance/hipaa-encryption/#:~:text=As%20a%20rule%2C%20a%20secure,email%20directly%2C%20Kiteworks%20sends%20a)). If using AWS RDS for Postgres, the storage is encrypted at rest with AES-256 by enabling that option. Similarly, any files (like user diaries or audio logs) stored on S3 are set to use Server-Side Encryption (SSE-S3 or SSE-KMS with AWS KMS keys).

- **Data Encryption in Transit:** All network communication is protected with the latest TLS (TLS 1.3) so that data between the mobile app, backend, and database is encrypted in transit ([HIPAA Encryption: Protect ePHI Protected Health Information](https://www.kiteworks.com/hipaa-compliance/hipaa-encryption/#:~:text=As%20a%20rule%2C%20a%20secure,email%20directly%2C%20Kiteworks%20sends%20a)). The mobile app communicates with the backend over HTTPS only. We use HSTS headers to enforce secure connections.

- **Authentication & Access Control:** JWTs are signed with strong secrets (256-bit keys) and have short lifetimes. We use refresh tokens or OAuth flows to renew sessions securely. Role-based access controls are in place on the backend (though in this app, most users are end-users, but if there's an admin interface, those endpoints are protected and only accessible with proper role claims in JWT). All access to data is scoped: e.g., one user cannot access another’s goals or insights due to checks in every query (the server uses the JWT’s user ID to filter database queries).

- **Protecting PHI and Personal Data:** For HIPAA compliance, we minimize stored PHI. For example, instead of storing raw journal text that might contain identifiable info, the system might store derived scores or anonymized tokens. Under GDPR, users have the right to delete their data, so we provide a mechanism for users to request account deletion, which will remove or anonymize their records in the database.

- **Audit Logging:** Access to sensitive operations (like if an admin feature exists or model outputs that could be considered medical advice) is logged. We maintain logs of when user data is accessed or modified, which is useful for compliance audits.

- **Compliance Measures:** We ensure Business Associate Agreements (BAA) with cloud providers if required (AWS is HIPAA compliant and will sign a BAA for using services like RDS, S3 in a compliant manner). The system’s design follows the principle of least privilege: each microservice or module has only the minimum access it needs (e.g., the ML module might not directly access the database, it only sees necessary input and produces output).

To summarize, **AES-256 encryption for data at rest and TLS for data in transit are standard** ([HIPAA Encryption: Protect ePHI Protected Health Information](https://www.kiteworks.com/hipaa-compliance/hipaa-encryption/#:~:text=As%20a%20rule%2C%20a%20secure,email%20directly%2C%20Kiteworks%20sends%20a)). Additionally, all secrets (JWT signing keys, OAuth client secrets, database passwords) are stored securely — in development via .env files (excluded from version control), and in production using environment variables or secret management services. The repository’s documentation includes a **Security.md** outlining how to configure these and recommendations for maintaining compliance (for example, regular security audits and using tools like OWASP ZAP to test the API for vulnerabilities).

## Cloud Deployment (AWS Architecture)

The project is set up to be deployed on **AWS** using free-tier eligible services (or other cost-effective solutions if needed). The architecture is designed for scalability and cost-awareness:

- **Compute (Backend Hosting):** The Node.js/Express backend can run on an **EC2** instance (t2.micro is free-tier eligible) or AWS **Elastic Beanstalk** for easier management. Elastic Beanstalk can manage environment configuration and auto-scaling for us. Alternatively, one could use **AWS Lambda** with API Gateway for a serverless deployment of the REST/GraphQL API (though WebSockets for GraphQL subscriptions would need additional setup). For simplicity, an EC2 or Beanstalk deployment is recommended, with auto-scaling groups configured to spawn more instances if load increases.

- **Database:** **AWS RDS (PostgreSQL)** is used for the Postgres database. In AWS free tier, one can run a small RDS instance (db.t2.micro) for free for 12 months. The `database.js` config will use the RDS endpoint and credentials from environment variables. Ensure to enable encryption and regular snapshots for backups. For development or a cost-free alternative, one can use a local Postgres or a Docker container.

- **Cache:** **Amazon ElastiCache (Redis)** can be used for a managed Redis service. However, ElastiCache is not free-tier. As a free alternative, one might run Redis on the same EC2 instance as the backend (for low traffic scenarios) or use a small Dockerized Redis. The config allows toggling the cache on/off or pointing to an external Redis service.

- **Static Storage:** **AWS S3** is used for storing any user-uploaded content (for example, if users upload an avatar, or if the app saves audio journal entries) and for storing the ML model files. S3 is cheap and scalable; plus, the **DSM-5 dataset** (if licensed for use) and trained model can be stored in a private S3 bucket so that the ML service or the backend can load the model at startup. S3 is configured with encryption at rest and appropriate bucket policies (only the app or admin can access, not public).

- **Content Delivery (CDN):** **AWS CloudFront** is set up as a CDN in front of the S3 bucket to deliver any static content globally with low latency. If the mobile app needs to download larger files (like meditation videos or AR exercises), CloudFront ensures faster delivery. CloudFront can also be used as a proxy to the API for global caching of certain GET requests, though for a dynamic API this is less common.

- **Scaling & High Availability:** The deployment uses an **Auto Scaling Group** for the EC2 instances running the backend. We configure a minimum of 1 instance (to stay within free tier) and allow scaling up to, say, 2-3 instances on high load. A load balancer (ELB/ALB) distributes traffic between instances. For the database, we could use a Multi-AZ RDS for failover (though not free-tier, it's a consideration for production). The stateless nature of the Node server (since session info is in JWT and cache) means we can easily scale horizontally.

- **Cost-Effective Alternatives:** If AWS costs are a concern beyond the free tier, the documentation also mentions alternatives like using **Heroku** (which offers free-tier hobby plans for Node and a free Postgres database for development) or **DigitalOcean** (which has low-cost droplets and managed databases). The codebase is not tied to AWS specifically; environment variables control the connection strings, so deploying on another platform is possible with minimal changes.

**Deployment Process:** The repository includes Infrastructure as Code (optional) to automate setup:
  - A Terraform or CloudFormation template (in `/infrastructure`) could be provided to create the EC2, RDS, etc., but if not, the README’s deployment guide shows how to manually provision resources.
  - Steps include: creating an RDS instance, an S3 bucket, setting environment variables (AWS access keys, DB URL, etc.) on the EC2 or Beanstalk environment, and then deploying the Node app (via `npm start` or Docker image).
  - The React Native app, being mobile, is distributed via app stores rather than through cloud – but we might have a **CodePush** or OTA update setup for pushing JS bundle updates via Microsoft App Center, which can be mentioned in documentation for convenience (not strictly required).

## Third-Party Integrations (Bubble.io, Make.com, Voiceflow)

To accelerate development and enhance functionality, AI Dreamweaver integrates with several third-party platforms:

- **Bubble.io (No-Code Frontend Prototyping):** Bubble is a no-code development platform that enables quick creation of web interfaces. We use Bubble.io for designing initial UI components and flows, which helped validate the concept early on. For example, a simple web dashboard for users or a landing page for the project can be built on Bubble without coding, using drag-and-drop UI elements ([Bubble’s new Component Library makes it easier to build UI - Announcements - Bubble Forum](https://forum.bubble.io/t/bubble-s-new-component-library-makes-it-easier-to-build-ui/232526#:~:text=Starting%20from%20a%20blank%20page,more%21%20Try%20it%20out%20yourself)). While the main mobile app is in React Native, Bubble could be used to create a companion web app or to embed certain dynamic forms via WebView. In our setup, Bubble might run automation for onboarding (e.g., a Bubble workflow that triggers when a new user signs up, sending a welcome email via SendGrid). Over time, critical functionality is implemented natively in the mobile app or backend, but Bubble remains useful for quick iterations or marketing pages.

- **Make.com (Workflow Automation):** Make.com (formerly Integromat) is a visual automation tool that connects various apps and services. We integrate Make.com to handle certain **workflow automations** without writing custom code ([Make Integration | Workflow Automation | Make](https://www.make.com/en/integrations/make#:~:text=Connect%20Make%20integrations)). For instance:
  - When a user achieves a goal in the app, we trigger a Make scenario (via a webhook or by Make polling an endpoint) to log this event in a Google Sheets (for internal tracking) and maybe post a congratulatory message to a community forum or send a celebratory email.
  - If the user’s stress level (detected by the ML module) goes above a threshold, a Make workflow could automatically create a support ticket or alert a human coach via email/SMS.
  - Make.com’s integration library (2000+ apps) ([Make Integration | Workflow Automation | Make](https://www.make.com/en/integrations/make#:~:text=Connect%20Make%20integrations)) means we can connect to services like Google Calendar (to add reminders for goals), Twilio (to send SMS reminders or emergency messages), etc., with minimal effort. The `make.com` workflows are configured on the Make platform, but the documentation includes how to set up the triggers (for example, an HTTPS endpoint on our backend that Make calls into, or vice versa).
  
- **Voiceflow (Conversational AI):** Voiceflow is a platform for building conversational agents (chatbots and voice assistants) without coding. We leverage Voiceflow to create an **AI conversational experience** within AI Dreamweaver ([How To Build an AI to Voice Chat With [3 Steps]](https://www.voiceflow.com/blog/ai-voice-chat#:~:text=Voiceflow%20allows%20you%20to%20create%2C,chatbot%20in%203%20easy%20steps)). This could be used for a guided meditation dialog, or a friendly chatbot that checks in on the user’s mood. The Voiceflow project (accessible on voiceflow.com) contains the dialogue logic – for example, it might ask the user how they feel today, and based on the answer, branch into different responses or exercises. We integrate Voiceflow in two possible ways:
  - **In-App Chatbot:** Using Voiceflow’s API (VAPI), the mobile app sends the user's message to Voiceflow and gets the chatbot’s reply, which we display in the app UI. This requires an internet connection but offloads NLP heavy lifting to Voiceflow’s managed service.
  - **Voice Assistant:** If using voice, Voiceflow can manage the speech recognition and response generation. The app could use the device’s microphone to capture audio, send it to Voiceflow (which might internally use ASR, NLU, etc.), and then get a text or audio response. Voiceflow allows creating these multi-turn conversations visually, and we just integrate the final agent.
  
  Voiceflow was chosen to speed up development of the conversational aspect – it allows creating and iterating on conversation flows without redeploying code ([How To Build an AI to Voice Chat With [3 Steps]](https://www.voiceflow.com/blog/ai-voice-chat#:~:text=Voiceflow%20allows%20you%20to%20create%2C,chatbot%20in%203%20easy%20steps)). This is especially useful for tweaking the wording of prompts or adding new dialogues (like a motivational quote of the day). Our codebase includes references on how to connect to Voiceflow:
    - For example, `services/voiceflowService.js` might hold the API calls to Voiceflow’s runtime API, with an API key. 
    - If Voiceflow provides an SDK or a web embed, we could integrate that into the app. (Voiceflow can deploy to platforms like Alexa or Google Assistant; for our case we use it as a backend service for chat.)

Each of these integrations (Bubble, Make, Voiceflow) is optional in production but can greatly enhance capabilities. We have included instructions to set up API keys or webhooks as needed. (For instance, obtaining a Voiceflow API key and agent ID, setting up Make webhooks to interface with our API, etc.) The codebase contains placeholders or configuration points for these: e.g., a `.env` value for `VOICEFLOW_API_KEY`, and a section in the README on how to connect a Make.com scenario to our system’s events.

## CI/CD Pipeline (GitHub Actions & Blue-Green Deployments)

Continuous integration and deployment are configured to ensure that any code changes are tested and deployed with minimal downtime:

- **GitHub Actions:** The repository uses GitHub Actions workflows for automating tests and deployments. Whenever code is pushed or a pull request is made, an Action is triggered to run the backend tests (e.g., `npm test` for Node, perhaps using Jest) and mobile app tests (if any, e.g., unit tests for functions or Detox tests for RN). Only on a successful test run will the deployment steps proceed.

  We have separate workflows defined, for example:
  - `backend-ci.yml` – runs on every push, testing the Node backend (and possibly building a Docker image).
  - `mobile-ci.yml` – runs linting/build for the React Native app (though actual mobile deployment to app stores is manual, we ensure the app builds without errors on CI).
  - `deploy.yml` – triggered on push to `main` or a tag, which handles deployment (this could be combined with backend-ci using job conditions).

- **Blue-Green Deployment Strategy:** To achieve zero/minimal downtime, we implement a blue-green deployment for the backend API. In AWS terms, this could mean maintaining two environments (for example, two Elastic Beanstalk environments or two sets of EC2 instances behind a load balancer) ([Top 40+ Jenkins Interview Questions and Answers [2025]](https://www.fdaytalk.com/jenkins-interview-questions-and-answers/#:~:text=Blue,To%20implement%20this%20using%20Jenkins)). The process is:
  1. **Blue Environment (Current Production)** – running the current stable version.
  2. **Green Environment (Staging/New Release)** – where the new version will be deployed.
  
  When a new version is ready:
  - The CI pipeline deploys the new version to the **Green** environment (without affecting users on Blue). For instance, if using Elastic Beanstalk, we deploy to a separate EB environment configured identically to production.
  - We run health checks and maybe a small subset of automated tests against Green to ensure it's working with the new code.
  - Then, we **switch traffic** to Green. If using Route53, we update DNS or if using a load balancer, we swap the target groups behind the LB so that Green gets the production traffic ([Top 40+ Jenkins Interview Questions and Answers [2025]](https://www.fdaytalk.com/jenkins-interview-questions-and-answers/#:~:text=Blue,To%20implement%20this%20using%20Jenkins)). Elastic Beanstalk has a feature to swap environment CNAMEs which effectively directs users to the new environment.
  - Monitor the Green (now live) environment. If any critical issue is detected, we can quickly rollback by switching traffic back to Blue, since it’s still running the previous version (this rollback can even be automated in the GitHub Action if a health check fails).
  - Once confident, we can terminate or recycle the Blue environment (or keep it idle as the new staging for the next deployment).

  The GitHub Actions workflow uses AWS CLI or specialized actions to perform these steps. For example, an Action might use a job to deploy to Beanstalk Green, then another job to swap environment URLs. We reference an open-source GitHub Action for blue-green deployments on AWS for guidance ([Top 40+ Jenkins Interview Questions and Answers [2025]](https://www.fdaytalk.com/jenkins-interview-questions-and-answers/#:~:text=Blue,To%20implement%20this%20using%20Jenkins)).

- **Testing and Linting:** The CI pipeline also ensures code quality. We run ESLint/Prettier on the JS code and perhaps Flake8 or similar on Python code. Any issues will fail the build. We also have basic tests for the ML module (ensuring the model can load and produce an output given sample input).

- **App Store Deployment (Future):** While not fully automated (because mobile app store deployments often require manual review), the CI could integrate with tools like Fastlane to simplify preparing builds. For example, a GitHub Action might create an Android APK/Bundle on each release. This artifact can then be manually uploaded to Google Play Console. Similar for iOS, it could trigger an App Center build.

By using CI/CD, developers can push changes with confidence that they will be automatically tested and deployed, and the blue-green setup ensures users never experience more than a momentary switch with no downtime. This approach **minimizes risk during updates by keeping two production environments (current “Blue” and new “Green”)** ([Top 40+ Jenkins Interview Questions and Answers [2025]](https://www.fdaytalk.com/jenkins-interview-questions-and-answers/#:~:text=Blue,To%20implement%20this%20using%20Jenkins)).

## Documentation & Setup Guide

This project comes with comprehensive documentation to assist with setup, deployment, and maintenance:

### Code Structure Overview

The repository is organized into multiple top-level folders as described above. Here’s a recap of the structure:

```bash
DreamWeaverAI/
├── mobile/            # React Native mobile app code
├── backend/           # Node.js backend API code
├── ml_model/          # Machine learning model training and inference code
├── infrastructure/    # (Optional) Infrastructure as code scripts (Terraform/CloudFormation)
├── .github/workflows/ # CI/CD pipeline definitions for GitHub Actions
└── README.md          # Primary documentation and setup instructions
```

Each folder contains its own README or notes:
- The **mobile** folder has a README for how to run the app, requirements (Node, Watchman, Xcode/Android Studio for building).
- The **backend** folder has instructions on environment variables and how to launch in dev and prod.
- The **ml_model** folder’s README explains how to set up a Python environment (e.g., using `virtualenv` or Conda), and how to train the model.

### Setup and Deployment Steps

**1. Prerequisites:** Ensure you have Node.js (>=14.x), Python (for ML, >=3.8), and Postgres installed (for local dev). Also, install the React Native CLI and Android/iOS build tools if you plan to run the app on a device/emulator.

**2. Cloning the Project:** Clone the repository from version control (GitHub). Install dependencies:
   - For the backend: `cd backend && npm install`.
   - For the mobile app: `cd mobile && npm install`.
   - For the ML module: `cd ml_model && pip install -r requirements.txt` (possibly in a Python virtual environment).

**3. Environment Configuration:** Copy the provided `.env.example` files to `.env` in the respective folders and fill in credentials.
   - In `backend/.env`: set `DATABASE_URL` (Postgres connection string), `REDIS_URL`, `JWT_SECRET`, `OAUTH_CLIENT_ID`/`SECRET` (for Google etc.), and any other needed config (like `VOICEFLOW_API_KEY` if using Voiceflow, `AWS_ACCESS_KEY` & `SECRET` for AWS SDK if it needs to upload to S3, etc.).
   - In `mobile/`, configure any API endpoints or keys in a config file (e.g., `config.js` for API base URL, which in dev might be `http://localhost:3000` and in prod the AWS endpoint).
   - In `ml_model/.env` (if needed): paths to dataset, any ML hyperparameters, etc.

**4. Running Locally:** Start each component:
   - Start Postgres locally and run the migration SQL scripts to create tables (or use an ORM’s migration tool).
   - (Optional) Start a local Redis server if caching is to be tested.
   - **Backend:** `npm run dev` (which might use nodemon for hot-reload). The GraphQL playground would be at `http://localhost:3000/graphql`.
   - **ML Module:** If using a separate service, `python ml_model/server.py` to start the ML API on a local port (ensure the backend knows this endpoint). If the backend calls a script directly, this step isn’t needed.
   - **Mobile App:** Run `npm start` (or `npx react-native start`) to launch the Metro bundler. Then run `npx react-native run-android` or `run-ios` to launch the app in an emulator. The app should connect to the locally running backend (if configured correctly in the API client module).

**5. Running Tests:** Execute `npm test` in `backend/` to run the backend test suite. Similarly, any unit tests for the mobile app can be run with `npm test` in `mobile/` if configured. Ensure all tests pass before deployment.

**6. Deployment to AWS:** Follow these summarized steps (detailed instructions are in the README Deployment section):
   - Set up AWS resources: using the AWS Console or CLI, create an RDS Postgres instance and an S3 bucket. (If using Elastic Beanstalk, create an environment for Node.js.)
   - In AWS Secrets Manager or Parameter Store, store sensitive config (DB password, etc.) and configure the EC2/Beanstalk to load those into environment variables.
   - For the backend, build a production bundle: e.g., `npm run build` (if we transpile or bundle) or simply zip the project and use EB CLI to deploy. Alternatively, build a Docker image (Dockerfile provided) and push to ECR, then use that in AWS (Elastic Beanstalk can directly use Docker images).
   - Migrate the database on first deploy: run the migration SQL or the ORM migration command on the RDS instance.
   - Update the mobile app to point to the production backend URL and build release versions for Android/iOS to publish on app stores.

The documentation includes screenshots and examples for some steps, like setting environment variables on Elastic Beanstalk, or sample AWS CLI commands to upload files to S3.

**7. Fine-Tuning the AI Model:** The README outlines how to retrain the model:
   - Obtain or prepare a dataset (e.g., a CSV with user text and labels like stress level or recommended action).
   - Place the data in `ml_model/dsm5_dataset/` and adjust any paths in `train.py`.
   - Run `python train.py`. This will output new model weights (and possibly some metrics).
   - Test the new model with `predict.py` on some sample inputs.
   - If satisfied, update the deployed model: for instance, upload the new `model_weights.pt` to the server or S3. The ML service might load the model at startup from a known location (so replacing the file and restarting the service is enough). Always keep a backup of the previous model in case rollback is needed.
   - The model training can be resource-intensive, so for large datasets you might use a machine with a GPU or use a cloud service (like an EC2 GPU instance or a SageMaker notebook). This is beyond the scope of the deployable package, but the code is provided to facilitate it.

### Configuration and Security Notes

- **Dependency Management:** All dependencies are pinned to specific versions in `package.json` and `requirements.txt` to ensure reproducible builds. We recommend running `npm audit` and `pip audit` periodically to catch vulnerabilities.
- **Monitoring & Logging:** The codebase integrates basic logging (using Winston or a similar library for Node). For production, consider using AWS CloudWatch logs or an external service (Datadog, etc.) to aggregate logs. Health check endpoints (`/health`) are provided for AWS to monitor instance health.
- **Hosting Recommendations:** For a small-scale deployment within free-tier:
  - 1 EC2 t2.micro for backend + maybe the ML service (could be same instance or separate if resources allow).
  - 1 RDS micro instance for Postgres.
  - (Optional) 1 t2.micro running a Redis (or use a small Elasticache if affordable).
  - CloudFront in front of the S3 (free-tier includes 50 GB egress which is plenty for starting).
  - Use AWS SES or a third-party service if the app needs to send emails (Make.com can also send emails).
  
  As usage grows, you can move the ML service to a separate instance or container (to allow using more CPU/GPU), and upgrade the database size or add read replicas.

- **Security Settings:** We enforce secure practices:
  - HTTP -> HTTPS redirects are enabled (either via load balancer or Express middleware).
  - CORS is locked down to known domains (e.g., the mobile app’s packager domain or website domain).
  - Rate limiting middleware on the API to prevent abuse (especially on auth endpoints).
  - All default passwords or secrets are changed before production. No secret is committed in the repo.

- **GDPR Considerations:** The documentation also explains how a user's data can be exported or deleted on request. Administrators can run a script to anonymize a user (replace personal identifiers with random IDs but keep data for aggregate analysis, if needed).
