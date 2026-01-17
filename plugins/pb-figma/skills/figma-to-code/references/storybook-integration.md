# Storybook Entegrasyonu

Figma-to-code dönüşümünde Storybook, hem dokümantasyon hem de görsel test için kritik öneme sahiptir.

## Story Template

Her yeni bileşen için otomatik oluşturulacak Storybook story şablonu:

**Dosya:** `assets/templates/component.stories.tsx.hbs`

```tsx
import type { Meta, StoryObj } from '@storybook/react';
import { {{componentName}} } from './{{componentName}}';

const meta: Meta<typeof {{componentName}}> = {
  title: 'Components/{{componentName}}',
  component: {{componentName}},
  tags: ['autodocs'],
  parameters: {
    layout: 'centered',
    design: {
      type: 'figma',
      url: '{{figmaUrl}}',
    },
    // Chromatic için snapshot ayarları
    chromatic: {
      viewports: [320, 768, 1024],
      delay: 300,
    },
  },
  argTypes: {
    {{#each props}}
    {{name}}: {
      control: '{{control}}',
      description: '{{description}}',
      {{#if options}}
      options: [{{#each options}}'{{this}}'{{#unless @last}}, {{/unless}}{{/each}}],
      {{/if}}
    },
    {{/each}}
  },
};

export default meta;
type Story = StoryObj<typeof {{componentName}}>;

/**
 * Default state - Figma tasarımındaki varsayılan görünüm
 */
export const Default: Story = {
  args: {
    {{#each defaultProps}}
    {{name}}: {{value}},
    {{/each}}
  },
};

/**
 * Tüm varyantları gösteren showcase
 */
export const AllVariants: Story = {
  render: () => (
    <div className="flex flex-col gap-4">
      {{#each variants}}
      <{{../componentName}} {{props}} />
      {{/each}}
    </div>
  ),
};

/**
 * Responsive davranış testi
 */
export const Responsive: Story = {
  parameters: {
    viewport: {
      defaultViewport: 'mobile1',
    },
  },
  args: {
    ...Default.args,
  },
};

/**
 * Dark mode varyantı
 */
export const DarkMode: Story = {
  parameters: {
    backgrounds: { default: 'dark' },
  },
  decorators: [
    (Story) => (
      <div className="dark">
        <Story />
      </div>
    ),
  ],
  args: {
    ...Default.args,
  },
};

/**
 * Accessibility testi için focus state
 */
export const FocusState: Story = {
  play: async ({ canvasElement }) => {
    const button = canvasElement.querySelector('button');
    button?.focus();
  },
  args: {
    ...Default.args,
  },
};
```

## Storybook Konfigürasyonu

### Figma Addon Kurulumu

```bash
npm install @storybook/addon-designs --save-dev
```

**Dosya:** `.storybook/main.ts`

```typescript
import type { StorybookConfig } from '@storybook/nextjs';

const config: StorybookConfig = {
  stories: ['../src/**/*.mdx', '../src/**/*.stories.@(js|jsx|mjs|ts|tsx)'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',           // Erişilebilirlik
    '@storybook/addon-designs',        // Figma embed
    '@chromatic-com/storybook',        // Chromatic CI
  ],
  framework: {
    name: '@storybook/nextjs',
    options: {},
  },
  docs: {
    autodocs: 'tag',
  },
  staticDirs: ['../public'],
};

export default config;
```

### Preview Konfigürasyonu

**Dosya:** `.storybook/preview.ts`

```typescript
import type { Preview } from '@storybook/react';
import '../src/styles/globals.css'; // Tailwind CSS

const preview: Preview = {
  parameters: {
    actions: { argTypesRegex: '^on[A-Z].*' },
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/i,
      },
    },
    backgrounds: {
      default: 'light',
      values: [
        { name: 'light', value: '#ffffff' },
        { name: 'dark', value: '#1a1a1a' },
        { name: 'gray', value: '#f5f5f5' },
      ],
    },
    viewport: {
      viewports: {
        mobile: { name: 'Mobile', styles: { width: '320px', height: '568px' } },
        tablet: { name: 'Tablet', styles: { width: '768px', height: '1024px' } },
        desktop: { name: 'Desktop', styles: { width: '1280px', height: '800px' } },
      },
    },
  },
  // Global decorators
  decorators: [
    (Story) => (
      <div className="font-sans antialiased">
        <Story />
      </div>
    ),
  ],
};

export default preview;
```

