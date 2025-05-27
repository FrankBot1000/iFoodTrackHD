# iFoodTrack HD
iFoodTrack HD is the iPad version of my iFoodTrack macOS App (Swift/AppKit), built from the ground up using Swift and SwiftUI and designed in the simplist possible way with the iPad's touch interface in mind. It also integrates the USDA FoodCentral database. You can look up food details, save favorites, create meals, build a food diary and track nutrient and food count totals in charts. 

Like the macOS version, it is organized with a left navigation bar, a middle detail view, for charts and lists, and a right extended details view. However, the right view instead of describing data averages and trends, it describes the nutrient data for the currently selected item in the food list, or describes the current food totals for a selected diary date.

Currently, the mac version have more features, like file saving/exporting capabilities and more complex chart types. The mac version also uses custom built charts, while this iPadOS version is using Apple's Swift Charts API.
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
* Documentation (DocC)

## Other
* Git Version Control
<br></br>


# iFoodTrack HD Animation
[//]: # "NB: For README.md Github videos, Use GitHub asset urls eg. https://github.com/user-attachments/assets/xxxxxPlaceholderFileNameHerexxxxx as video source (derived first by dragging-dropping a video within the README.md file to get the url)."
<video width="500" src="https://github.com/user-attachments/assets/8b58453f-f182-4ddb-9d8a-86c44d8a078b">
</video>




[//]: # "For webpage, use embedded below figure instead"
<!-- 
<figure>
	<div>
		<video width="500" controls poster="videos/02b iFTiPad Favorites Restaurant Fish Fillet ABS 051725.png" muted preload="auto">
			<source src="videos/iFTiPadOS_iPad12_9inch_landscape_compressed.mp4" type="video/mp4">
			<!- - For non-HTML5 browsers: - ->
			Your browser doesn't support the video tag. Click <a href=http://www.firefox.com>here</a> 
			to download the Firefox browser for your operating system.
		</video>
	</div>
</figure>
 -->

# Rebuilding a macOS AppKit project as an iPadOS SwiftUI project...
iFoodTrack was originally designed and built for macOS using Swift, AppKit and Programmatic-UI. It's navigation layout and complexity favours a desktop-style window application. I finally gave in to the many requests for the iPad version, and decided to start from the ground up using SwiftUI. It was an opportunity to give the App a fresh start that demanded a more simplified layout on the iPad. It was also a chance to implement SwiftUI-like declarative programming paradigms.

I assumed that there would be brick walls to break through, given the newer nature of SwiftUI, but that using this declarative programming style, with less UI code writing, would compensate for the slow downs in development. It was definitely a bit of a mind f__k at the start, when switching for the more traditional imperative programming style, but overall the processes has been a good investment.
<br></br>


## Sample Code:
### Plotting daily nutrient totals in time-bar charts:
```swift
/// Will draw nutrient-specific nutrient total value per date bar of bar graph.
    func drawBarGraphDataSegments(_ cgContext: CGContext?) {
        let lineWidth    = 2.0                              // initialize with zero value
        var y            = lineWidth - 1.0 + xLabelHeight   // start half way up width of axis line
        
        // total the value of all bars ( = "segments") by using reduce to sum them
        guard let maxValue  = segments.map({Double($0.value)}).max() else { return }
        var maxBarHeight    = maxValue > 0 ? maxValue : 100.0   // protects against dividing by zero.
        
        // Check if drv100Percent value is greater than maxValue, then set max to drv100Percent value
        maxBarHeight        = drv100PercentValue >= maxValue ? drv100PercentValue : maxValue
        
        let roundedRect     = FBRoundedRect(radius: 2.5, corners: [.bottomLeft, .bottomRight])
        // To round the top of bar, use bottomLeft and bottomRight, since drawing starts from bottom to top
        
        // Loop through the values array:
        for (index, segment) in segments.enumerated() {
            let i: CGFloat  = CGFloat(index)
            let x: CGFloat  = i * (barWidth + spacing) + offset + yLabelHeight
            let value       = segment.value > 0 ? segment.value : 0 // set to zero if value is negative.
            
            let barHeight   = (value/maxBarHeight * (viewHeight - xLabelHeight))
            if barHeight == 0 { continue }  // If barHeight was zero, then skip drawing the bar! Prevents drawing artifacts on X-Axis.
            
            let bar         = CGRect(x: x, y: y, width: barWidth, height: barHeight)
            var path        = CGPath(rect: .zero, transform: nil)
            path            = roundedRect.path(in: bar) ?? CGPath(roundedRect: bar, cornerWidth: 2.5, cornerHeight: 2.5, transform: .none)
            
            cgContext?.setFillColor(self.nutrientColor.cgColor)
            cgContext?.addPath(path)
            cgContext?.fillPath()
            
            y = lineWidth - 1.0 + xLabelHeight     // reset y
        }
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


### A test to confirm conversion of double values to scientific notation, for display in chart y-axis labels:
```swift
    func testConvertDoubleToSciNotationForGraph() {
        // Given
        let segmentValues = [0,
                            1,
                             1.2,
                             1.23,
                             1.2345678,
                            1234,
                             1234.5678,
                             0.12345678,
                             0.012345678,
                             0.0012345678]

        let numFormatter = NumberFormatter()
        numFormatter.maximumFractionDigits = 1
        numFormatter.numberStyle = .scientific

        for segmentValue in segmentValues {
            let myNum_sciString     = numFormatter.string(for: segmentValue) ?? ""
            
            // When
            let myNum_beforeDecimal   = myNum_sciString.prefix(3)
            
            // Will include "E" if was a whole number or zero, so only keep first character:
            var myNum_coefficientPart = myNum_beforeDecimal // NB: will contain "E" when is a whole number or zero.
            if myNum_coefficientPart.contains("E") {
                myNum_coefficientPart.removeLast(2)
            }
            
            let exponentEIndex      = myNum_sciString.firstIndex(of: "E") // will need to access original 'myNum_sciString' with it's "E" part.
            var myNum_exponent      = myNum_sciString.suffix(from: exponentEIndex!)
            _                       = myNum_exponent.removeFirst()      // removes, and returns, first element
            let exponentWithoutE    = String(myNum_exponent)
            
            print("\(myNum_coefficientPart) x 10E\(exponentWithoutE)")
//            0 x 10E0
//            1 x 10E0
//            1.2 x 10E0
//            1.2 x 10E0
//            1.2 x 10E0
//            1.2 x 10E3
//            1.2 x 10E3
//            1.2 x 10E-1
//            1.2 x 10E-2
//            1.2 x 10E-3
            
            // Then
            XCTAssert(myNum_beforeDecimal.count != 0, "There's nothing to represent the decimal part!")
            XCTAssert(exponentWithoutE != "", "There's no exponent value")
        }
    }
```
<br></br>


### Sample code structure for App Navigation:
```swift
// MARK: - FBNavigationDelegate Methods
extension FBMainSplitVC: FBNavigationDelegate {
    
    /// Set the middleVC based on selected Navigation Bar in leftVC.
    func setMiddleVC_forNavBar(at navBarIndex: kNavBarIndex) {
        var middleVCs: (graph: FBMiddleGraphVC, list: FBMiddleListVC)
        switch navBarIndex {
            case .library:      middleVCs = libraryGraphAndFoodListVCs()
            case .favorites:    middleVCs = favoritesGraphAndFoodListVCs()
            case .diary:        middleVCs = diaryGraphAndFoodListVCs()
            case .timeChart:    middleVCs = timeChartGraphAndFoodListVCs()
            case .trash:        middleVCs = trashGraphAndFoodListVCs()
        }
        setMiddleVCSplitViewItems(topVC: middleVCs.graph, bottomVC: middleVCs.list)
    }
    
    
    /// Sets up Library Graph and Library FoodList view controllers.
    ///
    /// A "mini" library of all_SRRFoods_library is loaded from disk.
    func libraryGraphAndFoodListVCs() -> (FBMiddleGraphVC, FBMiddleListVC) {
        let graphVC             = FBLibraryGraphVC(chartView: FBPieGraphView(frame: .zero))
        let graphTopToolBarVC   = FBGraphTopToolBarVC(identifier: .topToolBarVCID_libraryGraph)
        
        let listVC              = FBLibraryListVC(fdrfoods: all_FDRFoods_library,
                                                  identifier: .listVCID_library)
        let listTopToolBarVC    = FBLibraryListTopToolBarVC()
        let listBottomToolBarVC = FBFoodListBottomToolBarVC(popUpTitles: kFoodCategoryName.allNames)
        
        setGraphVC_FoodListVC_infoPanel_Delegates(graphVC: graphVC, 
                                                  graphTopToolBarVC: graphTopToolBarVC,
                                                  listVC: listVC,
                                                  listTopToolBarVC: listTopToolBarVC,
                                                  listBottomToolBarVC: listBottomToolBarVC)
        
        let topVC               = FBMiddleGraphVC(graphVC: graphVC, 
                                                  topToolBarVC: graphTopToolBarVC,
                                                  foodListTopToolBarVC: listTopToolBarVC)
        let bottomVC            = FBMiddleListVC(listVC: listVC,
                                                 topToolBarVC: listTopToolBarVC,
                                                 bottomToolBarVC: listBottomToolBarVC)
        return (topVC, bottomVC)
    }
    ... ...
```
<br></br>


# Sample Screen Shots
<table>
	<tr>
		<td>
		<img src="images/screenshots/01 Log Meals In Your Food Diary 051725b.png" alt="iFoodTrack Diary View" width="500"/>
		</td>
		<td>
		<img src="images/screenshots/02 Search Foods and Save Favorites 051725b.png" alt="iFoodTrack Diary Dark" width="500"/>
		</td>
	</tr>
	<tr>
		<td>
		<img src="images/screenshots/03 Create Meals for Quick Logging 051725b.png" alt="iFoodTrack Time Chart" width="500"/>
		</td>
		<td>
		<img src="images/screenshots/04 View Daily Weekly Monthly Charts 051725b.png" alt="iFoodTrack Trash View" width="500"/>
		</td>
	</tr>
	<tr>
		<td>
		<img src="images/screenshots/05 Fine-Tune Nutrient Settings and Look 051725b.png" alt="Settings Appearance" width="500"/>
		</td>
		<td>
		<img src="images/screenshots/06 Customize Look Switch to Dark Mode 051725b.png" alt="Settings Advanced" width="500"/>
		</td>
	</tr>
</table>


## I've implemented the following:
#### Code Structure
* Model-View-View-Model (MVVM) Design Pattern
* SwiftUI Bindings, @StateObject, @ObservableObject, @EnvironmentObject
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

#### Security
* Validation checks, in Assembly

#### User Customizations
* Settings Startup Options
* Unit Conversions
* Theme Colors
* Custom Threshold Values
* Light and Dark Mode Selections

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




