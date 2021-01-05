# Add SwiftUI Component : Star Rater View

* Proposal: [SE-NNNN](NNNN-filename.md)
* Authors: Anamika Sharma
* Review Manager: TBD
* Status: **Awaiting implementation**

*During the review process, add the following fields as needed:*

* Implementation: [apple/swift#NNNNN](https://github.com/apple/swift/pull/NNNNN) or [apple/swift-evolution-staging#NNNNN](https://github.com/apple/swift-evolution-staging/pull/NNNNN)
* Decision Notes: [Rationale](https://forums.swift.org/), [Additional Commentary](https://forums.swift.org/)
* Bugs: [SR-NNNN](https://bugs.swift.org/browse/SR-NNNN), [SR-MMMM](https://bugs.swift.org/browse/SR-MMMM)
* Previous Revision: [1](https://github.com/apple/swift-evolution/blob/...commit-ID.../proposals/NNNN-filename.md)
* Previous Proposal: [SE-XXXX](XXXX-filename.md)

## Introduction

Add a SwiftUI API that allows user to design a star rating UI. 

## Motivation

At the moment, we do not have any SwiftUI API to generate a rater view which would allow user to rate a review, text or their experience. We propose a solution that creates a list of subviews of image with each star tagged to a rating number.

This new API will allow users to design the rating view easily by just setting the properties. 

## Proposed solution
We create Rating View by providing rating binding, a label, and the max rating. 

* The ```rating``` is an integer parameter that stores the current rating. Set the ```rating``` parameter to a state property that provides the value to display as the current rating. 
* The ```label``` is a custom text message that visually describes the purpose of Rating View.

We allow users to customize the rating view through various properties to add more flexibility. The customizable properties here mean that:

* Label : The label should be placed before the rating , default  is an empty string
* Maximum integer rating: The maximum rating the user is allowed to tap. [Default is 5 and minimum is 1]
* The selected and unselected colors signify the states (selected or not) of the star. [Default : yellow for selected and gray for unselected]
* The selected and unselected image, means the images to use when the star if highlighted/selected or not. [Default :selected is a star filled and unselcted is nil image.]

The following example shows a Rating View that displays star rater, a local state variable ```rating``` , to set the control's rating given by user. The default rating ranges from 1 to 5.

```
struct ContentView: View {
    @State private var rating = 4
    
    var body: some View {
        NavigationView {
            Section {
                VStack{
                    RatingView(
                     "Current rating \(rating)",
                      rating: $rating,
                      maxRating: 8
                    )
                    .padding(10)
                }  
            }.navigationBarTitle("Swift UI Rater")
        }
    }
}

```

## Detailed design

This section contains the design of the Rating View. 

### Declaration

```
struct RatingView: View{
    @Binding var rating: Int
    
    var titleText : String?
    var maxRating  = 5
    var unselectedImage : Image?
    var selectedImage = Image(systemName: "star.fill")
    
    var unselectedColor = Color.gray
    var selectedColor = Color.yellow
}
```

### Creating Rating View

```init(rating:Binding<Int>)```

Creates a Rating View that displays a star rating with default maximum rating to 5.


```init<S>(_ title: S, rating: Binding<Int>, maxRating: Int) where S : StringProtocol```

Creates a Rating View that displays a star Rating View with a custom label and user specified maximum rating as shown in example above.

```init<S>(_ title: S, rating: Binding<Int>, unselectedImage: Image?, selectedImage: Image?,  unselectedColor : Color , selectedColor : Color) where S : StringProtocol```
This is an elaborate API which provids the user maximum flexibility in desiging the UI of the Rating View. It allows user to define custom image and colors for selected and unselected state.

### Implementation of init methods

```
init(rating:Binding<Int>){
    self._rating = rating
}
```
``` 
init<S>(_ title: S, rating: Binding<Int>, maxRating: Int) where S : StringProtocol {
    self.titleText = title as? String
    self._rating = rating
    self.maxRating = maxRating
}
```
``` 
init<S>(_ title: S, rating: Binding<Int>, unselectedImage: Image?, selectedImage: Image?,  unselectedColor : Color , selectedColor : Color) where S : StringProtocol{
    self.titleText = title as? String
    self._rating = rating
    if let image = selectedImage {
        self.selectedImage = image
    }
    if let image = unselectedImage {
        self.unselectedImage = image
    }
    self.selectedColor = selectedColor
    self.unselectedColor = unselectedColor
}

```
### Implementation of Rating View 
The body of the Rating View is HStack with label and maximum stars or any other image provided by user. The way we show the image of each rating is: 

* If the rating tapped by user is less than or equal to the current rating, then return the selected image. 
* If the rating tapped by user is greater than the current rating, return the unselected image if it was set, otherwise return the selected Image.

Mentioned in the ```private func image(for number:Int) -> Image```

```
var body: some View {
    if let label = titleText {
        Text(label).padding(10)
    }
    HStack{
        ForEach(1..<maxRating+1){
            number in self.image(for: number).foregroundColor(number > self.rating ? self.unselectedColor : self.selectedColor)
                .onTapGesture {
                    self.rating = number
                }
        }
    }

}

private func image(for number:Int) -> Image {
    if number > rating {
        return unselectedImage  ?? selectedImage
    }else{
        return selectedImage
    }
}
```

## Availability
iOS 13.0+, 
macOS 10.15+, 
tvOS 13.0+, 
watchOS 6.0+

## Effect Access Control Modifiers

All the access view modifiers are applicable mentioned in [Access Control Modifiers](https://developer.apple.com/documentation/swiftui/stepper-view-modifiers)
with deprecated modifiers as 

``` func accessibility(selectionIdentifier: AnyHashable) -> ModifiedContent<RatingView<Label>, AccessibilityAttachmentModifier>```

Sets a selection identifier for this view’s accessibility element.

``` frame()```

Positions the view in an invisible frame

## Future Evolution

Currently, the stars are binded to intger ratings. Possibly in future we can add star updating half and also in decimal points. 
We can also use sliders to update the star ratings in decimal points.

