# Test Stratejisi ve Template'ler

> **Used by:** code-generator-*

Figma-to-code dönüşümünde üretilen bileşenlerin test coverage'ı enterprise kalite için kritiktir.

## Test Piramidi

```
         ╱╲
        ╱  ╲         E2E / Visual (Playwright)
       ╱────╲        - Screenshot comparison
      ╱      ╲       - User flow tests
     ╱────────╲      
    ╱          ╲     Integration (Testing Library)
   ╱────────────╲    - Component interactions
  ╱              ╲   - Event handling
 ╱────────────────╲  
╱                  ╲ Unit (Vitest/Jest)
╱────────────────────╲ - Props validation
                       - Utility functions
```

## Unit Test Template

**Dosya:** `templates/component.test.tsx.hbs`

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { axe, toHaveNoViolations } from 'jest-axe';
import { {{componentName}} } from './{{componentName}}';

expect.extend(toHaveNoViolations);

describe('{{componentName}}', () => {
  // ==========================================
  // RENDERING TESTS
  // ==========================================
  
  describe('Rendering', () => {
    it('renders without crashing', () => {
      render(<{{componentName}} {{#each requiredProps}}{{name}}={{{value}}} {{/each}}/>);
      expect(screen.getByTestId('{{kebabCase componentName}}')).toBeInTheDocument();
    });

    it('renders with default props', () => {
      render(<{{componentName}} {{#each requiredProps}}{{name}}={{{value}}} {{/each}}/>);
      // Varsayılan değerlerin kontrolü
      {{#each defaultChecks}}
      expect(screen.getByText('{{text}}')).toBeInTheDocument();
      {{/each}}
    });

    it('renders children correctly', () => {
      render(
        <{{componentName}} {{#each requiredProps}}{{name}}={{{value}}} {{/each}}>
          <span data-testid="child">Child content</span>
        </{{componentName}}>
      );
      expect(screen.getByTestId('child')).toBeInTheDocument();
    });
  });

  // ==========================================
  // PROPS TESTS
  // ==========================================

  describe('Props', () => {
    {{#each props}}
    {{#if isVariant}}
    describe('{{name}} prop', () => {
      {{#each options}}
      it('renders {{this}} variant correctly', () => {
        render(<{{../../../componentName}} {{../name}}="{{this}}" {{#each ../../../requiredProps}}{{name}}={{{value}}} {{/each}}/>);
        expect(screen.getByTestId('{{kebabCase ../../../componentName}}')).toHaveClass('{{expectedClass}}');
      });
      {{/each}}
    });
    {{/if}}
    {{/each}}

    it('applies custom className', () => {
      render(
        <{{componentName}} 
          className="custom-class" 
          {{#each requiredProps}}{{name}}={{{value}}} {{/each}}
        />
      );
      expect(screen.getByTestId('{{kebabCase componentName}}')).toHaveClass('custom-class');
    });
  });

  // ==========================================
  // INTERACTION TESTS
  // ==========================================

  describe('Interactions', () => {
    {{#if hasClickHandler}}
    it('calls onClick when clicked', async () => {
      const handleClick = vi.fn();
      render(
        <{{componentName}} 
          onClick={handleClick} 
          {{#each requiredProps}}{{name}}={{{value}}} {{/each}}
        />
      );
      
      await userEvent.click(screen.getByTestId('{{kebabCase componentName}}'));
      expect(handleClick).toHaveBeenCalledTimes(1);
    });

    it('does not call onClick when disabled', async () => {
      const handleClick = vi.fn();
      render(
        <{{componentName}} 
          onClick={handleClick} 
          disabled 
          {{#each requiredProps}}{{name}}={{{value}}} {{/each}}
        />
      );
      
      await userEvent.click(screen.getByTestId('{{kebabCase componentName}}'));
      expect(handleClick).not.toHaveBeenCalled();
    });
    {{/if}}

    {{#if hasHoverState}}
    it('shows hover state on mouse enter', async () => {
      render(<{{componentName}} {{#each requiredProps}}{{name}}={{{value}}} {{/each}}/>);
      const element = screen.getByTestId('{{kebabCase componentName}}');
      
      await userEvent.hover(element);
      // Hover state kontrolü - Tailwind class veya style
      expect(element).toHaveClass('hover-state'); // veya computed style kontrolü
    });
    {{/if}}

    it('is keyboard accessible', async () => {
      render(<{{componentName}} {{#each requiredProps}}{{name}}={{{value}}} {{/each}}/>);
      const element = screen.getByTestId('{{kebabCase componentName}}');
      
      await userEvent.tab();
      expect(element).toHaveFocus();
    });
  });

  // ==========================================
  // ACCESSIBILITY TESTS
  // ==========================================

  describe('Accessibility', () => {
    it('has no accessibility violations', async () => {
      const { container } = render(
        <{{componentName}} {{#each requiredProps}}{{name}}={{{value}}} {{/each}}/>
      );
      const results = await axe(container);
      expect(results).toHaveNoViolations();
    });

    it('has correct ARIA attributes', () => {
      render(<{{componentName}} {{#each requiredProps}}{{name}}={{{value}}} {{/each}}/>);
      const element = screen.getByTestId('{{kebabCase componentName}}');
      
      {{#each ariaAttributes}}
      expect(element).toHaveAttribute('{{name}}', '{{value}}');
      {{/each}}
    });

    {{#if hasDisabledState}}
    it('is properly disabled for screen readers', () => {
      render(
        <{{componentName}} 
          disabled 
          {{#each requiredProps}}{{name}}={{{value}}} {{/each}}
        />
      );
      expect(screen.getByTestId('{{kebabCase componentName}}')).toHaveAttribute('aria-disabled', 'true');
    });
    {{/if}}
  });

  // ==========================================
  // RESPONSIVE TESTS
  // ==========================================

  describe('Responsive', () => {
    it('renders correctly at mobile viewport', () => {
      // viewport mock ile test
      Object.defineProperty(window, 'innerWidth', { value: 320 });
      window.dispatchEvent(new Event('resize'));
      
      render(<{{componentName}} {{#each requiredProps}}{{name}}={{{value}}} {{/each}}/>);
      // Mobile-specific assertions
    });
  });

  // ==========================================
  // SNAPSHOT TESTS
  // ==========================================

  describe('Snapshots', () => {
    it('matches snapshot', () => {
      const { container } = render(
        <{{componentName}} {{#each requiredProps}}{{name}}={{{value}}} {{/each}}/>
      );
      expect(container.firstChild).toMatchSnapshot();
    });

    {{#each variants}}
    it('matches snapshot for {{name}} variant', () => {
      const { container } = render(
        <{{../componentName}} {{props}} />
      );
      expect(container.firstChild).toMatchSnapshot();
    });
    {{/each}}
  });
});
```

## Visual Test Template (Playwright)

**Dosya:** `templates/component.visual.spec.ts.hbs`

```typescript
import { test, expect } from '@playwright/test';

test.describe('{{componentName}} Visual Tests', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/test-preview?component={{componentName}}');
    await page.waitForSelector('[data-testid="preview-component"]');
  });

  test('matches Figma design', async ({ page }) => {
    const component = page.locator('[data-testid="preview-component"]');
    
    await expect(component).toHaveScreenshot('{{kebabCase componentName}}-default.png', {
      maxDiffPixels: 100,
      threshold: 0.2,
    });
  });

  test('matches at mobile viewport', async ({ page }) => {
    await page.setViewportSize({ width: 320, height: 568 });
    const component = page.locator('[data-testid="preview-component"]');
    
    await expect(component).toHaveScreenshot('{{kebabCase componentName}}-mobile.png', {
      maxDiffPixels: 100,
    });
  });

  test('matches at tablet viewport', async ({ page }) => {
    await page.setViewportSize({ width: 768, height: 1024 });
    const component = page.locator('[data-testid="preview-component"]');
    
    await expect(component).toHaveScreenshot('{{kebabCase componentName}}-tablet.png', {
      maxDiffPixels: 100,
    });
  });

  {{#if hasHoverState}}
  test('hover state matches design', async ({ page }) => {
    const component = page.locator('[data-testid="preview-component"]');
    await component.hover();
    
    await expect(component).toHaveScreenshot('{{kebabCase componentName}}-hover.png', {
      maxDiffPixels: 50,
    });
  });
  {{/if}}

  {{#if hasFocusState}}
  test('focus state matches design', async ({ page }) => {
    const focusable = page.locator('[data-testid="preview-component"] button, [data-testid="preview-component"] a');
    await focusable.first().focus();
    
    await expect(page.locator('[data-testid="preview-component"]')).toHaveScreenshot('{{kebabCase componentName}}-focus.png', {
      maxDiffPixels: 50,
    });
  });
  {{/if}}

  {{#each variants}}
  test('{{name}} variant matches design', async ({ page }) => {
    await page.goto('/test-preview?component={{../componentName}}&variant={{name}}');
    const component = page.locator('[data-testid="preview-component"]');
    
    await expect(component).toHaveScreenshot('{{kebabCase ../componentName}}-{{kebabCase name}}.png', {
      maxDiffPixels: 100,
    });
  });
  {{/each}}
});
```

## Test Konfigürasyonu

### Vitest Setup

**Dosya:** `vitest.config.ts`

```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./tests/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.stories.tsx',
        '**/*.d.ts',
      ],
      thresholds: {
        statements: 80,
        branches: 80,
        functions: 80,
        lines: 80,
      },
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### Test Setup

**Dosya:** `tests/setup.ts`

```typescript
import '@testing-library/jest-dom';
import { vi } from 'vitest';

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});

// Mock ResizeObserver
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}));

// Mock IntersectionObserver
global.IntersectionObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn(),
}));
```

### Playwright Config

**Dosya:** `playwright.config.ts`

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/visual',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
  ],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
  expect: {
    toHaveScreenshot: {
      maxDiffPixels: 100,
      threshold: 0.2,
    },
  },
});
```

## Coverage Requirements

| Metrik | Minimum | Hedef |
|--------|---------|-------|
| Statements | 80% | 90% |
| Branches | 80% | 85% |
| Functions | 80% | 90% |
| Lines | 80% | 90% |

## Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:watch": "vitest watch",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui",
    "test:visual": "playwright test",
    "test:visual:update": "playwright test --update-snapshots",
    "test:a11y": "vitest run --grep accessibility",
    "test:all": "npm run test:coverage && npm run test:visual"
  }
}
```

## Pre-commit Hook

**Dosya:** `.husky/pre-commit`

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Sadece değişen component'ları test et
CHANGED_COMPONENTS=$(git diff --cached --name-only | grep -E '^src/components/.*\.tsx$' | sed 's|src/components/||;s|/.*||' | sort -u)

if [ -n "$CHANGED_COMPONENTS" ]; then
  for component in $CHANGED_COMPONENTS; do
    echo "Testing $component..."
    npm run test -- --run "src/components/$component"
  done
fi

# Lint
npm run lint-staged
```

## Best Practices

1. **Her bileşen için test dosyası zorunlu** — PR'da kontrol
2. **Accessibility test zorunlu** — jest-axe ile
3. **Visual snapshot** — Playwright ile
4. **Coverage threshold** — %80 minimum
5. **Pre-commit hook** — Değişen bileşenleri test et
6. **CI'da tüm testler** — PR merge için zorunlu
