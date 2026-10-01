# IIT Goa Cultural Calendar

A static, single-page cultural events calendar for the IIT Goa community. It uses plain HTML, CSS, and JavaScript, with Firebase Authentication and Cloud Firestore. There is no build step.

## Firebase setup

1. Create a project in the [Firebase console](https://console.firebase.google.com/) and register a Web app in the project settings.
2. Copy the Firebase web app configuration into the clearly marked `firebaseConfig` object near the top of the module script in `index.html`. The web configuration is intended for client use; do not put service-account credentials or other private keys in this repository.
3. In **Authentication → Sign-in method**, enable **Email/Password**.
4. In **Authentication → Users**, reuse the existing Email/Password account for an organiser if you already created one. Create any additional organiser accounts needed for the General Secretary and Originals Head. A Google Cloud/Firebase project member account is not automatically an Authentication user. Viewers do not need accounts.
5. Create a **Cloud Firestore** database.
6. In **Firestore Database → Rules**, publish the rules from [`firestore.rules`](firestore.rules).
7. For each organiser, find the Authentication user's UID and create a document in the `roles` collection with that UID as the document ID. An Auth user by itself can sign in but has no organiser permissions until this document exists. Set its `role` string to `gensec` for the General Secretary or `originals` for the Originals Head. Do not add client-side role write permissions; create or change these documents through the Firebase console or a trusted admin environment.
8. Add the site's GitHub Pages hostname (for example, `yourname.github.io`) under **Authentication → Settings → Authorized domains**. Add the repository Pages hostname if it is hosted under a project path; the hostname is still `yourname.github.io`.

## Event data

Create and edit events through the General Secretary account in the website. Each `events/{id}` document has these fields:

| Field | Type | Example |
| --- | --- | --- |
| `title` | string | `Open Mic Night` |
| `date` | string (`YYYY-MM-DD`) | `2026-10-24` |
| `time` | string (`HH:MM`, 24-hour), optional | `18:30` or blank when to be announced |
| `venue` | string | `Student Activity Centre` |
| `category` | string | `music` |
| `desc` | string | `An evening of student performances.` |
| `notes` | string | `Bring your campus ID.` |
| `photoLinks` | array of strings | `https://example.org/album` |
| `eventEnd` | Firestore timestamp | Two hours after the scheduled start |

Supported categories are `dance`, `music`, `drama`, `art`, `literary`, `fest`, and `other`. Leave `time` blank when it has not been announced; the site displays “Time to be announced” and downloads the event as an all-day calendar item. The General Secretary can add, edit, and delete events, including their public notes. The Originals Head can append exactly one HTTPS photo album link per update, only after `eventEnd`. For this calendar, an event is considered complete two hours after its scheduled start because event duration is not entered separately. Firestore rules enforce the completion timestamp and prevent Originals from removing links or changing other event fields.

Events created before `eventEnd` was added need a one-time General Secretary edit and save before their photo-link control becomes available. This writes their completion timestamp. Publish the updated [`firestore.rules`](firestore.rules) to Firebase after deploying this version of the site.

## Run locally

Because the page loads Firebase SDK modules, serve the folder over HTTP rather than opening `index.html` as a `file://` URL. For example, with Node.js installed:

```sh
npx http-server . -p 8080
```

Then open `http://localhost:8080`. You can also use the Live Server extension in VS Code.

## Publish with GitHub Pages

1. Push `index.html`, `firestore.rules`, and this README to the repository's `main` branch.
2. In GitHub, open **Settings → Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**, select `main` and the repository root (`/`), then save.
4. Wait for GitHub Pages to publish the site and open the URL shown in the Pages settings. Confirm that the same hostname has been added to Firebase Authorized domains.

The site is static: Firebase configuration is public web-app configuration, while authentication and Firestore Security Rules protect organiser operations and role documents.