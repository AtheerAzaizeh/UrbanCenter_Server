# UrbanCenter — Civic Issue Reporting Platform (Server)

The REST API and real-time server behind UrbanCenter, a platform where citizens report city problems and city departments handle them. Built with Node.js, Express, MongoDB, Socket.IO and AWS S3.

The React client is in **[UrbanCenter_Client](https://github.com/AtheerAzaizeh/UrbanCenter_Client)**.

---

## Features

- **REST API** for users, reports and comments, built with Express and Mongoose.
- **Authentication and roles.** Passwords are hashed with bcrypt and sessions use JSON Web Tokens. Users are either citizens or authority users, and each authority user is linked to one report category and only receives the reports in that category.
- **Report lifecycle.** Every report has a category and issue type (7 categories, 21 issue types, validated in the schema), a status (`pending`, `in_progress`, `resolved`) and a priority (`low`, `medium`, `high`). Status updates can carry a comment and an image.
- **Image pipeline.** Uploads are received with Multer, resized with Sharp, stored in AWS S3 and returned to clients through time-limited pre-signed URLs. Deleting a report also deletes its image from S3.
- **Real-time notifications.** The server keeps a map of online users and pushes Socket.IO events only to the people concerned: the authority users of a report's category when a report is created, and the citizen when their report is updated.

## Tech stack

Node.js · Express 4 · MongoDB with Mongoose 8 · Socket.IO 4 · JSON Web Tokens · bcrypt · Multer · Sharp · AWS SDK v3 (S3 and pre-signed URLs) · moment-timezone

## API

### Users

| Method | Endpoint | Description |
| --- | --- | --- |
| POST | `/api/users/register` | Register a citizen or an authority user |
| POST | `/api/users/login` | Sign in and receive a JWT |

### Reports

| Method | Endpoint | Description |
| --- | --- | --- |
| POST | `/api/reports/report` | Create a report (multipart form with an optional image) |
| GET | `/api/reports` | List all reports |
| GET | `/api/reports/report/user/:user_id` | Reports submitted by a citizen |
| GET | `/api/reports/report/category/:user_id` | Reports in an authority user's category |
| GET | `/api/reports/report/:id` | Pre-signed URL for a report's image |
| PUT | `/api/reports/report/:id` | Update a report (multipart) |
| PUT | `/api/reports/report/status/:id` | Update status and priority, with an optional comment image (requires JWT) |
| DELETE | `/api/reports/report/:id` | Delete a report and its image |

### Comments

| Method | Endpoint | Description |
| --- | --- | --- |
| POST | `/api/comments/add` | Add a comment to a report |
| GET | `/api/comments/report/category_comment/:user_id/:category` | Comments by user and category |
| DELETE | `/api/comments/report/category_comment/:report_id` | Delete the comments of a report |

### Socket.IO events

| Direction | Event | Purpose |
| --- | --- | --- |
| Client → server | `registerUser` | Link the user's id to their socket |
| Server → client | `reportAdded` | A new report arrived in the authority user's category |
| Server → client | `reportUpdated` | The status or priority of a report changed |
| Server → client | `reportDeleted` | A report was removed |

## Getting started

**Prerequisites:** Node.js 18+, a MongoDB database (local or Atlas) and an AWS S3 bucket.

```bash
git clone https://github.com/AtheerAzaizeh/UrbanCenter_Server.git
cd UrbanCenter_Server
npm install
cp .env.example .env    # then fill in your own values
npm run dev             # or: npm start
```

### Environment variables

| Variable | Description |
| --- | --- |
| `MONGO_URL` | MongoDB connection string |
| `JWT_SECRET_KEY` | Secret used to sign and verify JWTs |
| `JWT_SECRET` | Same value as `JWT_SECRET_KEY` (used by the User model) |
| `AWS_REGION` | Region of the S3 bucket |
| `AWS_ACCESS_KEY_ID` | Access key of an IAM user with access to the bucket |
| `AWS_SECRET_ACCESS_KEY` | Secret key of that IAM user |
| `S3_BUCKET_NAME` | Bucket where report images are stored |
| `PORT` | Port to listen on (defaults to 3000) |

Never commit your `.env` file; it is already listed in `.gitignore`.

## Deployment

The server was deployed on Render. It also serves the React production build from `view/build`, so the whole app runs from one service.

## Project structure

```
├── controllers/   # Users, reports and comments logic
├── middlewares/   # JWT authentication, Multer upload
├── models/        # Mongoose schemas: User, Report, Comment
├── routers/       # Express routes
├── view/build/    # React production build served by Express
├── config.js      # MongoDB connection
└── index.js       # Express app, Socket.IO server and route mounting
```

## Author

**Atheer Azaizeh** · [github.com/AtheerAzaizeh](https://github.com/AtheerAzaizeh)
