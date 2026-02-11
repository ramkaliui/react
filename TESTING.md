# Testing Guide

Comprehensive guide for testing the Dashboard application.

## Testing Stack

- **Jest** - Unit testing framework
- **React Testing Library** - Component testing
- **Playwright** - End-to-end testing
- **@testing-library/user-event** - User interaction simulation
- **@testing-library/jest-dom** - Custom Jest matchers

## Running Tests

### Unit Tests

```bash
# Run in watch mode (development)
npm test

# Run once with coverage
npm run test:ci

# Run specific test file
npm test -- src/components/ui/input.test.tsx

# Update snapshots
npm test -- -u
```

### E2E Tests

```bash
# Run all E2E tests
npm run test:e2e

# Run with UI mode (interactive)
npm run test:e2e:ui

# Run specific test file
npx playwright test e2e/auth.spec.ts

# Run in specific browser
npx playwright test --project=chromium

# Debug mode
npx playwright test --debug
```

### Type Checking

```bash
npm run type-check
```

## Writing Tests

### Unit Tests

Create test files next to components with `.test.tsx` or `.test.ts` extension.

**Example: Component Test**

```typescript
import { render, screen } from '@/test-utils';
import { Input } from './input';

describe('Input', () => {
  it('renders correctly', () => {
    render(<Input placeholder="Enter text" />);
    expect(screen.getByPlaceholderText('Enter text')).toBeInTheDocument();
  });

  it('handles user input', async () => {
    const { user } = render(<Input />);
    const input = screen.getByRole('textbox');

    await user.type(input, 'Hello');
    expect(input).toHaveValue('Hello');
  });
});
```

**Example: API Route Test**

```typescript
import { POST } from './route';

describe('POST /api/auth/login', () => {
  it('returns token for valid credentials', async () => {
    const request = new Request('http://localhost:3000/api/auth/login', {
      method: 'POST',
      body: JSON.stringify({
        email: 'test@test.com',
        password: 'Test123@123',
      }),
    });

    const response = await POST(request);
    const data = await response.json();

    expect(response.status).toBe(200);
    expect(data).toHaveProperty('token');
  });
});
```

### E2E Tests

Create test files in the `e2e/` directory with `.spec.ts` extension.

**Example: Authentication Flow**

```typescript
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('user can login', async ({ page }) => {
    await page.goto('/');

    await page.getByLabel(/email/i).fill('test@test.com');
    await page.getByLabel(/password/i).fill('Test123@123');
    await page.getByRole('button', { name: /sign in/i }).click();

    await expect(page).toHaveURL(/\/dashboard/);
  });
});
```

## Test Coverage

### Viewing Coverage

```bash
# Run tests with coverage
npm run test:ci

# Open coverage report
open coverage/lcov-report/index.html
```

### Coverage Thresholds

Minimum coverage requirements (configured in `jest.config.js`):

- **Branches**: 70%
- **Functions**: 70%
- **Lines**: 70%
- **Statements**: 70%

## Best Practices

### Unit Tests

1. **Test Behavior, Not Implementation**

   ```typescript
   // ❌ Bad - testing implementation
   expect(component.state.count).toBe(1);

   // ✅ Good - testing behavior
   expect(screen.getByText('Count: 1')).toBeInTheDocument();
   ```

2. **Use Testing Library Queries**

   ```typescript
   // Priority order:
   screen.getByRole('button', { name: /submit/i });
   screen.getByLabelText(/email/i);
   screen.getByPlaceholderText(/enter email/i);
   screen.getByText(/welcome/i);
   screen.getByTestId('custom-element'); // Last resort
   ```

3. **Test User Interactions**

   ```typescript
   import { render, screen } from '@/test-utils';
   import userEvent from '@testing-library/user-event';

   test('form submission', async () => {
     const user = userEvent.setup();
     render(<LoginForm />);

     await user.type(screen.getByLabelText(/email/i), 'test@test.com');
     await user.click(screen.getByRole('button', { name: /submit/i }));

     expect(screen.getByText(/success/i)).toBeInTheDocument();
   });
   ```

### E2E Tests

1. **Use Page Object Model**

   ```typescript
   class LoginPage {
     constructor(private page: Page) {}

     async login(email: string, password: string) {
       await this.page.getByLabel(/email/i).fill(email);
       await this.page.getByLabel(/password/i).fill(password);
       await this.page.getByRole('button', { name: /sign in/i }).click();
     }
   }
   ```

2. **Wait for Elements**

   ```typescript
   // ✅ Good - wait for element
   await expect(page.getByText('Welcome')).toBeVisible();

   // ❌ Bad - arbitrary timeout
   await page.waitForTimeout(1000);
   ```

3. **Use Fixtures for Setup**
   ```typescript
   test.beforeEach(async ({ page }) => {
     // Login before each test
     await page.goto('/');
     await page.getByLabel(/email/i).fill('test@test.com');
     await page.getByLabel(/password/i).fill('Test123@123');
     await page.getByRole('button', { name: /sign in/i }).click();
   });
   ```

## Debugging Tests

### Jest

```bash
# Run tests in debug mode
node --inspect-brk node_modules/.bin/jest --runInBand

# Use Chrome DevTools
# Open chrome://inspect
# Click "inspect" on the Node process
```

### Playwright

```bash
# Debug mode with browser
npx playwright test --debug

# Headed mode (see browser)
npx playwright test --headed

# Slow motion
npx playwright test --headed --slow-mo=1000
```

### VS Code Debugging

Add to `.vscode/launch.json`:

```json
{
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Jest Debug",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": ["--runInBand"],
      "console": "integratedTerminal"
    }
  ]
}
```

## CI/CD Integration

Tests run automatically on:

- Every pull request
- Every push to main/master

See `.github/workflows/ci.yml` for configuration.

## Common Issues

### Jest

**Issue: Module not found**

```bash
# Clear Jest cache
npx jest --clearCache
```

**Issue: Tests timeout**

```typescript
// Increase timeout for specific test
test('slow test', async () => {
  // test code
}, 10000); // 10 seconds
```

### Playwright

**Issue: Browser not installed**

```bash
npx playwright install
```

**Issue: Tests fail in CI but pass locally**

```typescript
// Use waitForLoadState
await page.waitForLoadState('networkidle');
```

## Resources

- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Playwright Documentation](https://playwright.dev/docs/intro)
- [Testing Best Practices](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
