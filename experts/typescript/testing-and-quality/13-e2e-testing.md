# E2E Testing

## Profile

### What It Is
End-to-end (E2E) testing verifies application behavior from the user's perspective — interacting with the UI through a real browser, making API calls, and validating the complete flow. Playwright (Microsoft) and Cypress are the two dominant tools, with Playwright increasingly preferred for its speed, multi-browser support, and TypeScript-first design.

### Origin and Evolution
- Selenium (2004) — Original browser automation
- Protractor (2013) — Angular E2E (deprecated)
- Cypress (2017) — Developer-friendly E2E testing
- Playwright (2020, Microsoft) — Multi-browser, fast, reliable
- Current: Playwright is the de facto standard for new projects

### Key Authors and References
- **Microsoft** — Playwright team
- **Cypress.io** — Cypress documentation
- **Kent C. Dodds** — Testing Trophy (integration > E2E)
- **Playwright documentation** — comprehensive reference

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Playwright | Multi-browser (Chrome, Firefox, Safari), auto-wait, fast |
| Cypress | Chrome-focused, time-travel debugging, developer-friendly |
| Page Object Model | Pattern: encapsulate page interactions in classes |
| Test Fixtures | Reusable setup (auth state, test data) |
| Visual Testing | Screenshot comparison for visual regression |
| Trace Viewer | Debug failed tests with recorded traces |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Test user flows, not implementation** — E2E tests simulate what users do: click, type, navigate, verify content. Don't test internal state.
2. **Playwright for new projects** — Multi-browser, fast parallel execution, auto-waiting, excellent TypeScript support. Best-in-class.
3. **Fewer E2E, more integration** — E2E tests are slow and flaky. Cover critical paths (login, checkout, signup). Use unit/integration tests for coverage.
4. **Page Objects for maintainability** — Encapsulate page selectors and actions. When UI changes, update one place.
5. **Stable selectors** — Use `data-testid`, ARIA roles, or text content. Never use CSS classes or auto-generated IDs.

### When to Use vs. When to Avoid

**E2E tests for:**
- Critical user flows (auth, checkout, onboarding)
- Cross-page navigation and state persistence
- Multi-step forms and wizards
- Smoke tests for deployment verification

**Unit/integration tests for:**
- Component logic and rendering
- API handler logic
- Business rules and validation
- Edge cases and error paths

---

## Frameworks and Methodologies

### 1. Playwright Setup — step-by-step
1. Install: `npm init playwright@latest`
2. Configure `playwright.config.ts` (browsers, base URL, timeouts)
3. Create test files in `tests/` or `e2e/`
4. Use Page Objects for page interactions
5. Set up test fixtures for authentication
6. Add to CI with `npx playwright install --with-deps`

### 2. Page Object Design — step-by-step
1. Create a class per page or major component
2. Define locators as properties
3. Define actions as methods
4. Define assertions as verification methods
5. Compose page objects for multi-page flows

---

## How to Apply

### Decision Checklist
- [ ] Are E2E tests covering critical user flows?
- [ ] Are selectors using data-testid or ARIA roles?
- [ ] Are page objects used for maintainability?
- [ ] Are auth fixtures used to avoid login in every test?
- [ ] Do E2E tests run in CI with proper browser setup?

### Implementation Patterns

**Playwright Tests:**
```typescript
import { test, expect } from "@playwright/test";

test.describe("Order Flow", () => {
  test("user can create and view an order", async ({ page }) => {
    await page.goto("/orders/new");

    // Fill form
    await page.getByLabel("Customer").fill("Alice");
    await page.getByRole("button", { name: "Add Item" }).click();
    await page.getByLabel("Product").fill("Widget");
    await page.getByLabel("Quantity").fill("3");

    // Submit
    await page.getByRole("button", { name: "Create Order" }).click();

    // Verify redirect and content
    await expect(page).toHaveURL(/\/orders\/[\w-]+/);
    await expect(page.getByText("Order created")).toBeVisible();
    await expect(page.getByText("Widget")).toBeVisible();
    await expect(page.getByText("Qty: 3")).toBeVisible();
  });
});
```

