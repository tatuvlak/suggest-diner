# Suggest Diner - Technical Architecture

## 1. System Architecture Diagram

### 1.1 Component Overview

```
┌────────────────────────────────────────────────────────────────────┐
│                         MOBILE APP LAYER                            │
│                        (Android - Kotlin)                           │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                  PRESENTATION LAYER                           │ │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐             │ │
│  │  │   Jetpack  │  │  Activity/ │  │  Navigation│             │ │
│  │  │  Compose   │  │  Fragment  │  │  Component │             │ │
│  │  │    UI      │  │            │  │            │             │ │
│  │  └────────────┘  └────────────┘  └────────────┘             │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                           │                                         │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                    VIEWMODEL LAYER                            │ │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐             │ │
│  │  │   Recipe   │  │   Pantry   │  │   Family   │             │ │
│  │  │ ViewModel  │  │ ViewModel  │  │ ViewModel  │             │ │
│  │  └────────────┘  └────────────┘  └────────────┘             │ │
│  │          │              │              │                       │ │
│  │          └──────────────┴──────────────┘                       │ │
│  │                         │                                       │ │
│  │               (Kotlin Coroutines + Flow)                       │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                           │                                         │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │                   REPOSITORY LAYER                            │ │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐             │ │
│  │  │   Recipe   │  │   Pantry   │  │    User    │             │ │
│  │  │ Repository │  │ Repository │  │ Repository │             │ │
│  │  └────────────┘  └────────────┘  └────────────┘             │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                │                                    │               │
│         ┌──────┴───────┐                    ┌─────┴──────┐        │
│         ▼              ▼                    ▼            ▼        │
│  ┌─────────────┐ ┌──────────┐       ┌──────────┐ ┌──────────┐   │
│  │  Room DB    │ │  Shared  │       │  Retrofit│ │  Camera  │   │
│  │  (SQLite)   │ │   Prefs  │       │  Client  │ │  ML Kit  │   │
│  │  - Recipes  │ │          │       │          │ │          │   │
│  │  - Pantry   │ │          │       │          │ │          │   │
│  │  - Cache    │ │          │       │          │ │          │   │
│  └─────────────┘ └──────────┘       └──────────┘ └──────────┘   │
│                                           │                        │
└───────────────────────────────────────────┼────────────────────────┘
                                            │
                                            │ HTTPS/REST
                                            │
┌───────────────────────────────────────────┼────────────────────────┐
│                          BACKEND SERVICES LAYER                     │
├───────────────────────────────────────────┼────────────────────────┤
                                            ▼
                                  ┌──────────────────┐
                                  │   API Gateway    │
                                  │   (Kong/AWS)     │
                                  │  - Auth          │
                                  │  - Rate Limiting │
                                  │  - Load Balancer │
                                  └──────────────────┘
                                            │
                   ┌────────────────────────┼────────────────────────┐
                   │                        │                        │
                   ▼                        ▼                        ▼
         ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
         │   Recipe Service │    │   User Service   │    │  LLM Service     │
         │                  │    │                  │    │                  │
         │  - CRUD recipes  │    │  - Authentication│    │  - GPT-4 API     │
         │  - Search        │    │  - Profiles      │    │  - Claude API    │
         │  - Ratings       │    │  - Preferences   │    │  - Prompt Mgmt   │
         │                  │    │  - Families      │    │  - Response Cache│
         └──────────────────┘    └──────────────────┘    └──────────────────┘
                   │                        │                        │
                   │                        │                        │
         ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
         │  Pantry Service  │    │   Poll Service   │    │  ML Model Service│
         │                  │    │                  │    │                  │
         │  - Inventory     │    │  - Create polls  │    │  - Preference    │
         │  - Expiry Track  │    │  - Vote tracking │    │    learning      │
         │  - Suggestions   │    │  - Results       │    │  - Embeddings    │
         │                  │    │                  │    │  - Ranking       │
         └──────────────────┘    └──────────────────┘    └──────────────────┘
                   │                        │                        │
                   └────────────────────────┴────────────────────────┘
                                            │
                                            ▼
                              ┌──────────────────────────┐
                              │    MESSAGE QUEUE         │
                              │    (RabbitMQ/Kafka)      │
                              │  - Async processing      │
                              │  - Event streaming       │
                              └──────────────────────────┘
                                            │
                   ┌────────────────────────┼────────────────────────┐
                   │                        │                        │
                   ▼                        ▼                        ▼
         ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
         │  Notification    │    │  Analytics       │    │  Background Jobs │
         │    Service       │    │   Service        │    │                  │
         │                  │    │                  │    │  - Expiry alerts │
         │  - FCM           │    │  - User metrics  │    │  - ML training   │
         │  - Email         │    │  - Recipe stats  │    │  - Data cleanup  │
         │  - Push          │    │  - A/B testing   │    │                  │
         └──────────────────┘    └──────────────────┘    └──────────────────┘
                                            │
                                            ▼
                              ┌──────────────────────────┐
                              │     DATA LAYER           │
                              ├──────────────────────────┤
                              │  PostgreSQL (Primary)    │
                              │  - Users, Recipes        │
                              │  - Families, Votes       │
                              │  - Pantry items          │
                              ├──────────────────────────┤
                              │  MongoDB (Documents)     │
                              │  - LLM responses         │
                              │  - User interactions     │
                              │  - Analytics logs        │
                              ├──────────────────────────┤
                              │  Redis (Cache)           │
                              │  - Session data          │
                              │  - Rate limiting         │
                              │  - Real-time polls       │
                              ├──────────────────────────┤
                              │  S3/Cloud Storage        │
                              │  - Recipe images         │
                              │  - User uploads          │
                              └──────────────────────────┘
                                            │
┌───────────────────────────────────────────┼────────────────────────┐
│                    EXTERNAL SERVICES LAYER                          │
├───────────────────────────────────────────┴────────────────────────┤
│                                                                     │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
│  │   OpenAI   │  │ Spoonacular│  │   USDA     │  │  Weather   │  │
│  │   GPT-4    │  │  Recipe    │  │ Nutrition  │  │    API     │  │
│  │    API     │  │    API     │  │    API     │  │            │  │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘  │
│                                                                     │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
│  │  Firebase  │  │  Barcode   │  │   Google   │  │   Payment  │  │
│  │    Auth    │  │  Lookup    │  │   Cloud    │  │  Gateway   │  │
│  │    FCM     │  │   UPC DB   │  │  Services  │  │  (Stripe)  │  │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Data Flow Diagrams

### 2.1 Recipe Suggestion Flow

```
┌─────────┐
│  User   │
└────┬────┘
     │ 1. Request suggestion
     ▼
