# Bucket List App

A full-stack web application for creating and managing personal bucket list items with image uploads. Built as a learning project to explore AWS Amplify Gen2 capabilities for rapid full-stack development.

## 🎯 Project Purpose

This is a sample application designed to understand and demonstrate:
- **AWS Amplify Gen2** for full-stack development
- **Backend deployment** with automatic infrastructure setup
- **Database management** with secure, user-specific data access
- **Authentication** and authorization patterns
- **File storage** with S3 integration
- **Frontend-backend integration** with type-safe APIs

---

## 🛠️ Tech Stack

### **Frontend**
- **React** 19.2.6 - UI library
- **Vite** 8.0.12 - Fast build tool and dev server
- **AWS Amplify UI** 6.15.3 - Pre-built UI components for AWS services
- **TypeScript** 5.9.3 - Type safety
- **ESLint** 10.3.0 - Code quality

### **Backend (AWS Amplify Gen2)**
- **AWS AppSync** - GraphQL API with real-time capabilities
- **Amazon DynamoDB** - NoSQL database for bucket list items
- **Amazon S3** - Cloud storage for images
- **Amazon Cognito** - User authentication and authorization
- **AWS CDK** - Infrastructure as Code for backend definition

### **Development Tools**
- **AWS Amplify Backend CLI** 1.8.2 - Local backend development
- **TypeScript** - Type-safe backend configuration
- **tsx** 4.21.0 - TypeScript execution for backend code

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Frontend (React + Vite)                  │
│  - React Components                                           │
│  - AWS Amplify UI Components                                  │
│  - Local State Management                                     │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
        ┌───────────────────────────────┐
        │   AWS Amplify Data Client     │
        │   (GraphQL API Client)        │
        └───────────────────┬───────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
    ┌─────────┐         ┌─────────┐       ┌─────────┐
    │ AppSync │         │ Cognito │       │    S3   │
    │(GraphQL)│         │(Auth)   │       │(Storage)│
    └────┬────┘         └────┬────┘       └────┬────┘
         │                   │                 │
         ▼                   ▼                 ▼
    ┌─────────────────────────────────────────────┐
    │        AWS Backend Infrastructure           │
    │  - DynamoDB (Database)                      │
    │  - Lambda (API Logic)                       │
    │  - IAM (Authorization)                      │
    └─────────────────────────────────────────────┘
```

---

## ✨ Features

### **User Authentication**
- Email-based sign-up and login via Amazon Cognito
- Secure password management
- Session handling

### **Bucket List Management**
- **Create** new bucket list items with title and description
- **Add images** to bucket list items via S3 upload
- **View** your bucket list items
- **Update** item details
- **Delete** items from your list

### **Data Security**
- **Owner-based authorization** - Users can only access their own items
- **Role-based access control** - Image storage is restricted by user identity
- **GraphQL API** - Type-safe, efficient data fetching

---

## 📁 Project Structure

```
bucketlistapp/
├── amplify/                          # Backend definition (Amplify Gen2)
│   ├── backend.ts                   # Main backend configuration
│   ├── auth/
│   │   └── resource.ts              # Cognito auth setup (email login)
│   ├── data/
│   │   └── resource.ts              # GraphQL schema & DynamoDB config
│   └── storage/
│       └── resource.ts              # S3 storage setup for images
│
├── src/                              # Frontend source code
│   ├── main.jsx                     # App entry point
│   ├── App.jsx                      # Main React component
│   ├── App.css                      # Styles
│   ├── index.css                    # Global styles
│   └── assets/                      # Static assets
│
├── public/                           # Static files served as-is
├── package.json                      # Dependencies & scripts
├── vite.config.js                   # Vite configuration
├── eslint.config.js                 # Code linting rules
├── amplify_outputs.json             # Auto-generated backend outputs
└── README.md                         # This file
```

---

## 🚀 Getting Started

### **Prerequisites**
- **Node.js** 18+ and npm (comes with Node.js)
- **AWS Account** (free tier eligible)
- **Git** (optional, for version control)

### **Installation**

1. **Clone or download** the project:
   ```bash
   cd bucketlistapp
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Set up the Amplify backend** (creates AWS resources):
   ```bash
   npx amplify sandbox
   ```
   This command:
   - Authenticates with your AWS account
   - Creates a temporary backend environment
   - Generates `amplify_outputs.json` for frontend connection
   - Keeps backend running locally for development

### **Running Locally**

Start the development server:
```bash
npm run dev
```

- Opens at `http://localhost:5173`
- Hot Module Replacement (HMR) enabled - changes reflect instantly
- Backend automatically synced via Amplify sandbox

### **Building for Production**

Create an optimized production build:
```bash
npm run build
```

Output: `dist/` folder with minified files ready for deployment

### **Linting**

Check code quality:
```bash
npm run lint
```

Preview production build locally:
```bash
npm run preview
```

---

## 🔄 How It Works

### **1. Authentication Flow**

```
User → Sign Up/Login Form → AWS Cognito → JWT Token → Logged In
                                 ↓
                          Verify Email
```

- Users sign up with email and password
- Cognito sends verification email
- Upon verification, users can log in
- JWT token stored and used for API authentication

### **2. Data Flow: Creating a Bucket Item**

```
React Component
    ↓
User Input (title, description, image)
    ↓
Amplify Data Client
    ↓
GraphQL Mutation → AppSync
    ↓
Lambda → DynamoDB (save with owner ID)
S3 (save image)
    ↓
Response → React Component → UI Update
```

