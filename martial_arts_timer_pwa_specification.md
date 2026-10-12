# Software Specification: Martial Arts Round Timer (PWA)

## 1. Product Overview
**Type:** Progressive Web App (PWA)
**Objective:** A reliable, installable web application for martial artists, boxers, and combat sports athletes to manage training rounds, rest intervals, and preparation time. The app functions entirely offline, prevents the device screen from sleeping, and features a highly customizable audio engine that cuts through noisy gym environments.

## 2. Core Functional Requirements
### 2.1 Timer Configuration
*   **Preparation Time:** 0 to 60 seconds (countdown before the first round begins).
*   **Round Duration:** 15 seconds to 10 minutes.
*   **Rest Duration:** 0 seconds to 5 minutes.
*   **Number of Rounds:** 1 to 50, or an "Infinite" continuous loop mode.
*   **Push Intervals / Action Cues:** Optional periodic signals during an active round (e.g., a beep every 30 seconds to switch stances) or random intervals (e.g., a cue every 15-40 seconds to simulate unpredictable fight conditions).

### 2.2 Progressive Web App (PWA) Capabilities
*   **Installability:** Must include a valid `manifest.json` with appropriate icons (192x192, 512x512, maskable) to trigger the "Add to Home Screen" prompt on iOS and Android.
*   **Offline First:** A Service Worker must cache the HTML, CSS, JavaScript, and the default audio library. The app must load and function 100% without an internet connection.
*   **Screen Wake Lock:** Must utilize the Screen Wake Lock API (`navigator.wakeLock.request('screen')`) to prevent the device from going to sleep while the timer is running.
*   **Background Execution:** Because mobile browsers heavily throttle `setInterval`/`setTimeout` when a tab is minimized, the timer logic must rely on timestamp deltas (`Date.now()` or `performance.now()`) and utilize a dedicated **Web Worker** to ensure the timer does not drift or lose time if the user switches apps.

## 3. Audio & Sound System (Key Feature)
The app must provide a multi-sensory experience with low-latency audio triggering via the Web Audio API.

### 3.1 Default Sound Library
The app will ship with pre-loaded, optimized audio files mapped to specific combat sports:
*   **Martial Arts:** Traditional Gong, Wooden Clappers (Hyoshigi).
*   **Boxing:** Classic ringside mechanical bell.
*   **MMA / Fitness:** Digital buzzer, Referee whistle.
*   **Voice Prompts:** Synthesized or pre-recorded phrases ("Box!", "Rest", "Ten Seconds", "Time").

### 3.2 Sound Customization Engine
Users can assign specific sounds to distinct timer phases. 
*   **Trigger Events:** Round Start, 10-Second Warning, Round End, Rest Start, Action Cue/Push Interval.
*   **Custom Audio Uploads:** Users can upload their own `.mp3`, `.wav`, or `.ogg` files (capped at 1MB per file). 
*   **Local Storage:** Because `localStorage` cannot efficiently handle binary blobs, custom audio files must be stored locally on the device using **IndexedDB**. 
*   **Volume Control:** An independent in-app volume slider that overrides media volume to ensure the bell can be heard over external gym music.

## 4. User Interface & Experience (UI/UX)
*   **High-Visibility Display:** Massive, screen-filling typography for the countdown, designed to be readable from across a room or dojo. 
*   **Color-Coded States:** The interface provides immediate visual feedback:
    *   *Preparation:* Yellow / Orange background.
    *   *Active Round:* Black / Dark Grey background.
    *   *Rest Period:* Blue / Green background.
    *   *10-Second Warning:* Screen background flashes Red.
*   **Gloved-Hand Accessibility:** Extra-large touch targets for the Play, Pause, Reset, and Skip (Next/Previous Round) buttons, ensuring they can be tapped easily by users wearing hand wraps or boxing gloves.

## 5. Technical Architecture & Stack
*   **Frontend Framework:** Vue.js, Svelte, or React (Svelte is recommended for minimal bundle size and fast PWA load times).
*   **Styling:** Tailwind CSS or raw CSS with CSS Variables to easily handle the state-based background color transitions.
*   **Data Persistence:**
    *   `localStorage` for saving the user's custom Timer Presets (e.g., "Muay Thai 5x3", "Tabata").
    *   `IndexedDB` (via a wrapper like *localForage* or *idb*) for storing user-uploaded audio assets.
*   **Audio Playback:** Use the `Web Audio API` rather than standard `<audio>` HTML tags to guarantee zero-delay playback and allow for concurrent sound overlapping (e.g., a background timer ticking while a bell rings).

## 6. Data Structure (Timer Presets)
```json
{
  "presetId": "mma-championship",
  "name": "MMA Championship",
  "prepTimeSeconds": 10,
  "rounds": 5,
  "roundLengthSeconds": 300,
  "restLengthSeconds": 60,
  "audioConfig": {
    "roundStart": "default_bell_1",
    "tenSecondWarning": "default_wood_block",
    "roundEnd": "default_gong",
    "actionCue": "custom_user_upload_1241"
  }
}
```