# Social Network - CS50 Project 4

A Django-based social network application built for CS50's Web Programming course. Users can create posts, follow other users, like posts, edit their own posts, and view a personalized feed of posts from users they follow.

## Features

### Core Functionality
- **Authentication**: User registration, login, and logout
- **Posts**: Create, view, and edit text-based posts (max 256 characters)
- **Follow System**: Follow/unfollow other users with real-time follower/following counts
- **Likes**: Asynchronous like/unlike functionality with live count updates
- **Profile Pages**: View any user's profile with their posts, follower count, and follow button
- **Following Feed**: Dedicated page showing posts only from followed users
- **Pagination**: All post listings paginated at 10 posts per page

### Technical Highlights

#### React-Powered Interactive Components
The project uses React (via CDN) for two key interactive features without full page reloads:
- **Inline Post Editing**: Click "Edit" on your own post → textarea appears → click "Save" → PUT request to `/edit` endpoint → UI updates instantly
- **Like/Unlike Toggle**: Click like button → PUT request to `/like` endpoint → count and button state update immediately

Both components are mounted via `ReactDOM.render()` targeting specific DOM elements with data attributes.

#### Data Models
```
User (AbstractUser)
  ├── followers (ManyToMany via Follow)
  ├── followings (ManyToMany via Follow)
  └── liked_posts (ManyToMany via Like)

Post
  ├── user (ForeignKey)
  ├── text (CharField, 256 max)
  ├── likes (ManyToMany via Like)
  └── timestamp (auto_now_add)

Like (through model)
  ├── user + post (unique_together)
  └── prevents duplicate likes

Follow (through model)
  ├── followed + following (unique_together)
  └── prevents duplicate follows
```

#### API Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | All posts (paginated) |
| GET | `/profile/<username>` | User profile + posts |
| GET | `/following` | Posts from followed users (auth required) |
| POST | `/post` | Create new post |
| POST | `/follow` | Follow/unfollow user |
| PUT | `/edit` | Edit own post (JSON) |
| PUT | `/like` | Like/unlike post (JSON) |

#### Security
- `@login_required` decorators on all mutating endpoints
- Ownership verification in `edit()` view prevents editing others' posts
- CSRF protection on forms, `@csrf_exempt` only on JSON API endpoints
- `unique_together` constraints prevent duplicate likes/follows at DB level

## Setup

```bash
# Create virtual environment
python -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install django

# Run migrations
python manage.py migrate

# Start server
python manage.py runserver
```

Visit `http://127.0.0.1:8000/`

## Project Structure
```
network/
├── models.py          # User, Post, Like, Follow models
├── views.py           # All view logic + API endpoints
├── urls.py            # URL routing
├── templates/network/
│   ├── layout.html    # Base template + React CDN includes
│   ├── index.html     # All posts + new post form
│   ├── profile.html   # User profile + posts
│   ├── following.html # Following feed
│   ├── login.html
│   └── register.html
└── static/network/
    ├── styles.css     # Custom styling
    └── networkReact.js # React components (TextApp, LikesApp)
```

## What Makes This Project Unique

1. **Hybrid Django + React Architecture**: Instead of a full SPA or pure server-rendered templates, it uses Django for routing/auth/data and React only for the two interactive components that benefit from client-side state (edit/like). This demonstrates pragmatic technology selection.

2. **Through Models for Complex Relationships**: Both `Like` and `Follow` use explicit through models with `unique_together` constraints, enabling metadata (timestamps could be added) and preventing race conditions at the database level.

3. **Efficient Query Patterns**: Views use `.values()` for list views to avoid unnecessary object instantiation, while still fetching related data (usernames, like counts, like status) in loops - a practical balance between ORM convenience and performance.

4. **Pagination Everywhere**: All three post views (index, profile, following) implement consistent 10-per-page pagination with previous/next navigation.

5. **CS50 Specification Compliance**: Implements every requirement from the spec including the specific UI behaviors (edit-in-place, async likes, follow toggle, pagination, following feed).