## Chromatic Entegrasyonu

### Kurulum

```bash
npm install chromatic --save-dev
npx chromatic --project-token=<your-token>
```

### Package.json Scripts

```json
{
  "scripts": {
    "storybook": "storybook dev -p 6006",
    "build-storybook": "storybook build",
    "chromatic": "chromatic --exit-zero-on-changes",
    "chromatic:ci": "chromatic --auto-accept-changes main"
  }
}
```

### Chromatic Konfigürasyonu

**Dosya:** `chromatic.config.json`

```json
{
  "projectId": "your-project-id",
  "autoAcceptChanges": "main",
  "exitZeroOnChanges": true,
  "onlyChanged": true,
  "externals": ["public/**"],
  "skip": "dependabot/**"
}
```

## Visual Test ile Storybook Entegrasyonu

### Test Runner Kurulumu

```bash
npm install @storybook/test-runner --save-dev
```

**Dosya:** `package.json`

```json
{
  "scripts": {
    "test-storybook": "test-storybook",
    "test-storybook:ci": "test-storybook --coverage"
  }
}
```

### Playwright Konfigürasyonu

**Dosya:** `test-runner-jest.config.js`

```javascript
const { getJestConfig } = require('@storybook/test-runner');

module.exports = {
  ...getJestConfig(),
  testEnvironmentOptions: {
    'jest-playwright': {
      browsers: ['chromium'],
      launchOptions: {
        headless: true,
      },
    },
  },
};
```

## Figma Embed Örneği

Story'de Figma tasarımını göstermek:

```tsx
export const WithFigmaDesign: Story = {
  parameters: {
    design: {
      type: 'figma',
      url: 'https://www.figma.com/file/abc123/MyDesign?node-id=1-2',
      // Alternatif: embed olarak göster
      // type: 'iframe',
      // url: 'https://www.figma.com/embed?embed_host=storybook&url=...',
    },
  },
};
```

## Otomatik Story Üretimi

Figma-to-code dönüşümünde story otomatik oluşturulmalı:

```typescript
// scripts/generate-story.ts
import Handlebars from 'handlebars';
import fs from 'fs';
import path from 'path';

interface StoryConfig {
  componentName: string;
  figmaUrl: string;
  props: Array<{
    name: string;
    control: string;
    description: string;
    options?: string[];
  }>;
  defaultProps: Array<{
    name: string;
    value: string;
  }>;
  variants: Array<{
    props: string;
  }>;
}

export function generateStory(config: StoryConfig): string {
  const templatePath = path.join(__dirname, '../templates/component.stories.tsx.hbs');
  const template = fs.readFileSync(templatePath, 'utf-8');
  const compiled = Handlebars.compile(template);
  
  return compiled(config);
}

// Kullanım
const storyContent = generateStory({
  componentName: 'HeroCard',
  figmaUrl: 'https://figma.com/file/...',
  props: [
    { name: 'title', control: 'text', description: 'Card title' },
    { name: 'variant', control: 'select', description: 'Visual variant', options: ['default', 'featured'] },
  ],
  defaultProps: [
    { name: 'title', value: '"Welcome"' },
    { name: 'variant', value: '"default"' },
  ],
  variants: [
    { props: 'variant="default" title="Default"' },
    { props: 'variant="featured" title="Featured"' },
  ],
});

fs.writeFileSync('src/components/HeroCard/HeroCard.stories.tsx', storyContent);
```

## Best Practices

1. **Her bileşen için story zorunlu** — PR review'da kontrol et
2. **Figma URL'i meta'ya ekle** — Tasarım-kod bağlantısı
3. **Tüm varyantları dokümante et** — AllVariants story'si
4. **Responsive test** — viewport parametresi
5. **Erişilebilirlik** — a11y addon aktif
6. **Dark mode** — Her bileşen için test
7. **Chromatic baseline** — Visual regression CI'da