┌─────────────────┐
│  Mobile App     │
│  ViewModel      │
└────┬────────────┘
     │ 2. Get user context
     ▼
┌─────────────────┐
│  Local Storage  │
│  - Preferences  │
│  - Pantry       │
│  - History      │
└────┬────────────┘
     │ 3. User profile + pantry data
     ▼
┌─────────────────┐
│  Recipe Repo    │
└────┬────────────┘
     │ 4. API request with context
     ▼
┌─────────────────┐
│  API Gateway    │
└────┬────────────┘
     │ 5. Authenticated request
     ▼
┌─────────────────┐
│  Recipe Service │
└────┬────────────┘
     │ 6. Check cache
     ▼
┌─────────────────┐     ┌─────────────────┐
│  Redis Cache    │ No  │  ML Model       │
│  (miss)         │────→│  Service        │
└─────────────────┘     └────┬────────────┘
                             │ 7. Rank existing recipes
                             ▼
                        ┌─────────────────┐
                        │  LLM Service    │
                        └────┬────────────┘
                             │ 8. Generate prompt
                             ▼
                        ┌─────────────────┐
                        │  OpenAI API     │
                        │  (GPT-4)        │
                        └────┬────────────┘
                             │ 9. AI response
                             ▼
                        ┌─────────────────┐
                        │  Parse & Store  │
                        │  in DB/Cache    │
                        └────┬────────────┘
                             │ 10. Return recipes
                             ▼
                        ┌─────────────────┐
                        │  Mobile App     │
                        └────┬────────────┘
                             │ 11. Display cards
                             ▼
                        ┌─────────────────┐
                        │  User sees      │
                        │  suggestions    │
                        └─────────────────┘
```

### 2.2 Family Poll Flow

```
User A                  User B                  User C
  │                       │                       │
  │ 1. Create poll        │                       │
  ├──────────────────────►│                       │
  │                       │                       │
  │  2. Select recipes    │                       │
  │                       │                       │
  │  3. Set deadline      │                       │
  │                       │                       │
  │  4. Submit            │                       │
  │                       │                       │
  ▼                       ▼                       ▼
