<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"
  />
  <title>Pulse Tracker</title>

  <link
    rel="stylesheet"
    href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
    integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY="
    crossorigin=""
  />

  <style>
    :root {
      --bg: #10151d;
      --card: #18212d;
      --muted: #8ca0b8;
      --text: #f4f8fc;
      --primary: #35d48c;
      --danger: #ef4444;
      --border: #253244;
    }

    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: var(--bg);
      color: var(--text);
    }

    .hidden { display: none !important; }

    .screen-center {
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 16px;
    }

    .card {
      width: 100%;
      max-width: 420px;
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: 18px;
      padding: 24px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.25);
    }

    .title {
      font-size: 30px;
      font-weight: 700;
      margin: 0 0 8px;
      text-align: center;
    }

    .subtitle {
      font-size: 14px;
      color: var(--muted);
      text-align: center;
      margin-bottom: 20px;
    }

    .input {
      width: 100%;
      height: 48px;
      border-radius: 12px;
      border: 1px solid var(--border);
      background: #101823;
      color: var(--text);
      padding: 0 14px;
      font-size: 16px;
      outline: none;
    }

    .btn {
      width: 100%;
      height: 48px;
      border: none;
      border-radius: 12px;
      font-size: 16px;
      font-weight: 700;
      cursor: pointer;
      margin-top: 14px;
    }

    .btn-primary {
      background: var(--primary);
      color: #052814;
    }

    .btn-danger {
      background: var(--danger);
      color: white;
    }

    .note {
      font-size: 12px;
      color: var(--muted);
      text-align: center;
      margin-top: 14px;
    }

    .app {
      min-height: 100vh;
      display: flex;
      flex-direction: column;
    }

    .topbar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 12px;
      padding: 14px 16px;
      border-bottom: 1px solid var(--border);
      background: rgba(24,33,45,0.95);
    }

    .topbar-left {
      display: flex;
      align-items: center;
      gap: 12px;
    }

    .dot-wrap {
      width: 40px;
      height: 40px;
      border-radius: 12px;
      background: rgba(53,212,140,0.12);
      display: flex;
      align-items: center;
      justify-content: center;
      position: relative;
    }

    .dot {
      width: 12px;
      height: 12px;
      border-radius: 50%;
      background: var(--primary);
    }

    .dot.live::before {
      content: "";
      position: absolute;
      inset: 10px;
      border-radius: 999px;
      background: rgba(53,212,140,0.3);
      animation: pulse 1.8s infinite;
    }

    @keyframes pulse {
      0% { transform: scale(1); opacity: 0.8; }
      100% { transform: scale(2.6); opacity: 0; }
    }

    .phone-label {
      font-size: 12px;
      color: var(--muted);
    }

    .phone-value {
      font-size: 15px;
      font-weight: 700;
    }

    #map {
      height: 60vh;
      width: 100%;
    }

    .overlay-msg {
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      top: 16px;
      z-index: 1000;
      background: var(--danger);
      color: white;
      padding: 10px 14px;
      border-radius: 10px;
      font-size: 14px;
      display: none;
    }

    .map-wrap {
      position: relative;
      flex: 1;
    }

    .follow-btn {
      position: absolute;
      right: 16px;
      bottom: 16px;
      z-index: 1000;
      border: none;
      background: var(--card);
      color: var(--text);
      border: 1px solid var(--border);
      border-radius: 10px;
      padding: 10px 14px;
      cursor: pointer;
      font-weight: 600;
    }

    .stats {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 12px;
      padding: 16px;
      border-top: 1px solid var(--border);
      background: rgba(24,33,45,0.7);
    }

    .stat {
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: 14px;
      padding: 14px;
    }

    .stat-label {
      font-size: 12px;
      color: var(--muted);
      margin-bottom: 6px;
    }

    .stat-value {
      font-size: 18px;
      font-weight: 700;
      word-break: break-word;
    }

    .leaflet-container {
      background: #0f1720;
    }

    @media (min-width: 768px) {
      .stats {
        grid-template-columns: repeat(4, 1fr);
      }
      #map {
        height: 68vh;
      }
    }
  </style>
