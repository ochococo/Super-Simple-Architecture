![Super Simple Architecture](SuperSimple.png)

![Swift5](https://img.shields.io/badge/%20in-swift%205.0-orange.svg)

# Super Simple

### Application Architecture you can learn in minutes.

## Why?

> Nobody is really smart enough to program computers.
> 
> Steve McConnell (Code Complete 2.0)

We believe we should compose software from components that are: 

- accurately named,
- simple,
- small,
- responsible for one thing,
- reusable.

# Components

## Types:

- Active Components (ex. Interaction, Navigation, Mediator etc.)
- Passive Components (ex. Renderable)
- Views (ex. UILabel)

## 🖼 Renderable

> Renderable
> (*noun*)
> 
> Something (information) that view is able to show.

### Definitions

### TL;DR

Renderable is a struct that you pass to a View which should show its contents.

#### Renderable is:
- Passive, 
- immutable struct,
- often converted from model or models,

```swift
struct BannerRenderable {
    let message: String
}
```

#### View should:
- always look the same if ordered to render the same renderable,
- have code answering to one question: "how should I look?",
- have no state,
- accept infinite number of renderables in random order.

### Example

**Step 1 - Create Renderable:**

```swift
struct BannerRenderable {
    let message: String
    
    init(message: String) {
        self.message = message
    }
}
```

**Step 2 - Expose View ability in Protocol:**

```swift
protocol BannerRendering: AnyObject {
    func render(_ renderable: BannerRenderable)
}
```

**Step 2 - Implement View:**

```swift
final class BannerView: UIView {
    @IBOutlet fileprivate var bannerLabel: UILabel!
}

extension BannerView: BannerRendering {
    func render(_ renderable: BannerRenderable) {
        bannerLabel.text = renderable.message
    }
}
```

## 🛰 Interaction

> Interaction
> (*noun*)
> 
> Component answering the question: What should I do in reaction to user action. 

### Definitions

### TL;DR

Interaction is a final class that ViewController or View owns, which performs actions in reaction to user input - gestures, text input, device movement, geographical location change etc.

#### Interaction:
- Active component (has a lifecycle), 
- final class,
- can use other components for data retrieval,
- should be literally between two actions, the one causing and the one caused,
- can call navigation methods when other ViewController should be presented,
- can call render methods on weakly held views (seen via protocol).

📌 RULE: There is no good reason for inheritance of custom classes, ever.

#### ViewControllers and Views:

- Can have many Interactions for separate functions,
- should OWN Interaction and see it via protocol,
- subviews of VC could own separate Interaction but it’s not mandatory for simple VC.

### Example

**Step 1 - Create Interaction protocols:**

```swift
protocol PodBayDoorsInteracting: AnyObject {
    func use(_ banner: BannerRendering)
    func didTapMainButton()
}
```

**Step 2 - Add Interaction implementation:**

📌 RULE: Do not tell Interaction what to do, tell what happened, it should decide what should happen next.

```swift
final class PodBayDoorsInteraction {
    fileprivate let killDave = true
    fileprivate weak var banner: BannerRendering?
}

extension PodBayDoorsInteraction: PodBayDoorsInteracting {
    private enum Strings {
        static let halsAnswer = "I know you and Frank were planning to disconnect me, and that is something I cannot allow to happen."
    }

    func didTapMainButton() {
        guard let banner = banner else { fatalError() }

        if killDave { // Just to show the business logic is resolved here.
            banner.render(BannerRenderable(message: Strings.halsAnswer))
        } else {
            // Open doors. Not implemented :P 
        }
    }

    func use(_ banner: BannerRendering) {
        self.banner = banner
    }
}
```

**Step 3 - Use it in ViewController (via nib):**

```swift
final class DiscoveryOneViewController: UIViewController {
    
    fileprivate let podBayInteraction: PodBayDoorsInteracting
    
    @IBOutlet private var bannerView: BannerRendering!
    
    init(podBayInteraction: PodBayDoorsInteracting) {
        self.podBayInteraction = podBayInteraction
        super.init(nibName: nil, bundle: nil)
    }
    
    override func awakeFromNib() {
        super.awakeFromNib()
        podBayInteraction.use(bannerView)
    }
    
    @IBAction private func didTapMainButton() {
        podBayInteraction.didTapMainButton()
    }
}
```

## 🛠 Assembler

> Assembler
> (*noun*)
> 
> Assembler is gathering dependencies and injecting them into a single instantiated component.

### Definitions

### TL;DR

Its purpose is to assemble dependencies for one class or struct. 

#### Assembler:
- a struct,
- has only one method called `assemble()`. 
- can call ONLY ONE initializer directly,
- will call other Assemblers to instantiate dependencies.

### Example

**Step 1 - Create Interaction protocols:**

📌 RULE: When assembled component is dependent on other instances, you should use it's Assemblers, do not initialize more than one class directly.

```swift
protocol PodBayDoorsInteractionAssembling {
	func assemble() -> PodBayDoorsInteracting
}

struct PodBayDoorsInteractionAssembler {
    func assemble() -> PodBayDoorsInteracting {
        return PodBayDoorsInteraction()
    }
}

protocol DiscoveryOneAssembling {
    func assemble() -> UIViewController
}

struct DiscoveryOneAssembler: DiscoveryOneAssembling {

    let interactionAssembler: PodBayDoorsInteractionAssembling

    func assemble() -> UIViewController {
        return DiscoveryOneViewController(interaction: interactionAssembler.assemble())
    }
}

let interactionAssembler = PodBayDoorsInteractionAssembler()
let viewControllerAssembler = DiscoveryOneAssembler(interactionAssembler: interactionAssembler)
let viewController = viewControllerAssembler.assemble()
```
## 🧭 Navigation

> **Navigation**
> (*noun*)
> 
> Component responsible for presenting views (here: View Controllers).

### Definitions

### TL;DR

Interaction may need to open another View Controller in response to user's action, it has to use Navigation component to do that.

#### Navigation is:
- A class,
- owned by Interaction,
- can push or open views modally,
- will use Assemblers to create ViewControllers to show.

### Example

📌 RULE: We should never pass classes directly, always via protocol or Adapter.

**NavigationControlling.swift**
```swift
protocol NavigationControlling: AnyObject {
    func pushViewController(_ viewController: UIViewController, animated: Bool)
}

// `NavigationControlling` is a protocol convering (some) methods of `UINavigationController`
extension UINavigationController: NavigationControlling { }
```

**Step 1 - Create Navigating protocol:**

📌 RULE: When A uses B but A is not an owner of B, pass B via `use(_)` method, not in `init`.

**PodBayNavigation.swift**
```swift
protocol DiscoveryOneNavigating: AnyObject {
    func use(_ navigationController: NavigationControlling)
    func presentDiscoveryOneInterface()
}
```

**Step 2 - Implement Navigation:**

**DiscoveryOneNavigation.swift**
```swift
final class DiscoveryOneNavigation: DiscoveryOneNavigating {

    private let discoveryAssembler: DiscoveryOneAssembling
    private weak var navigationController: NavigationControlling?

    init(navigationController: NavigationControlling, discoveryAssembler: DiscoveryOneAssembling) {
        self.navigationController = navigationController
        self.discoveryAssembler = discoveryAssembler
    }

    func use(_ navigationController: NavigationControlling) {
        self.navigationController = navigationController
    }

    func presentDiscoveryOneInterface() {
        let discovery = discoveryAssembler.assemble()
        navigationController.pushViewController(discovery, animated: true)
    }
}
```

# Additional assets

- [Cookie Cutter Template](https://github.com/bart-g/Super-Simple-Module-Template) by my friend [@bart-g](https://github.com/bart-g)