┌──────────────────────────────────────────────────────┐
│              Poll Service (Backend)                   │
└───────────────────┬──────────────────────────────────┘
                    │ 5. Create poll in DB
                    ▼
              ┌──────────┐
              │ Database │
              └──────────┘
                    │ 6. Trigger notifications
                    ▼
         ┌──────────────────────┐
         │ Notification Service │
         └──────────────────────┘
                    │
      ┌─────────────┼─────────────┐
      │             │             │
      ▼             ▼             ▼
   User A        User B        User C
      │             │             │
      │             │ 7. Vote     │
      │             ├────────────►│
      │             │             │
      │                     8. Vote│
      ├─────────────────────────►│
      │                           │
      ▼                           ▼
┌──────────────────────────────────────┐
│     Poll aggregation (real-time)     │
│              Redis                    │
└──────────────────────────────────────┘
                    │
                    │ 9. Deadline reached
                    ▼
           ┌─────────────────┐
           │ Background Job  │
           │ - Calculate win │
           │ - Notify family │
           └─────────────────┘
                    │
      ┌─────────────┼─────────────┐
      │             │             │
      ▼             ▼             ▼
   User A        User B        User C
   "Winner: Recipe C!"
```

### 2.3 Pantry Management Flow

```
User scans barcode
       │
       ▼
┌──────────────┐
│  Camera API  │
│  ML Kit      │
└──────┬───────┘
       │ Detected: 012345678901
       ▼
┌──────────────┐
│  Barcode     │
│  Service     │
└──────┬───────┘
       │ Lookup product info
       ▼
┌──────────────┐
│  UPC Database│
│  API         │
└──────┬───────┘
       │ Product: "Milk, 2%"
       ▼
┌──────────────┐
│  Pantry      │
│  Repository  │
└──────┬───────┘
       │ Add with expiry
       ▼
┌──────────────┐
│  Local DB    │
│  Room        │
└──────┬───────┘
       │ Sync to cloud
       ▼
┌──────────────┐
│  Backend     │
│  Pantry Svc  │
└──────┬───────┘
       │ Store + check expiry
       ▼
┌──────────────┐
│  PostgreSQL  │
└──────┬───────┘
       │ Trigger expiry check
       ▼
┌──────────────┐
│  Background  │
│  Job         │
└──────┬───────┘
       │ Items expiring soon?
       ▼
┌──────────────┐
│  Notification│
│  Service     │
└──────┬───────┘
       │ Push notification
       ▼
     User
  "Milk expires in 2 days!"
```

---

## 3. Database Schema

### 3.1 PostgreSQL Schema (Relational Data)

```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    family_id UUID REFERENCES families(id),
    subscription_tier VARCHAR(50) DEFAULT 'free',
    CONSTRAINT email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- User profiles
CREATE TABLE user_profiles (
    user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    skill_level VARCHAR(20) CHECK (skill_level IN ('beginner', 'intermediate', 'advanced')),
    cooking_time_pref VARCHAR(20) CHECK (cooking_time_pref IN ('quick', 'medium', 'elaborate')),
    spice_tolerance INT CHECK (spice_tolerance BETWEEN 1 AND 10),
    dietary_restrictions JSONB DEFAULT '[]',
    allergies JSONB DEFAULT '[]',
    cuisine_preferences JSONB DEFAULT '[]',
    disliked_ingredients JSONB DEFAULT '[]',
    preferred_proteins JSONB DEFAULT '[]',
    go_crazy_frequency INT DEFAULT 20 CHECK (go_crazy_frequency BETWEEN 0 AND 100)
);

-- Families
CREATE TABLE families (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    shared_inventory BOOLEAN DEFAULT true,
    invite_code VARCHAR(20) UNIQUE NOT NULL
);

