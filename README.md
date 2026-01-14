# iFoodTrack Daily
iFoodTrack Daily is the iPad version of the iFoodTrack macOS App (Swift/AppKit), built from the ground up using Swift and SwiftUI and designed in the simplist possible way with the iPad's touch interface in mind. It also integrates the USDA FoodCentral database. You can look up food details, save favorites, create meals, build a food diary and track nutrient and food count totals in charts. 

Like the macOS version, it is organized with a left navigation bar, a middle detail view, for charts and lists, and a right extended details view. However, the right view describes the nutrient data for the currently selected item in the food list, or describes the current food totals for a selected diary date, instead of describing averages and nutrient trends.

Currently, the mac version has more features, like file saving/exporting capabilities and more complex chart types. The mac version also uses custom built charts, while this iPadOS version is using Apple's Swift Charts API.
<br></br>


<h2>Available on the iOS AppStore:</h2>
<a href="https://apps.apple.com/us/app/ifoodtrack-hd/id6593659637"> 
	<img src="images/pics/Download_on_the_App_Store_Badge_US-UK_RGB_blk_092917.svg" alt="Download on iOS App Store"</img> 
</a>
<br></br>


# Technologies Used
## Languages and Frameworks							
* Swift Programming language
* Assembly Language
* SwiftUI framework

## Apple Technologies
* Swift Charts
* StoreKit
* Unit Testing
* Accessibility (Voice Over)
* Documentation (DocC)

## Other
* Git Version Control
<br></br>


