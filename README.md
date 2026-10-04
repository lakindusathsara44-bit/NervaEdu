# NervaEdu

NervaEdu is a small, self-hosted learning management system for a school. It includes student and teacher accounts, teacher profiles, subject discovery, videos and PDF resources, and six-question multiple-choice quizzes.

## Run it

1. Install Node.js 22 or newer on the computer that will host the app.
2. Copy `.env.example` to `.env`, then set a private account-manager phone number and password. The server creates the protected admin account on first start and stores only its salted password hash in the database.
3. On Windows, double-click `start-nervaedu.bat`. It starts the local server and opens the app.

You can also open a terminal in this folder, run `node server.js` (or `npm start`), then visit [http://localhost:4173](http://localhost:4173). Do not open `index.html` directly as a `file://` page; the account and upload features require the server.

Without Firebase configuration, the server creates `data/nervaedu.json` for accounts and learning records. Uploaded profile photos, videos and PDFs are stored in `data/uploads/`. Keep the `data/` folder when backing up the local edition. Do not publish it or commit real student information to a public repository.

## Use Cloud Firestore

The backend can store users, teacher choices, resource metadata and quizzes in Cloud Firestore. Configure ImageKit below for cloud-hosted videos and papers before deploying multiple server instances.

1. Create a Firebase project and enable Cloud Firestore in the Firebase console.
2. Copy `.env.example` to `.env` and set `NERVAEDU_DATABASE=firestore` and `FIREBASE_PROJECT_ID`. Set the account-manager phone and private password in `.env`; the server stores only a salted password hash in its database.
3. For local development, create a Firebase service account key and store it outside this repository. Set `GOOGLE_APPLICATION_CREDENTIALS` in `.env` to its full path. Never commit the key. On Google Cloud, use Application Default Credentials from the server's service identity instead.
4. Install dependencies with `npm install`, then start NervaEdu with `npm start`.

The Admin SDK uses privileged server credentials. `firestore.rules` denies all direct browser access; keep that protection in place so password hashes and student details are not exposed. The first run reads Firestore as the source of truth. To deliberately import existing local records into an empty Firestore database, set `FIREBASE_IMPORT_LOCAL=true` for that first run. Review the local records before enabling this because it uploads account and student information to your Firebase project.

Firestore collections are `users`, `resources`, `quizzes`, `studentChoices`, `videoPacks`, `videoAccessRequests`, `videoUnlocks`, and `teacherPlanRequests`. Store service-account credentials securely and grant the server only the Google Cloud permissions it needs.

## Use ImageKit for media

Set `IMAGEKIT_PUBLIC_KEY`, `IMAGEKIT_PRIVATE_KEY`, and `IMAGEKIT_URL_ENDPOINT` in the server's environment. The private key must only exist in the server's secret configuration. Teacher video uploads go directly from the browser to ImageKit; videos are uploaded as private files, and NervaEdu returns a short-lived signed URL only to the teacher or a student whose teacher has unlocked access. PDFs are stored in ImageKit and shared as public links. When ImageKit is not configured, small local uploads continue to use `data/uploads/`.

## Included

- Student sign-up: name, age, phone, home address, school and password.
- Teacher sign-up: name, subjects, call number, WhatsApp number, highest qualification, other qualification when applicable, and an optional profile photo.
- New teacher profiles stay hidden until the NervaEdu account manager verifies them. The free plan permits up to 250 students per subject. Premium supports up to 1,400 students for LKR 1,000/month; Premium Plus supports up to 10,000 for LKR 2,100/month. Teachers send a plan receipt via WhatsApp; the account manager activates the plan after checking payment.
- Teacher directory filtered by subject; students can select a teacher for each subject.
- Sri Lankan G.C.E. O/L and A/L subject lists plus custom subjects for teachers.
- Teacher uploads: up to five videos per subject, plus PDF papers and notes. Teachers set an LKR price on each video or one price for the subject video pack. Students send a payment receipt to the teacher in WhatsApp; the teacher reviews it and unlocks the video or pack. Video limit is 100 MB per file; PDF limit is 25 MB per file.
- Six-question quizzes with four choices per question and server-checked answers.
- Passwords stored as salted scrypt hashes. Sessions use an HttpOnly cookie.

## Notes

The app listens on `127.0.0.1` by default. Set `HOST=0.0.0.0` in `.env` when deploying on a platform that requires an externally reachable port. Put it behind HTTPS and add school-managed backups, access controls, and an appropriate privacy process for student contact details. Use a non-production dataset while evaluating it.

The interface uses an optional Google Fonts stylesheet. Local JSON mode uses Node.js built-in modules; Firestore mode uses the Firebase Admin SDK.