-- Family members
CREATE TABLE family_members (
    family_id UUID REFERENCES families(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    role VARCHAR(20) CHECK (role IN ('admin', 'member', 'child')),
    voting_weight DECIMAL(3,2) DEFAULT 1.0 CHECK (voting_weight BETWEEN 0 AND 1),
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (family_id, user_id)
);

-- Recipes
CREATE TABLE recipes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    cuisine_type VARCHAR(50),
    difficulty VARCHAR(20) CHECK (difficulty IN ('easy', 'medium', 'hard')),
    prep_time_minutes INT CHECK (prep_time_minutes >= 0),
    cook_time_minutes INT CHECK (cook_time_minutes >= 0),
    total_time_minutes INT GENERATED ALWAYS AS (prep_time_minutes + cook_time_minutes) STORED,
    servings INT CHECK (servings > 0),
    ingredients JSONB NOT NULL,
    instructions JSONB NOT NULL,
    nutrition JSONB,
    tags JSONB DEFAULT '[]',
    image_url VARCHAR(500),
    source VARCHAR(50) CHECK (source IN ('llm_generated', 'community', 'external_api', 'curated')),
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    rating_avg DECIMAL(3,2) DEFAULT 0.0 CHECK (rating_avg BETWEEN 0 AND 5),
    rating_count INT DEFAULT 0,
    is_public BOOLEAN DEFAULT true
);

-- Recipe ratings
CREATE TABLE recipe_ratings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    recipe_id UUID REFERENCES recipes(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    rating INT CHECK (rating BETWEEN 1 AND 5),
    review TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(recipe_id, user_id)
);

-- Pantry inventory
CREATE TABLE inventory_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    category VARCHAR(50),
    quantity DECIMAL(10,2) CHECK (quantity >= 0),
    unit VARCHAR(50),
    purchase_date DATE,
    expiry_date DATE,
    location VARCHAR(20) CHECK (location IN ('fridge', 'freezer', 'pantry', 'other')),
    barcode VARCHAR(50),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT expiry_after_purchase CHECK (expiry_date IS NULL OR expiry_date >= purchase_date)
);

-- Family polls
CREATE TABLE family_polls (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    family_id UUID REFERENCES families(id) ON DELETE CASCADE,
    created_by UUID REFERENCES users(id),
    title VARCHAR(255),
    recipe_options JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    closes_at TIMESTAMP NOT NULL,
    status VARCHAR(20) DEFAULT 'open' CHECK (status IN ('open', 'closed', 'executed')),
    winning_recipe_id UUID,
    CONSTRAINT closes_after_creation CHECK (closes_at > created_at)
);

-- Poll votes
CREATE TABLE poll_votes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    poll_id UUID REFERENCES family_polls(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    recipe_id UUID NOT NULL,
    vote_type VARCHAR(20) DEFAULT 'yes' CHECK (vote_type IN ('yes', 'no', 'maybe')),
    rank INT,
    voted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(poll_id, user_id, recipe_id)
);

-- User interactions (for ML)
CREATE TABLE user_interactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    recipe_id UUID REFERENCES recipes(id) ON DELETE CASCADE,
    interaction_type VARCHAR(50) CHECK (interaction_type IN 
        ('viewed', 'saved', 'cooked', 'rated', 'shared', 'dismissed', 'customized')),
    rating INT CHECK (rating BETWEEN 1 AND 5),
    feedback TEXT,
    context JSONB,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Meal plans
CREATE TABLE meal_plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    recipe_id UUID REFERENCES recipes(id),
    planned_date DATE NOT NULL,
    meal_type VARCHAR(20) CHECK (meal_type IN ('breakfast', 'lunch', 'dinner', 'snack')),
    is_cooked BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Shopping lists
CREATE TABLE shopping_lists (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) DEFAULT 'My Shopping List',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT true
);

CREATE TABLE shopping_list_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    list_id UUID REFERENCES shopping_lists(id) ON DELETE CASCADE,
    recipe_id UUID REFERENCES recipes(id),
    name VARCHAR(255) NOT NULL,
    quantity DECIMAL(10,2),
    unit VARCHAR(50),
    category VARCHAR(50),
    is_purchased BOOLEAN DEFAULT false,
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_family ON users(family_id);
CREATE INDEX idx_recipes_cuisine ON recipes(cuisine_type);
CREATE INDEX idx_recipes_difficulty ON recipes(difficulty);
CREATE INDEX idx_recipes_rating ON recipes(rating_avg DESC);
CREATE INDEX idx_recipes_created ON recipes(created_at DESC);
CREATE INDEX idx_inventory_user ON inventory_items(user_id);
CREATE INDEX idx_inventory_expiry ON inventory_items(expiry_date);
CREATE INDEX idx_interactions_user ON user_interactions(user_id);
CREATE INDEX idx_interactions_recipe ON user_interactions(recipe_id);
CREATE INDEX idx_interactions_timestamp ON user_interactions(timestamp DESC);
CREATE INDEX idx_polls_family ON family_polls(family_id);
CREATE INDEX idx_polls_status ON family_polls(status);
CREATE INDEX idx_meal_plans_user_date ON meal_plans(user_id, planned_date);