### **3. Authorization**

**BucketItem Model:**
```typescript
.authorization((allow) => [
  allow.owner()  // Only the item owner can read/write/delete
])
```

**Storage Access:**
```typescript
"media/{entity_id}/*": [
  allow.entity("identity").to(["read", "write", "delete"])
]
```

---

## 📊 API Documentation

### **GraphQL Schema**

#### **BucketItem Model**
```graphql
type BucketItem {
  id: ID!                  # Unique identifier (auto-generated)
  title: String!           # Item title
  description: String      # Detailed description
  image: String            # S3 URL to the bucket item image
  owner: String!          # Cognito user ID (auto-set)
  createdAt: AWSDateTime   # Creation timestamp
  updatedAt: AWSDateTime   # Last update timestamp
}
```

#### **Available Operations**

**Create a bucket item:**
```graphql
mutation CreateBucketItem($input: CreateBucketItemInput!) {
  createBucketItem(input: $input) {
    id
    title
    description
    image
  }
}
```

**Fetch your items:**
```graphql
query ListBucketItems {
  listBucketItems {
    items {
      id
      title
      description
      image
    }
  }
}
```

**Update an item:**
```graphql
mutation UpdateBucketItem($input: UpdateBucketItemInput!) {
  updateBucketItem(input: $input) {
    id
    title
  }
}
```

**Delete an item:**
```graphql
mutation DeleteBucketItem($input: DeleteBucketItemInput!) {
  deleteBucketItem(input: $input) {
    id
  }
}
```

### **Image Upload**

Images are stored in S3 bucket: `amplifyBucketTrackerImages`

Path pattern: `media/{user_id}/`

Only authenticated users can upload/access their own images.

---

## 🌐 Deployment

### **Local Development**
```bash
npx amplify sandbox
npm run dev
```

### **Deploy to AWS (Production)**

1. **Create Amplify App** (first time):
   ```bash
   npx amplify deploy
   ```

2. **Subsequent deployments**:
   ```bash
   npx amplify deploy
   ```

This will:
- Build frontend (`npm run build`)
- Deploy to AWS S3 + CloudFront
- Create backend infrastructure on AWS
- Provide production URL

### **Deployment Outputs**
- Frontend hosted on **CloudFront** (CDN)
- Backend API endpoints generated automatically
- Database and storage ready for production scale
- SSL/TLS certificates auto-managed

---

## 📚 Learning Resources

### **AWS Amplify Documentation**
- [Amplify Gen2 Overview](https://docs.amplify.aws/gen2/)
- [Authentication Setup](https://docs.amplify.aws/gen2/build-a-backend/auth/)
- [Database & API (Data)](https://docs.amplify.aws/gen2/build-a-backend/data/)
- [File Storage](https://docs.amplify.aws/gen2/build-a-backend/storage/)
- [Frontend Integration](https://docs.amplify.aws/gen2/build-a-frontend/)

### **Related AWS Services**
- [AppSync GraphQL API](https://docs.aws.amazon.com/appsync/)
- [Amazon Cognito](https://docs.aws.amazon.com/cognito/)
- [DynamoDB](https://docs.aws.amazon.com/dynamodb/)
- [Amazon S3](https://docs.aws.amazon.com/s3/)

### **Tools & Technologies**
- [React Documentation](https://react.dev/)
- [Vite Guide](https://vitejs.dev/guide/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

---

## 🎓 Key Learning Outcomes

After working through this project, you'll understand:
1. ✅ How to set up a full-stack app with Amplify Gen2
2. ✅ Infrastructure as Code (IaC) with AWS CDK
3. ✅ GraphQL APIs and real-time data synchronization
4. ✅ User authentication and authorization patterns
5. ✅ Building scalable, serverless applications
6. ✅ Type-safe frontend-backend integration
7. ✅ Deploying to production on AWS

---

## 💡 Next Steps & Enhancement Ideas

### **Frontend Enhancements**
- [ ] Add item categories/tags
- [ ] Search and filter functionality
- [ ] Dark mode toggle
- [ ] Responsive mobile design
- [ ] Item priority levels

### **Backend Enhancements**
- [ ] Add item completion status (boolean field)
- [ ] User profile management
- [ ] Sharing bucket lists with others (authorization)
- [ ] Real-time collaboration
- [ ] Timestamps for when items were added/completed

### **Testing**
- [ ] Unit tests with Vitest
- [ ] E2E tests with Cypress/Playwright
- [ ] Backend tests for authorization rules

### **DevOps & Monitoring**
- [ ] CloudWatch logs and monitoring
- [ ] Error tracking with Sentry
- [ ] Performance metrics
- [ ] API usage analytics

---

## 🤝 Contributing

This is a learning project! Feel free to:
- Experiment with the features
- Add new functionality
- Test edge cases
- Deploy your own version to AWS
- Share improvements and learnings

---

## 📝 License

This project is provided for educational purposes.

---

## 🔗 Quick Links

- [Amplify Sandbox Command](https://docs.amplify.aws/gen2/deploy-and-host/sandbox-environments/)
- [Amplify Deployment Guide](https://docs.amplify.aws/gen2/deploy-and-host/fullstack-branching/)
- [AWS Free Tier](https://aws.amazon.com/free/)

---

**Happy learning! 🚀**
