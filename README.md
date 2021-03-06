# EventBox [![Circle CI](https://circleci.com/gh/s-aska/EventBox.svg?style=svg)](https://circleci.com/gh/s-aska/EventBox)

Provides an interface for use the `addObserverForName` safely and easily.

## Features

- [x] Comprehensive Unit Test Coverage
- [x] Carthage support
- [x] Thread-safe

## Requirements

- iOS 8+
- Xcode 6.3

## Installation

Create a Cartfile that lists the frameworks you’d like to use in your project.

```bash
echo 'github "s-aska/EventBox"' >> Cartfile
```

Run `carthage update`

```bash
carthage update
```

On your application targets’ “General” settings tab, in the “Embedded Binaries” section, drag and drop each framework you want to use from the Carthage.build folder on disk.

## Usage

### Minimum

```swift
EventBox.onMainThread(target, name: "reload") { _ in
    // UI
}

EventBox.onBackgroundThread(target, name:"reload") { _ in
    // API Access
}

EventBox.post("reload")

EventBox.off(target) // Remove all the observers of the target

EventBox.off(target, "reload") // Remove observers of the same name of the target
```

### Sender

```swift
EventBox.onMainThread(target, name:"getStatus") { n in
    // API Access
    let status = n.object as TwitterStatus // sender
}

let status = TwitterStatus()
EventBox.post("getStatus", sender: status)
```

### Handling the keyboard notifications

```swift
// MARK: - View Life Cycle

override func viewWillAppear(animated: Bool) {
    super.viewWillAppear(animated)

    EventBox.onMainThread(self, name: UIKeyboardWillShowNotification, handler: { n in
        self.keyboardWillChangeFrame(n, showsKeyboard: true) })

    EventBox.onMainThread(self, name: UIKeyboardWillHideNotification, handler: { n in
        self.keyboardWillChangeFrame(n, showsKeyboard: false) })
}

override func viewDidDisappear(animated: Bool) {
    super.viewDidDisappear(animated)

    EventBox.off(self)
}

// MARK: - Keyboard Event Notifications

func keyboardWillChangeFrame(notification: NSNotification, showsKeyboard: Bool) {
    let userInfo = notification.userInfo!
    let animationDuration = (userInfo[UIKeyboardAnimationDurationUserInfoKey] as NSNumber).doubleValue
    let keyboardScreenEndFrame = (userInfo[UIKeyboardFrameEndUserInfoKey] as NSValue).CGRectValue()

    if showsKeyboard {
        let orientation: UIInterfaceOrientation = UIApplication.sharedApplication().statusBarOrientation
        if (orientation.isLandscape) {
            containerViewButtomConstraint.constant = keyboardScreenEndFrame.size.width
        } else {
            containerViewButtomConstraint.constant = keyboardScreenEndFrame.size.height
        }
    } else {
        containerViewButtomConstraint.constant = 0
    }

    self.view.setNeedsUpdateConstraints()

    UIView.animateWithDuration(animationDuration, delay: 0, options: .BeginFromCurrentState, animations: {
        self.containerView.alpha = showsKeyboard ? 1 : 0
        self.view.layoutIfNeeded()
    }, completion: { finished in
        if !showsKeyboard {
            self.view.hidden = true
        }
    })
}
```


## License

EventBox is released under the MIT license. See LICENSE for details.
