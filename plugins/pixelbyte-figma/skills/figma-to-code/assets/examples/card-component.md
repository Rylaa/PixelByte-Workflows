# Example Conversion: Card Component

Bu örnek, Figma'dan alınan tasarım bilgisinin nasıl React/Tailwind koduna dönüştürüldüğünü gösterir.

## Figma Design Context (Input)

```json
{
  "node": {
    "name": "Card",
    "type": "FRAME",
    "width": 320,
    "height": "auto",
    "layoutMode": "VERTICAL",
    "primaryAxisSizingMode": "AUTO",
    "counterAxisSizingMode": "FIXED",
    "paddingTop": 24,
    "paddingBottom": 24,
    "paddingLeft": 24,
    "paddingRight": 24,
    "itemSpacing": 16,
    "cornerRadius": 12,
    "fills": [{ "type": "SOLID", "color": { "r": 1, "g": 1, "b": 1 } }],
    "effects": [{
      "type": "DROP_SHADOW",
      "color": { "r": 0, "g": 0, "b": 0, "a": 0.1 },
      "offset": { "x": 0, "y": 4 },
      "radius": 6
    }],
    "children": [
      {
        "name": "Image",
        "type": "RECTANGLE",
        "width": 272,
        "height": 180,
        "cornerRadius": 8,
        "fills": [{ "type": "IMAGE", "scaleMode": "FILL" }]
      },
      {
        "name": "Content",
        "type": "FRAME",
        "layoutMode": "VERTICAL",
        "itemSpacing": 8,
        "children": [
          {
            "name": "Title",
            "type": "TEXT",
            "characters": "Card Title",
            "style": {
              "fontFamily": "Inter",
              "fontWeight": 600,
              "fontSize": 18,
              "lineHeightPercent": 133,
              "letterSpacing": -0.3
            },
            "fills": [{ "type": "SOLID", "color": { "r": 0.1, "g": 0.1, "b": 0.1 } }]
          },
          {
            "name": "Description",
            "type": "TEXT",
            "characters": "This is a description of the card content that spans multiple lines.",
            "style": {
              "fontFamily": "Inter",
              "fontWeight": 400,
              "fontSize": 14,
              "lineHeightPercent": 157,
              "letterSpacing": 0
            },
            "fills": [{ "type": "SOLID", "color": { "r": 0.4, "g": 0.4, "b": 0.4 } }]
          }
        ]
      },
      {
        "name": "Button",
        "type": "FRAME",
        "layoutMode": "HORIZONTAL",
        "primaryAxisAlignItems": "CENTER",
        "counterAxisAlignItems": "CENTER",
        "paddingTop": 12,
        "paddingBottom": 12,
        "paddingLeft": 24,
        "paddingRight": 24,
        "cornerRadius": 8,
        "fills": [{ "type": "SOLID", "color": { "r": 0.2, "g": 0.4, "b": 1 } }],
        "children": [{
          "name": "Label",
          "type": "TEXT",
          "characters": "Learn More",
          "style": {
            "fontFamily": "Inter",
            "fontWeight": 500,
            "fontSize": 14,
            "lineHeightPercent": 100
          },
          "fills": [{ "type": "SOLID", "color": { "r": 1, "g": 1, "b": 1 } }]
        }]
      }
    ]
  }
}
```

## Conversion Analysis

### Token Dönüşümleri

| Figma Değeri | Hesaplama | Tailwind |
|--------------|-----------|----------|
| padding: 24px | 24/4 = 6 | `p-6` |
| gap: 16px | 16/4 = 4 | `gap-4` |
| gap: 8px | 8/4 = 2 | `gap-2` |
| cornerRadius: 12px | - | `rounded-xl` |
| cornerRadius: 8px | - | `rounded-lg` |
| fontSize: 18px | 18/16 = 1.125 | `text-lg` |
| fontSize: 14px | 14/16 = 0.875 | `text-sm` |
| fontWeight: 600 | - | `font-semibold` |
| fontWeight: 500 | - | `font-medium` |
| fontWeight: 400 | - | `font-normal` |
| lineHeight: 133% | 133/100 = 1.33 | `leading-[1.33]` |
| lineHeight: 157% | 157/100 = 1.57 | `leading-relaxed` (~1.625) |
| letterSpacing: -0.3 | -0.3/1000 = -0.0003 | `tracking-tight` |