**Page Object Model:**
```typescript
// page-objects/order-page.ts
import { type Page, type Locator, expect } from "@playwright/test";

export class OrderPage {
  readonly customerInput: Locator;
  readonly addItemButton: Locator;
  readonly createButton: Locator;

  constructor(private readonly page: Page) {
    this.customerInput = page.getByLabel("Customer");
    this.addItemButton = page.getByRole("button", { name: "Add Item" });
    this.createButton = page.getByRole("button", { name: "Create Order" });
  }

  async goto() {
    await this.page.goto("/orders/new");
  }

  async fillCustomer(name: string) {
    await this.customerInput.fill(name);
  }

  async addItem(product: string, quantity: number) {
    await this.addItemButton.click();
    await this.page.getByLabel("Product").last().fill(product);
    await this.page.getByLabel("Quantity").last().fill(String(quantity));
  }

  async submit() {
    await this.createButton.click();
  }

  async expectSuccess() {
    await expect(this.page.getByText("Order created")).toBeVisible();
  }
}

// Usage in test
test("create order flow", async ({ page }) => {
  const orderPage = new OrderPage(page);
  await orderPage.goto();
  await orderPage.fillCustomer("Alice");
  await orderPage.addItem("Widget", 3);
  await orderPage.submit();
  await orderPage.expectSuccess();
});
```

**Authentication Fixture:**
```typescript
// fixtures/auth.ts
import { test as base, type Page } from "@playwright/test";

type AuthFixtures = {
  authenticatedPage: Page;
  adminPage: Page;
};

export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ page }, use) => {
    // Login once, reuse state
    await page.goto("/login");
    await page.getByLabel("Email").fill("user@example.com");
    await page.getByLabel("Password").fill("password");
    await page.getByRole("button", { name: "Sign In" }).click();
    await page.waitForURL("/dashboard");
    await use(page);
  },

  adminPage: async ({ page }, use) => {
    await page.goto("/login");
    await page.getByLabel("Email").fill("admin@example.com");
    await page.getByLabel("Password").fill("admin-password");
    await page.getByRole("button", { name: "Sign In" }).click();
    await use(page);
  },
});

// Usage
test("authenticated user sees dashboard", async ({ authenticatedPage }) => {
  await authenticatedPage.goto("/dashboard");
  await expect(authenticatedPage.getByText("Welcome")).toBeVisible();
});
```

**Playwright Configuration:**
```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [["html", { open: "never" }]],
  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
  },
  projects: [
    { name: "chromium", use: { ...devices["Desktop Chrome"] } },
    { name: "firefox", use: { ...devices["Desktop Firefox"] } },
    { name: "webkit", use: { ...devices["Desktop Safari"] } },
    { name: "mobile", use: { ...devices["iPhone 13"] } },
  ],
  webServer: {
    command: "npm run dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

### Common Mistakes
1. **Too many E2E tests** — Testing everything with E2E; slow, flaky. Use unit/integration for most testing.
2. **CSS selectors** — `page.click(".btn-primary")` breaks when class changes; use data-testid or ARIA roles
3. **No Page Objects** — Selectors scattered across tests; one UI change breaks many tests
4. **Login in every test** — Slow; use auth fixtures with stored state
5. **No retries in CI** — E2E tests can be flaky in CI; use retries (2) for resilience

### Integration with Other Topics
- **Testing Strategy** — E2E at the top of the testing pyramid
- **CI/CD** — E2E runs in CI with browser installation
- **Frontend Architecture** — Tests verify user-facing behavior
- **Accessibility** — Test with ARIA selectors; verify accessibility
- **Deployment** — Smoke tests after deployment

---

## Resources

### Essential Reading
- Playwright documentation (playwright.dev)
- Cypress documentation (docs.cypress.io)
- Kent C. Dodds — "Testing Trophy" article

### Online Resources
- Playwright trace viewer
- Playwright codegen (record and generate tests)
- Testing Library Playwright (@testing-library/playwright, hypothetical)

### Practice Exercises
1. Set up Playwright with multi-browser configuration
2. Write E2E tests for a login → dashboard → action flow
3. Create Page Objects for 3 pages and compose them
4. Set up auth fixtures with stored session state
5. Configure CI to run E2E tests with screenshots on failure
