https://x.com/willing/status/1957641530375901217?t=yo1pEkqEKOhiUJ5pOz-9UA&s=19

How to make custom layout in swiftui and how to make it load lazy (Swift lazy feature) 

> Grok response


### Creating a Custom Layout in SwiftUI

In SwiftUI (available since iOS 16), you can create custom layouts by defining a struct that conforms to the `Layout` protocol. This allows you to control how subviews are sized and positioned, enabling complex arrangements like grids, waterfalls, or morphing menus (as seen in the Waymo app example from the X post you linked).

The `Layout` protocol requires two main methods:
- `sizeThatFits`: Calculates the overall size of the layout based on the proposed size and subviews.
- `placeSubviews`: Positions each subview within the given bounds.

Here's a basic example of a custom layout that arranges subviews in a horizontal row with equal spacing:

```swift
import SwiftUI

struct EqualSpacingLayout: Layout {
    var spacing: CGFloat = 10
    
    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
        let totalWidth = subviews.reduce(0) { $0 + $1.dimensions(in: proposal).width } + spacing * CGFloat(max(0, subviews.count - 1))
        let maxHeight = subviews.map { $0.dimensions(in: proposal).height }.max() ?? 0
        return CGSize(width: totalWidth, height: maxHeight)
    }
    
    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
        var x = bounds.minX
        for subview in subviews {
            let subviewSize = subview.dimensions(in: proposal)
            subview.place(at: CGPoint(x: x, y: bounds.midY), anchor: .center, proposal: ProposedViewSize(subviewSize))
            x += subviewSize.width + spacing
        }
    }
}

// Usage
EqualSpacingLayout {
    Text("Item 1").background(Color.blue)
    Text("Item 2").background(Color.green)
    Text("Item 3").background(Color.red)
}
```

This layout measures each subview's size and places them horizontally with the specified spacing. You can extend this for more complex behaviors, like the morphing menu in the Waymo example, by animating positions or sizes based on scroll gestures (using `ScrollView` with modifiers like `onScrollGeometryChange` and `onScrollPhaseChange`, as mentioned in the X post).

For the morphing effect in the post, the custom layout likely handles transitioning between a compact bottom bar and an expanded view, temporarily adjusting content size to match the screen for bounce feedback during gestures.

### Making a Custom Layout Load Lazily (Using Swift's Lazy Features and View Recycling Tricks)

By default, custom layouts in SwiftUI are *eager*: All subviews are instantiated and measured upfront to determine the layout's size, which can hurt performance with large datasets (e.g., lists or grids). This contrasts with built-in lazy containers like `LazyVStack` or `LazyHGrid`, which only create views as they become visible.

To make a custom layout lazy, you can use "tricks" like:
- **Breaking into chunks**: Wrap sections of your custom layout in a `LazyVStack` to defer loading non-visible parts.<grok:render card_id="6a666c" card_type="citation_card" type="render_inline_citation">
<argument name="citation_id">10</argument>
</grok:render> This is useful for vertical scrolling but may require manual pagination.
- **View recycling**: Combine a `ScrollView` with `onScrollGeometryChange` to track visible ranges, then only render visible subviews while reusing view identities (via modulo indexing) to recycle them efficiently. This mimics lazy behavior in a fully custom setup.
- **Swift's `lazy` keyword**: For non-view elements, use `lazy var` properties in your layout struct to defer expensive computations (e.g., caching sizes) until first access. However, this doesn't directly lazy-load views—it's more for optimizing internal state.

A powerful trick for true lazy loading in custom layouts is to implement view recycling, as detailed in a performance-focused approach.<grok:render card_id="a2781f" card_type="citation_card" type="render_inline_citation">
<argument name="citation_id">11</argument>
</grok:render> Here's how it works:

1. Use `ScrollView` to detect the visible range via `onScrollGeometryChange`.
2. Generate only the visible subviews, but assign recycled IDs (e.g., index % visibleCount) to maintain consistent identities for SwiftUI's diffing.
3. Use a custom `Layout` to position them at absolute offsets, even for high indices.

Example code for a custom lazy vertical list with recycling (adapted for simplicity; assumes fixed row height):

```swift
import SwiftUI

struct RecyclingLazyList<ID: Hashable, Content: View>: View {
    let rowIDs: [ID]  // Data source
    let rowHeight: CGFloat
    @ViewBuilder let content: (ID) -> Content
    @State private var visibleRange: Range<Int> = 0..<1
    
    var body: some View {
        ScrollView {
            OffsetLayout(totalRowCount: rowIDs.count, rowHeight: rowHeight) {
                ForEach(visibleRows) { row in
                    content(row.value)
                        .frame(height: rowHeight)
                        .id(row.id)  // Recycled ID for reuse
                        .layoutValue(key: LayoutIndex.self, value: row.index)
                }
            }
        }
        .onScrollGeometryChange(for: Range<Int>.self, of: { geo in
            computeVisibleRange(in: geo.visibleRect)
        }, action: { _, newValue in
            visibleRange = newValue
        })
    }
    
    private func computeVisibleRange(in rect: CGRect) -> Range<Int> {
        let lower = max(0, Int(floor(rect.minY / rowHeight)))
        let rowsInView = Int(ceil(rect.height / rowHeight)) + 2  // Buffer for smoothness
        let upper = min(rowIDs.count, lower + rowsInView)
        return lower..<upper
    }
    
    private var visibleRows: [RowData] {
        let range = visibleRange
        return range.map { index in
            RowData(id: index % range.count,  // Recycle ID
                    index: index,
                    value: rowIDs[index])
        }
    }
    
    struct RowData: Identifiable {
        let id: Int
        let index: Int
        let value: ID
    }
}

struct OffsetLayout: Layout {
    let totalRowCount: Int
    let rowHeight: CGFloat
    
    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
        CGSize(width: proposal.width ?? 0, height: rowHeight * CGFloat(totalRowCount))
    }
    
    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
        for subview in subviews {
            let index = subview[LayoutIndex.self]
            subview.place(at: CGPoint(x: bounds.minX, y: bounds.minY + rowHeight * CGFloat(index)),
                          proposal: ProposedViewSize(width: proposal.width, height: rowHeight))
        }
    }
}

struct LayoutIndex: LayoutValueKey {
    static let defaultValue: Int = 0
}

// Usage
RecyclingLazyList(rowIDs: Array(0..<1000), rowHeight: 50) { id in
    Text("Row \(id)").background(Color.gray.opacity(0.2))
}
```

This setup loads only visible rows lazily, recycles views as you scroll (preserving state like text fields), and uses a custom layout for precise positioning. For variable heights, you'd need to precompute or estimate sizes.

In the context of the X post, the "tricks to make it lazy" likely involve similar recycling or chunking for the expandable menu's content, ensuring smooth performance during morphing and scrolling. If your menu has dynamic content, integrate this with gesture handlers for the minimize/expand behavior.

If this doesn't match your exact needs, consider wrapping in `UIViewRepresentable` with `UICollectionView` for even better lazy performance in complex cases.<grok:render card_id="661679" card_type="citation_card" type="render_inline_citation">
<argument name="citation_id">10</argument>
</grok:render>