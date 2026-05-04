# Setup Guide: Klaviyo Abandoned Cart Recovery Workflow with Smart Discount Tiers

## Before you start

**Required accounts:**
- **Shopify store with Admin access** - You need to be able to create custom apps or private apps in your Shopify admin panel
- **Klaviyo account** (any plan) - You'll need access to Settings where you can generate Private API Keys
- **Google account** - For creating a Google Sheet to log recovered carts and errors
- **n8n account** - Either n8n Cloud (recommended for beginners, starts at $20/month) or self-hosted (free but requires technical setup)

**Estimated setup time:** 20-25 minutes

**What this template DOES:**
This workflow automatically monitors your Shopify store for abandoned carts, then sends personalized recovery emails through Klaviyo with smart discount offers based on cart value. Higher-value carts get bigger discounts to maximize recovery rate. It checks every hour for carts that were abandoned in the last 24 hours, creates or updates customer profiles in Klaviyo, sends the cart data as an event that triggers your Klaviyo email flow, and logs everything to a Google Sheet so you can track performance. The discount tiers are automatically calculated (for example: 10% off for carts under $50, 15% off for $50-$100, 20% off for $100+).

**What this template does NOT do:**
This workflow does NOT create the actual email templates in Klaviyo - you'll need to design those yourself in Klaviyo's Flow builder using the "Abandoned Cart" event this workflow sends. It does not automatically apply discount codes to carts (you'll need to create those codes in Shopify first). It does not send emails directly - it only sends data to Klaviyo, which then triggers your email sequence. It does not track whether customers actually used the discount or completed their purchase (though you can see that data in your Google Sheet by cross-referencing order data). This is a behind-the-scenes automation that feeds your Klaviyo marketing, not a complete standalone solution.

## Step 1 - Get your n8n instance running

You have two options for running n8n:

**Option A: n8n Cloud (Recommended for beginners)**
1. Go to https://n8n.io/cloud
2. Click "Start free trial" and create your account
3. After signup, you'll land in your n8n dashboard immediately
4. No installation needed - you're ready to import the workflow

**Option B: Self-hosted (Free but technical)**
1. Go to https://docs.n8n.io/hosting/
2. Follow the installation guide for your preferred method (Docker is easiest)
3. Once installed, access n8n at http://localhost:5678 (or your server URL)

For this guide, we'll assume you're using n8n Cloud since you're a small business owner who values time over technical complexity.

## Step 2 - Import the workflow

1. In your n8n dashboard, look at the left sidebar
2. Click **"Workflows"** (should be near the top)
3. Click the **"Add Workflow"** button (top right corner)
4. Click the three dots menu (⋮) in the top right
5. Select **"Import from File"**
6. Choose the `klaviyo-abandoned-cart-recovery.json` file you received with your purchase
7. Click **"Import"**

**What you should see:** The workflow will open with 12 nodes visible on your canvas. You should see nodes labeled "Schedule Trigger," "Get Abandoned Carts," "Filter Recent Carts," and several others connected by lines. If you see a warning triangle on some nodes, that's normal - it means credentials haven't been set up yet.

## Step 3 - Connect credentials

### Shopify API Credentials

1. Log into your Shopify admin at `https://[your-store].myshopify.com/admin`
2. Go to **Settings** (bottom left) > **Apps and sales channels**
3. Click **"Develop apps"** at the top
4. If prompted, click **"Allow custom app development"**
5. Click **"Create an app"** and name it "n8n Abandoned Cart Recovery"
6. Click on **"Configure Admin API scopes"**
7. Scroll down and enable these permissions:
   - `read_orders`
   - `read_customers`
8. Click **"Save"**, then click **"Install app"**, then **"Install"** again
9. Click **"Reveal token once"** and copy the Admin API access token
10. Also copy your store name (the part before .myshopify.com)

**Now in n8n:**
1. Click on the **"Get Abandoned Carts"** node in your workflow
2. Under "Credentials," click the dropdown that says "Create New"
3. Select **"Header Auth"**
4. Name it "Shopify Admin API"
5. For Name, enter: `X-Shopify-Access-Token`
6. For Value, paste your Admin API access token
7. Click **"Save"**

### Klaviyo API Credentials

1. Log into Klaviyo at https://www.klaviyo.com/login
2. Click your account name (bottom left) > **Settings**
3. Click **"API Keys"** in the left menu
4. Under "Private API Keys," click **"Create Private API Key"**
5. Give it a name like "n8n Abandoned Cart" and click **"Create"**
6. Copy the Private API Key that appears (starts with "pk_")

**Now in n8n:**
1. Click on the **"Check Klaviyo Profile Exists"** node
2. Under "Credentials," click "Create New"
3. Select **"Klaviyo API"** (if you don't see this, select "Header Auth" and proceed)
4. If using Header Auth:
   - Name: `Authorization`
   - Value: `Klaviyo-API-Key [paste your key here]`
   - Replace `[paste your key here]` with your actual key
5. Click **"Save"**
6. You'll need to select this same credential for these nodes:
   - **"Create Klaviyo Profile"**
   - **"Send Abandoned Cart Event"**

### Google Sheets Credentials

1. Open your workflow in n8n
2. Click on the **"Log to Google Sheets"** node
3. Under "Credentials," click "Create New"
4. Select **"Google Sheets OAuth2 API"**
5. Click the **"Sign in with Google"** button
6. Select your Google account and grant permissions
7. Click **"Save"**
8. Select this same credential for the **"Log Error to Sheets"** node

## Step 4 - Configure your data

### Create your Google Sheet

1. Go to https://sheets.google.com
2. Create a new blank spreadsheet
3. Name it "Klaviyo Abandoned Cart Log"
4. In Row 1, add these exact column headers:
   - A1: `Timestamp`
   - B1: `Email`
   - C1: `Cart Value`
   - D1: `Discount Tier`
   - E1: `Status`
   - F1: `Error Message`
5. Copy the spreadsheet URL from your browser
6. In n8n, click the **"Log to Google Sheets"** node
7. Paste the spreadsheet URL in the "Document" field
8. Make sure "Sheet" is set to "Sheet1"
9. Repeat for the **"Log Error to Sheets"** node

### Configure the Schedule

1. Click the **"Schedule Trigger"** node
2. The default is set to run every hour - you can keep this or adjust it
3. Recommended: Keep "Every hour" for the first week, then adjust based on your cart abandonment patterns

### Configure Discount Tiers (Optional)

1. Click the **"Calculate Discount Tier"** node
2. Click **"Edit Code"**
3. You'll see discount tiers like this:
   - Under $50: 10% off
   - $50-$100: 15% off
   - Over $100: 20% off
4. To change these values, modify the numbers in the code (look for `discount: 10`, `discount: 15`, etc.)
5. Also update the `code` field with actual discount codes you've created in Shopify

### Configure Shopify Store URL

1. Click the **"Get Abandoned Carts"** node
2. In the URL field, replace `YOUR_STORE` with your actual Shopify store name
3. Example: `https://my-awesome-store.myshopify.com/admin/api/2024-01/checkouts.json`

## Step 5 - Test the workflow

### Run a safe test

1. Make sure you have at least one abandoned cart in your Shopify store (you can create one by adding items to cart as a test customer and not checking out)
2. In n8n, click the **"Execute Workflow"** button at the bottom
3. Watch the nodes light up green as they execute

**What success looks like:**
- All nodes should turn green (or gray if no data passed through them)
- Check your Google Sheet - you should see a new row with timestamp, email, cart value, and discount tier
- Check Klaviyo - go to Profiles, find the customer email, and look for an "Abandoned Cart" event in their timeline
- No red error nodes should appear

**If something fails:**
1. Click the red node to see the error message
2. Check that the credential for that node is properly set
3. Verify you have the correct permissions enabled
4. Check the "Common errors" section below

### Activate the workflow

Once your test succeeds:
1. Toggle the switch at the top left from **"Inactive"** to **"Active"**
2. Your workflow will now run automatically every hour

## Common errors

**Error: "401 Unauthorized" on Shopify nodes**
- **Fix:** Your Shopify API token is incorrect or expired. Go back to Step 3 and generate a new token. Make sure you clicked "Reveal token once" and copied the full token.

**Error: "403 Forbidden" on Klaviyo nodes**
- **Fix:** Your Klaviyo API key doesn't have the right permissions. Make sure you created a Private API Key (not a Public API Key). The key should start with "pk_" not "pk_public_".

**Error: "Could not find the item to update" on Google Sheets**
- **Fix:** The spreadsheet URL is wrong or the column names don't match exactly. Make sure you copied the full URL including the spreadsheet ID, and verify your column headers match exactly (including capitalization): Timestamp, Email, Cart Value, Discount Tier, Status, Error Message.

**Error: "No abandoned carts found" (not an error, just no data)**
- **Fix:** This is normal if you don't have any carts abandoned in the last 24 hours. Create a test abandoned cart by adding items to your store and not checking out.

**Error: "The resource you are requesting could not be found" on Create Klaviyo Profile**
- **Fix:** The email format from Shopify is incorrect or the Klaviyo API URL is wrong. Open the "Create Klaviyo Profile" node and verify the URL is exactly: `https://a.klaviyo.com/api/profiles/` - no typos or extra characters.

## Need help?

Reply to your Gumroad receipt email - we respond within 24h. If we can't fix it, we refund. No questions asked.

Include in your email:
- A screenshot of the error (if any)
- Which step you're stuck on
- What you've already tried

We're here to make sure this works for you.