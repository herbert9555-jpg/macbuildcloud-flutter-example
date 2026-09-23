# MacBuildCloud Flutter example

A plain Flutter app with a build file that works on
[MacBuildCloud](https://macbuildcloud.com) as it stands. One build, on one
real Mac, gives you an Android `.apk` and an iOS app.

## Your first build

1. **Make your own copy.** Press **Use this template** at the top of this page
   (or fork it). Private or public both work.
2. **Sign up** at [macbuildcloud.com](https://macbuildcloud.com/signup) and
   verify a card when the dashboard asks. It is not charged; it keeps the free
   tier from being farmed for crypto-mining.
3. **Connect your copy.** On **Repositories**, choose **Install the GitHub app**
   (or **Choose repositories** if you have installed it before) and pick the
      repository you just made. We ask for read-only access to your code, plus
   permission to post build statuses (Checks) back to GitHub — we can never
   modify your code.
4. **Press Build.** A fresh Mac is ready in about 20 seconds and you can watch
   the log live.

When it finishes, the build page has two downloads:

| File | What it is |
|---|---|
| `app-release.apk` | The Android app, ready to install on a device |
| `Runner.app.tar.gz` | The iOS app, compiled for real devices but not signed |

Folders such as `Runner.app` are packed into a `.tar.gz` for download.

## What the build does

Everything is in [`.macbuildcloud.yml`](.macbuildcloud.yml):

```yaml
script:
  - flutter --version
  - flutter pub get
  - flutter test
  - flutter build apk --release --build-number=$MBC_BUILD_NUMBER
  - flutter build ios --release --no-codesign --build-number=$MBC_BUILD_NUMBER
```

Commands run in order and the build stops at the first one that fails.
`$MBC_BUILD_NUMBER` is set by MacBuildCloud and goes up by one or more with
every build, so each upload gets a build number the stores have not seen.

## Ship to TestFlight

The iOS half above is not signed, so it cannot be installed from TestFlight.
To get a signed `.ipa` uploaded for you:

1. Change the bundle id to one registered to your Apple team
   (Xcode → Runner target → Signing & Capabilities).
2. Fill in [`ios/ExportOptions.plist`](ios/ExportOptions.plist) with your team
   id, bundle id and the name of your App Store provisioning profile.
3. On MacBuildCloud → **Secrets**, add:
   - `APPLE_DIST_CERT` — your Apple Distribution certificate, exported as a
     `.p12` **with its private key**
   - `APPSTORE_PROFILE` — the App Store provisioning profile for that bundle id
   - `ASC_KEY` — an App Store Connect API key (`.p8`) with the **App Manager** role
4. Copy [`ci/testflight.macbuildcloud.yml`](ci/testflight.macbuildcloud.yml)
   over `.macbuildcloud.yml` and push.

The app is archived without signing and signed only while it is exported, so
your certificate is used for exactly one step and nothing is ever changed in
your Apple Developer account. The full walkthrough, and what each Xcode
signing error actually means, is at
[macbuildcloud.com/docs/signing](https://macbuildcloud.com/docs/signing/).

## See what a failed build looks like

Worth doing once, before a real failure catches you out:

1. On **Repositories**, turn on **Debug on failure** for this repository.
2. In `test/widget_test.dart`, change `expect(find.text('1'), findsOneWidget);`
   to expect `'2'`, and push.
3. The build stops at `flutter test` and the log shows which expectation
   failed. Because debug on failure is on, the machine waits 15 minutes and a
   live terminal on it appears under the log on the build page — look around
   the Mac the build actually ran on. That time is billed like build time.
   (For a single build you can also press **Keep this machine if the build
   fails** while it is still running.)
4. Change the test back.

## More
test push build
- [Flutter guide](https://macbuildcloud.com/docs/flutter/) — Android and iOS
  in one run, tests, pinning a Flutter version
- [Build configuration reference](https://macbuildcloud.com/docs/build-config/)
- [Troubleshooting](https://macbuildcloud.com/docs/troubleshooting/)

Questions: [support@macbuildcloud.com](mailto:support@macbuildcloud.com)