-- Full-text search
CREATE INDEX idx_recipes_name_fts ON recipes USING GIN(to_tsvector('english', name));
CREATE INDEX idx_recipes_description_fts ON recipes USING GIN(to_tsvector('english', description));
```

### 3.2 MongoDB Collections (Document Data)

```javascript
// llm_responses collection
{
  _id: ObjectId,
  user_id: UUID,
  prompt: String,
  prompt_hash: String,  // for deduplication
  response: {
    recipes: Array,
    metadata: Object
  },
  model: String,  // "gpt-4", "claude-3", etc.
  tokens_used: Number,
  response_time_ms: Number,
  created_at: ISODate,
  ttl: Number  // Time to live in seconds
}

// user_activity_logs collection
{
  _id: ObjectId,
  user_id: UUID,
  session_id: UUID,
  events: [
    {
      timestamp: ISODate,
      event_type: String,  // "page_view", "recipe_view", "search", etc.
      event_data: Object,
      device_info: Object
    }
  ],
  created_at: ISODate
}

// ml_training_data collection
{
  _id: ObjectId,
  user_id: UUID,
  feature_vector: Array,  // Embedded features for ML
  interaction_history: Array,
  preferences_snapshot: Object,
  model_version: String,
  last_updated: ISODate
}

// recipe_cache collection (TTL index)
{
  _id: ObjectId,
  cache_key: String,
  recipe_data: Object,
  created_at: ISODate,
  expires_at: ISODate
}
```

### 3.3 Redis Data Structures

```
# Session management
session:{user_id} -> {
  "token": "jwt_token",
  "expires_at": timestamp,
  "device_id": "device_123"
}
TTL: 24 hours

# Rate limiting
rate_limit:{user_id}:{endpoint} -> counter
TTL: 1 hour

# Real-time poll data
poll:{poll_id}:votes -> Hash {
  "recipe_uuid_1": "5",
  "recipe_uuid_2": "3",
  "recipe_uuid_3": "7"
}
TTL: Until poll closes

# LLM response cache
llm_cache:{prompt_hash} -> "JSON response"
TTL: 7 days

# Active users (for analytics)
active_users:{date} -> Set of user_ids
TTL: 30 days

# Pantry expiry notifications queue
expiry_notifications -> List of {user_id, item_id, expiry_date}
```

---

## 4. API Endpoints

### 4.1 Authentication & Users

```
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
POST   /api/v1/auth/refresh-token
POST   /api/v1/auth/forgot-password
POST   /api/v1/auth/reset-password

GET    /api/v1/users/me
PUT    /api/v1/users/me
DELETE /api/v1/users/me
GET    /api/v1/users/me/profile
PUT    /api/v1/users/me/profile
GET    /api/v1/users/me/preferences
PUT    /api/v1/users/me/preferences
```

### 4.2 Recipes

```
GET    /api/v1/recipes                    # List/search recipes
POST   /api/v1/recipes                    # Create custom recipe
GET    /api/v1/recipes/:id                # Get recipe details
PUT    /api/v1/recipes/:id                # Update recipe
DELETE /api/v1/recipes/:id                # Delete recipe

GET    /api/v1/recipes/suggestions        # AI-powered suggestions
POST   /api/v1/recipes/suggestions/crazy  # "Go Crazy" mode
GET    /api/v1/recipes/suggestions/pantry # Based on pantry
GET    /api/v1/recipes/:id/similar        # Similar recipes

POST   /api/v1/recipes/:id/rate           # Rate recipe
GET    /api/v1/recipes/:id/ratings        # Get ratings
POST   /api/v1/recipes/:id/save           # Save to favorites
DELETE /api/v1/recipes/:id/save           # Remove from favorites
GET    /api/v1/recipes/saved              # Get saved recipes
```

### 4.3 Pantry/Inventory

```
GET    /api/v1/pantry                     # Get all inventory
POST   /api/v1/pantry                     # Add item
GET    /api/v1/pantry/:id                 # Get item
PUT    /api/v1/pantry/:id                 # Update item
DELETE /api/v1/pantry/:id                 # Remove item
POST   /api/v1/pantry/batch               # Batch add items

