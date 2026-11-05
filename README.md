# Chore Kingdom

A family chore chart for iPhone and iPad. Kids check off their chores and earn points, parents approve the bigger jobs and hand out rewards, and everyone can see streaks and a weekly summary.

Published by Luminae Crown Studio LLC. Sold as a paid app: one purchase on the App Store, no in-app purchases, no subscription.

## Features

- **Kids:** a name, an avatar (SF Symbol) and a color for each child. No accounts.
- **Chores:** a title, an icon, a point value, the weekdays it repeats, a time of day (morning, after school or evening), one or more assigned kids, and an optional "needs parent approval" setting.
- **Today:** each kid taps their avatar and sees today's chores grouped by time of day, with large tap targets, a check-off animation and a haptic.
- **Approvals:** chores that need approval wait in a queue until a parent approves or rejects them.
- **Rewards:** parents create rewards with a point cost. Kids redeem them with their points, and parents mark them delivered.
- **Streaks and weekly summary:** consecutive days a kid finished everything, plus a "This Week" card on the Parent tab and a full week-by-week summary.
- **Reminders:** optional daily reminders for each time of day, sent only on days that have chores.
- **Widgets:** small and medium Home Screen widgets showing a chosen kid's chores left today.
- **Parent PIN:** a 4-digit PIN in the Keychain guards the Parent and Settings tabs. It can be changed, or turned off completely in Settings.
- **iCloud sync (optional, on by default):** keeps the family's data in step across devices signed in to the same Apple Account.
- **Child's own device:** a device can be set to show only one child's chores and rewards.

## Requirements

- A current Xcode that supports Swift 6 and the iOS 18 SDK
- iOS 18.0 or later, iPhone and iPad
- Swift 6 language mode (strict concurrency)
- Apple frameworks only: SwiftUI, SwiftData, WidgetKit, AppIntents, UserNotifications, Security, Foundation, Observation and UIKit (for one "Open Settings" link). No third-party packages or SDKs.

## Getting started

1. Open `ChoreKingdom.xcodeproj`.
2. For each of the three targets (ChoreKingdom, ChoreKingdomWidget, ChoreKingdomTests), open **Signing & Capabilities** and choose your Team. The bundle IDs are `com.luminaecrown.chorechart`, `.widget` and `.tests`. Change them there if yours differ.
3. Confirm the **App Group** `group.com.luminaecrown.chorechart` is on both the app and the widget. The widget reads the app's database through it.
4. To try sync, add these to the **app target only** (not the widget):
   - **iCloud**, with **CloudKit** ticked and the container `iCloud.com.luminaecrown.chorechart`
   - **Background Modes**, with **Remote notifications** ticked

   Without them the app still runs; it just stays on the device.
5. Choose an iPhone or iPad simulator and run. Run all tests with Cmd+U.

In Debug builds a ladybug menu (Parent, then Kids) offers Load Sample Data, Replay Onboarding, Reset Parent PIN and Delete All Data. It is compiled out of Release builds.

## How it's organised

```
ChoreKingdom/
├── App/           App entry, root tabs, system sync, launch-failure screen, privacy manifest, asset catalog
├── Models/        SwiftData models: Kid, Chore, Completion, Reward, Redemption (+ TimeOfDay, WeekdaySet)
├── Services/      Logic with no UI: points, streaks, schedules, approvals, redemptions, PIN, reminders
├── Features/      Screens, one folder per feature (Today, Parent, Settings, Rewards, Summary, Device, Sync…)
├── Views/         Small shared components (avatar badge, pickers, colors)
├── Widget/        The widget extension
└── Tests/         Swift Testing unit tests
```

Views stay small. Anything involving points, streaks, scheduling, PINs or reminders lives in a testable type under `Services/`.

### Data model

| Model | Holds |
|---|---|
| `Kid` | name, avatar symbol, color, sort order |
| `Chore` | title, symbol, points, weekdays, time of day, requires approval, active, created date, assigned kids |
| `Completion` | chore, kid, date, status (pending, approved, rejected), points awarded |
| `Reward` | title, symbol, cost, active |
| `Redemption` | reward, kid, date, cost, delivered |

Rules the code relies on:

- **No stored balance.** A kid's points are approved completions minus redemptions, always calculated.
- **History is frozen.** `pointsAwarded` and `cost` are copied at the moment of the action, so editing a chore or reward later never rewrites history.
- **One check-off per chore per kid per day.** Duplicates (from a double-tap, or two devices before they sync) are prevented, and are counted once if they do appear.
- **CloudKit-compatible schema.** Every property has a default or is optional, relationships are optional with inverses, and nothing is marked unique.

### Streaks

A streak is consecutive days a kid finished everything scheduled for them. Days with nothing scheduled are skipped, a chore waiting for approval counts as finished, a rejected one does not, and today only counts once it is complete (an unfinished today does not break the streak). Chores are ignored on days before they existed.

## The widget: a rule worth knowing

The widget target compiles only a handful of shared files: everything in `Models/`, plus `Services/AppModelContainer.swift`, `Services/TodayPlanner.swift`, `Services/WidgetSnapshot.swift` and `Views/Components/ColorHex.swift`.

Because the project uses folder-synchronised groups, **any new file added to `Models/`, `Services/` or `Views/` is compiled into the widget unless it is added to that target's exclusion list.** Put app-only code in `Features/` or `App/` (the widget does not include them), or exclude the file from the widget in Xcode (select the file, then the File inspector, then Target Membership).

## Sync and child devices

- Sync uses SwiftData's private CloudKit database. Devices must be signed in to the **same Apple Account**; a child with their own Apple Account will not see the family's data.
- The parent PIN, reminder settings and each device's role (family device or a child's device) are stored per device and do not sync.
- Before releasing, deploy the CloudKit schema from Development to Production in the CloudKit Console, or sync will fail for customers.

## Privacy

Chore Kingdom has no accounts, no analytics, no ads and no tracking, and the developer never receives your data. See [PRIVACY.md](PRIVACY.md). The privacy manifest (`App/PrivacyInfo.xcprivacy`) declares no tracking and no collected data, plus the one required-reason API the app uses (UserDefaults, reason CA92.1).

## Known limitations

- Children with their own Apple Accounts cannot join the family's data yet. That would need CloudKit sharing, which SwiftData does not provide.
- There is no recovery for a forgotten parent PIN yet.
- Two devices redeeming the same reward at the same instant can overspend a balance; sync cannot prevent it.
- Day boundaries follow the time zone of the device being used.
- There is no versioned database schema yet. Add one before the first schema change after release.

## Before submitting to the App Store

- [ ] Add the 1024×1024 app icon in `App/Assets.xcassets/AppIcon`
- [ ] Add the iCloud and Background Modes capabilities and deploy the CloudKit schema to Production
- [ ] Set the price in App Store Connect (no in-app purchase is needed)
- [ ] Publish the privacy policy at a public URL and add it to the App Store listing
- [ ] Answer the App Store privacy questions ("Data Not Collected") and set the age rating
- [ ] Archive a Release build; test on real devices, in airplane mode, with VoiceOver and at the largest text size
