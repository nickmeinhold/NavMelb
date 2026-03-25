# NavMelb

A mobile navigation app for Melbourne that combines car routing with public transport (PTV) into a single trip. Instead of choosing between driving *or* taking the train, NavMelb lets you plan a route that drives to a station and then rides the rest of the way.

Forked from [D3lK1ch1/NavMelb](https://github.com/D3lK1ch1/NavMelb).

## How it works

The app has three routing strategies:

- **Car** -- drives origin to destination using road-aware routing via [OSRM](https://project-osrm.org/).
- **PTV** -- public transport only, connecting stations you select as waypoints. Uses Victoria's [GTFS schedule data](https://discover.data.vic.gov.au/dataset/gtfs-schedule) for stop locations and departure times.
- **Park & Ride** -- the main idea: drive to a station, then take public transport the rest of the way. You pick the station; the app routes the car leg via OSRM and draws the transit leg between stations.

The frontend is a React Native (Expo) app that renders an OpenStreetMap via Leaflet inside a WebView. The backend is an Express server that loads GTFS data at startup, geocodes addresses through Nominatim, and calculates routes.

## What works

- Place search (geocoding via Nominatim) and station search (from GTFS stops.txt)
- Setting origin and destination by search or by tapping the map
- Car routing with real road geometry and duration from OSRM
- Adding station waypoints for PTV and Park & Ride strategies
- GTFS stop index covering train, tram, and bus stops across Melbourne
- Next-departure lookup from GTFS timetable data
- Route visualization on the map with distance and duration

## What doesn't work yet

- **PTV legs draw straight lines between stations** -- there is no rail/tram geometry, so the transit segments are just point-to-point. Duration estimates for PTV legs are rough.
- **No real-time PTV data** -- departures come from static GTFS schedules, not live feeds.
- **Station parking info is stubbed** -- the `Station` type has `hasParking` / `parkingCapacity` fields but nothing populates them.
- **No lane guidance or traffic lights** -- these are aspirational features listed in the original project scope.
- **Web rendering does not work** -- Expo + WebView means the map only runs on iOS/Android via Expo Go.
- **GTFS data is not included in the repo** -- the zip files are too large for Git. You need to download them yourself (see setup below).

## Tech stack

| Layer | Technology |
|-------|-----------|
| Frontend | React Native 0.81, Expo 54, expo-router |
| Map | Leaflet 1.9 via react-native-webview |
| Backend | Node.js, Express 5, TypeScript |
| Routing | OSRM (car), GTFS static data (PTV) |
| Geocoding | Nominatim (OpenStreetMap) |
| Data | Victoria GTFS schedule (trains, trams, buses) |

## Setup

### Prerequisites

- Node.js
- Expo Go installed on your phone (iOS or Android)

### GTFS data

Download the GTFS schedule from [data.vic.gov.au](https://discover.data.vic.gov.au/dataset/gtfs-schedule) and extract the feed folders into a `gtfs/` directory at the project root. Each subfolder should contain a `google_transit.zip`. The folder numbering maps to transport types (1, 2, 10 = train; 3 = tram; 4, 5, 6, 11 = bus).

### Backend

```bash
cd backend
npm install
npm run dev
```

Runs on port 3000. Set `GTFS_ROOT` in a `.env` file if your GTFS data is not at `../gtfs` relative to the backend directory.

### Frontend

Create a `.env` file in `frontend/` with your machine's local IP:

```
EXPO_PUBLIC_API_BASE_URL=http://<YOUR_IP>:3000/api/map
```

Then:

```bash
cd frontend
npm install
npx expo start --lan
```

Scan the QR code with Expo Go on your phone. Both your phone and your machine must be on the same network.