GET    /api/v1/pantry/expiring            # Items expiring soon
GET    /api/v1/pantry/categories          # Group by category
POST   /api/v1/pantry/barcode             # Lookup by barcode
GET    /api/v1/pantry/suggestions         # Recipes with available items
```

### 4.4 Family & Polls

```
POST   /api/v1/families                   # Create family
GET    /api/v1/families/:id               # Get family details
PUT    /api/v1/families/:id               # Update family
DELETE /api/v1/families/:id               # Delete family

POST   /api/v1/families/:id/invite        # Generate invite code
POST   /api/v1/families/join              # Join family via code
DELETE /api/v1/families/:id/members/:uid  # Remove member

GET    /api/v1/families/:id/polls         # List polls
POST   /api/v1/families/:id/polls         # Create poll
GET    /api/v1/polls/:id                  # Get poll details
POST   /api/v1/polls/:id/vote             # Cast vote
GET    /api/v1/polls/:id/results          # Get results
DELETE /api/v1/polls/:id                  # Delete poll
```

### 4.5 Meal Planning

```
GET    /api/v1/meal-plans                 # Get meal plans
POST   /api/v1/meal-plans                 # Create meal plan
GET    /api/v1/meal-plans/:date           # Get plans for date
PUT    /api/v1/meal-plans/:id             # Update meal plan
DELETE /api/v1/meal-plans/:id             # Delete meal plan

POST   /api/v1/meal-plans/generate        # AI generate week plan
POST   /api/v1/meal-plans/:id/cooked      # Mark as cooked
```

### 4.6 Shopping Lists

```
GET    /api/v1/shopping-lists             # Get shopping lists
POST   /api/v1/shopping-lists             # Create list
GET    /api/v1/shopping-lists/:id         # Get list details
DELETE /api/v1/shopping-lists/:id         # Delete list

POST   /api/v1/shopping-lists/:id/items   # Add item
PUT    /api/v1/shopping-lists/:id/items/:item_id  # Update item
DELETE /api/v1/shopping-lists/:id/items/:item_id  # Remove item
POST   /api/v1/shopping-lists/:id/items/:item_id/check  # Mark purchased

POST   /api/v1/shopping-lists/from-recipe # Generate from recipe
POST   /api/v1/shopping-lists/from-meal-plan  # Generate from plan
```

### 4.7 LLM Integration

```
POST   /api/v1/llm/suggest                # General suggestion
POST   /api/v1/llm/generate-recipe        # Generate custom recipe
POST   /api/v1/llm/modify-recipe          # Modify existing recipe
POST   /api/v1/llm/substitute             # Ingredient substitution
POST   /api/v1/llm/chat                   # General cooking chat
```

### 4.8 Analytics & Interactions

```
POST   /api/v1/interactions               # Log interaction
GET    /api/v1/analytics/my-stats         # User's cooking stats
GET    /api/v1/analytics/family-stats     # Family cooking stats
GET    /api/v1/analytics/trends           # Recipe trends
```

---

## 5. Security Implementation

### 5.1 Authentication Flow

```
1. User Registration
   ├─> Client: Email + Password
   ├─> Server: Hash password (bcrypt)
   ├─> Server: Create user record
   ├─> Server: Generate JWT token
   └─> Client: Store JWT securely

2. User Login
   ├─> Client: Email + Password
   ├─> Server: Verify credentials
   ├─> Server: Generate access token (15 min expiry)
   ├─> Server: Generate refresh token (7 day expiry)
   ├─> Client: Store tokens in Android Keystore
   └─> Client: Include access token in API requests

3. Token Refresh
   ├─> Client: Send refresh token
   ├─> Server: Validate refresh token
   ├─> Server: Generate new access token
   └─> Client: Update stored access token

4. API Request
   ├─> Client: Include Authorization: Bearer {token}
   ├─> Server: Validate JWT signature
   ├─> Server: Check expiration
   ├─> Server: Extract user_id from claims
   └─> Server: Process request
