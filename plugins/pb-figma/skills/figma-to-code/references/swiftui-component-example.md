# SwiftUI Component Structure Example

> **Used by:** code-generator-swiftui

This reference shows the canonical structure for a generated SwiftUI component including MARK comments, DocC documentation, property wrappers, accessibility modifiers, and #Preview.

---

## CardView Example

```swift
import SwiftUI

/// A card component displaying title, description, and optional image
struct CardView: View {
  // MARK: - Properties

  /// Card title
  let title: String

  /// Card description
  let description: String?

  /// Optional image name from asset catalog
  let imageName: String?

  /// Card variant style
  let variant: CardVariant

  // MARK: - Body

  var body: some View {
    VStack(alignment: .leading, spacing: 16) {
      if let imageName = imageName {
        Image(imageName)
          .resizable()
          .aspectRatio(contentMode: .fill)
          .frame(height: 200)
          .clipped()
          .cornerRadius(12)
      }

      VStack(alignment: .leading, spacing: 8) {
        Text(title)
          .font(.title2)
          .fontWeight(.semibold)
          .foregroundColor(Color("TextPrimary"))

        if let description = description {
          Text(description)
            .font(.body)
            .foregroundColor(Color("TextSecondary"))
            .lineLimit(3)
        }
      }
    }
    .padding(24)
    .background(backgroundColor)
    .cornerRadius(16)
    .shadow(color: Color.black.opacity(0.1), radius: 8, x: 0, y: 4)
    .accessibilityElement(children: .combine)
    .accessibilityLabel(accessibilityDescription)
  }

  // MARK: - Computed Properties

  private var backgroundColor: Color {
    switch variant {
    case .elevated:
      return Color("CardBackground")
    case .outlined:
      return Color.clear
    }
  }

  private var accessibilityDescription: String {
    var desc = "Card: \(title)"
    if let description = description {
      desc += ". \(description)"
    }
    return desc
  }
}

// MARK: - Supporting Types

enum CardVariant {
  case elevated
  case outlined
}

// MARK: - Preview (iOS 17+)

#Preview("Light Mode") {
  CardView(
    title: "Sample Card",
    description: "This is a sample description for the card component.",
    imageName: "sample-image",
    variant: .elevated
  )
  .padding()
}

#Preview("Dark Mode") {
  CardView(
    title: "Sample Card",
    description: "This is a sample description for the card component.",
    imageName: "sample-image",
    variant: .elevated
  )
  .preferredColorScheme(.dark)
  .padding()
}

// MARK: - Preview (iOS 13-16 fallback)

struct CardView_Previews: PreviewProvider {
  static var previews: some View {
    Group {
      CardView(
        title: "Sample Card",
        description: "This is a sample description for the card component.",
        imageName: "sample-image",
        variant: .elevated
      )
      .previewLayout(.sizeThatFits)
      .padding()
      .previewDisplayName("Light Mode")

      CardView(
        title: "Sample Card",
        description: "This is a sample description for the card component.",
        imageName: "sample-image",
        variant: .elevated
      )
      .preferredColorScheme(.dark)
      .previewLayout(.sizeThatFits)
      .padding()
      .previewDisplayName("Dark Mode")
    }
  }
}
```
