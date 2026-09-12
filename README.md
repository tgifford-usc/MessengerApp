# Messenger — app

The phone half of a tiny chat app: a single `index.html`, hosted on GitHub Pages. It needs a backend to talk to; that's the separate `Messenger` repo, deployed on Vercel.

## Set it up

1. Deploy the backend first (see its README) and note its address, something like `https://my-messenger.vercel.app`.
2. Open `index.html` and change the `API` line at the top of the `<script>` to that address.
3. Put this folder in a GitHub repository. In the repo go to **Settings → Pages**, set Source to **Deploy from a branch**, choose `main` and `/ (root)`, and save. After a minute your app is at `https://YOUR-NAME.github.io/YOUR-REPO/`.
4. Open that on two phones, type your names and the same room code, and chat.

**Testing on your laptop first:** run the backend locally (`npm run dev` in its folder), set `API` to `http://localhost:3000`, and open `index.html` straight from disk. Two browser windows make two "phones".

If the app says "Can't reach the server", the message after the colon is the clue. "Failed to fetch" means the browser never got a usable answer: check the `API` address for typos, and open it in a new tab to see whether the backend's status page loads. Anything else is the backend telling you what's wrong. (If the browser console mentions CORS, it's nearly always the address, not CORS itself.)

## How it works

- **Joining.** There are no accounts. Both people type the same room code. The code goes into the URL (`?room=blue-fox-42`), so you can also just send someone the link. Your name is remembered in `localStorage`.
- **Receiving.** Every 2 seconds the app fetches `GET /api/messages?room=…` and redraws the list if anything changed. This is called **polling**. Polling stops while the app is in the background to save battery (and the backend's free quota).
- **Sending.** `POST /api/messages` with `{ room, from, text }`, then fetch straight away so your own message appears immediately.

## Things worth knowing

**Never put user text into `innerHTML`.** The app builds each bubble with `textContent` and `append()`, so a message like `<img src=x onerror=alert(1)>` shows up as text instead of running as code. Try changing it to `innerHTML` and sending that message to see why this matters.

**Mobile details that are easy to miss:** inputs are `16px` so iOS doesn't zoom when you tap them; the page is `100dvh` tall so it fits as the browser's address bar slides away; the send bar has `env(safe-area-inset-bottom)` padding so it clears the iPhone home bar.

## Ideas to extend it

- **The same-name problem.** Two people called "Sam" both see each other's messages as their own. Generate a random ID on first visit, keep it in `localStorage`, and send it with every message.
- **Typing indicator.** Send a "typing" message with a short expiry (the backend README shows the Redis trick).
- **Sounds or vibration** (`navigator.vibrate`) when a new message from someone else arrives. You already know when that is: `render()` runs only when the list has changed.
- **Emoji reactions, colours per person, a QR code for the room link**, or anything else. The whole app is one file; go wild.