```

### 5.2 Data Encryption

```
At Rest:
- Database: TLS encryption
- Passwords: bcrypt (cost factor 12)
- Sensitive fields: AES-256-GCM
- API keys: Stored in secrets manager

In Transit:
- All API calls: HTTPS/TLS 1.3
- Certificate pinning in mobile app
- HSTS headers enabled

On Device:
- JWT tokens: Android Keystore
- User data: EncryptedSharedPreferences
- Database: SQLCipher for Room
```

### 5.3 Input Validation & Sanitization

```kotlin
// Example validation middleware
class RecipeValidator {
    fun validateRecipeInput(recipe: RecipeInput): ValidationResult {
        return ValidationResult(
            name = validateString(recipe.name, 1, 255),
            prepTime = validateInt(recipe.prepTime, 0, 1440),
            ingredients = validateArray(recipe.ingredients, 1, 100),
            instructions = validateArray(recipe.instructions, 1, 50),
            // Sanitize HTML/SQL injection attempts
            description = sanitizeHtml(recipe.description)
        )
    }
}
```

### 5.4 Rate Limiting

```
Free Tier:
- 10 AI suggestions / day
- 100 API requests / hour
- 5 family polls / week

Premium Tier:
- Unlimited AI suggestions
- 1000 API requests / hour
- Unlimited family polls

Implementation:
- Token bucket algorithm
- Redis-based counter
- 429 Too Many Requests response
- Retry-After header
```

---

## 6. Deployment Architecture

### 6.1 Cloud Infrastructure (AWS Example)

```
┌─────────────────────────────────────────────────────────┐
│                    CloudFront CDN                        │
│  - Static assets (images, JS, CSS)                      │
│  - Edge caching                                          │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────┴─────────────────────────────────┐
│                 Application Load Balancer                │
│  - SSL Termination                                       │
│  - Health checks                                         │
│  - Traffic distribution                                  │
└───────────────────────┬─────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
   ┌─────────┐    ┌─────────┐    ┌─────────┐
   │  ECS    │    │  ECS    │    │  ECS    │
   │Container│    │Container│    │Container│
   │ (API)   │    │ (API)   │    │ (API)   │
   └─────────┘    └─────────┘    └─────────┘
        │               │               │
        └───────────────┼───────────────┘
                        │
        ┌───────────────┴───────────────┐
        │                               │
        ▼                               ▼
   ┌──────────┐                    ┌──────────┐
   │   RDS    │                    │  Redis   │
   │PostgreSQL│                    │ElastiCache│
   │Multi-AZ  │                    └──────────┘
   └──────────┘
        │
        │
   ┌──────────┐
   │    S3    │
   │  Bucket  │
   │ (Images) │
   └──────────┘
```

### 6.2 CI/CD Pipeline

```
GitHub Push
    │
    ▼
┌─────────────────┐
│ GitHub Actions  │
│ - Run tests     │
│ - Lint code     │
│ - Security scan │
└────────┬────────┘
         │ Pass
         ▼
┌─────────────────┐
│ Build Docker    │
│ Image           │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Push to ECR     │
│ (Container Reg) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Deploy to       │
│ Staging (ECS)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Run E2E Tests   │
└────────┬────────┘
         │ Pass
         ▼
┌─────────────────┐
│ Manual Approval │
└────────┬────────┘
         │ Approved
         ▼
┌─────────────────┐
│ Deploy to       │
│ Production      │
│ (Blue-Green)    │
└─────────────────┘
```

### 6.3 Monitoring & Observability

```
Application Monitoring:
- CloudWatch Logs
- CloudWatch Metrics
- Custom dashboards
- APM: New Relic / Datadog

Mobile App:
- Firebase Crashlytics
- Firebase Analytics
- Performance Monitoring
- Custom events

Alerts:
- Error rate > 1%
- Response time > 2s
- Database CPU > 80%
- Failed LLM calls
- Low disk space
- SSL cert expiry
```

---

## 7. Performance Optimization

### 7.1 Caching Strategy

```
Layer 1: Client-Side (Mobile App)
- Room database for offline data
- In-memory cache for frequently accessed data
- Image caching (Coil/Glide)
- TTL: User preferences (24h), Recipes (1h)

Layer 2: API Gateway
- Redis cache for common queries
- TTL: Recipe lists (15m), User profiles (5m)

Layer 3: Database
- Query result caching
- Materialized views for complex queries
- Read replicas for analytics

