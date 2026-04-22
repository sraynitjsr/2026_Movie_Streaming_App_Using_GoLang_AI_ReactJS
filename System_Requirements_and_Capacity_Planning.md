# System Design Analysis - Movie Streaming Platform
## System Requirements & Capacity Planning

---

## 📋 Table of Contents
1. [Problem Statement](#problem-statement)
2. [Functional Requirements](#functional-requirements)
3. [Non-Functional Requirements](#non-functional-requirements)
4. [Capacity Estimation & Constraints](#capacity-estimation--constraints)
5. [API Design](#api-design)
6. [Database Schema Design](#database-schema-design)
7. [High-Level Architecture](#high-level-architecture)
8. [Detailed Component Design](#detailed-component-design)
9. [Trade-offs & Design Decisions](#trade-offs--design-decisions)
10. [Scalability & Performance](#scalability--performance)
11. [Security Considerations](#security-considerations)
12. [Monitoring & Observability](#monitoring--observability)

---

## 🎯 Problem Statement

**Design a scalable movie/video streaming platform similar to Netflix, Hulu, or Disney+**

The platform should allow users to:
- Browse and search for movies/TV shows
- Stream video content with adaptive bitrate
- Get personalized recommendations
- Resume watching from where they left off
- Upload content (admin/content creators)

---

## ✅ Functional Requirements

### Must-Have (P0)
1. **User Management**
   - User registration and authentication (email/password, OAuth)
   - User profile management
   - Multi-profile support (family accounts)

2. **Content Catalog**
   - Browse movies and TV series
   - View content details (title, description, cast, genre, rating)
   - Content categorization by genre, year, rating
   - Trending/popular content listing

3. **Video Playback**
   - Stream video with adaptive bitrate streaming (HLS/DASH)
   - Play/pause/seek controls
   - Quality selection (auto, 480p, 720p, 1080p, 4K)
   - Resume playback from last position
   - Subtitles/closed captions support

4. **Search**
   - Full-text search for movies/shows
   - Filter by genre, year, rating, language
   - Search suggestions/autocomplete

5. **Content Upload (Admin)**
   - Upload video files
   - Add metadata (title, description, cast, etc.)
   - Manage content library

### Should-Have (P1)
6. **Recommendations**
   - Personalized content recommendations
   - "Continue Watching" row
   - "Because you watched X" suggestions
   - Trending and popular content

7. **User Interactions**
   - Add to "My List" / Watchlist
   - Rating/reviewing content
   - Watch history

8. **Analytics**
   - Track viewing metrics (watch time, completion rate)
   - User engagement analytics
   - Content performance metrics

### Nice-to-Have (P2)
9. **Social Features**
   - Watch parties (synchronized viewing)
   - Comments and discussions
   - Share content on social media

10. **Advanced Features**
    - Download for offline viewing
    - Parental controls
    - Multiple audio tracks
    - Interactive content (branching narratives)

---

## 🔧 Non-Functional Requirements

### 1. **Performance**
- **Video startup latency**: < 2 seconds
- **API response time**: 
  - p95 < 200ms for metadata APIs
  - p99 < 500ms for search APIs
- **Video buffering**: < 5% rebuffering ratio
- **CDN cache hit rate**: > 95%

### 2. **Scalability**
- **User base**: Support 100M registered users
- **Concurrent viewers**: Handle 10M concurrent streams
- **Content library**: Store 100K+ titles (500K+ hours of video)
- **Upload rate**: Process 1000+ video uploads per day
- **Horizontal scalability**: All services must be stateless and horizontally scalable

### 3. **Availability**
- **Uptime SLA**: 99.9% (< 8.76 hours downtime/year)
- **Disaster recovery**: RPO < 1 hour, RTO < 4 hours
- **Multi-region deployment**: Active-active in 3+ regions
- **Graceful degradation**: Core playback works even if recommendations/search fail

### 4. **Reliability**
- **Data durability**: 99.999999999% (11 9's) for video files
- **Transcoding success rate**: > 99.5%
- **Payment processing**: Zero data loss, ACID compliance
- **Idempotency**: All write operations must be idempotent

### 5. **Consistency**
- **User data**: Strong consistency (auth, subscriptions, payments)
- **Content metadata**: Eventual consistency acceptable (< 1 minute lag)
- **Watch progress**: Eventually consistent (< 5 seconds)
- **Analytics**: Eventual consistency (< 15 minutes lag)

### 6. **Security**
- **Authentication**: OAuth 2.0, JWT tokens
- **Authorization**: Role-based access control (RBAC)
- **Data encryption**: TLS 1.3 in transit, AES-256 at rest
- **DRM**: Widevine, FairPlay for premium content
- **PII protection**: GDPR/CCPA compliance
- **API security**: Rate limiting, WAF, DDoS protection

### 7. **Cost Efficiency**
- **Storage costs**: Cold storage for old/unpopular content
- **Bandwidth costs**: Optimize CDN usage, peer-to-peer delivery (optional)
- **Compute costs**: Spot instances for batch processing
- **Transcoding costs**: On-demand vs pre-transcoding trade-off

### 8. **Maintainability**
- **Code quality**: > 80% test coverage
- **Documentation**: OpenAPI specs, architecture diagrams
- **Deployment**: Blue-green deployments, canary releases
- **Rollback**: < 5 minutes to rollback failed deployments

### 9. **Observability**
- **Metrics**: Prometheus with 1-minute granularity
- **Logging**: Centralized, structured JSON logs
- **Tracing**: Distributed tracing with Jaeger (100% critical paths, 1% sample others)
- **Alerting**: < 2 minutes to detect and alert on incidents

### 10. **Compliance**
- **Content licensing**: Region-based content restrictions
- **Age ratings**: Parental controls, age verification
- **Accessibility**: WCAG 2.1 AA compliance
- **Privacy**: Cookie consent, data deletion requests

---

## 📊 Capacity Estimation & Constraints

### Traffic Estimation

**Assumptions:**
- Total users: 100M registered users
- Daily active users (DAU): 30M (30% of total)
- Concurrent peak viewers: 10M (33% of DAU)
- Average watch time per day: 2 hours per DAU
- Average video bitrate: 4 Mbps (adaptive, average across qualities)

**Daily Metrics:**
- Total watch hours/day: 30M users × 2 hours = 60M hours
- Peak concurrent streams: 10M
- API requests: ~500M/day (browsing, metadata, auth, etc.)

**QPS (Queries Per Second):**
- Average API QPS: 500M / 86400 = ~5,800 QPS
- Peak API QPS (3x average): ~17,500 QPS
- Video segment requests: 10M streams × 6 segments/min = 60M segments/min = 1M QPS

### Storage Estimation

**Content Storage:**
- Total titles: 100,000
- Average movie duration: 2 hours
- Storage per quality:
  - 4K (25 Mbps): 22.5 GB/movie
  - 1080p (8 Mbps): 7.2 GB/movie
  - 720p (5 Mbps): 4.5 GB/movie
  - 480p (2.5 Mbps): 2.25 GB/movie
  - Total per movie: ~36 GB

**Total Content Storage:**
- 100K titles × 36 GB = 3.6 PB
- With redundancy (3x): ~11 PB
- Metadata + thumbnails: ~100 TB
- **Total: ~11 PB**

**Database Storage:**
- User records: 100M × 5 KB = 500 GB
- Content metadata: 100K × 50 KB = 5 GB
- Watch history: 100M users × 100 entries × 1 KB = 10 TB
- Analytics (1 year): ~50 TB
- **Total DB: ~60 TB**

### Bandwidth Estimation

**Egress Bandwidth (CDN):**
- Peak concurrent: 10M streams × 4 Mbps = 40 Tbps
- Average (24h): 60M hours × 4 Mbps / 24 = ~10 Tbps
- Daily data transfer: 10 Tbps × 86400 sec = 108 PB/day

**Ingress Bandwidth (Uploads):**
- Daily uploads: 1000 videos × 50 GB (source) = 50 TB/day
- Upload bandwidth: 50 TB / 86400 sec = ~5 Gbps

### Compute Estimation

**Transcoding:**
- 1000 videos/day × 2 hours × 4 qualities = 8000 transcoding hours/day
- Transcoding speed: 0.5x realtime (2 hours video = 4 hours compute)
- Total compute: 8000 × 4 = 32,000 CPU-hours/day
- With 96-core servers: ~14 servers running 24/7

**API Servers:**
- Peak QPS: 17,500
- Assuming 500 RPS per server: ~35 API servers
- With redundancy and headroom: ~100 servers

**Recommendation Engine:**
- Batch jobs: Nightly processing for 100M users
- Compute: ~5000 CPU-hours/day (spot instances)

### Cost Estimation (Monthly - AWS Pricing)

| Resource | Amount | Unit Cost | Monthly Cost |
|----------|--------|-----------|--------------|
| S3 Storage (11 PB) | 11,000 TB | $0.023/GB | $253,000 |
| CloudFront (108 PB/day) | 3,240 PB/month | $0.085/GB (first 10 PB) | $275,400 |
| RDS Postgres (60 TB) | db.r6g.16xlarge | $6.912/hour | $5,000 |
| ElastiCache Redis | 100 nodes × cache.r6g.large | $0.226/hour | $16,000 |
| EKS Nodes (API) | 100 × m6g.2xlarge | $0.308/hour | $22,000 |
| Transcoding (EC2 Spot) | 14 × c6g.16xlarge (spot) | $0.80/hour | $8,000 |
| Elasticsearch | 20 nodes × r6g.2xlarge | $0.504/hour | $7,300 |
| Data transfer (other) | | | $50,000 |
| **Total** | | | **~$637,000/month** |

**Per-user cost:** $637k / 30M DAU = **$0.02 per DAU per month**

---

## 🔌 API Design

### REST API Endpoints (External - API Gateway)

#### Authentication & Users
```http
POST   /api/v1/auth/register          # Register new user
POST   /api/v1/auth/login             # Login (returns JWT)
POST   /api/v1/auth/refresh           # Refresh access token
POST   /api/v1/auth/logout            # Logout (invalidate token)
GET    /api/v1/users/me               # Get current user profile
PUT    /api/v1/users/me               # Update profile
GET    /api/v1/users/me/profiles      # Get all profiles (multi-profile)
POST   /api/v1/users/me/profiles      # Create new profile
```

#### Content Catalog
```http
GET    /api/v1/content                # Browse content (paginated)
                                       # ?genre=action&year=2024&page=1&limit=20
GET    /api/v1/content/:id            # Get content details
GET    /api/v1/content/trending       # Get trending content
GET    /api/v1/content/popular        # Get popular content
GET    /api/v1/genres                 # List all genres
```

#### Search
```http
GET    /api/v1/search                 # Search content
                                       # ?q=inception&genre=scifi&year=2010
GET    /api/v1/search/suggest         # Autocomplete suggestions
                                       # ?q=inc
```

#### Playback
```http
POST   /api/v1/playback/start         # Start playback (get streaming URL)
       # Body: { content_id, profile_id, quality }
       # Returns: { manifest_url, drm_token, expires_at }
POST   /api/v1/playback/progress      # Update watch progress
       # Body: { content_id, position_seconds }
GET    /api/v1/playback/progress/:id  # Get watch progress
```

#### Recommendations
```http
GET    /api/v1/recommendations        # Get personalized recommendations
GET    /api/v1/recommendations/similar/:id  # Similar content
GET    /api/v1/continue-watching      # Continue watching row
```

#### User Interactions
```http
POST   /api/v1/watchlist              # Add to watchlist
       # Body: { content_id }
DELETE /api/v1/watchlist/:id          # Remove from watchlist
GET    /api/v1/watchlist              # Get user's watchlist
POST   /api/v1/ratings                # Rate content
       # Body: { content_id, rating }
```

#### Admin APIs
```http
POST   /api/v1/admin/content          # Create content entry
PUT    /api/v1/admin/content/:id      # Update content metadata
DELETE /api/v1/admin/content/:id      # Delete content
POST   /api/v1/admin/upload/presign   # Get presigned S3 upload URL
       # Body: { filename, size, content_type }
POST   /api/v1/admin/transcode        # Trigger transcoding job
       # Body: { content_id, s3_key, qualities }
```

### gRPC Service Contracts (Internal)

#### Auth Service (auth.proto)
```protobuf
service AuthService {
  rpc Authenticate(AuthRequest) returns (AuthResponse);
  rpc ValidateToken(TokenRequest) returns (TokenResponse);
  rpc RefreshToken(RefreshRequest) returns (AuthResponse);
  rpc RevokeToken(RevokeRequest) returns (RevokeResponse);
}

message AuthRequest {
  string email = 1;
  string password = 2;
}

message AuthResponse {
  string access_token = 1;
  string refresh_token = 2;
  int64 expires_at = 3;
  User user = 4;
}
```

#### Content Service (content.proto)
```protobuf
service ContentService {
  rpc GetContent(GetContentRequest) returns (Content);
  rpc ListContent(ListContentRequest) returns (ListContentResponse);
  rpc CreateContent(CreateContentRequest) returns (Content);
  rpc UpdateContent(UpdateContentRequest) returns (Content);
  rpc DeleteContent(DeleteContentRequest) returns (DeleteContentResponse);
  rpc GetContentByIds(GetContentByIdsRequest) returns (GetContentByIdsResponse);
}

message Content {
  string id = 1;
  string title = 2;
  string description = 3;
  string type = 4; // movie, series, episode
  int32 duration_seconds = 5;
  int32 release_year = 6;
  repeated string genres = 7;
  string thumbnail_url = 8;
  string s3_key = 9;
  map<string, string> metadata = 10;
}
```

#### Playback Service (playback.proto)
```protobuf
service PlaybackService {
  rpc StartPlayback(StartPlaybackRequest) returns (StartPlaybackResponse);
  rpc UpdateProgress(UpdateProgressRequest) returns (UpdateProgressResponse);
  rpc GetProgress(GetProgressRequest) returns (PlaybackProgress);
}

message StartPlaybackRequest {
  string user_id = 1;
  string content_id = 2;
  string quality = 3; // auto, 480p, 720p, 1080p, 4K
}

message StartPlaybackResponse {
  string manifest_url = 1; // HLS m3u8 URL (presigned)
  string drm_token = 2;
  int64 expires_at = 3;
  int32 resume_position_seconds = 4;
}
```

#### Search Service (search.proto)
```protobuf
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
  rpc Suggest(SuggestRequest) returns (SuggestResponse);
  rpc IndexContent(IndexContentRequest) returns (IndexContentResponse);
  rpc DeleteFromIndex(DeleteFromIndexRequest) returns (DeleteFromIndexResponse);
}

message SearchRequest {
  string query = 1;
  repeated string genres = 2;
  int32 min_year = 3;
  int32 max_year = 4;
  int32 page = 5;
  int32 limit = 6;
}
```

#### Recommendation Service (recommendation.proto)
```protobuf
service RecommendationService {
  rpc GetRecommendations(GetRecommendationsRequest) returns (GetRecommendationsResponse);
  rpc GetSimilarContent(GetSimilarContentRequest) returns (GetSimilarContentResponse);
  rpc GetContinueWatching(GetContinueWatchingRequest) returns (GetContinueWatchingResponse);
}

message GetRecommendationsRequest {
  string user_id = 1;
  int32 limit = 2;
  string context = 3; // homepage, genre_page, etc.
}

message GetRecommendationsResponse {
  repeated RecommendationItem items = 1;
  string algorithm_version = 2;
}

message RecommendationItem {
  string content_id = 1;
  float score = 2;
  string reason = 3; // "Because you watched X", "Trending now"
}
```

---

## 🗄️ Database Schema Design

### PostgreSQL Schema

```sql
-- Users & Authentication
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    subscription_tier VARCHAR(50) DEFAULT 'free', -- free, basic, premium
    subscription_expires_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP NULL
);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_subscription ON users(subscription_tier, subscription_expires_at);

CREATE TABLE user_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    avatar_url VARCHAR(500),
    is_kids_profile BOOLEAN DEFAULT false,
    preferences JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(user_id, name)
);
CREATE INDEX idx_user_profiles_user_id ON user_profiles(user_id);

CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) NOT NULL,
    ip_address INET,
    user_agent TEXT,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX idx_sessions_token ON sessions(token_hash);
CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);

-- Content Catalog
CREATE TABLE content (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(500) NOT NULL,
    slug VARCHAR(500) UNIQUE NOT NULL,
    description TEXT,
    type VARCHAR(50) NOT NULL, -- movie, series
    duration_seconds INT,
    release_year INT,
    rating VARCHAR(10), -- PG, PG-13, R, etc.
    imdb_rating DECIMAL(3,1),
    language VARCHAR(10) DEFAULT 'en',
    country VARCHAR(50),
    director VARCHAR(255),
    cast TEXT[], -- array of actor names
    thumbnail_url VARCHAR(500),
    poster_url VARCHAR(500),
    trailer_url VARCHAR(500),
    s3_key VARCHAR(500), -- for movies
    status VARCHAR(50) DEFAULT 'draft', -- draft, processing, published, archived
    view_count BIGINT DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    published_at TIMESTAMP
);
CREATE INDEX idx_content_type ON content(type);
CREATE INDEX idx_content_status ON content(status);
CREATE INDEX idx_content_published_at ON content(published_at);
CREATE INDEX idx_content_slug ON content(slug);
CREATE INDEX idx_content_view_count ON content(view_count DESC);

CREATE TABLE genres (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL
);

CREATE TABLE content_genres (
    content_id UUID REFERENCES content(id) ON DELETE CASCADE,
    genre_id INT REFERENCES genres(id) ON DELETE CASCADE,
    PRIMARY KEY (content_id, genre_id)
);
CREATE INDEX idx_content_genres_genre_id ON content_genres(genre_id);

CREATE TABLE series (
    id UUID PRIMARY KEY REFERENCES content(id) ON DELETE CASCADE,
    total_seasons INT NOT NULL DEFAULT 1,
    total_episodes INT NOT NULL DEFAULT 0,
    status VARCHAR(50) DEFAULT 'ongoing' -- ongoing, completed, cancelled
);

CREATE TABLE episodes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    series_id UUID NOT NULL REFERENCES series(id) ON DELETE CASCADE,
    season_number INT NOT NULL,
    episode_number INT NOT NULL,
    title VARCHAR(500) NOT NULL,
    description TEXT,
    duration_seconds INT NOT NULL,
    thumbnail_url VARCHAR(500),
    s3_key VARCHAR(500) NOT NULL,
    air_date DATE,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(series_id, season_number, episode_number)
);
CREATE INDEX idx_episodes_series_id ON episodes(series_id);

-- Streaming & Playback
CREATE TABLE streaming_assets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    content_id UUID REFERENCES content(id) ON DELETE CASCADE,
    episode_id UUID REFERENCES episodes(id) ON DELETE CASCADE,
    quality VARCHAR(20) NOT NULL, -- 480p, 720p, 1080p, 4k
    codec VARCHAR(50) NOT NULL, -- h264, h265, vp9
    bitrate_kbps INT NOT NULL,
    resolution VARCHAR(20), -- 1920x1080
    s3_key VARCHAR(500) NOT NULL, -- path to HLS manifest
    size_bytes BIGINT,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(content_id, episode_id, quality)
);
CREATE INDEX idx_streaming_assets_content_id ON streaming_assets(content_id);

CREATE TABLE playback_progress (
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    profile_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
    content_id UUID REFERENCES content(id) ON DELETE CASCADE,
    episode_id UUID REFERENCES episodes(id) ON DELETE CASCADE,
    position_seconds INT NOT NULL DEFAULT 0,
    duration_seconds INT NOT NULL,
    completed BOOLEAN DEFAULT false,
    last_watched_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (profile_id, content_id, episode_id)
);
CREATE INDEX idx_playback_progress_profile_id ON playback_progress(profile_id, last_watched_at DESC);
CREATE INDEX idx_playback_progress_content_id ON playback_progress(content_id);

-- Recommendations & Interactions
CREATE TABLE watchlist (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    profile_id UUID NOT NULL REFERENCES user_profiles(id) ON DELETE CASCADE,
    content_id UUID NOT NULL REFERENCES content(id) ON DELETE CASCADE,
    added_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(profile_id, content_id)
);
CREATE INDEX idx_watchlist_profile_id ON watchlist(profile_id, added_at DESC);

CREATE TABLE ratings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    content_id UUID NOT NULL REFERENCES content(id) ON DELETE CASCADE,
    rating INT NOT NULL CHECK (rating >= 1 AND rating <= 5),
    review TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(user_id, content_id)
);
CREATE INDEX idx_ratings_content_id ON ratings(content_id);
CREATE INDEX idx_ratings_user_id ON ratings(user_id);

-- Analytics & Events
CREATE TABLE playback_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    profile_id UUID REFERENCES user_profiles(id) ON DELETE SET NULL,
    content_id UUID REFERENCES content(id) ON DELETE SET NULL,
    episode_id UUID REFERENCES episodes(id) ON DELETE SET NULL,
    event_type VARCHAR(50) NOT NULL, -- play, pause, seek, complete, buffer
    position_seconds INT,
    quality VARCHAR(20),
    bitrate_kbps INT,
    buffer_duration_ms INT,
    session_id UUID,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX idx_playback_events_user_id ON playback_events(user_id, created_at);
CREATE INDEX idx_playback_events_content_id ON playback_events(content_id, created_at);
CREATE INDEX idx_playback_events_created_at ON playback_events(created_at);

-- Transcoding Jobs
CREATE TABLE transcode_jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    content_id UUID REFERENCES content(id) ON DELETE SET NULL,
    episode_id UUID REFERENCES episodes(id) ON DELETE SET NULL,
    input_s3_key VARCHAR(500) NOT NULL,
    output_s3_prefix VARCHAR(500) NOT NULL,
    status VARCHAR(50) NOT NULL DEFAULT 'pending', -- pending, processing, completed, failed
    qualities TEXT[] NOT NULL, -- ['480p', '720p', '1080p']
    progress_percent INT DEFAULT 0,
    error_message TEXT,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX idx_transcode_jobs_status ON transcode_jobs(status, created_at);
CREATE INDEX idx_transcode_jobs_content_id ON transcode_jobs(content_id);
```

### Redis Data Structures

```redis
# Session Cache
sessions:{token_hash} -> JSON(user_id, profile_id, expires_at)
TTL: 15 minutes (access token)

# User Cache
users:{user_id} -> JSON(user object)
TTL: 1 hour

# Content Cache
content:{content_id} -> JSON(content object)
TTL: 6 hours

# Playback Progress Cache (write-through)
progress:{profile_id}:{content_id} -> JSON(position_seconds, last_watched_at)
TTL: 7 days

# Rate Limiting
ratelimit:api:{user_id}:{window} -> counter
TTL: 1 minute

ratelimit:ip:{ip_address}:{window} -> counter
TTL: 1 minute

# Trending Content (sorted set)
trending:content -> ZSET(content_id, view_count_last_24h)
TTL: 1 hour

# Search Suggestions Cache
search:suggest:{prefix} -> LIST(suggestions)
TTL: 1 hour

# Presigned URL Cache
presigned:{content_id}:{quality} -> JSON(url, expires_at)
TTL: 50 minutes
```

### Elasticsearch Schema

```json
{
  "mappings": {
    "properties": {
      "content_id": { "type": "keyword" },
      "title": { 
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": { "type": "keyword" },
          "autocomplete": {
            "type": "text",
            "analyzer": "autocomplete",
            "search_analyzer": "standard"
          }
        }
      },
      "description": { "type": "text" },
      "type": { "type": "keyword" },
      "genres": { "type": "keyword" },
      "cast": { "type": "text" },
      "director": { "type": "text" },
      "release_year": { "type": "integer" },
      "rating": { "type": "keyword" },
      "language": { "type": "keyword" },
      "country": { "type": "keyword" },
      "imdb_rating": { "type": "float" },
      "view_count": { "type": "long" },
      "published_at": { "type": "date" }
    }
  },
  "settings": {
    "analysis": {
      "analyzer": {
        "autocomplete": {
          "tokenizer": "autocomplete",
          "filter": ["lowercase"]
        }
      },
      "tokenizer": {
        "autocomplete": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10,
          "token_chars": ["letter", "digit"]
        }
      }
    }
  }
}
```

### Vector Database Schema (Qdrant)

```json
{
  "collections": [
    {
      "name": "content_embeddings",
      "vectors": {
        "size": 512,
        "distance": "Cosine"
      },
      "payload_schema": {
        "content_id": "keyword",
        "title": "text",
        "genres": "keyword[]",
        "release_year": "integer",
        "view_count": "integer"
      }
    },
    {
      "name": "user_embeddings",
      "vectors": {
        "size": 512,
        "distance": "Cosine"
      },
      "payload_schema": {
        "user_id": "keyword",
        "watched_genres": "keyword[]",
        "favorite_actors": "text[]",
        "avg_rating": "float"
      }
    }
  ]
}
```

---

## 🏗️ High-Level Architecture

See [architecture.md](./architecture.md) for the detailed Mermaid diagram.

**Key Components:**
1. **Client Layer**: React web app, mobile apps, admin dashboard
2. **Edge Layer**: CDN (CloudFront/Cloudflare), WAF
3. **API Gateway**: REST endpoints, rate limiting, authentication
4. **Microservices**: 8 Go gRPC services (Auth, User, Content, Playback, Search, Reco, Upload, Analytics)
5. **Workers**: Transcoding, thumbnail generation, ASR, moderation, embeddings
6. **Event Bus**: Kafka for async processing
7. **Data Layer**: PostgreSQL, Redis, S3, Elasticsearch, Vector DB
8. **AI/ML**: Recommendation engine, ASR (Whisper), Vision API
9. **Observability**: Prometheus, Grafana, Jaeger, ELK

---

## 🔍 Detailed Component Design

### 1. Video Transcoding Pipeline

**Flow:**
```
Upload → S3 → Kafka Event → Transcode Worker → ffmpeg
  → Multiple qualities (480p, 720p, 1080p, 4K)
  → HLS segments (.ts files) + manifest (.m3u8)
  → Upload to S3
  → Update database
  → Kafka completion event
```

**Transcode Worker (Go):**
```go
type TranscodeWorker struct {
    kafka   *kafka.Consumer
    s3      *s3.Client
    db      *sql.DB
    ffmpeg  *FFmpegClient
}

func (w *TranscodeWorker) ProcessJob(job TranscodeJob) error {
    // 1. Download source from S3
    inputPath := w.downloadFromS3(job.InputS3Key)
    
    // 2. Transcode to multiple qualities in parallel
    qualities := []string{"480p", "720p", "1080p", "4k"}
    var wg sync.WaitGroup
    for _, quality := range qualities {
        wg.Add(1)
        go func(q string) {
            defer wg.Done()
            w.transcodeQuality(inputPath, q, job.OutputS3Prefix)
        }(quality)
    }
    wg.Wait()
    
    // 3. Upload HLS files to S3
    // 4. Update database with streaming_assets
    // 5. Publish completion event to Kafka
}
```

**ffmpeg Command:**
```bash
ffmpeg -i input.mp4 \
  -c:v libx264 -preset medium -crf 23 \
  -vf scale=1920:1080 -b:v 5000k -maxrate 5350k -bufsize 7500k \
  -c:a aac -b:a 192k -ar 48000 \
  -hls_time 6 -hls_playlist_type vod \
  -hls_segment_filename "1080p_%03d.ts" \
  1080p.m3u8
```

### 2. Playback Service Design

**Responsibilities:**
- Verify user entitlement (subscription, geo-restrictions)
- Generate presigned S3 URLs for HLS manifest
- Track playback sessions
- Resume from last position

**Code Flow:**
```go
func (s *PlaybackService) StartPlayback(ctx context.Context, req *StartPlaybackRequest) (*StartPlaybackResponse, error) {
    // 1. Validate user subscription
    user := s.authClient.GetUser(req.UserId)
    if !user.HasActiveSubscription() {
        return nil, ErrSubscriptionRequired
    }
    
    // 2. Check geo-restrictions
    if !s.isAvailableInRegion(req.ContentId, user.Country) {
        return nil, ErrContentNotAvailable
    }
    
    // 3. Get streaming asset (quality-based)
    asset := s.db.GetStreamingAsset(req.ContentId, req.Quality)
    
    // 4. Generate presigned URL (1 hour expiry)
    manifestURL := s.s3.PresignURL(asset.S3Key, 1*time.Hour)
    
    // 5. Get resume position from cache/DB
    progress := s.getPlaybackProgress(req.UserId, req.ContentId)
    
    // 6. Generate DRM token (if premium content)
    drmToken := s.drmClient.GenerateToken(req.ContentId, req.UserId)
    
    // 7. Log playback start event
    s.analyticsClient.TrackEvent("playback_start", map[string]interface{}{
        "user_id": req.UserId,
        "content_id": req.ContentId,
        "quality": req.Quality,
    })
    
    return &StartPlaybackResponse{
        ManifestURL: manifestURL,
        DRMToken: drmToken,
        ExpiresAt: time.Now().Add(1 * time.Hour).Unix(),
        ResumePositionSeconds: progress.PositionSeconds,
    }, nil
}
```

### 3. Recommendation Engine

**Approaches:**

**A. Collaborative Filtering (User-based)**
- Find similar users based on watch history
- Recommend content watched by similar users

**B. Content-based Filtering**
- Use vector embeddings (genres, cast, plot)
- Find similar content via cosine similarity

**C. Hybrid Approach**
- Combine both methods
- Use ML model to weight results

**Implementation:**
```go
func (s *RecommendationService) GetRecommendations(ctx context.Context, userID string) ([]Recommendation, error) {
    // 1. Get user watch history
    watchHistory := s.db.GetWatchHistory(userID)
    
    // 2. Get user embedding vector
    userVector := s.getUserEmbedding(userID, watchHistory)
    
    // 3. Query vector DB for similar content
    similarContent := s.vectorDB.Search(userVector, limit=100)
    
    // 4. Filter out already watched
    filtered := s.filterWatched(similarContent, watchHistory)
    
    // 5. Rank by predicted rating
    ranked := s.mlModel.RankByPredictedRating(userID, filtered)
    
    // 6. Diversify results (genre diversity)
    diversified := s.diversifyByGenre(ranked, targetGenres=5)
    
    // 7. Add trending content for cold start
    if len(watchHistory) < 5 {
        trending := s.cache.GetTrending(limit=10)
        diversified = append(trending, diversified...)
    }
    
    return diversified[:20], nil
}
```

### 4. Search Service Design

**Elasticsearch Query:**
```json
{
  "query": {
    "bool": {
      "must": [
        {
          "multi_match": {
            "query": "inception",
            "fields": ["title^3", "description", "cast", "director"],
            "type": "best_fields",
            "fuzziness": "AUTO"
          }
        }
      ],
      "filter": [
        { "term": { "status": "published" } },
        { "terms": { "genres": ["sci-fi", "thriller"] } },
        { "range": { "release_year": { "gte": 2010, "lte": 2024 } } }
      ]
    }
  },
  "sort": [
    { "_score": "desc" },
    { "view_count": "desc" },
    { "imdb_rating": "desc" }
  ],
  "from": 0,
  "size": 20
}
```

**Autocomplete Query:**
```json
{
  "suggest": {
    "title-suggest": {
      "prefix": "inc",
      "completion": {
        "field": "title.autocomplete",
        "size": 10,
        "fuzzy": { "fuzziness": 1 }
      }
    }
  }
}
```

---

## ⚖️ Trade-offs & Design Decisions

### 1. REST vs gRPC
**Decision**: REST for external APIs, gRPC for internal microservices

**Trade-offs:**
- ✅ REST: Browser-friendly, easy debugging, HTTP caching
- ✅ gRPC: 7x faster, type-safe, bi-directional streaming
- ❌ REST: Slower, no type safety, larger payloads
- ❌ gRPC: Requires HTTP/2, harder to debug (binary)

### 2. HLS vs DASH
**Decision**: Use HLS (with DASH support for Android)

**Trade-offs:**
- ✅ HLS: Native iOS support, simpler, better CDN caching
- ✅ DASH: Open standard, more flexible DRM
- ❌ HLS: Proprietary (Apple), limited on older Android
- ❌ DASH: Requires polyfills, less CDN-optimized

**Solution**: Transcode to both formats, serve based on device

### 3. Pre-transcoding vs On-demand Transcoding
**Decision**: Pre-transcode all qualities on upload

**Trade-offs:**
- ✅ Pre-transcode: Instant playback, predictable costs
- ✅ On-demand: Save storage, only transcode popular content
- ❌ Pre-transcode: High storage costs (4x per video)
- ❌ On-demand: Slow first playback, unpredictable costs

**Hybrid Solution**: 
- Pre-transcode popular content (top 20%)
- On-demand for long-tail content
- Delete unpopular qualities after 90 days

### 4. Microservices vs Monolith
**Decision**: Microservices with gRPC

**Trade-offs:**
- ✅ Microservices: Independent scaling, team autonomy, fault isolation
- ✅ Monolith: Simpler deployment, lower latency, easier transactions
- ❌ Microservices: Distributed complexity, network overhead, debugging harder
- ❌ Monolith: Tight coupling, difficult to scale independently

**When to use monolith**: Early MVP, small team (<10 devs)
**When to use microservices**: Scale (>10M users), multiple teams

### 5. PostgreSQL vs MongoDB
**Decision**: PostgreSQL

**Trade-offs:**
- ✅ Postgres: ACID, complex queries, mature, JSON support (JSONB)
- ✅ MongoDB: Flexible schema, horizontal sharding, faster writes
- ❌ Postgres: Harder to shard, slower for very high write throughput
- ❌ MongoDB: No ACID across collections, complex queries harder

**When to use MongoDB**: High write throughput, flexible schema, simple queries

### 6. Kafka vs RabbitMQ
**Decision**: Kafka

**Trade-offs:**
- ✅ Kafka: Event log, replay, high throughput, durable
- ✅ RabbitMQ: Simpler, better for work queues, lower latency
- ❌ Kafka: Complex ops, overkill for simple queues, higher resource usage
- ❌ RabbitMQ: No event replay, harder to scale, messages deleted on consumption

**When to use RabbitMQ**: Simple queues, request-reply patterns, low volume

### 7. Self-hosted vs Managed Services
**Decision**: Managed services (RDS, ElastiCache, EKS, S3)

**Trade-offs:**
- ✅ Managed: Less ops, auto-backups, HA out of box, faster setup
- ✅ Self-hosted: Lower cost at scale, full control, no vendor lock-in
- ❌ Managed: Higher cost (2-3x), less control, vendor lock-in
- ❌ Self-hosted: Complex ops, need DBA/SRE, slower setup

**Hybrid**: Use managed for MVP, migrate to self-hosted at scale (>$100k/month)

---

## 📈 Scalability & Performance

### Horizontal Scaling

**API Gateway:**
- Stateless design (JWT validation from cache)
- Load balancer (ALB/NLB) in front
- Auto-scaling group (3-100 instances)
- Scale on CPU (target: 70%)

**Microservices:**
- Stateless gRPC servers
- Kubernetes HPA (Horizontal Pod Autoscaler)
- Scale on custom metrics (RPS, gRPC latency)
- Circuit breakers for cascading failures

**Workers:**
- Kafka consumer groups (multiple consumers per topic)
- Scale based on consumer lag
- Spot instances for cost savings

### Database Scaling

**PostgreSQL:**
- **Vertical scaling**: Up to 128 vCPUs (db.r6g.32xlarge)
- **Read replicas**: 5+ replicas for read-heavy workloads
- **Partitioning**: Partition `playback_events` by date (monthly)
- **Sharding** (if >10TB): Shard by `user_id` or `content_id`

**Redis:**
- **Cluster mode**: 90 shards (max 500 nodes)
- **Sharding**: Hash-based on key prefix
- **Read replicas**: 5 per shard

**Elasticsearch:**
- **Sharding**: 5 primary shards, 2 replicas = 15 shards total
- **Index per day/week**: For time-series data (logs, events)
- **Hot-warm-cold**: Hot nodes (SSD) for recent, cold (HDD) for old

### Caching Strategy

**Multi-layer caching:**
1. **Browser cache**: Static assets (7 days)
2. **CDN cache**: Video segments (24 hours), thumbnails (7 days)
3. **Redis cache**: API responses (1-60 min), sessions (15 min)
4. **Application cache**: In-memory LRU for hot data

**Cache invalidation:**
- Content metadata: 6 hours TTL, invalidate on update
- User data: 1 hour TTL, invalidate on profile update
- Playback progress: Write-through cache (sync to DB every 10 seconds)

### CDN Optimization

**CloudFront distribution:**
- **Origin**: S3 bucket (private, presigned URLs)
- **Cache behaviors**: 
  - Video segments: Cache 24 hours
  - Manifests: Cache 1 minute (for live streams)
  - Thumbnails: Cache 7 days
- **Geo-restrictions**: Block countries without licensing
- **Price class**: Use all edge locations for premium, cheaper for free tier
- **Compression**: Enable gzip for JSON APIs

---

## 🔒 Security Considerations

### Authentication & Authorization
- **JWT tokens**: RS256 signing, 15-min access, 7-day refresh
- **Token rotation**: Refresh before expiry
- **Revocation**: Store revoked tokens in Redis (TTL = expiry)
- **MFA**: TOTP (Time-based One-Time Password) for admin accounts

### API Security
- **Rate limiting**: 100 req/min per IP, 1000 req/min per user
- **API keys**: For mobile apps, rotate every 90 days
- **WAF rules**: Block SQL injection, XSS, known bad IPs
- **CORS**: Whitelist domains (*.yourapp.com)
- **CSRF**: SameSite cookies, CSRF tokens for state-changing requests

### Data Protection
- **Encryption at rest**: AES-256 for S3, RDS, EBS
- **Encryption in transit**: TLS 1.3 everywhere
- **PII handling**: Hash/encrypt email, mask credit cards
- **Secrets management**: AWS Secrets Manager, rotate every 90 days
- **Database access**: IAM roles, no hardcoded passwords

### DRM & Content Protection
- **Widevine** (Android/Chrome), **FairPlay** (iOS/Safari)
- **License server**: Issue time-bound licenses (24 hours)
- **Watermarking**: Forensic watermarking to trace leaks
- **HDCP**: Require HDCP for 4K content
- **Screen capture blocking**: On mobile apps

### Compliance
- **GDPR**: Right to deletion, data export, consent management
- **CCPA**: Opt-out of data sales, privacy policy
- **PCI-DSS**: If handling payments (use Stripe/Braintree)
- **SOC 2**: Audit logging, access controls, incident response

---

## 📊 Monitoring & Observability

### Metrics (Prometheus)

**Golden Signals:**
- **Latency**: p50, p95, p99 for all API endpoints
- **Traffic**: Requests per second (RPS)
- **Errors**: Error rate (5xx, 4xx breakdown)
- **Saturation**: CPU, memory, disk, network utilization

**Custom Metrics:**
- `video_startup_latency_seconds{quality="1080p"}`: Time to first frame
- `buffering_ratio{content_id="xyz"}`: % of time buffering
- `transcoding_jobs_pending`: Queue depth
- `recommendation_click_through_rate`: CTR on recommendations
- `cdn_cache_hit_ratio`: CDN efficiency

### Logging (ELK Stack)

**Structured JSON logs:**
```json
{
  "timestamp": "2026-04-22T10:30:45Z",
  "level": "info",
  "service": "playback-service",
  "trace_id": "a1b2c3d4...",
  "span_id": "e5f6g7h8...",
  "user_id": "user-123",
  "content_id": "movie-456",
  "event": "playback_started",
  "quality": "1080p",
  "latency_ms": 145
}
```

**Log levels:**
- **DEBUG**: Verbose, dev only
- **INFO**: Normal operations (playback started, user logged in)
- **WARN**: Recoverable errors (cache miss, retry)
- **ERROR**: Application errors (DB connection failed, external API timeout)
- **FATAL**: Critical failures (service crash)

### Tracing (Jaeger)

**Distributed trace example:**
```
API Gateway [200ms]
  └─> Auth Service (gRPC) [10ms]
  └─> Playback Service (gRPC) [150ms]
      └─> PostgreSQL query [20ms]
      └─> Redis cache [5ms]
      └─> S3 presign [2ms]
      └─> DRM service (gRPC) [100ms]
```

**Sampling:**
- 100% for errors and slow requests (>1s)
- 10% for successful requests
- 1% for internal health checks

### Alerts

**Critical (PagerDuty):**
- Service down (0 healthy instances)
- Error rate > 1% (5-minute window)
- p95 latency > 1s
- Database CPU > 90%
- Kafka consumer lag > 100k

**Warning (Slack):**
- Error rate > 0.5%
- p99 latency > 2s
- Transcode queue depth > 1000
- CDN cache hit rate < 90%

### Dashboards (Grafana)

1. **Platform Overview**: RPS, error rate, latency, active users
2. **Video Performance**: Startup latency, buffering ratio, quality distribution
3. **Service Health**: CPU, memory, pods, error rate per service
4. **Data Layer**: DB connections, query time, cache hit ratio
5. **Business Metrics**: New signups, watch hours, revenue

---

## 📝 Summary

This system design analysis covers:

✅ **Functional Requirements**: 10 core features (auth, catalog, playback, search, recommendations, etc.)  
✅ **Non-Functional Requirements**: Performance, scalability, availability, security, cost  
✅ **Capacity Planning**: 100M users, 10M concurrent, 11 PB storage, $637k/month  
✅ **API Design**: REST (external) + gRPC (internal) with detailed contracts  
✅ **Database Schema**: PostgreSQL, Redis, Elasticsearch, Vector DB schemas  
✅ **Component Design**: Transcoding pipeline, playback service, recommendations, search  
✅ **Trade-offs**: REST vs gRPC, HLS vs DASH, microservices vs monolith  
✅ **Scalability**: Horizontal scaling, database sharding, caching, CDN  
✅ **Security**: Auth, DRM, encryption, compliance (GDPR, CCPA)  
✅ **Observability**: Metrics, logging, tracing, alerting

---

**Next Steps:**
1. Review and refine requirements with stakeholders
2. Create protobuf files for gRPC services → [See proto-definitions.md](./proto-definitions.md)
3. Implement Phase 1 (Core Infrastructure) → [See implementation-plan.md](./implementation-plan.md)
4. Set up CI/CD pipeline and infrastructure (Terraform)
5. Build MVP and iterate based on user feedback

---

**Last Updated**: April 22, 2026  
**Version**: 1.0  
