# Pricklet: Barcode Labels — proof of fix for review 152708 (1.2.2)

Fix deployed 21 Nov 2026 (build 5170f14). Tested end to end on development store `pricklet-dev`.

**Root cause:** after a reinstall, an in-app navigation could reach `/app/plans` before App Bridge attached a session token; the library's auth handshake response (status 200) was not recognised by our error boundary in the minified production build, so React Router showed "Unhandled Thrown Response! 200" and "See plans" appeared to do nothing.

**Fix:** the app is on Shopify App Pricing, so the Plans page now reads the current plan from `currentAppInstallation.activeSubscriptions` on every load and every plan button opens Shopify's hosted plan selection page (top frame). Accept, decline and re-approval after reinstall are handled by Shopify. Token-less data requests now get a 401 retry so App Bridge retries with a fresh token, stale tokens are re-authenticated, error boundaries work in production, and uninstall resets the plan to Free. No Billing API calls remain.

## Test run (install → upgrade → decline → downgrade → uninstall → reinstall → upgrade)

1. Plans page via sidebar navigation (no error, plan picker shown) ![](img/01-plans-nav.png)
2. Plans page after a full reload ![](img/02-plans-reload.png)
3. "Choose Pro" opens Shopify's plan selection page ![](img/03-hosted-plan-page.png)
4. Pro approved, app shows Pro ![](img/04-pro-approved.png)
5. Unlimited approval screen (then declined with Cancel) ![](img/05-unlimited-approval-screen.png)
6. After declining, app still on Pro ![](img/06-after-decline-still-pro.png)
7. Downgrade to Free ![](img/07-downgrade-free.png)
8. App uninstalled (while on Unlimited) ![](img/08-uninstalled.png)
9. Reinstalled: app opens normally ![](img/09-reinstalled-print-labels.png)
10. After reinstall: Plans via nav and reload, no error, plan picker shown ![](img/10-plans-after-reinstall-reload.png)
11. After reinstall: upgrade to Pro re-approved through Shopify, app shows Pro ![](img/11-pro-after-reinstall.png)

Monthly label usage is tracked per shop and intentionally survives reinstall (it resets each calendar month).