Layer 4: CDN
- CloudFront for images and static assets
- Edge caching with max-age headers
```

### 7.2 Database Optimization

```sql
-- Partitioning large tables
CREATE TABLE user_interactions_y2026m01 PARTITION OF user_interactions
FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

-- Covering indexes
CREATE INDEX idx_recipes_search_covering ON recipes(cuisine_type, difficulty)
INCLUDE (name, prep_time_minutes, rating_avg);

-- Partial indexes
CREATE INDEX idx_active_polls ON family_polls(family_id)
WHERE status = 'open';
```

### 7.3 Async Processing

```
Use Cases for Message Queues:
1. Email notifications (order confirmation, expiry alerts)
2. ML model training (batch processing)
3. Image processing (thumbnail generation)
4. Analytics aggregation
5. LLM response caching
6. Recipe import from external APIs

Queue: RabbitMQ / AWS SQS
Workers: Auto-scaling based on queue depth
```

---

## 8. Testing Strategy

### 8.1 Mobile App Testing

```kotlin
// Unit Tests (JUnit + Mockk)
class RecipeViewModelTest {
    @Test
    fun `loadSuggestions should update UI state with recipes`() {
        // Arrange
        val mockRepo = mockk<RecipeRepository>()
        coEvery { mockRepo.getSuggestions() } returns flowOf(Result.Success(recipes))
        
        // Act
        viewModel.loadSuggestions()
        
        // Assert
        assertEquals(recipes, viewModel.uiState.value.recipes)
    }
}

// Integration Tests (Espresso)
@Test
fun testRecipeSuggestionFlow() {
    onView(withId(R.id.suggest_button)).perform(click())
    onView(withId(R.id.recipe_card)).check(matches(isDisplayed()))
}

// UI Tests (Compose)
@Test
fun testRecipeCard() {
    composeTestRule.setContent {
        RecipeCard(recipe = testRecipe)
    }
    composeTestRule.onNodeWithText("Pasta Carbonara").assertIsDisplayed()
}
```

### 8.2 Backend Testing

```javascript
// Unit Tests (Jest)
describe('RecipeService', () => {
  test('should generate recipe suggestion', async () => {
    const suggestion = await recipeService.generateSuggestion(userContext);
    expect(suggestion).toHaveProperty('name');
    expect(suggestion.ingredients).toBeInstanceOf(Array);
  });
});

// Integration Tests
describe('POST /api/v1/recipes/suggestions', () => {
  test('should return 200 with recipes', async () => {
    const response = await request(app)
      .post('/api/v1/recipes/suggestions')
      .set('Authorization', `Bearer ${token}`)
      .send({ count: 3 });
    
    expect(response.status).toBe(200);
    expect(response.body.recipes).toHaveLength(3);
  });
});

// E2E Tests (Cypress)
describe('Recipe Suggestion Flow', () => {
  it('should suggest recipes based on pantry', () => {
    cy.login();
    cy.visit('/pantry');
    cy.addItem('Chicken');
    cy.visit('/suggestions');
    cy.contains('Chicken').should('be.visible');
  });
});
```

### 8.3 Load Testing

```javascript
// k6 load test
import http from 'k6/http';

export let options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp-up
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 200 }, // Spike
    { duration: '5m', target: 200 },
    { duration: '2m', target: 0 },   // Ramp-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
    http_req_failed: ['rate<0.01'],   // Error rate < 1%
  },
};

export default function () {
  http.get('https://api.suggestdiner.com/recipes/suggestions', {
    headers: { 'Authorization': `Bearer ${__ENV.TOKEN}` },
  });
}
```

---

## 9. Conclusion

This technical architecture provides a scalable, secure, and performant foundation for the Suggest Diner application. Key highlights:

1. **Microservices Architecture**: Modular services allow independent scaling
2. **LLM Integration**: Flexible integration with multiple AI providers
3. **Offline-First Mobile**: Better user experience with local caching
4. **Strong Security**: JWT auth, encryption, input validation
5. **Scalable Infrastructure**: Auto-scaling, load balancing, CDN
6. **Comprehensive Testing**: Unit, integration, E2E, and load tests
7. **Observability**: Monitoring, logging, and alerting at every layer

The architecture is designed to handle growth from MVP to millions of users while maintaining performance and reliability.