</head>
<body>
  <div id="registerScreen" class="screen-center">
    <div class="card">
      <h1 class="title">Pulse Tracker</h1>
      <p class="subtitle">Enter your phone number to start broadcasting your live location.</p>
      <form id="registerForm">
        <input id="phoneInput" class="input" type="tel" placeholder="+1 (555) 123-4567" maxlength="20" />
        <button class="btn btn-primary" type="submit">Continue</button>
      </form>
      <p class="note">Your location stays on this device. Nothing is uploaded.</p>
    </div>
  </div>

  <div id="appScreen" class="app hidden">
    <header class="topbar">
      <div class="topbar-left">
        <div id="statusWrap" class="dot-wrap">
          <div class="dot"></div>
        </div>
        <div>
          <div class="phone-label">Tracking</div>
          <div id="phoneDisplay" class="phone-value"></div>
        </div>
      </div>
      <button id="toggleTrackBtn" class="btn btn-primary" style="width:auto;padding:0 18px;margin:0;">
        Start
      </button>
    </header>

    <div class="map-wrap">
      <div id="errorBox" class="overlay-msg"></div>
      <div id="map"></div>
      <button id="followBtn" class="follow-btn hidden">Following</button>
    </div>

    <section class="stats">
      <div class="stat">
        <div class="stat-label">Speed</div>
        <div id="speedValue" class="stat-value">—</div>
      </div>
      <div class="stat">
        <div class="stat-label">Heading</div>
        <div id="headingValue" class="stat-value">—</div>
      </div>
      <div class="stat">
        <div class="stat-label">Accuracy</div>
        <div id="accuracyValue" class="stat-value">—</div>
      </div>
      <div class="stat">
        <div class="stat-label">Coordinates</div>
        <div id="coordsValue" class="stat-value">—</div>
      </div>
    </section>
  </div>

  <script
    src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
    integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo="
    crossorigin=""
  ></script>

  <script>
    const registerScreen = document.getElementById("registerScreen");
    const appScreen = document.getElementById("appScreen");
    const registerForm = document.getElementById("registerForm");
    const phoneInput = document.getElementById("phoneInput");
    const phoneDisplay = document.getElementById("phoneDisplay");
    const toggleTrackBtn = document.getElementById("toggleTrackBtn");
    const errorBox = document.getElementById("errorBox");
    const followBtn = document.getElementById("followBtn");
    const statusWrap = document.getElementById("statusWrap");
    const speedValue = document.getElementById("speedValue");
    const headingValue = document.getElementById("headingValue");
    const accuracyValue = document.getElementById("accuracyValue");
    const coordsValue = document.getElementById("coordsValue");

    let map;
    let marker;
    let accuracyCircle;
    let watchId = null;
    let follow = true;
    let currentPhone = "";
    let firstFix = false;

    function formatPhone(v) {
      const d = v.replace(/\D/g, "").slice(0, 15);
      if (!d) return "";
      if (d.length <= 3) return `+${d}`;
      if (d.length <= 6) return `+${d.slice(0, 1)} (${d.slice(1)}`;
      if (d.length <= 10) return `+${d.slice(0, 1)} (${d.slice(1, 4)}) ${d.slice(4)}`;
      return `+${d.slice(0, 1)} (${d.slice(1, 4)}) ${d.slice(4, 7)}-${d.slice(7)}`;
    }

    phoneInput.addEventListener("input", (e) => {
      e.target.value = formatPhone(e.target.value);
    });

    function showError(msg) {
      errorBox.textContent = msg;
      errorBox.style.display = "block";
      setTimeout(() => {
        errorBox.style.display = "none";
      }, 3000);
    }

    function initMap() {
      map = L.map("map").setView([20, 0], 2);

      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution: "&copy; OpenStreetMap"
      }).addTo(map);
    }

    function updateStats(coords) {
      const speedKmh = coords.speed != null ? Math.max(0, coords.speed * 3.6) : 0;
      speedValue.textContent = speedKmh.toFixed(1) + " km/h";
      headingValue.textContent = coords.heading != null ? Math.round(coords.heading) + "°" : "—";
      accuracyValue.textContent = "±" + Math.round(coords.accuracy) + " m";
      coordsValue.textContent = coords.latitude.toFixed(4) + ", " + coords.longitude.toFixed(4);
    }

    function updateMap(position) {
      const { latitude, longitude, accuracy } = position.coords;
      const latlng = [latitude, longitude];

      if (!marker) {
        marker = L.marker(latlng).addTo(map);
      } else {
        marker.setLatLng(latlng);
      }

      if (!accuracyCircle) {
        accuracyCircle = L.circle(latlng, {
          radius: accuracy,
          color: "#35d48c",
          fillColor: "#35d48c",
          fillOpacity: 0.15,
          weight: 1
        }).addTo(map);
      } else {
        accuracyCircle.setLatLng(latlng);
        accuracyCircle.setRadius(accuracy);
      }

      if (follow || !firstFix) {
        map.setView(latlng, 16, { animate: true });
      }

      firstFix = true;
      followBtn.classList.remove("hidden");
      updateStats(position.coords);
    }

    function startTracking() {
      if (!navigator.geolocation) {
        showError("Geolocation is not supported in this browser.");
        return;
      }

      watchId = navigator.geolocation.watchPosition(
        (position) => {
          updateMap(position);
        },
        (err) => {
          showError(err.message);
          stopTracking();
        },
        {
          enableHighAccuracy: true,
          maximumAge: 1000,
          timeout: 15000
        }
      );

      toggleTrackBtn.textContent = "Stop";
      toggleTrackBtn.classList.remove("btn-primary");
      toggleTrackBtn.classList.add("btn-danger");
      statusWrap.classList.add("live");
    }

    function stopTracking() {
      if (watchId !== null) {
        navigator.geolocation.clearWatch(watchId);
        watchId = null;
      }
      toggleTrackBtn.textContent = "Start";
      toggleTrackBtn.classList.remove("btn-danger");
      toggleTrackBtn.classList.add("btn-primary");
      statusWrap.classList.remove("live");
    }

    registerForm.addEventListener("submit", (e) => {
      e.preventDefault();
      const digits = phoneInput.value.replace(/\D/g, "");
      if (digits.length < 7) {
        showError("Enter a valid phone number.");
        return;
      }

      currentPhone = phoneInput.value;
      phoneDisplay.textContent = currentPhone;

      registerScreen.classList.add("hidden");
      appScreen.classList.remove("hidden");

      if (!map) {
        initMap();
      }

      setTimeout(() => map.invalidateSize(), 200);
    });

    toggleTrackBtn.addEventListener("click", () => {
      if (watchId === null) {
        startTracking();
      } else {
        stopTracking();
      }
    });

    followBtn.addEventListener("click", () => {
      follow = !follow;
      followBtn.textContent = follow ? "Following" : "Follow";
    });

    window.addEventListener("beforeunload", () => {
      if (watchId !== null) {
        navigator.geolocation.clearWatch(watchId);
      }
    });
  </script>
</body>
</html>
