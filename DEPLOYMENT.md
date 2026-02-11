# Deployment Guide

This guide covers deploying your Dashboard application to production using Vercel and Neon PostgreSQL.

## Prerequisites

- GitHub account with your code pushed
- Vercel account ([sign up](https://vercel.com/signup))
- Neon account ([sign up](https://neon.tech))

## Step 1: Set Up Neon Database

### Using Neon Console

1. Go to [Neon Console](https://console.neon.tech)
2. Click "Create Project"
3. Choose a name for your project
4. Select a region (choose closest to your users)
5. Click "Create Project"
6. Copy the connection string from the dashboard
   - Format: `postgresql://user:password@host/database?sslmode=require`

### Using Neon CLI

```bash
# Install and authenticate
npx neonctl@latest init

# The CLI will:
# - Open browser for authentication
# - Create a new project
# - Set up database
# - Update your .env file
```

## Step 2: Run Database Migrations

Once you have your `DATABASE_URL`:

```bash
# Set environment variable
export DATABASE_URL="postgresql://..."

# Run migrations
npm run db:migrate

# Seed database with admin user
npm run db:seed
```

## Step 3: Deploy to Vercel

### Option A: Vercel Dashboard (Recommended)

1. **Import Project**
   - Go to [Vercel Dashboard](https://vercel.com/new)
   - Click "Import Project"
   - Select your GitHub repository
   - Click "Import"

2. **Configure Environment Variables**

   Add these in the Vercel project settings:

   ```
   DATABASE_URL=postgresql://user:password@host/database?sslmode=require
   JWT_SECRET=your-super-secret-jwt-key-minimum-32-characters
   NEXTAUTH_URL=https://your-domain.vercel.app
   ```

   Optional variables:

   ```
   OPENAI_API_KEY=sk-...
   RESEND_API_KEY=re_...
   ```

3. **Deploy**
   - Click "Deploy"
   - Wait for build to complete
   - Visit your production URL

### Option B: Vercel CLI

```bash
# Install Vercel CLI
npm install -g vercel

# Login to Vercel
vercel login

# Deploy to preview
vercel

# Deploy to production
vercel --prod
```

## Step 4: Configure Production Environment

### Set Environment Variables via CLI

```bash
# Set production environment variables
vercel env add DATABASE_URL production
vercel env add JWT_SECRET production
vercel env add NEXTAUTH_URL production

# Optional
vercel env add OPENAI_API_KEY production
vercel env add RESEND_API_KEY production
```

### Update NEXTAUTH_URL

After first deployment, update `NEXTAUTH_URL` to your production domain:

```bash
vercel env rm NEXTAUTH_URL production
vercel env add NEXTAUTH_URL production
# Enter: https://your-project.vercel.app
```

## Step 5: Verify Deployment

1. **Visit Your Site**
   - Go to your Vercel URL
   - Should see the login page

2. **Test Authentication**
   - Login with admin credentials:
     - Email: `test@test.com`
     - Password: `Test123@123`

3. **Check Database Connection**
   - Navigate to dashboard
   - Verify data loads correctly

4. **Test Admin Features**
   - Go to `/admin`
   - Verify admin panel loads

## Step 6: Set Up Automated Deployment

### GitHub Integration

Vercel automatically deploys when you push to GitHub:

- **Push to any branch** → Preview deployment
- **Push to main/master** → Production deployment

### GitHub Actions (Optional)

For more control, use GitHub Actions:

1. **Get Vercel Token**
   - Go to [Vercel Account Settings](https://vercel.com/account/tokens)
   - Create new token
   - Copy the token

2. **Add GitHub Secrets**

   In your GitHub repository settings → Secrets and variables → Actions:

   ```
   VERCEL_TOKEN=your-vercel-token
   DATABASE_URL=your-neon-connection-string
   JWT_SECRET=your-jwt-secret
   NEXTAUTH_URL=https://your-domain.vercel.app
   ```

3. **Push to Main**
   - GitHub Actions will automatically run tests
   - If tests pass, deploys to Vercel

## Step 7: Custom Domain (Optional)

1. Go to Vercel project settings → Domains
2. Add your custom domain
3. Update DNS records as instructed
4. Update `NEXTAUTH_URL` environment variable to your custom domain

## Monitoring and Maintenance

### Vercel Dashboard

Monitor your deployment:

- **Analytics** - Page views, performance
- **Logs** - Runtime logs and errors
- **Deployments** - Deployment history

### Neon Dashboard

Monitor your database:

- **Metrics** - Connection count, query performance
- **Branches** - Database branches for development
- **Backups** - Automatic backups

## Troubleshooting

### Build Fails

**Error: Prisma Client not generated**

```bash
# Add to package.json scripts
"postinstall": "prisma generate"
```

**Error: Environment variables not found**

- Check Vercel environment variables are set
- Ensure they're set for "Production" environment

### Database Connection Issues

**Error: Connection timeout**

- Verify `DATABASE_URL` includes `?sslmode=require`
- Check Neon database is active (not suspended)

**Error: Authentication failed**

- Verify connection string is correct
- Check password doesn't contain special characters that need URL encoding

### Runtime Errors

**Error: JWT_SECRET not defined**

- Ensure `JWT_SECRET` is set in Vercel environment variables
- Redeploy after adding environment variables

**Error: NEXTAUTH_URL mismatch**

- Update `NEXTAUTH_URL` to match your production domain
- Include protocol (https://)

## Rollback

If deployment has issues:

```bash
# Via Vercel CLI
vercel rollback

# Or via Vercel Dashboard
# Go to Deployments → Select previous deployment → Promote to Production
```

## Best Practices

1. **Environment Variables**
   - Never commit `.env` files
   - Use different secrets for production
   - Rotate secrets regularly

2. **Database**
   - Use Neon branching for testing migrations
   - Enable connection pooling for better performance
   - Monitor query performance

3. **Deployment**
   - Test in preview before promoting to production
   - Use semantic versioning for releases
   - Keep deployment logs for debugging

4. **Security**
   - Use strong JWT secrets (32+ characters)
   - Enable Vercel's security headers
   - Monitor for suspicious activity

## Support

- **Vercel Docs**: https://vercel.com/docs
- **Neon Docs**: https://neon.tech/docs
- **Next.js Docs**: https://nextjs.org/docs
