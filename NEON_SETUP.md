# Neon Database Setup Instructions

Since the Neon CLI authentication timed out, here are manual setup instructions:

## Option 1: Using Neon Console (Easiest)

1. **Go to Neon Console**
   - Visit: https://console.neon.tech
   - Sign up or log in

2. **Create a New Project**
   - Click "Create Project"
   - Choose a name (e.g., "dashboard-production")
   - Select a region closest to your users
   - Click "Create Project"

3. **Get Connection String**
   - After creation, you'll see the connection string
   - Format: `postgresql://user:password@host/database?sslmode=require`
   - Copy this string

4. **Update .env File**
   ```bash
   # Replace the DATABASE_URL in your .env file
   DATABASE_URL="postgresql://user:password@host/database?sslmode=require"
   ```

5. **Run Migrations**
   ```bash
   npm run db:push
   npm run db:seed
   ```

## Option 2: Using Neon CLI (Retry)

```bash
# Try the CLI again
npx neonctl@latest init

# This will:
# 1. Open browser for authentication
# 2. Create a new Neon project
# 3. Set up PostgreSQL database
# 4. Automatically update your .env file
```

**Important:** Make sure to complete the browser authentication within 60 seconds.

## Option 3: Use SQLite for Development

If you want to test locally first:

```bash
# Your .env already has SQLite configured
DATABASE_URL="file:./dev.db"

# Just run:
npm run db:push
npm run db:seed
npm run dev
```

**Note:** SQLite is fine for development but use PostgreSQL (Neon) for production.

## Verification

After setting up the database:

```bash
# Check if database is accessible
npm run db:studio

# This opens Prisma Studio where you can view your data
```

## Troubleshooting

**Error: Connection timeout**
- Ensure `?sslmode=require` is at the end of the connection string
- Check that your Neon database is active (not suspended)

**Error: Authentication failed**
- Verify the connection string is correct
- Check for special characters in password (may need URL encoding)

**Error: Database does not exist**
- Neon automatically creates the database
- If using manual connection string, ensure database name is correct
