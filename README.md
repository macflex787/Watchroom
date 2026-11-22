# WatchRoom — Google Drive watch party (Version 1)

A static, browser-based private watch room for **videos you have permission to share**. GitHub Pages hosts the frontend, Google Drive hosts the video, and Firebase Realtime Database powers rooms, participants, chat, and WebRTC voice signaling.

## Included

- Create/join a room using a long random invitation URL
- Host selects a Google Drive video link; guests see the same embedded preview
- Live room chat, participant presence, optional peer-to-peer voice (2–5 people)
- Responsive dark interface; no guest account form (Firebase anonymous sign-in happens automatically)
- Google Drive caption tracks can be selected through its embedded player where supported

**Not included:** synchronized play/pause/seek; automatic matching of `.srt` files; uploading files to Google Drive; guaranteed video access or voice connectivity across restricted networks; persistent room password; moderation; file downloading.

## Setup (about 15 minutes)

1. Create a project at https://console.firebase.google.com/.
2. Under **Build → Authentication → Sign-in method**, enable **Anonymous**.
3. Under **Build → Realtime Database**, create a database. Choose a region and start in **locked mode** (do not use public test rules).
4. Open Realtime Database → **Rules** and paste the contents of `database.rules.json`. Click **Publish**.
5. In Firebase **Project settings → General → Your apps**, register a **Web app**. Copy the web app's `firebaseConfig` object values into `config.js` (which is provided with placeholder values). **Be sure to include the correct `databaseURL` shown under Realtime Database.**
6. On your computer, open this folder with a local web server (not by double-clicking `index.html`). For example, from the folder run `python -m http.server 8080` and visit http://localhost:8080.
7. Enter a display name and click **Create a room**. Paste a Google Drive link in the video field. Click **Load video**.
8. Upload all website files (`index.html`, `styles.css`, `app.js`, `config.js`) to a GitHub repository. In **Settings → Pages**, choose **Deploy from a branch**, select `main` and `/ (root)`, then Save. Your site will be at `https://YOUR_USERNAME.github.io/YOUR_REPOSITORY/`.
9. Open the GitHub Pages URL, create a room, and send the invite link to a friend. Test chat and voice on two separate devices/browsers.

**Important:** `database.rules.json` is deployed through the Firebase Console, not by GitHub Pages. `config.example.js` is a template. `config.js` is loaded by the site and must contain your actual Firebase web configuration. Firebase web config is intentionally public; do not put service-account keys or admin secrets in the website.

## Google Drive video preparation

1. Upload a supported video (MP4/H.264/AAC is a good choice).
2. In Drive sharing settings, allow your intended viewers access. "Anyone with the link" is convenient but means that anyone holding the Drive link can access the file; the room link does not restrict the underlying Drive file.
3. Copy the file's sharing link, for example `https://drive.google.com/file/d/FILE_ID/view?usp=sharing`.
4. Paste it into WatchRoom. Google Drive's preview iframe is embedded, so each viewer independently controls playback. Video processing, playback and captions depend on Drive and viewer access.
5. For subtitles, use Drive's **Manage caption tracks** feature to upload an `.srt` track to the video (if offered in your Drive interface). A similarly named SRT in the folder is **not** detected automatically. Test CC availability in the embedded preview on your devices.

## Security and known limitations

- The 36-character random room ID is a **bearer secret**: anyone with the link can read/join that room. Firebase Anonymous Authentication is not identity verification. The rules are designed for a small trusted group, **not a hardened public service**.
- The example rules restrict writes to the host for video changes, to each user for their own presence, and to signed-in participants for chat. They do **not** enforce a participant cap, room expiration, rate limits, comprehensive data validation, or strong access control for reading rooms.
- Firebase may require configuration changes to authorized domains depending on your Firebase setup. Confirm GitHub Pages is permitted if authentication fails.
- Voice uses a public STUN server but **no TURN relay**. Some NATs, firewalls, mobile browsers, or restricted networks will prevent connections. Each peer must allow microphone permission. A 2–5 participant mesh is a prototype, not production voice infrastructure.
- Voice signaling is a minimal proof of concept. Network races (such as ICE candidates arriving before remote descriptions), reconnects, and browser autoplay restrictions may cause audio failure. For reliable voice, consider a managed service such as LiveKit or a TURN-backed WebRTC implementation.
- Firebase room data/messages are not automatically deleted when the room is empty. Presence uses `onDisconnect`, but stale data may remain after unusual disconnects. Add room expiration and cleanup before broad use.
- Drive iframe playback cannot be controlled by your site, so automatic synchronization and overlay subtitle rendering are not possible with this embedded player.
- Google Drive is not a guaranteed streaming CDN; its quotas, previews, permissions and accessibility can change. Avoid using the project to redistribute material without authorization.

## Files

- `index.html` — interface
- `styles.css` — responsive styling
- `app.js` — Firebase, rooms, chat, voice
- `config.js` — Firebase project settings **you must fill in**
- `config.example.js` — configuration template
- `database.rules.json` — Firebase security rules to paste into the console

## Suggested next milestones

1. Test and harden room creation and chat; add server-side validation and rate limiting.
2. Improve voice signaling and add TURN or a managed voice service.
3. Add room passwords, moderation, room expiry, and file access guidance.
4. Replace the Drive iframe with a controllable video source to support automatic sync and external `.vtt` subtitle tracks (convert `.srt` to `.vtt`).