### Renk Dönüşümleri

| Figma RGB | Hex | Tailwind |
|-----------|-----|----------|
| r:1, g:1, b:1 | #FFFFFF | `bg-white` |
| r:0.1, g:0.1, b:0.1 | #1A1A1A | `text-[#1A1A1A]` |
| r:0.4, g:0.4, b:0.4 | #666666 | `text-[#666666]` |
| r:0.2, g:0.4, b:1 | #3366FF | `bg-[#3366FF]` |

### Semantik Mapping

| Figma Element | HTML Element | Reasoning |
|---------------|--------------|-----------|
| Card (Frame) | `<article>` | Bağımsız içerik birimi |
| Image | `<img>` | Görsel içerik |
| Title | `<h3>` | Başlık |
| Description | `<p>` | Paragraf |
| Button | `<button>` | Tıklanabilir aksiyon |

## Generated Code (Output)

```tsx
import React from 'react';

interface CardProps {
  imageUrl: string;
  imageAlt: string;
  title: string;
  description: string;
  onLearnMore?: () => void;
}

export const Card: React.FC<CardProps> = ({
  imageUrl,
  imageAlt,
  title,
  description,
  onLearnMore,
}) => {
  return (
    <article className="w-[320px] p-6 flex flex-col gap-4 rounded-xl bg-white shadow-md">
      {/* Image */}
      <img
        src={imageUrl}
        alt={imageAlt}
        className="w-full h-[180px] object-cover rounded-lg"
      />
      
      {/* Content */}
      <div className="flex flex-col gap-2">
        <h3 className="font-inter text-lg font-semibold leading-[1.33] tracking-tight text-[#1A1A1A]">
          {title}
        </h3>
        <p className="font-inter text-sm font-normal leading-relaxed text-[#666666]">
          {description}
        </p>
      </div>
      
      {/* Button */}
      <button
        onClick={onLearnMore}
        className="w-full py-3 px-6 flex items-center justify-center rounded-lg bg-[#3366FF] hover:bg-[#2952CC] transition-colors"
      >
        <span className="font-inter text-sm font-medium text-white">
          Learn More
        </span>
      </button>
    </article>
  );
};

export default Card;
```

## Responsive Version

```tsx
export const ResponsiveCard: React.FC<CardProps> = ({
  imageUrl,
  imageAlt,
  title,
  description,
  onLearnMore,
}) => {
  return (
    <article className="w-full max-w-[320px] sm:w-[320px] p-4 sm:p-6 flex flex-col gap-3 sm:gap-4 rounded-xl bg-white shadow-md">
      {/* Image - daha kısa mobile'da */}
      <img
        src={imageUrl}
        alt={imageAlt}
        className="w-full h-[140px] sm:h-[180px] object-cover rounded-lg"
      />
      
      {/* Content */}
      <div className="flex flex-col gap-1.5 sm:gap-2">
        <h3 className="font-inter text-base sm:text-lg font-semibold leading-[1.33] tracking-tight text-[#1A1A1A]">
          {title}
        </h3>
        <p className="font-inter text-xs sm:text-sm font-normal leading-relaxed text-[#666666]">
          {description}
        </p>
      </div>
      
      {/* Button */}
      <button
        onClick={onLearnMore}
        className="w-full py-2.5 sm:py-3 px-4 sm:px-6 flex items-center justify-center rounded-lg bg-[#3366FF] hover:bg-[#2952CC] focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transition-colors"
        aria-label={`Learn more about ${title}`}
      >
        <span className="font-inter text-sm font-medium text-white">
          Learn More
        </span>
      </button>
    </article>
  );
};
```

## Validation Checklist

- [x] Semantik HTML kullanıldı (`<article>`, `<h3>`, `<p>`, `<button>`)
- [x] Tüm spacing değerleri Tailwind scale'den
- [x] Tipografi formülleri doğru uygulandı
- [x] Renkler hex olarak aktarıldı
- [x] Hover state eklendi
- [x] Focus state eklendi (erişilebilirlik)
- [x] aria-label eklendi
- [x] Props typed (TypeScript)
- [x] Responsive varyant oluşturuldu
