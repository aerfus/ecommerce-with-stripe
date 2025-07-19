### End-to-End Test for Ordering Onigiri

Based on the testing plan for Use case #1, here's an end-to-end test for ordering onigiri using Cypress:

```javascript
describe('Onigiri Purchase End-to-End Test', () => {
  it('allows a user to purchase onigiri through the complete checkout flow', () => {
    // Visit the homepage
    cy.visit('/');
    
    // Click the "Proceed" button on the intro screen (if applicable)
    cy.contains('button', 'Proceed').click();
    
    // Find the onigiri product
    cy.contains('.product', 'おにぎり').as('onigiriProduct');
    
    // Test the +/- buttons for adjusting quantity
    cy.get('@onigiriProduct').within(() => {
      // Initial quantity should be 0 or 1
      cy.get('.quantity-display').then($qty => {
        const initialQty = parseInt($qty.text());
        
        // Increase quantity by 2
        cy.get('.increase-button').click().click();
        cy.get('.quantity-display').should('contain', initialQty + 2);
        
        // Decrease quantity by 1
        cy.get('.decrease-button').click();
        cy.get('.quantity-display').should('contain', initialQty + 1);
        
        // Add to cart
        cy.contains('button', 'Add to Cart').click();
      });
    });
    
    // Verify cart icon shows updated count
    cy.get('[data-testid="cart-icon"]').should('contain', '1');
    
    // Open the cart
    cy.get('[data-testid="cart-icon"]').click();
    
    // Verify onigiri is in the cart
    cy.get('.cart-items').within(() => {
      cy.contains('おにぎり').should('be.visible');
      cy.contains('(1)').should('be.visible'); // Quantity
    });
    
    // Verify total price is displayed
    cy.get('.total-price').should('be.visible');
    
    // Proceed to checkout
    cy.contains('button', 'Proceed to checkout').click();
    
    // Verify redirect to payment screen
    cy.url().should('include', 'checkout');
    
    // If there are 20 or more items, verify cart items are displayed
    // (In this test we only have 1 item, so this check is optional)
    
    // Fill in shipping information
    cy.get('input[name="name"]').type('Test User');
    cy.get('input[name="email"]').type('test@example.com');
    cy.get('input[name="address"]').type('123 Test Street');
    cy.get('input[name="city"]').type('Test City');
    cy.get('input[name="postal_code"]').type('123-4567');
    
    // Fill in payment information (using Stripe test card)
    cy.get('input[name="cardNumber"]').type('4242424242424242');
    cy.get('input[name="cardExpiry"]').type('1230'); // December 2030
    cy.get('input[name="cardCvc"]').type('123');
    
    // Complete payment
    cy.contains('button', 'Pay').click();
    
    // Verify successful payment (redirect to success page)
    cy.url().should('include', 'success');
    
    // Verify cart is now empty
    cy.get('[data-testid="cart-icon"]').click();
    cy.get('.cart-items').should('contain', 'Your cart is empty');
  });
});
```

### Alternative Implementation Using Playwright

If you prefer using Playwright for end-to-end testing, here's the equivalent test:

```javascript
import { test, expect } from '@playwright/test';

test('Complete onigiri purchase flow', async ({ page }) => {
  // Visit the homepage
  await page.goto('/');
  
  // Click the "Proceed" button on the intro screen (if applicable)
  await page.getByRole('button', { name: 'Proceed' }).click();
  
  // Find the onigiri product
  const onigiriProduct = page.locator('.product', { hasText: 'おにぎり' });
  
  // Test the +/- buttons for adjusting quantity
  const quantityDisplay = onigiriProduct.locator('.quantity-display');
  const initialQty = parseInt(await quantityDisplay.textContent() || '0');
  
  // Increase quantity by 2
  await onigiriProduct.locator('.increase-button').click();
  await onigiriProduct.locator('.increase-button').click();
  await expect(quantityDisplay).toHaveText(`${initialQty + 2}`);
  
  // Decrease quantity by 1
  await onigiriProduct.locator('.decrease-button').click();
  await expect(quantityDisplay).toHaveText(`${initialQty + 1}`);
  
  // Add to cart
  await onigiriProduct.getByRole('button', { name: 'Add to Cart' }).click();
  
  // Verify cart icon shows updated count
  await expect(page.locator('[data-testid="cart-icon"]')).toContainText('1');
  
  // Open the cart
  await page.locator('[data-testid="cart-icon"]').click();
  
  // Verify onigiri is in the cart
  await expect(page.locator('.cart-items')).toContainText('おにぎり');
  await expect(page.locator('.cart-items')).toContainText('(1)'); // Quantity
  
  // Verify total price is displayed
  await expect(page.locator('.total-price')).toBeVisible();
  
  // Proceed to checkout
  await page.getByRole('button', { name: 'Proceed to checkout' }).click();
  
  // Verify redirect to payment screen
  await expect(page).toHaveURL(/checkout/);
  
  // Fill in shipping information
  await page.locator('input[name="name"]').fill('Test User');
  await page.locator('input[name="email"]').fill('test@example.com');
  await page.locator('input[name="address"]').fill('123 Test Street');
  await page.locator('input[name="city"]').fill('Test City');
  await page.locator('input[name="postal_code"]').fill('123-4567');
  
  // Fill in payment information (using Stripe test card)
  await page.locator('input[name="cardNumber"]').fill('4242424242424242');
  await page.locator('input[name="cardExpiry"]').fill('1230'); // December 2030
  await page.locator('input[name="cardCvc"]').fill('123');
  
  // Complete payment
  await page.getByRole('button', { name: 'Pay' }).click();
  
  // Verify successful payment (redirect to success page)
  await expect(page).toHaveURL(/success/);
  
  // Verify cart is now empty
  await page.locator('[data-testid="cart-icon"]').click();
  await expect(page.locator('.cart-items')).toContainText('Your cart is empty');
});
```

### Test Notes

1. This test follows the business scenario outlined in the testing plan for Use case #1.
2. The test verifies all key functionality:
   - Adjusting product quantity with +/- buttons
   - Adding products to cart
   - Viewing cart contents and total price
   - Proceeding to checkout
   - Entering shipping and payment information
   - Completing payment
   - Verifying cart is cleared after purchase

3. The selectors used (like `.product`, `.quantity-display`, etc.) should be adjusted to match your actual application's DOM structure.

4. For Stripe testing, the test uses the standard Stripe test card number (4242 4242 4242 4242) which will always succeed in test mode.