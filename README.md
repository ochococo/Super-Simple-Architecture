![Super Simple Architecture](SuperSimple.png)

![Swift5](https://img.shields.io/badge/%20in-swift%205.0-orange.svg)

# Super Simple

### Application Architecture you can learn in minutes.

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
- dead, 
- immutable struct,
- threadsafe,
- often converted from model or models,
- responsible for formatting data for presentation text (ex. dates, numbers).

```swift
struct BannerRenderable {
    let message: String
}
```

#### View should:
- always look the same if ordered to render the same renderable,
- have code answering to the question: "how should I look?",
- have no state,
- accept infinite number of renderables in random order.

### Example

**Step 1 - Create Renderable:**

```swift
struct BannerRenderable {
    let message: String
    
    init(what: String) {
        message = "Only 🍎 has \(what)" // #courage ;)
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
- active component (has a lifecycle), 
- final class (1),
- can use other components for data retrieval,
- can convert Model(s) to Renderable(s) ex. via map,
- should call render methods on weakly held views (seen via protocol).

*(1) There is no good reason for inheritance of custom classes, ever.*

#### ViewControllers and Views:

- can have many Interactions for separate functions,
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

**Step 3 - Use it in ViewController (via nib + init):**

```swift
final class HAL9000ViewController: UIViewController {
    
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
> Assembler is assembling dependencies and instantiating one component.

### Definitions

### TL;DR

Its purpose is to assemble dependencies for one class or struct. 

#### Assembler:
- a struct,
- as only one method called `assemble()`. 
- can call ONLY ONE initializer directly,
- will call other Assemblers to instantiate dependencies.

### Example

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