# iFoodTrack Daily Animation
[//]: # "NB: For README.md Github videos, Use GitHub asset urls eg. https://github.com/user-attachments/assets/xxxxxPlaceholderFileNameHerexxxxx as video source (derived first by dragging-dropping a video within the README.md file to get the url)."
<video width="500" src="https://github.com/user-attachments/assets/216ec31b-b227-4491-8f4f-d80cedd22189">
</video>


[//]: # "For webpage, use embedded below figure instead"
<!-- 
 <div style="display: inline-block">
     <figure>
         <div>
             <video width="500" controls poster="videos/01a2 iFTiPadOS26 Diary Milk Pie Chart PB.png" muted preload="auto">
                 <source src="videos/iFTiPadOS26_iPad12_9inch landscape compressed 092925.mp4" type="video/mp4">
                 <!- - For non-HTML5 browsers: - ->
                     Your browser doesn't support the video tag. Click <a href=http://www.firefox.com>here</a>
                     to download the Firefox browser for your operating system.
             </video>
         </div>
     </figure>
 </div>
 -->

# Rebuilding a macOS AppKit project as an iPadOS SwiftUI project...
iFoodTrack was originally designed and built for macOS using Swift, AppKit and Programmatic-UI. It's navigation layout and complexity favours a desktop-style window application. I finally gave in to the many requests for the iPad version, and decided to start from the ground up using SwiftUI. It was an opportunity to give the App a fresh start that demanded a more simplified layout on the iPad. It was also a chance to implement SwiftUI-like declarative programming paradigms.

I assumed that there would be brick walls to break through, given the newer nature of SwiftUI, but that using this declarative programming style, with less UI code writing, would compensate for the slow downs in development. It was definitely a bit of a mind f__k at the start, when switching from the more traditional imperative programming style, but overall the processes has been a good learning experience.
<br></br>


## Sample Code:
### A test for reading HealthKit Sample Data::
```swift
/// Read HKSampleType data. Will fail if permissions not set in test Simulator Clone's System Settings for the Health App, and/or if Privacy key-value pairs not set for FBHealthKitTests unit testing bundle info.plist.
    ///
    /// In Health App, set the Nutrition/Dietary Energy Burned value for today to "1350" for the Simulator Clone that runs the test. Is same value used by kMockData.fdrFoodsWithRandomDates.first FDRFood that has a calories value of 1350.
    ///
    /// Also, within the Clone's System Settings, enable permissions for reading Health Nutrition data (for dietary Energy Burned value that's read in this method).
    ///
    /// And, for the FBHealthKitTests target unit test bundle, include Privacy - Health Share Usage Description and Update Usage Description Info.plist key-values.
    func testGetHKSampleType_readHealthKitSampleData() {
        // Given
        let identifier: HKQuantityTypeIdentifier = .dietaryEnergyConsumed
        var quantity: Double = 0.0
        
        // When
        guard let sampleType = HKSampleType.quantityType(forIdentifier: identifier) else {
            print("\(identifier.rawValue) sample Type is no longer available in HealthKit")
            return
        }
        
        let asyncDataSetAssetExpectation = expectation(description: "asyncHealthKitExpectation")
        
        FBHealthKit.getHKSampleType(sampleType, date: Date()) { (sample, error) in
            guard let sample = sample else {
                if let error = error {
                    print("There was an error retreiving sample type: \(error)")
                }
                // Then
                asyncDataSetAssetExpectation.fulfill()
                quantity = sample?.quantity.doubleValue(for: HKUnit.kilocalorie()) ?? 0.0
                print("error... quantity: \(quantity)")
                XCTAssert(quantity == 1350, "Quantity was not 1350.0 kCal.")
                return
            }
            
            // Then
            asyncDataSetAssetExpectation.fulfill()
            print("sample: \(sample)")
            quantity = sample.quantity.doubleValue(for: HKUnit.kilocalorie())
            print("quantity: \(quantity)")
        }
        self.wait(for: [asyncDataSetAssetExpectation], timeout: 10)
        
        // Then
        XCTAssert(quantity == 1350.0, "Quantity was not 1350.0 kCal.")
    }
```
<br></br>


### A test for validating nutrient value calculations:
```swift
/// Test will test Food Nutrient DRV percent calculation using DRV nutrient defaults. The result should yield 100% for all DRV Nutrient calculations.
    func testCalculateNutValues_recalculateDRVPercent() {
        // Arrange
        let other6NutItemIndexes: [kNutrientNameIndex] = FBDRVSettingsHelper.shared.nutNameIndexes
        
        // MARK: Set up NutIDs:
        // topNutIDs, midNutIDs, and bottomNutIDs split 'other6NutIems' into subarray item pairs:
        var otherNutIDs: (topPair:[kNutrientID], midPair:[kNutrientID], bottomPair:[kNutrientID]) {
            let topPair = other6NutItemIndexes[0...1].map({$0.rawValue})
            let midPair = other6NutItemIndexes[2...3].map({$0.rawValue})
            let btmPair = other6NutItemIndexes[4...5].map({$0.rawValue})
            let NutPairs = [topPair, midPair, btmPair]
            let NutIDPairs = NutPairs.map{$0.map{kNutrientID.allNutrientIDs[$0]}}
            return (NutIDPairs[0], NutIDPairs[1], NutIDPairs[2])
        }
        
        var topNutIDs: [kNutrientID]    { otherNutIDs.topPair } // recompute .topPair
        var midNutIDs: [kNutrientID]    { otherNutIDs.midPair } // recompute .midPair
        var bottomNutIDs: [kNutrientID] { otherNutIDs.bottomPair } // recompute .bottomPair
        
        let otherNutIDsArrays   = [topNutIDs, midNutIDs, bottomNutIDs]
        
        // MARK: Create DRV Food (a token food):
        let nutIDs      = kNutrientID.allCases.map{$0.rawValue}
        let nutNames    = kNutrientName.allCases.map{$0.rawValue}
        let nutNumbers  = kNutrientNameIndex.allNutrientIndexes
        let nutModifers = kNutrientNameIndex.allUnitModifiers
        let nutDRV      = NutrientDRV()

        var nutrients: [FDRNutrient] = []

        // Iterate through all nutrients, based on nutrient name case number (index) in kNutrientNameIndex enum:
        for n in nutNumbers {
            nutrients.append(FDRNutrient(amount: nutDRV.nutrientDRVDict[kNutrientNameIndex(rawValue: n)!], nutrient: FDRNutInfo(id: nutIDs[n], number: "0", name: nutNames[n], rank: 0, unitName: nutModifers[n])))
        }
        
        
        var drvFood  = FDRFood(nutrients: nutrients)
        var drvAbsAmountLabelsArray: [FBNutAmountsAndLabels] = []
        let graphVC = FBGraphVC()
        
//        let expectedOtherNutIDsArrays: [FBNutAmountsAndLabels] = [
//        FBNutAmountsAndLabels(nutAmounts: (drvAmounts: [100, 100], absAmounts: [20, 28]), nutLabels: (drvLabels: ["%", "%"], absLabels: ["g", "g"])),
//        FBNutAmountsAndLabels(nutAmounts: (drvAmounts: [100, 100], absAmounts: [90, 300]), nutLabels: (drvLabels: ["%", "%"], absLabels: ["g", "mg"])),
//        FBNutAmountsAndLabels(nutAmounts: (drvAmounts: [100, 100], absAmounts: [1300, 2300]), nutLabels: (drvLabels: ["%", "%"], absLabels: ["mg", "mg"]))
//        ]
        
        // Act
        // Calculate selected Food amounts:
        for otherNutIDsArray in otherNutIDsArrays {
            guard let nutAmountsAndLabels = drvFood.calculateNutValues(nutrientIDs: otherNutIDsArray) else { return }
            drvAbsAmountLabelsArray.append(nutAmountsAndLabels)
        }
        
        let drvOtherNutIdsAmounts = drvAbsAmountLabelsArray.map{$0.nutAmounts}.map{$0.drvAmounts}
        let expectedDRVOtherNutIdsAmounts = [[100.0, 100.0], [100.0, 100.0], [100.0, 100.0]]
        
        // Assert
        XCTAssert(drvOtherNutIdsAmounts == expectedDRVOtherNutIdsAmounts)
        
    }
```
<br></br>


### Building a Calendar Date Picker for the Food Diary that includes Voice-over accessibility::
```swift
struct CalendarDatePicker: View {
    @Binding var selectedDate: Date
    
    var selectedDate_MDY: String {
        selectedDate.formatted(date: .abbreviated, time: .omitted)
    }
    
    var body: some View {
        HStack {
            Spacer()
            Button {
                print("Date Back Button.")
                selectedDate = selectedDate.increment(by: -1)
            } label: {
                Image(systemName: kSFSymbolName.backward)
                    .font(.system(size: 20.0))
            }
            .accessibilityRemoveTraits(.isButton)
            .accessibilityLabel(Text("Decrease Date Button. Date: \(selectedDate_MDY)."))
            
            DatePicker("", selection: $selectedDate, displayedComponents: [.date])
                .frame(width: 100.0) //...need a fixed width for system to center it.
                .padding([.leading, .trailing], 20.0)
            
            Button {
                print("Date Forward Button.")
                selectedDate = selectedDate.increment(by: 1)
            } label: {
                Image(systemName: kSFSymbolName.forward)
                    .font(.system(size: 20.0))
            }
            .accessibilityRemoveTraits(.isButton)
            .accessibilityLabel(Text("Increase Date Button. Date: \(selectedDate_MDY)."))
            
            Spacer()
        }
    }
}
```
<br></br>


### A ProgressThresholds ObservableObject that sets and saves threshold values for Linear ProgressViews:
```swift
class ProgressThresholds: ObservableObject {
    @AppStorage("lowValue") private var lowValueSetting: Double = 20.0
    @AppStorage("midValue") private var midValueSetting: Double = 60.0
    @AppStorage("hiValue") private var hiValueSetting: Double   = 80.0
    
    @Published var lowValue: Double = 20.0
    @Published var midValue: Double = 60.0
    @Published var hiValue: Double  = 80.0
    
    
    init() {
        getLowValue()
        getMidValue()
        getHiValue()
    }
    
    
    // --------------------------------------------------------
    // MARK: - Save/Get ProgressView lowValue threshold setting
    func saveLowValue() {
        lowValueSetting = lowValue
    }
    
    func getLowValue() {
        lowValue = lowValueSetting
    }
    
    
    // --------------------------------------------------------
    // MARK: - Save/Get ProgressView midValue threshold setting
    func saveMidValue() {
        midValueSetting = midValue
    }
    
    func getMidValue() {
        midValue = midValueSetting
    }
    
    
    // --------------------------------------------------------
    // MARK: - Save/Get ProgressView hiValue threshold setting
    func saveHiValue() {
        hiValueSetting = hiValue
    }
    
    func getHiValue() {
        hiValue = hiValueSetting
    }
}
```
<br></br>


### Linear ProgressViews that are thicker in style:
```swift
struct LinearProgressView: View {
    @EnvironmentObject var progressThresholds: ProgressThresholds
    
    @Binding var progress: Double
    let width: Double
    
    @State private var lowValue: Double = 0.3
    @State private var midValue: Double = 0.8
    @State private var hiValue: Double  = 0.81
    
    
    
    var strokeColor: Color {
        switch progress {
        case 0..<lowValue:          Color.red
        case lowValue...midValue:   Color.yellow
        case hiValue...:            Color.green
        default:                    Color.blue
        }
    }
    
    var body: some View {
        ProgressView(value: progress, total: 1.0)
            .progressViewStyle(ThickProgressViewStyle(width: width))
            .tint(strokeColor)
            .task {
                lowValue = progressThresholds.lowValue / 100
                midValue = progressThresholds.midValue / 100
                hiValue = progressThresholds.hiValue / 100
            }
    }
}

/// - Note: Unable to change the frame height alone, so use the .scaleEffect together with the .frame height and .clipShape, instead. Note, the drawbrack is that you can't include a label without also stretching it.
struct ThickProgressViewStyle: ProgressViewStyle {
    let width: Double
    func makeBody(configuration: Configuration) -> some View {
        ProgressView(configuration)
            .progressViewStyle(.linear)
            .frame(width: width, height: 10.0)
            .scaleEffect(x: 1, y: 10, anchor: .center)
//            .clipShape(RoundedRectangle(cornerRadius: 6))
            .clipShape(Capsule())
    }
```
<br></br>


# Sample Screen Shots
<table>
    <tr>
        <td>
            <img src="images/screenshots/02 iPadOS26 Add Meals to Your Food Diary 092925.png" alt="Add Meals to Your Food Diary" width="500" />
        </td>
        <td>
            <img src="images/screenshots/03 iPadOS26 Create Meals for Quick Logging 092925.png" alt="Create Meals for Quick Logging" width="500" />
        </td>
    </tr>
    <tr>
        <td>
            <img src="images/screenshots/04 iPadOS26 Discover Foods Save Favorites 092925.png" alt="Discover Foods Save Favorites" width="500" />
        </td>
        <td>
            <img src="images/screenshots/05 iPadOS26 View Detailed Meal Nutrition 092925.png" alt="View Detailed Meal Nutrition" width="500" />
        </td>
    </tr>
    <tr>
        <td>
            <img src="images/screenshots/06 iPadOS26 Customize Nutrition Goals and Tracking 092925.png" alt="Customize Nutrition Goals and Tracking" width="500" />
        </td>
        <td>
            <img src="images/screenshots/07 iPadOS26 Accessibility VoiceOver and DarkMode 092925.png" alt="Accessibility VoiceOver and DarkMode" width="500" />
        </td>
    </tr>
</table>


## I've implemented the following:
#### Code Structure
* Model-View-View-Model (MVVM) Design Pattern
* SwiftUI components: 
	* ObservableObject, @EnvironmentObject, @ObservedObject, @StateObject
	* @State, @Binding, @Published
* Delegates and Protocols
* Navigation Split Views
* Generics for Core Data objects
* Swift Charts
* Linear Progress Bar Views
* Apple Health Syncing
* Voice Over Accessibility

#### Testing/Error Handling
* Unit Testing
* Error Handling
* Alerts
* Empty States
* Random Generation of Sample Test Data
* Text Input Validation

#### Accessibility
* Light and Dark Mode Selections
* VoiceOver Accessibility (buttons & charts)

#### User Customizations
* Settings Startup Options
* Unit Conversions
* Theme Colors
* Custom Threshold Values

#### Project Organization
* Code Documentation (DocC)
* Privacy Manifest
* Group Folder Organization


# Future Considerations
* More Swift Charts, with trends over time.
* Implement Core Data's CloudKit syncing
<br></br>


# FeedBack
<p class="contact-message">If you have any feedback or suggestions you can reach out via <a class="btn" href="mailto:fbotlogic@fbotlogicsolutions.com?subject=Blue Marble Weather Support">Email</a>.</p>




