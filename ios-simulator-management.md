---
description: 'Sun May 27, 2018'
---

# iOS Simulator Management

## Simulator Deivce Control

### List Simulators / Device Types / Runtimes

```bash
$ xcrun simctl list [-j | -json] [devices | devicetypes | runtimes]
```

### Open Simulator

```bash
$ open -a Simulator [--args -CurrentDeviceUDID <device>]
```

### Create / Delete

```bash
$ xcun simctl create <name> <devicetype id> <runtime id>
```

```bash
$ xcrun simctl create 'My iPhoneX' \
    com.apple.CoreSimulator.SimDeviceType.iPhone-X \
    com.apple.CoreSimulator.SimRuntime.iOS-11-3
```

```bash
$ xcrun simctl delete <device | unavailable>
```

### Rename / Clone

```bash
$ xcrun simctl rename <device> <new name>
$ xcrun simctl clone <device> <new name>
```

### Boot / Shutdown

```bash
$ xcrun simctl boot <device>
$ xcrun simctl shutdown <device | booted | all>
```

## Simulator Content Control

### Install / Uninstall

```bash
$ xcrun simctl install <device> <path>
```

{% hint style="info" %}
Use `xcodebuild -sdk iphonesimulator` to create a simulator app bundle.
{% endhint %}

```bash
$ xcrun simctl uninstall booted <app identifier>
```

### Launch / Terminate

```bash
$ simctl launch [-w | --wait-for-debugger] \
    [--console] [--stdout=<path>] [--stderr=<path>] \
    <device> <app identifier> \
    [<argv 1> <argv 2> ... <argv n>]
```

```bash
$ xcrun simctl terminate <device> <app identifier>
```

### Erase

```bash
$ xcrun simctl erase <device | all>
```

{% hint style="info" %}
Shutdown device required
{% endhint %}

### Record Video / Screenshot

```bash
$ xcrun simctl io <device> <recordVideo | screenshot> \
    [--type=<type>] [--display=<display>] <file>
```

### Open URL

```bash
$ xcrun simctl openurl <device> <URL>
```

#### Add Media

```bash
$ xcrun simctl <addmedia | addphoto | addvideo> <device> <path> [... <path>]
```

### List Apps / App Info

```bash
$ xcrun simctl listapps <device>
```

```bash
$ xcrun simctl appinfo <device> <app identifier>
```

### Spawn

```bash
$ simctl spawn [-w | --wait-for-debugger] \
    [-s | --standalone] \
    [-a <arch> | --arch=<arch>] \
    <device> <path to executable> [<argv 1> <argv 2> ... <argv n>]
```

This command is for spawning a process on a device. For example, to attach device log stream:

```bash
$ xcrun simctl spawn booted log stream
```

## More

| bootstatus | notify\_post |
| --- | --- | --- | --- | --- | --- | --- | --- |
| darwinup | notify\_get\_state / notify\_set\_state |
| diagnose | pair / unpair |
| get\_app\_container | pair\_ activate |
| getenv | pbinfo / pbsync |
| icloud\_sync | pbcopy / pbpaste |
| logverbose | register / unregister |
| monitor | upgrade |

## Testing

```bash
$ xcrun simctl list devices
```

```text
== Devices ==
-- iOS 11.3 --
    iPhone 6s (86DD2531-0731-4F59-84A5-CC651F685C06) (Shutdown)
    iPhone 6s Plus (84D55204-A978-47E3-9868-8D93A5FFF26E) (Shutdown)
    iPhone 7 (ED2CCFC7-B7EF-45CD-87D8-FBD5C36F8AA4) (Shutdown)
    iPhone 7 Plus (BEFBC108-9A6C-4781-8154-C7E0458A1554) (Shutdown)
    iPhone 8 (F9682D7A-961B-428D-BE0E-0518417F061C) (Shutdown)
    iPhone 8 Plus (2D55F86E-9B75-443F-BC25-884F1C9F9236) (Shutdown)
    iPhone X (C088E278-461F-4035-8B2C-AC2BC5A7E0F6) (Shutdown)
    iPad Air (22758588-0532-4780-9234-778D9A8B074B) (Shutdown)
    iPad Air 2 (269F7A45-6B56-47C9-9AE3-B6E5E11D181A) (Shutdown)
    iPad (5th generation) (CB5D322F-7D56-4224-9CC7-CB9B3C15CE3E) (Shutdown)
    iPad Pro (9.7-inch) (4547823A-DCB9-47ED-B10D-71D2BC1ADE63) (Shutdown)
    iPad Pro (12.9-inch) (C22DA1AF-8DFE-478E-847C-B1666986451D) (Shutdown)
    iPad Pro (10.5-inch) (5AA51BA3-4FEE-4CC3-863E-3C767EF09D5A) (Shutdown)
-- tvOS 11.3 --
    Apple TV (6B376AD0-588C-41AE-BF8F-22F2E2E22327) (Shutdown)
    Apple TV 4K (C0EA8C7C-E691-4500-BD83-2D18FB302CB5) (Shutdown)
-- watchOS 4.3 --
    Apple Watch - 38mm (64CFB69D-8E01-4F98-84CC-E5CDA9C5A4FE) (Shutdown)
    Apple Watch Series 3 - 42mm (301B9910-296D-41E6-BE82-A3D88AFF7541) (Shutdown)
...
```

### Find UDIDs

iPhone X \(iOS 11+\)

```bash
$ xcrun simctl list devices \
    | sed -n '{ 1,/iOS 11/d; /iPhone X/p; }' \
    | egrep -o '([A-Z0-9-]){36}' \
```

iPhone 8/8+/X

```bash
$ xcrun simctl list devices \
    | grep 'iPhone [X]' \
    | awk -F "[()]" '{ print $2 }'
```

```bash
$ xcrun simctl list devices \
    | awk '/iPhone/{ if ($2 > 7 && $2 != "SE") print $(NF-1)}' \
    | cut -c 2-37
```

#### Boot the device

```bash
$ xcrun simctl list devices \
    | sed -n '{ 1,/iOS 11/d; /iPhone X/p; }' \
    | egrep -o '([A-Z0-9-]){36}' \
    | xargs -I udid xcrun simctl boot udid
```

### Localization

```bash
$ xcrun simctl launch booted com.acme.example \
    -AppleLanguages "(en)" \
    -AppleLocale "en_US" \
    -NSDouleLocalaizedString YES -NSForceRightToLeftWritingDirection YES \
    -NSShowNonLocalizedStrings YES
```

### Arguments

```bash
$ xcrun simctl launch booted com.acme.example foo bar -swift true
```

#### Get environment argument in code

```swift
let args = ProcessInfo.processInfo.arguments
for (idx, arg) in args.enumerated() {
    print("idx: \(idx), argument: \(arg)")
}
```

#### "-Argument" are stored in UserDefaults

```swift
let isSwift = UserDefaults.standard.bool(forKey: "swift")
print(isSwift)
```

Primitive value types can be recognized.

```bash
-array "('a', 'b', 'c')"
-dict "{'a'='foo'; 'b'='bar'}"
-xml "<dict><key>foo</key><string>bar</string></dict>"
```

```swift
let arr = UserDefaults.standard.array(forKey: "array")
let dict = UserDefaults.standard.dict(forKey: "dict")
let xml = UserDefaults.standard.object(forKey: "xml")
```

## Reference

> [https://www.slideshare.net/itpersons/simulator-customizing-testing-for-xcode-9](https://www.slideshare.net/itpersons/simulator-customizing-testing-for-xcode-9)



