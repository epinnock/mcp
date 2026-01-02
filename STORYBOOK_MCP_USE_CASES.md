# Storybook MCP Use Cases - Detailed Documentation

This guide provides comprehensive documentation on how to use Storybook MCP for six key use cases. Each section includes setup instructions, practical examples, and best practices.

---

## Table of Contents

1. [Component Development: AI Agents Build Components with Stories](#1-component-development)
2. [Documentation: Auto-generated Stories as Living Documentation](#2-documentation)
3. [Testing: Stories as Component Tests with Play Functions](#3-testing)
4. [Design Systems: Discovering and Using Existing Components](#4-design-systems)
5. [Research: Evaluating AI Agent Capabilities](#5-research)
6. [Benchmarking: Comparing Models and Agents](#6-benchmarking)

---

## 1. Component Development

### Overview

The Component Development use case enables AI agents to build UI components and automatically generate Storybook stories for them. This creates a workflow where every component comes with visual examples in different states, making verification and iteration faster.

### How It Works

When an AI agent (like Claude Code or GitHub Copilot) builds a component with Storybook MCP:

1. **Agent calls `get-ui-building-instructions`** → Receives project-specific guidelines
2. **Agent builds the component** → Creates the React/Vue/Angular/etc. component
3. **Agent creates stories automatically** → Generates CSF3 stories for different states
4. **Agent calls `get-story-urls`** → Gets direct links to visual previews
5. **Developer clicks links** → Instantly sees component in all its states

### Setup

#### Prerequisites

- Storybook 9.1.16+ or 10.0.0+
- Node.js 20+
- An MCP-compatible AI agent (Claude Code, GitHub Copilot, etc.)

#### Installation

```bash
# Install the addon
npx storybook add @storybook/addon-mcp

# Start Storybook
npm run storybook
```

#### Configure Your AI Agent

**For Claude Code:**

```bash
# Add MCP server
claude mcp add storybook-mcp --transport http http://localhost:6006/mcp --scope project
```

**For GitHub Copilot:**

Create or update `.github/copilot/agents.json`:

```json
{
  "storybook-mcp": {
    "type": "http",
    "url": "http://localhost:6006/mcp"
  }
}
```

#### Add Agent Instructions (Recommended)

Add this to your project or system prompt:

```
Before doing any UI, frontend or React development, ALWAYS call the storybook MCP server to get further instructions.
```

### Practical Example

**Prompt to Agent:**

```
Build a SearchBar component with:
- A text input with placeholder "Search..."
- A search icon button
- Loading state while searching
- Error state for failed searches
- Success state showing results count
```

**What the Agent Does:**

1. **Calls `get-ui-building-instructions`** → Gets instructions like:

```markdown
# Writing User Interfaces

ALWAYS write a Storybook story for any component written.

## Story Format

Use CSF3 format with Meta and StoryObj:

import type { Meta, StoryObj } from '@storybook/react-vite';
import { SearchBar } from './SearchBar';

const meta: Meta<typeof SearchBar> = {
  title: 'Components/SearchBar',
  component: SearchBar,
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof SearchBar>;
```

2. **Creates the component** (`src/components/SearchBar.tsx`):

```tsx
interface SearchBarProps {
  onSearch: (query: string) => void;
  isLoading?: boolean;
  error?: string;
  resultsCount?: number;
}

export const SearchBar = ({ onSearch, isLoading, error, resultsCount }: SearchBarProps) => {
  const [query, setQuery] = useState('');

  return (
    <div className="search-bar">
      <input
        type="text"
        placeholder="Search..."
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <button onClick={() => onSearch(query)} disabled={isLoading}>
        {isLoading ? 'Searching...' : '🔍'}
      </button>
      {error && <div className="error">{error}</div>}
      {resultsCount !== undefined && <div>Found {resultsCount} results</div>}
    </div>
  );
};
```

3. **Creates stories** (`src/components/SearchBar.stories.ts`):

```typescript
import type { Meta, StoryObj } from '@storybook/react-vite';
import { fn } from 'storybook/test';
import { SearchBar } from './SearchBar';

const meta: Meta<typeof SearchBar> = {
  title: 'Components/SearchBar',
  component: SearchBar,
  tags: ['autodocs'],
  args: {
    onSearch: fn(),
  },
};

export default meta;
type Story = StoryObj<typeof SearchBar>;

export const Default: Story = {
  args: {},
};

export const Loading: Story = {
  args: {
    isLoading: true,
  },
};

export const WithError: Story = {
  args: {
    error: 'Failed to fetch results',
  },
};

export const WithResults: Story = {
  args: {
    resultsCount: 42,
  },
};
```

4. **Calls `get-story-urls`** with:

```json
{
  "stories": [
    {
      "absoluteStoryPath": "/path/to/src/components/SearchBar.stories.ts",
      "exportName": "Default"
    },
    {
      "absoluteStoryPath": "/path/to/src/components/SearchBar.stories.ts",
      "exportName": "Loading"
    },
    {
      "absoluteStoryPath": "/path/to/src/components/SearchBar.stories.ts",
      "exportName": "WithError"
    },
    {
      "absoluteStoryPath": "/path/to/src/components/SearchBar.stories.ts",
      "exportName": "WithResults"
    }
  ]
}
```

5. **Returns links to you:**

```
I've created the SearchBar component with the following stories:

- Default: http://localhost:6006/?path=/story/components-searchbar--default
- Loading: http://localhost:6006/?path=/story/components-searchbar--loading
- With Error: http://localhost:6006/?path=/story/components-searchbar--with-error
- With Results: http://localhost:6006/?path=/story/components-searchbar--with-results
```

### Best Practices

1. **Always Start with Instructions**: Ensure the agent calls `get-ui-building-instructions` first
2. **Request Multiple States**: Ask for different component states (loading, error, empty, filled)
3. **Use Descriptive Story Names**: Make story exports self-documenting (e.g., `LoadingWithLongText`)
4. **Click the Links**: Always verify the visual output by clicking the story URLs
5. **Iterate Visually**: Ask the agent to modify components based on what you see in Storybook

### Common Patterns

#### Building a Form Component

```
Build a LoginForm component with:
- Email and password inputs
- Remember me checkbox
- Submit button
- States: empty, filled, validating, error, success
```

#### Building a Data Display Component

```
Build a UserCard component showing:
- Avatar, name, role
- States: loading skeleton, populated, error, no data
```

#### Building an Interactive Widget

```
Build a DatePicker component with:
- Calendar popup
- Date range selection
- States: closed, open, selecting start date, selecting end date, complete
```

---

## 2. Documentation

### Overview

Auto-generated stories serve as living documentation for your components. Every story becomes a visual example that stays in sync with your code, eliminating the need for separate documentation that gets stale.

### How It Works

Stories automatically provide documentation through:

1. **Visual Examples**: Each story shows the component in a specific state
2. **Args Tables**: Storybook generates prop/attribute documentation
3. **Auto-generated Docs**: With the `autodocs` tag, Storybook creates a docs page
4. **JSDoc Comments**: Component descriptions and prop documentation appear in Storybook

### Setup

#### Enable Autodocs

In `.storybook/preview.ts`:

```typescript
export default {
  tags: ['autodocs'], // Generate docs for all stories
};
```

Or per story file:

```typescript
const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'], // Generate docs for this component
};
```

#### Add Component Documentation

In your component file:

```tsx
interface ButtonProps {
  /**
   * Is this the principal call to action on the page?
   */
  primary?: boolean;
  /**
   * How large should the button be?
   */
  size?: 'small' | 'medium' | 'large';
  /**
   * Button contents
   */
  label: string;
  /**
   * Optional click handler
   */
  onClick?: () => void;
}

/**
 * Primary UI component for user interaction
 *
 * @import import { Button } from '@my-org/my-component-library';
 * @summary A customizable button component for user interactions.
 */
export const Button = ({ primary, size, label, onClick }: ButtonProps) => {
  // ... implementation
};
```

### Practical Example

**Your Component with Documentation:**

```tsx
// src/components/Alert.tsx

interface AlertProps {
  /**
   * The severity level of the alert
   * @default 'info'
   */
  severity?: 'success' | 'info' | 'warning' | 'error';

  /**
   * The title of the alert message
   */
  title: string;

  /**
   * The detailed message content
   */
  message?: string;

  /**
   * Whether the alert can be dismissed
   * @default false
   */
  dismissible?: boolean;

  /**
   * Callback when alert is dismissed
   */
  onDismiss?: () => void;
}

/**
 * Alert component for displaying important messages to users
 *
 * Use alerts to communicate status, warnings, or errors to users in a prominent way.
 *
 * @import import { Alert } from '@my-org/ui-components';
 * @example
 * <Alert
 *   severity="success"
 *   title="Success!"
 *   message="Your changes have been saved."
 *   dismissible
 * />
 */
export const Alert = ({
  severity = 'info',
  title,
  message,
  dismissible = false,
  onDismiss
}: AlertProps) => {
  // ... implementation
};
```

**Stories as Documentation:**

```typescript
// src/components/Alert.stories.ts

import type { Meta, StoryObj } from '@storybook/react-vite';
import { fn } from 'storybook/test';
import { Alert } from './Alert';

const meta: Meta<typeof Alert> = {
  title: 'Components/Alert',
  component: Alert,
  tags: ['autodocs'],
  argTypes: {
    severity: {
      control: 'select',
      options: ['success', 'info', 'warning', 'error'],
      description: 'The visual style of the alert',
    },
  },
  args: {
    onDismiss: fn(),
  },
};

export default meta;
type Story = StoryObj<typeof Alert>;

// Each story is a documentation example
export const Info: Story = {
  args: {
    title: 'Information',
    message: 'This is an informational message.',
    severity: 'info',
  },
};

export const Success: Story = {
  args: {
    title: 'Success',
    message: 'Your operation completed successfully!',
    severity: 'success',
  },
};

export const Warning: Story = {
  args: {
    title: 'Warning',
    message: 'Please review this before continuing.',
    severity: 'warning',
  },
};

export const Error: Story = {
  args: {
    title: 'Error',
    message: 'Something went wrong. Please try again.',
    severity: 'error',
  },
};

export const Dismissible: Story = {
  args: {
    title: 'Dismissible Alert',
    message: 'You can close this alert.',
    dismissible: true,
  },
};

export const WithoutMessage: Story = {
  args: {
    title: 'Title Only',
  },
};

export const LongContent: Story = {
  args: {
    title: 'Long Message',
    message: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam.',
    severity: 'warning',
  },
};
```

**What This Provides:**

1. **Visual Documentation**: 7 examples showing all use cases
2. **Props Table**: Auto-generated from TypeScript/JSDoc
3. **Usage Examples**: Copy-paste ready code for developers
4. **Interactive Playground**: Developers can modify props in real-time
5. **Always Up-to-Date**: Documentation updates when code changes

### Documentation Workflow with AI

**Prompt:**

```
Document the Alert component by creating comprehensive stories that show all its variations.
```

**Agent Action:**

1. Reads the component and its props
2. Creates stories for each meaningful combination
3. Adds descriptive names and args
4. Calls `get-story-urls` to provide preview links

**Result:**

You get instant, visual, interactive documentation without writing a single markdown file.

### Best Practices

1. **Write JSDoc Comments**: Document props with `/** */` comments
2. **Use the `@import` Tag**: Show developers how to import the component
3. **Create Comprehensive Stories**: Cover all meaningful prop combinations
4. **Name Stories Descriptively**: `LoadingWithError` is better than `Story3`
5. **Add Controls**: Use `argTypes` to make docs interactive
6. **Group Related Components**: Use title like `Components/Forms/Input`

### Advanced Documentation Patterns

#### Documenting Variants

```typescript
export const PrimarySizes: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '1rem', alignItems: 'center' }}>
      <Button primary size="small" label="Small" />
      <Button primary size="medium" label="Medium" />
      <Button primary size="large" label="Large" />
    </div>
  ),
};
```

#### Documenting States

```typescript
export const AllStates: Story = {
  render: () => (
    <div style={{ display: 'flex', flexDirection: 'column', gap: '1rem' }}>
      <Alert severity="success" title="Success State" />
      <Alert severity="info" title="Info State" />
      <Alert severity="warning" title="Warning State" />
      <Alert severity="error" title="Error State" />
    </div>
  ),
};
```

#### Documenting Composition

```typescript
export const InContext: Story = {
  render: () => (
    <div className="page">
      <Header />
      <Alert severity="warning" title="Session Expiring" message="Please save your work." />
      <MainContent />
    </div>
  ),
};
```

---

## 3. Testing

### Overview

Stories double as component tests when you add **play functions**. Play functions simulate user interactions and make assertions, turning every story into an automated test that validates component behavior.

### How It Works

1. **Stories define initial state** → Via args
2. **Play functions simulate interactions** → Click, type, hover, etc.
3. **Assertions verify behavior** → Using Vitest/Jest assertions
4. **Tests run automatically** → During builds and in CI

### Setup

#### Install Test Dependencies

```bash
npm install --save-dev @storybook/test
```

#### Configure Test Runner

In `package.json`:

```json
{
  "scripts": {
    "test-storybook": "test-storybook"
  }
}
```

### Practical Example

#### Basic Interaction Test

```typescript
// src/components/Counter.stories.ts

import type { Meta, StoryObj } from '@storybook/react-vite';
import { expect, userEvent, within } from 'storybook/test';
import { Counter } from './Counter';

const meta: Meta<typeof Counter> = {
  title: 'Components/Counter',
  component: Counter,
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof Counter>;

export const Default: Story = {
  args: {
    initialCount: 0,
  },
};

export const ClickToIncrement: Story = {
  args: {
    initialCount: 0,
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    // Find elements
    const incrementButton = canvas.getByRole('button', { name: /increment/i });
    const counter = canvas.getByTestId('counter-value');

    // Verify initial state
    await expect(counter).toHaveTextContent('0');

    // Simulate interaction
    await userEvent.click(incrementButton);

    // Verify result
    await expect(counter).toHaveTextContent('1');

    // Multiple clicks
    await userEvent.click(incrementButton);
    await userEvent.click(incrementButton);
    await expect(counter).toHaveTextContent('3');
  },
};

export const DecrementButton: Story = {
  args: {
    initialCount: 5,
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const decrementButton = canvas.getByRole('button', { name: /decrement/i });
    const counter = canvas.getByTestId('counter-value');

    await expect(counter).toHaveTextContent('5');
    await userEvent.click(decrementButton);
    await expect(counter).toHaveTextContent('4');
  },
};
```

#### Form Validation Test

```typescript
// src/components/LoginForm.stories.ts

import type { Meta, StoryObj } from '@storybook/react-vite';
import { expect, userEvent, within } from 'storybook/test';
import { fn } from 'storybook/test';
import { LoginForm } from './LoginForm';

const meta: Meta<typeof LoginForm> = {
  title: 'Components/LoginForm',
  component: LoginForm,
  tags: ['autodocs'],
  args: {
    onSubmit: fn(),
  },
};

export default meta;
type Story = StoryObj<typeof LoginForm>;

export const ValidSubmission: Story = {
  play: async ({ canvasElement, args }) => {
    const canvas = within(canvasElement);

    // Fill out form
    await userEvent.type(
      canvas.getByLabelText(/email/i),
      'user@example.com'
    );
    await userEvent.type(
      canvas.getByLabelText(/password/i),
      'password123'
    );

    // Submit
    await userEvent.click(canvas.getByRole('button', { name: /submit/i }));

    // Verify callback was called with correct data
    await expect(args.onSubmit).toHaveBeenCalledWith({
      email: 'user@example.com',
      password: 'password123',
    });
  },
};

export const EmailValidation: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const emailInput = canvas.getByLabelText(/email/i);
    await userEvent.type(emailInput, 'invalid-email');
    await userEvent.tab(); // Trigger blur

    // Check for error message
    await expect(canvas.getByText(/invalid email/i)).toBeInTheDocument();
  },
};

export const EmptySubmission: Story = {
  play: async ({ canvasElement, args }) => {
    const canvas = within(canvasElement);

    // Try to submit empty form
    await userEvent.click(canvas.getByRole('button', { name: /submit/i }));

    // Verify errors are shown
    await expect(canvas.getByText(/email is required/i)).toBeInTheDocument();
    await expect(canvas.getByText(/password is required/i)).toBeInTheDocument();

    // Verify callback was NOT called
    await expect(args.onSubmit).not.toHaveBeenCalled();
  },
};
```

#### Async Behavior Test

```typescript
// src/components/SearchBar.stories.ts

import type { Meta, StoryObj } from '@storybook/react-vite';
import { expect, userEvent, within, waitFor } from 'storybook/test';
import { fn } from 'storybook/test';
import { SearchBar } from './SearchBar';

const meta: Meta<typeof SearchBar> = {
  title: 'Components/SearchBar',
  component: SearchBar,
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof SearchBar>;

export const SearchFlow: Story = {
  args: {
    onSearch: fn(async (query: string) => {
      // Simulate API delay
      await new Promise(resolve => setTimeout(resolve, 500));
      return { results: 42 };
    }),
  },
  play: async ({ canvasElement, args }) => {
    const canvas = within(canvasElement);

    const input = canvas.getByPlaceholderText(/search/i);
    const button = canvas.getByRole('button', { name: /search/i });

    // Type search query
    await userEvent.type(input, 'react components');

    // Verify input value
    await expect(input).toHaveValue('react components');

    // Click search
    await userEvent.click(button);

    // Verify loading state
    await expect(canvas.getByText(/searching/i)).toBeInTheDocument();

    // Wait for results
    await waitFor(() => {
      expect(canvas.getByText(/found 42 results/i)).toBeInTheDocument();
    });

    // Verify search was called
    await expect(args.onSearch).toHaveBeenCalledWith('react components');
  },
};
```

### Running Tests

```bash
# Run tests interactively
npm run test-storybook

# Run tests in CI
npm run test-storybook -- --ci

# Run with coverage
npm run test-storybook -- --coverage

# Watch mode
npm run test-storybook -- --watch
```

### Testing Workflow with AI

**Prompt:**

```
Add tests to the LoginForm component that verify:
- Email validation shows error for invalid emails
- Password must be at least 8 characters
- Form submits with valid data
- Loading state shows while submitting
```

**Agent Action:**

1. Calls `get-ui-building-instructions` (includes testing patterns)
2. Creates or updates stories with play functions
3. Adds assertions for each requirement
4. Calls `get-story-urls` to provide test preview links

**You Can:**

- Click links to see tests run live in Storybook
- Watch interactions happen in real-time
- Debug failing tests visually

### Best Practices

1. **One Test Per Story**: Each story should test one specific scenario
2. **Use Descriptive Names**: `EmailValidationError` not `Test1`
3. **Mock External Dependencies**: Use `fn()` for callbacks, mock API calls
4. **Test User Behavior**: Simulate real user interactions
5. **Verify Visual Feedback**: Check loading states, error messages, success states
6. **Use Accessibility Queries**: `getByRole`, `getByLabelText` over `getByTestId`
7. **Wait for Async**: Use `waitFor` for async updates

### Common Testing Patterns

#### Testing Accessibility

```typescript
export const KeyboardNavigation: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const firstButton = canvas.getByRole('button', { name: /option 1/i });
    firstButton.focus();

    await expect(firstButton).toHaveFocus();

    await userEvent.keyboard('{Tab}');
    await expect(canvas.getByRole('button', { name: /option 2/i })).toHaveFocus();
  },
};
```

#### Testing Error Boundaries

```typescript
export const ErrorHandling: Story = {
  args: {
    data: null, // Will cause error
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    await expect(canvas.getByText(/something went wrong/i)).toBeInTheDocument();
  },
};
```

#### Testing Responsive Behavior

```typescript
export const MobileView: Story = {
  parameters: {
    viewport: {
      defaultViewport: 'mobile1',
    },
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    // Mobile menu should be hidden initially
    const mobileMenu = canvas.queryByRole('navigation');
    await expect(mobileMenu).not.toBeVisible();

    // Click hamburger menu
    await userEvent.click(canvas.getByLabelText(/menu/i));

    // Menu should appear
    await expect(mobileMenu).toBeVisible();
  },
};
```

---

## 4. Design Systems

### Overview

The Design Systems use case enables AI agents to discover and use existing components from your component library through the **component manifest**. This prevents reinventing the wheel and ensures consistency across your application.

### How It Works

When you enable the experimental docs toolset:

1. **Storybook generates a component manifest** → Metadata about all components
2. **AI agent calls `list-all-components`** → Gets a list of available components
3. **AI agent calls `get-component-documentation`** → Gets detailed docs for specific components
4. **AI agent uses existing components** → Instead of building from scratch

### Setup

#### Requirements

- **Storybook 10.1.0+** (currently in prerelease: `storybook@next`)
- **React-based framework** (react-vite, nextjs, react-webpack5)
- **Experimental feature flag enabled**

#### Installation

```bash
# Upgrade to Storybook 10.1.0+
npm install storybook@next

# Addon should already be installed
npx storybook add @storybook/addon-mcp
```

#### Configuration

In `.storybook/main.ts`:

```typescript
export default {
  addons: [
    {
      name: '@storybook/addon-mcp',
      options: {
        toolsets: {
          dev: true,
          docs: true, // Enable docs tools
        },
      },
    },
  ],
  features: {
    experimentalComponentsManifest: true, // REQUIRED for docs tools
  },
};
```

### Practical Example

#### Scenario: Building a Dashboard

**Without Storybook MCP (Agent builds everything):**

```
Build a dashboard with:
- Card component for metrics
- Button component for actions
- Alert component for notifications
```

**Result:** Agent builds all three components from scratch, possibly with inconsistent styling.

**With Storybook MCP (Agent discovers existing components):**

**Agent's Workflow:**

1. **Lists available components:**

```typescript
// Agent calls list-all-components
```

**Response:**

```markdown
## Available Components

- **Alert** - Alert component for displaying important messages to users
- **Button** - Primary UI component for user interaction
- **Card** - A container component for grouping related content
- **Header** - Page header with navigation
- **Input** - Text input component with validation
```

2. **Gets documentation for specific components:**

```typescript
// Agent calls get-component-documentation
{
  componentId: "Card"
}
```

**Response:**

```markdown
# Card

A container component for grouping related content

## Import

import { Card } from '@my-org/ui-components';

## Props

| Name | Type | Default | Description |
|------|------|---------|-------------|
| title | string | - | The card title |
| children | ReactNode | - | The card content |
| variant | 'outlined' \| 'elevated' \| 'filled' | 'outlined' | Visual style |
| padding | 'none' \| 'small' \| 'medium' \| 'large' | 'medium' | Internal padding |
| onClick | () => void | - | Click handler for interactive cards |

## Usage Example

<Card title="User Stats" variant="elevated" padding="large">
  <p>Total Users: 1,234</p>
  <p>Active Today: 567</p>
</Card>
```

3. **Uses existing components:**

```tsx
// Agent creates Dashboard.tsx using existing components

import { Card } from '@my-org/ui-components';
import { Button } from '@my-org/ui-components';
import { Alert } from '@my-org/ui-components';

export const Dashboard = () => {
  return (
    <div className="dashboard">
      <Alert
        severity="info"
        title="Welcome!"
        message="Here's your daily overview."
      />

      <div className="metrics">
        <Card title="Total Users" variant="elevated">
          <div className="stat">1,234</div>
          <Button label="View Details" size="small" />
        </Card>

        <Card title="Revenue" variant="elevated">
          <div className="stat">$45,678</div>
          <Button label="View Details" size="small" />
        </Card>

        <Card title="Active Sessions" variant="elevated">
          <div className="stat">567</div>
          <Button label="View Details" size="small" />
        </Card>
      </div>
    </div>
  );
};
```

**Result:** Consistent styling, no duplicate components, faster development.

### Design System Workflow with AI

**Prompt:**

```
Build a user profile page with:
- User information card
- Edit button
- Settings section
```

**Agent Action:**

1. Calls `list-all-components` → Discovers Card, Button, and other components
2. Calls `get-component-documentation` for Card and Button → Gets usage details
3. Builds ProfilePage using existing components
4. Only creates new components if something doesn't exist (e.g., SettingsSection)
5. Creates stories for the new ProfilePage

**Result:** Faster development, consistent design, less code to maintain.

### Best Practices

1. **Document All Components**: Add JSDoc comments to all exported components
2. **Use Semantic Component Names**: `Card`, `Button`, `Alert` not `Component1`, `Box`
3. **Provide Import Examples**: Use the `@import` tag in JSDoc
4. **Organize by Category**: Use title prefixes like `Components/Forms/Input`
5. **Keep Manifest Updated**: Regenerate manifest when components change
6. **Prompt Agent to Check First**: "Before building, check if a similar component exists"

### Advanced Patterns

#### Building with a Design System

**Prompt:**

```
Before building anything, check what components are available in our design system.
Then build a checkout form using existing components where possible.
```

**Agent Workflow:**

1. Lists all components
2. Identifies reusable pieces (Input, Button, Card, Alert)
3. Builds CheckoutForm composing existing components
4. Only creates new domain-specific components (e.g., PaymentMethodSelector)

#### Extending Existing Components

**Prompt:**

```
I need a IconButton component. Check if we have a Button component that we can extend.
```

**Agent Workflow:**

1. Gets Button component documentation
2. Creates IconButton as a wrapper or variant of Button
3. Maintains consistency with existing Button API

#### Discovering Component Capabilities

**Prompt:**

```
What alert/notification components do we have?
```

**Agent Response:**

```
I found the following alert/notification components:

1. **Alert** - For displaying important messages
   - Severities: success, info, warning, error
   - Supports dismissible mode
   - Optional title and message

2. **Toast** - For temporary notifications
   - Auto-dismisses after timeout
   - Positions: top-right, top-left, bottom-right, bottom-left

Which one would you like to use?
```

### Component Manifest Structure

The manifest is automatically generated from your components and includes:

```json
{
  "components": {
    "Button": {
      "id": "Button",
      "name": "Button",
      "description": "Primary UI component for user interaction",
      "importStatement": "import { Button } from '@my-org/ui-components';",
      "props": [
        {
          "name": "primary",
          "type": "boolean",
          "description": "Is this the principal call to action?",
          "required": false,
          "defaultValue": "false"
        }
      ]
    }
  }
}
```

### Configuring Component Discovery

#### Limit to Specific Components

Create `components.json` to provide a curated manifest:

```json
{
  "components": {
    "Button": { /* ... */ },
    "Card": { /* ... */ },
    "Input": { /* ... */ }
  }
}
```

Then configure the agent to use it:

```json
{
  "storybook-mcp": {
    "url": "http://localhost:6006/mcp",
    "type": "http"
  }
}
```

---

## 5. Research

### Overview

The Research use case leverages the built-in **evaluation framework** to systematically measure AI coding agents' ability to build UI components. This is valuable for:

- Understanding agent capabilities and limitations
- Improving prompts and tooling
- Tracking progress over time
- Academic research on AI-assisted development

### How It Works

The evaluation framework:

1. **Prepares a fresh project** → Vite + React + Storybook
2. **Executes the agent** → With a specific prompt and context
3. **Evaluates the results** → Build success, tests, lint, types, a11y, coverage
4. **Generates metrics** → Cost, duration, turns, success rates

### Setup

#### Prerequisites

- Node.js 24+
- pnpm 10.19.0+
- Playwright: `npx playwright install`
- Claude Code CLI: `npm install -g @anthropic-ai/claude-code` (for Claude agents)
- GitHub Copilot CLI: `gh extension install github/gh-copilot` (for Copilot agents)

#### Navigate to Eval Directory

```bash
cd eval
```

### Running an Evaluation

#### Interactive Mode (Recommended)

```bash
node eval.ts
```

The CLI will prompt you for:

- Which eval to run (e.g., `100-flight-booking-plain`)
- Which agent (claude-code or copilot-cli)
- Which model (claude-sonnet-4.5, claude-opus-4.5, etc.)
- Which context (none, storybook-dev, components.json, etc.)
- Whether to upload results

#### Command-Line Mode

```bash
# Run specific evaluation
node eval.ts \
  --agent claude-code \
  --model claude-sonnet-4.5 \
  --context storybook-dev \
  --upload-id batch-1 \
  100-flight-booking-plain

# Run without context (baseline)
node eval.ts \
  --agent claude-code \
  --model claude-sonnet-4.5 \
  --no-context \
  100-flight-booking-plain

# Run with custom manifest
node eval.ts \
  --agent copilot-cli \
  --model gpt-5.2 \
  --context components.json \
  120-flight-booking-radix
```

### Available Evaluations

```bash
# List available evaluations
ls eval/evals/

# Common evals:
# - 100-flight-booking-plain - Build flight booking UI with plain HTML/CSS
# - 110-flight-booking-reshaped - Same but with Reshaped component library
# - 120-flight-booking-radix - Same but with Radix UI components
# - 130-flight-booking-rsuite - Same but with RSuite components
```

### Evaluation Metrics

Each evaluation produces comprehensive metrics:

#### Build Metrics

- **buildSuccess**: `true` | `false` - Whether the project builds without errors

#### Type Checking

- **typeCheckErrors**: `number` - Count of TypeScript errors

#### Linting

- **lintErrors**: `number` - Count of ESLint errors

#### Testing

- **test.passed**: `number` - Number of passing tests
- **test.failed**: `number` - Number of failing tests

#### Accessibility

- **a11y.violations**: `number` - Number of axe violations

#### Code Coverage

- **coverage.lines**: `number` - Line coverage percentage
- **coverage.statements**: `number` - Statement coverage percentage
- **coverage.branches**: `number` - Branch coverage percentage
- **coverage.functions**: `number` - Function coverage percentage

#### Performance

- **cost**: `number` - API cost in USD
- **duration**: `number` - Total time in seconds
- **apiTime**: `number` - API time in seconds
- **turns**: `number` - Number of conversation turns

### Viewing Results

#### Summary JSON

```bash
# View summary
cat evals/100-flight-booking-plain/experiments/{context}-{agent}-{timestamp}/results/summary.json
```

Example output:

```json
{
  "cost": 0.1234,
  "duration": 45.2,
  "apiTime": 12.3,
  "turns": 8,
  "buildSuccess": true,
  "typeCheckErrors": 0,
  "lintErrors": 2,
  "test": { "passed": 12, "failed": 0 },
  "a11y": { "violations": 1 },
  "coverage": {
    "lines": 87.5,
    "statements": 86.9,
    "branches": 75.0,
    "functions": 80.0
  }
}
```

#### Conversation Viewer

```bash
# Open conversation viewer
open conversation-viewer.html

# Then select the full-conversation.js file from the experiment
```

This shows:

- All messages and tool calls
- Token counts and costs per message
- Todo list progress
- Timeline of actions

#### Storybook

```bash
# View the generated Storybook
cd evals/100-flight-booking-plain/experiments/{context}-{agent}-{timestamp}/project
pnpm storybook
```

### Research Workflow Example

#### Research Question

"Does providing component documentation improve agent success rates?"

#### Experiment Design

1. **Baseline**: Run without any context
2. **Treatment**: Run with component manifest

```bash
# Run baseline
node eval.ts \
  --agent claude-code \
  --model claude-sonnet-4.5 \
  --no-context \
  --upload-id research-batch-1 \
  120-flight-booking-radix

# Run with manifest
node eval.ts \
  --agent claude-code \
  --model claude-sonnet-4.5 \
  --context components.json \
  --upload-id research-batch-1 \
  120-flight-booking-radix
```

#### Analyze Results

Compare metrics:

| Metric | No Context | With Manifest | Improvement |
|--------|------------|---------------|-------------|
| Build Success | 80% | 95% | +15% |
| Type Errors | 12 | 2 | -83% |
| Test Pass Rate | 75% | 92% | +17% |
| Duration | 52s | 38s | -27% |
| Cost | $0.15 | $0.11 | -27% |

#### Conclusion

Component manifests significantly improve success rates and reduce development time.

### Creating Custom Evaluations

#### 1. Create Eval Directory

```bash
mkdir eval/evals/200-my-custom-eval
cd eval/evals/200-my-custom-eval
```

#### 2. Write Prompt

Create `prompt.md`:

```markdown
Build a SearchResults component that displays:

- List of results with title, description, and link
- Pagination controls (prev, next, page numbers)
- Loading skeleton while fetching
- Empty state when no results

<technical_requirements>

1. Component MUST be default export in src/components/SearchResults.tsx
2. Component MUST accept results prop: Array<{id: string, title: string, description: string, url: string}>
3. Pagination controls SHOULD have data-testid="pagination"
4. Each result SHOULD have data-testid="result-{id}"

</technical_requirements>
```

#### 3. Optional: Add Context Files

```bash
# For component library evals
cp ../120-flight-booking-radix/components.json .

# For custom MCP servers
echo '{"server-name": {"url": "http://localhost:3000/mcp", "type": "http"}}' > mcp.config.json

# For extra prompts
echo "Use TypeScript strict mode" > extra-prompt-01.md
```

#### 4. Optional: Add Hooks

Create `hooks.ts`:

```typescript
import type { Hooks } from '../../types.ts';

export default {
  async postPrepareExperiment(args, log) {
    // Copy test fixtures
    await fs.copyFile(
      './fixtures/results.json',
      path.join(args.experimentPath, 'project/src/fixtures/results.json')
    );
  },
} satisfies Hooks;
```

#### 5. Run Evaluation

```bash
cd ../..
node eval.ts 200-my-custom-eval
```

### Best Practices for Research

1. **Control Variables**: Change one thing at a time (agent, model, or context)
2. **Run Multiple Times**: Results can vary; run 3-5 times for statistical significance
3. **Use Upload IDs**: Group related experiments for easier comparison
4. **Document Findings**: Keep notes on what works and what doesn't
5. **Share Results**: Upload to the public Google Sheet for community insights
6. **Version Your Evals**: Track changes to prompts and requirements

### Public Results

All uploaded evaluation results are available in this [Google Sheet](https://docs.google.com/spreadsheets/d/1TAvPyK6S6J-Flc1-gNrQpwmd6NWVXoTrQhaQ35y13vw/edit?usp=sharing).

You can:

- Compare different models and agents
- See trends over time
- Filter by context type
- Analyze success patterns

---

## 6. Benchmarking

### Overview

The Benchmarking use case extends the Research use case to systematically compare different models and agents across standardized tasks. This helps answer questions like:

- Which model is best for UI development?
- How does Claude Opus compare to GPT-5.2?
- Is the added cost of a larger model worth it?

### How It Works

Same infrastructure as Research, but focused on:

1. **Running identical prompts** across multiple models/agents
2. **Comparing performance metrics** side-by-side
3. **Analyzing cost vs. quality tradeoffs**
4. **Identifying model strengths and weaknesses**

### Setup

Same as Research use case (see above).

### Supported Models

| Model | Claude Code CLI | Copilot CLI | Typical Cost |
|-------|:---------------:|:-----------:|--------------|
| `claude-sonnet-4.5` | ✅ | ✅ | $$$ |
| `claude-opus-4.5` | ✅ | ✅ | $$$$ |
| `claude-haiku-4.5` | ✅ | ✅ | $$ |
| `gpt-5.1-codex` | ❌ | ✅ | $$$ |
| `gpt-5.1-codex-max` | ❌ | ✅ | $$$$ |
| `gpt-5.2` | ❌ | ✅ | $$$$ |
| `gemini-3-pro-preview` | ❌ | ✅ | $$$ |

### Running a Benchmark

#### Single Eval, Multiple Models

```bash
# Create a benchmark script
cat > run-benchmark.sh << 'EOF'
#!/bin/bash

EVAL="100-flight-booking-plain"
CONTEXT="storybook-dev"
UPLOAD_ID="benchmark-$(date +%Y%m%d)"

# Test Claude models
for MODEL in claude-haiku-4.5 claude-sonnet-4.5 claude-opus-4.5; do
  echo "Testing $MODEL..."
  node eval.ts \
    --agent claude-code \
    --model $MODEL \
    --context $CONTEXT \
    --upload-id $UPLOAD_ID \
    $EVAL
done

# Test GPT models (requires Copilot CLI)
for MODEL in gpt-5.1-codex gpt-5.2; do
  echo "Testing $MODEL..."
  node eval.ts \
    --agent copilot-cli \
    --model $MODEL \
    --context $CONTEXT \
    --upload-id $UPLOAD_ID \
    $EVAL
done
EOF

chmod +x run-benchmark.sh
./run-benchmark.sh
```

#### Multiple Evals, Multiple Models

```bash
# Comprehensive benchmark
cat > comprehensive-benchmark.sh << 'EOF'
#!/bin/bash

EVALS=(
  "100-flight-booking-plain"
  "110-flight-booking-reshaped"
  "120-flight-booking-radix"
)

MODELS=(
  "claude-code:claude-sonnet-4.5"
  "claude-code:claude-opus-4.5"
  "copilot-cli:gpt-5.2"
)

UPLOAD_ID="comprehensive-$(date +%Y%m%d)"

for EVAL in "${EVALS[@]}"; do
  for MODEL_COMBO in "${MODELS[@]}"; do
    AGENT="${MODEL_COMBO%%:*}"
    MODEL="${MODEL_COMBO##*:}"

    echo "Running $EVAL with $AGENT ($MODEL)..."
    node eval.ts \
      --agent $AGENT \
      --model $MODEL \
      --context storybook-dev \
      --upload-id $UPLOAD_ID \
      $EVAL
  done
done
EOF

chmod +x comprehensive-benchmark.sh
./comprehensive-benchmark.sh
```

### Analyzing Benchmark Results

#### 1. Collect Results

```bash
# Extract all summaries for a benchmark
UPLOAD_ID="benchmark-20260102"

find evals -name "summary.json" | while read file; do
  if grep -q "$UPLOAD_ID" "$file"; then
    echo "$file"
  fi
done
```

#### 2. Compare Metrics

Create a comparison script:

```bash
# compare-results.sh
#!/bin/bash

UPLOAD_ID=$1

echo "Model,Build,TypeErrors,LintErrors,TestsPassed,TestsFailed,A11yViolations,Cost,Duration"

find evals -name "summary.json" | while read file; do
  if grep -q "$UPLOAD_ID" "$file" 2>/dev/null; then
    # Extract model from path
    MODEL=$(echo $file | grep -oP '(?<=experiments/)[^/]+')

    # Parse JSON (requires jq)
    BUILD=$(jq -r '.buildSuccess' "$file")
    TYPE_ERR=$(jq -r '.typeCheckErrors' "$file")
    LINT_ERR=$(jq -r '.lintErrors' "$file")
    PASSED=$(jq -r '.test.passed' "$file")
    FAILED=$(jq -r '.test.failed' "$file")
    A11Y=$(jq -r '.a11y.violations' "$file")
    COST=$(jq -r '.cost' "$file")
    DURATION=$(jq -r '.duration' "$file")

    echo "$MODEL,$BUILD,$TYPE_ERR,$LINT_ERR,$PASSED,$FAILED,$A11Y,$COST,$DURATION"
  fi
done
```

Run it:

```bash
chmod +x compare-results.sh
./compare-results.sh benchmark-20260102 > results.csv

# View in spreadsheet or analyze further
```

#### 3. Visualize Results

Upload to Google Sheets or use the public results sheet to create charts comparing:

- Success rates by model
- Cost vs. quality tradeoffs
- Time to completion
- Error rates

### Benchmark Example Results

| Model | Build Success | Type Errors | Tests Passed | Cost | Duration |
|-------|---------------|-------------|--------------|------|----------|
| claude-haiku-4.5 | 75% | 8 | 8/12 | $0.08 | 38s |
| claude-sonnet-4.5 | 95% | 2 | 11/12 | $0.12 | 45s |
| claude-opus-4.5 | 100% | 0 | 12/12 | $0.24 | 52s |
| gpt-5.1-codex | 90% | 3 | 10/12 | $0.15 | 42s |
| gpt-5.2 | 95% | 1 | 11/12 | $0.20 | 48s |

### Insights from Benchmarking

#### Cost vs. Quality

- **Haiku**: Fast and cheap, but lower success rate
- **Sonnet**: Best balance of cost and quality
- **Opus**: Highest quality, but 2x cost of Sonnet

#### Task-Specific Performance

- **Simple forms**: Haiku sufficient
- **Complex interactions**: Sonnet or better
- **Accessibility-critical**: Opus recommended

#### Context Impact

Benchmark the same eval with different contexts:

```bash
for CONTEXT in false storybook-dev components.json; do
  node eval.ts \
    --agent claude-code \
    --model claude-sonnet-4.5 \
    --context $CONTEXT \
    --upload-id context-comparison \
    100-flight-booking-plain
done
```

**Results:**

| Context | Build Success | Duration | Cost |
|---------|---------------|----------|------|
| None | 80% | 62s | $0.18 |
| storybook-dev | 95% | 45s | $0.12 |
| components.json | 98% | 38s | $0.11 |

**Insight:** Providing context significantly improves both speed and success rate.

### Best Practices for Benchmarking

1. **Use Identical Prompts**: Ensure fair comparison
2. **Control External Factors**: Same network conditions, machine specs
3. **Run Multiple Iterations**: Account for variability (3-5 runs)
4. **Document Setup**: Note versions, configurations, date
5. **Share Results**: Upload to public sheet for community benefit
6. **Track Over Time**: Re-run benchmarks as models improve
7. **Consider Cost**: Factor in API costs for real-world usage
8. **Test Edge Cases**: Include evals with different complexity levels

### Creating Benchmark Suites

#### Difficulty Levels

```bash
# Easy: Simple component
100-button-component

# Medium: Form with validation
200-login-form

# Hard: Complex interaction
300-flight-booking

# Very Hard: Data visualization
400-analytics-dashboard
```

#### Specialized Benchmarks

```bash
# Accessibility focus
500-a11y-navigation

# Performance focus
600-optimized-table

# TypeScript focus
700-complex-types
```

### Continuous Benchmarking

Set up automated benchmarks:

```bash
# .github/workflows/benchmark.yml
name: Weekly Benchmark

on:
  schedule:
    - cron: '0 0 * * 0' # Every Sunday

jobs:
  benchmark:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 24
      - name: Run benchmarks
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          cd eval
          ./run-benchmark.sh
      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: benchmark-results
          path: eval/evals/**/results/
```

---

## Conclusion

Storybook MCP transforms how AI agents interact with UI development through six powerful use cases:

1. **Component Development**: Build components with automatic story generation
2. **Documentation**: Living, visual, always up-to-date docs
3. **Testing**: Stories double as automated tests
4. **Design Systems**: Discover and reuse existing components
5. **Research**: Measure agent capabilities systematically
6. **Benchmarking**: Compare models and optimize for cost/quality

Each use case builds on the Model Context Protocol to make AI agents more effective, efficient, and consistent in UI development workflows.

### Getting Started

1. Install Storybook MCP: `npx storybook add @storybook/addon-mcp`
2. Configure your AI agent to connect to `http://localhost:6006/mcp`
3. Start building components and let the agent handle stories automatically
4. Iterate based on visual feedback from story links

### Resources

- [Storybook MCP GitHub](https://github.com/storybookjs/mcp)
- [Public Evaluation Results](https://docs.google.com/spreadsheets/d/1TAvPyK6S6J-Flc1-gNrQpwmd6NWVXoTrQhaQ35y13vw/edit?usp=sharing)
- [Model Context Protocol Docs](https://modelcontextprotocol.io)
- [Storybook Documentation](https://storybook.js.org)

### Contributing

Have ideas for new use cases or improvements? [Start a discussion](https://github.com/storybookjs/mcp/discussions/new?category=ideas) or [open an issue](https://github.com/storybookjs/mcp/issues/new).
