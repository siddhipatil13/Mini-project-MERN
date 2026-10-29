# LearnSpace LMS

A responsive MERN learning platform starter. Administrators and instructors can manage courses and YouTube lessons from the Studio. Students can register, enroll, watch lessons, mark lessons complete, and track course progress.

## Requirements

- Node.js 20 or newer and npm
- MongoDB Community Server running locally, or a MongoDB Atlas connection string
- VS Code (optional)

## Windows setup

1. Extract the ZIP and open the `LearnSpace-LMS` folder in VS Code.
2. Open PowerShell in that folder.
3. Copy the environment template: `Copy-Item server/.env.example server/.env`
4. Edit `server/.env`. Set `MONGODB_URI` to your MongoDB connection string and replace `JWT_SECRET` with a long random value. Set an administrator email/password that you will remember.
5. Install dependencies: `npm run install:all`
6. Ensure MongoDB is running, then create the first admin, sample course and programming-language starter courses: `npm --prefix server run seed`
7. Start the app: `npm run dev`
8. Visit [http://localhost:5173](http://localhost:5173). Sign in with the admin email and password from `server/.env`.

The API runs at `http://localhost:5000`. The sample course is public and includes a sample YouTube lesson. Change its content from Studio.

## Add and remove YouTube videos

Sign in as admin, choose **Studio**, create or select a course, then choose **Add video**. Enter a lesson title and YouTube watch URL (or `youtu.be` link). You can edit lesson metadata/URL, move lessons up/down, and remove lessons. Publishing exposes the course in the catalog. Removing a lesson removes its database entry and clears progress references; it does not delete the video from YouTube.

To create an instructor, change a user's role to `instructor` in MongoDB for this starter. Public registration always creates students. Never expose a public admin-registration route.

## Useful commands

- `npm run dev` — start frontend and backend in development
- `npm --prefix server run seed` — create the initial admin and sample course (safe to rerun; existing admin password is not reset)
- `npm --prefix client run build` — build the frontend for production
- `npm --prefix server start` — run the API

## Current scope

Working core: account registration/login, role checks, public catalog/search, course CRUD, publishing, YouTube lesson CRUD/reordering, enrollment, sequential video completion, course quizzes, per-video coding tests, certificates, and responsive UI with a light/dark toggle.

This starter does not include assignments, password reset, file/video uploads, AI, or deployment automation. Every published course must have an instructor-authored final quiz. YouTube videos are embedded; the project does not host or own video media. Configure valid MongoDB and change the sample credentials before exposing the app online. Keep API secrets only in the server `.env` file.


## Learning rules

**Videos cannot be skipped.** The YouTube seek bar and keyboard shortcuts are hidden; the player only has play/pause, 10-second back/forward buttons and full screen. The browser reports playback every 5 seconds and the **server** decides what counts: only continuous 1x playback is credited, jumps earn nothing, and credit can never exceed real time spent playing. A lesson completes only after 100% of its duration has been watched through continuous, normal-speed playback. Progress is counted on the server and cannot be earned by skipping forward. Lessons unlock one by one. Leaving the tab pauses the video.

**Coding test.** Every video has a coding challenge after completion. Students can run a sample, submit code against hidden test cases and see their score. They must pass before the next lesson unlocks. Copy/paste is blocked, and detected tab/window switches or fullscreen exits end the attempt. In Studio, choose a lesson under Video coding tests to customize its language, prompt, starter code and grading cases. A default square-a-number task is supplied until customized.

**Quiz.** Admin/instructor creates it in Studio (select a course, then "Final quiz"). Students unlock it at 100% of lessons. It runs full screen with a timer; switching tabs/windows or leaving full screen ends the attempt as failed. Fullscreen must be available to start. Copy/paste/right-click are blocked. Answers are checked on the server and are never sent to the browser.

**Certificate.** Issued automatically when a student reaches 75% of the lessons. They can view it, download it as a real PDF, print it, or copy its link (lesson page > Your goals, or My learning > My certificates). Each has a unique ID that anyone can check at `/certificate/<ID>`. The certificate unlocks at exactly 75% course completion. Configure one quiz for each course in Studio before publishing; quizzes unlock when all lessons are complete.

**Video lengths.** When an admin opens Studio, the length of each new video is detected automatically (keep Studio open for a few seconds after adding lessons). Students see "lesson not ready" until then.

Video lessons count only when fully watched; certificates unlock at 75% completion. The final quiz unlocks when all video lessons are complete.

## Known limits (be honest in your viva)

- A website cannot truly stop someone switching tabs; it detects it and penalises it. For high-stakes exams use proctoring software.
- Someone who writes their own script could still send fake playback reports, but would need to spend the full video length in real time doing it.
- YouTube videos must allow embedding.

## Emails (v4)

| Who | When | Email |
|---|---|---|
| Admin(s) and the course owner | A student enrolls | Who joined, their email, and how many learners the course has now |
| Student | Enrolls | Welcome |
| Student | Reaches 50% | "Halfway there" |
| Student | Reaches 75% | Certificate ready (view link + PDF download link) |
| Student | Reaches 100% | Course complete (plus the quiz link, if there is a quiz) |

Each email is sent once. Without SMTP settings the app still works: emails are printed in the server terminal.

**Turn on real emails with Gmail**
1. Google Account > Security > turn on 2-Step Verification.
2. Security > App passwords > create one called LearnSpace. Copy the 16 letters (no spaces).
3. Add these lines to `server/.env` and restart the server:
```
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=yourname@gmail.com
SMTP_PASS=your16letterapppassword
MAIL_FROM=LearnSpace <yourname@gmail.com>
ADMIN_NOTIFY_EMAIL=yourname@gmail.com
API_PUBLIC_URL=http://localhost:5000
```
4. Open Studio and use "Send test email" on the Email notifications panel.

`ADMIN_NOTIFY_EMAIL` matters: placeholder addresses such as admin@example.com are skipped, so set it to your real email to receive "new enrollment" mails.

## Studio additions (v4)

- **Enrolled students** panel under each course: name, email, join date, progress, certificate, quiz result, and a Download CSV button.
- **Email notifications** panel with a test button.

## Known limits

- The PDF certificate uses built-in PDF fonts, so student names must use Latin letters (accents are fine). The on-screen certificate and Print work for any script.
- Students who were already past 75% before this update get their certificate when they next finish a lesson, or by pressing Get on the lesson page.

## History

- v2: locked lessons no longer leak YouTube IDs, expired logins sign out cleanly, admins get Studio buttons instead of Enroll, API URL via `client/.env`.
- v3: videos cannot be skipped, final quiz with tab-switch protection, certificates.
- v4: real PDF download, automatic certificate at 75%, email notifications, enrolled-students panel.


## Coding compiler setup

The app sends student programs to a Piston sandbox you run locally. This keeps submissions on your own machine. Install Docker Desktop with its Linux/WSL2 engine, then follow Piston setup from its official repository:

1. Clone the Piston repository at https://github.com/engineer-man/piston and start its API container with docker compose up -d api from the Piston folder.
2. Install and use Piston CLI to install runtimes for Python, JavaScript, C, C++ and Java.
3. Set CODE_RUNNER_URL=http://localhost:2000/api/v2 in this app server/.env, then restart LearnSpace.

The editor supports Python, JavaScript, C, C++ and Java. Install each runtime on Piston before using that language. Existing course records are preserved; the seed command adds Python, JavaScript, C++ and Java starter courses only when a course with the same title is absent.
