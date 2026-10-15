# Zenith

Flutter app (iOS + Android) that predicts and notifies about visible flyovers in the
sky: primarily the ISS and Starlink, optionally planes (and weather balloons). Meant
for fun / hobby sky-watching: everything in one app instead of separate ISS-tracker
and FlightRadar-style apps.

The project is at an empty Flutter template right now. Build features step by step, as the user asks for them.

## Planned features

MVP:
- **Pass predictions**: computed on the phone from TLE data and the user's location.
  For each pass: start/end time, max elevation, naked-eye visible or not.
- **Pass detail**: sky chart (CustomPainter: horizon circle, compass directions,
  path of the object across the sky).
- **Notifications** a few minutes before a pass; the user picks object types and a
  minimum elevation.
- **Live map** of tracked objects, filterable by type.
- **Account, watchlist, observation history.**

Later: weather/cloud forecast per pass, AR mode, home-screen widget.

Monetization: free tier (ISS + Starlink, live map, watchlist ≤ 5, small banner ads),
plus a one-time 120 CZK premium (unlimited watchlist/history, no ads).

## Tech

- State management: **Riverpod**
- Backend: the proposal says **Firebase**, but its data model uses Postgres terms
  (uuid FKs, `text[]`, Row Level Security, Edge Functions), which are Supabase
  concepts. Firebase uses Firestore, Security Rules and Cloud Functions. **Not
  decided yet; ask the user before building backend code.**
- Server jobs: a daily fetch of TLEs from CelesTrak into `objects`, and a job
  that sends FCM notifications before passes.
- Passes are **not stored**; they are computed on the device. Planes and balloons are not
  stored either; they are fetched live (planes via ADS-B, balloons via SondeHub).

## Data model (from the proposal)

- `profiles` (id uuid, username, latitude, longitude, is_premium, created_at): the owner can read/write it; only the server sets `is_premium`
- `objects` (id, name, type: iss|starlink|satellite|plane|balloon, norad_id, tle_line1, tle_line2, updated_at): everyone can read; only the server writes
- `watchlist` (id, user_id → profiles, object_id → objects, created_at): owner only
- `observations` (id, user_id, object_id, observed_at, max_elevation, note): owner only
- `notification_settings` (user_id PK, enabled, minutes_before, min_elevation, object_types text[], fcm_token): owner, plus server read

## Domain notes

- **SGP4**: no maintained Dart package exists (`sgp4_sdp4` and similar are pre-Dart 3).
  Expect to implement near-Earth SGP4 yourself and verify it against the Spacetrack
  Report #3 test vector (`88888`).
- **CelesTrak** (`celestrak.org/NORAD/elements/gp.php?CATNR=25544&FORMAT=tle`,
  `?GROUP=starlink`) updates about every 2 h. Repeat downloads in between get a
  plain-text "GP data has not updated…" reply instead of TLEs, so always cache.
- **Starlink**: about 10k satellites, too many to predict passes for on a phone. Only freshly
  launched "trains" (low orbit, mean motion ≳ 15.45 rev/day, grouped by launch
  designator) are interesting to watch.
- **Planes**: they can't be predicted hours ahead, only by extrapolating the live position
  a few minutes forward. Use adsb.lol (`api.adsb.lol/v2/point/{lat}/{lon}/{nm}`, free,
  ODbL, needs attribution). OpenSky's terms are non-commercial only, which conflicts
  with ads and premium.
- Visibility: the satellite must be sunlit and the observer's sun elevation must be below about −6°.

## Dev environment

- Flutter 3.44 / Dart 3.12, Xcode 27.
- iOS deployment target is raised to 15.0 (Xcode 27 rejects 13.0).
- `flutter build ios --simulator` currently fails on Xcode 27 (`lipo -verify_arch`
  accepts only one arch). Use `flutter run -d <simulator>` instead. It builds only the
  active arch, and it must stay attached, because debug builds freeze when it detaches.

## Commands

```bash
flutter pub get
flutter analyze
flutter test
flutter run
```
