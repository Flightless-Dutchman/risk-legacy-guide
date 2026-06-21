# Firebase Hosting & Integration Summary

This document describes how `risk-legacy-guide` is deployed to Firebase Hosting,
so the same setup can be reused or referenced from another repository.

## Firebase project

| Item | Value |
|------|-------|
| **Project ID** (`.firebaserc` default) | `paul-and-chester` |
| **Hosting target name** | `risk` |
| **Hosting site** | `risk-legacy-guide` |

The project uses a **named hosting target** (`risk` → site `risk-legacy-guide`),
the standard pattern for hosting multiple sites under one Firebase project.
To replicate the target mapping in another repo:

```bash
firebase target:apply hosting <target-name> <site-id>
```

## `firebase.json`

```json
{
  "hosting": {
    "target": "risk",
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**",
      "*.heic",
      "converted/**"
    ],
    "rewrites": [
      { "source": "**", "destination": "/index.html" }
    ]
  }
}
```

- **Public dir:** `public/` — only what's in here is deployed.
- **SPA rewrite:** all routes (`**`) fall back to `/index.html`.
- **Ignore rules:** strips dotfiles, `node_modules`, raw `*.heic` image
  originals, and the `converted/` working folder so they never ship.

## `.firebaserc`

```json
{
  "projects": { "default": "paul-and-chester" },
  "targets": {
    "paul-and-chester": {
      "hosting": { "risk": ["risk-legacy-guide"] }
    }
  },
  "etags": {}
}
```

## What's actually served

- A single static `public/index.html` — the RISK Legacy Battle Guide.
- External dependencies are just **Google Fonts** (`fonts.googleapis.com` —
  Black Ops One, Oswald, Roboto Condensed).
- **No Firebase SDK, Analytics/gtag, Functions, Firestore, or Auth.**
  It is hosting-only, fully static.

## Deploy

Deploys are done with the Firebase CLI:

```bash
firebase deploy --only hosting:risk
```

A GitHub Actions workflow is also provided at
`.github/workflows/firebase-hosting-deploy.yml` for automated deploys on push
to `main` (see below).

`.gitignore` excludes `.firebase/` (CLI cache), `*.heic`/`*.HEIC` originals,
and an old `risk-legacy-guide.html` draft.

## Reusing this setup in another repo

1. `firebase init hosting`, or copy `firebase.json` + `.firebaserc`.
2. Set your own project ID and, if using multiple sites,
   `firebase target:apply hosting <target> <site-id>` to match the `targets`
   block.
3. Put deployable assets in `public/`.
4. Keep the `**` → `/index.html` rewrite for SPA-style routing; drop it for a
   plain multi-page static site.
5. For auto-deploy, add the GitHub Actions workflow below and configure the
   service-account secret.

## CI/CD: GitHub Actions

The workflow at `.github/workflows/firebase-hosting-deploy.yml` deploys to the
live channel on every push to `main`.

### Required setup

1. Generate a Firebase service account key:
   - Firebase Console → Project Settings → Service accounts →
     **Generate new private key**, or run
     `firebase init hosting:github` to have the CLI create one.
2. Add the JSON key as a repository secret named
   **`FIREBASE_SERVICE_ACCOUNT`**.
3. Confirm the `projectId` in the workflow matches your Firebase project
   (`paul-and-chester` here).

The workflow uses the official
[`FirebaseExtended/action-hosting-deploy`](https://github.com/FirebaseExtended/action-hosting-deploy)
action.
